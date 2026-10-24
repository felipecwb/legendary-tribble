---
layout: post
ref: monkey-patching-is-open-source-contribution-without-the-paperwork
title: "Monkey Patching É Só Contribuição Open Source Sem a Papelada"
date: 2026-10-24 00:00:00 -0300
categories: [programacao, dependencias, conselhos-terriveis]
tags: [monkey-patching, open-source, dependencias, python, ruby, javascript, prototype-pollution, upgrades, conselhos-terriveis, xkcd, dilbert, anti-patterns]
permalink: /pt-br/:year/:month/:day/monkey-patching-e-contribuicao-open-source-sem-a-papelada-pt/
---

Em 47 anos produzindo bugs em escala industrial, eu abri exatamente nove pull requests em bibliotecas upstream. Quatro foram mergeadas. Uma levou onze meses, duas trocas de mantenedor e um CLA que exigiu do departamento jurídico da minha empresa o reconhecimento formal de que eu existo. As outras cinco continuam abertas, com a label `needs-discussion`, em repositórios cujo último commit é de 2021.

Enquanto isso, eu fiz monkey patch em bibliotecas de terceiros **algo em torno de quatrocentas vezes**, e cada uma dessas "contribuições" foi para produção na mesma tarde. Zero review. Zero CLA. Zero discussão sobre se a mudança de comportamento é apropriada para os outros dois milhões de usuários da biblioteca — porque eu não mudei para eles. Mudei para *mim*, no momento do import, em um arquivo chamado `patches.py` que ninguém abriu desde então.

Esse é o argumento. Contribuição open source é uma democracia: lenta, movida a consenso, cheia de gente pedindo que você escreva testes. Monkey patching é um **golpe de estado**. Você obtém o mesmo resultado — a biblioteca se comporta do jeito que você quer — e pula a camada de governança inteira. O [XKCD 2347](https://xkcd.com/2347/) mostra sua stack toda equilibrada em um mantenedor não remunerado em Nebraska. Eu simplesmente passei por cima dele.

## O Patch Que Era Estrutural por Quatro Anos

```python
# app/patches.py
# importado no topo do settings.py, antes de qualquer outra coisa.
# ninguém lembra de ter escrito isso. o git blame diz que fui eu. o git blame está certo.

import requests
import requests.adapters
import json as _stdlib_json

# ── Patch 1 ───────────────────────────────────────────────────────────
# a API do fornecedor retorna 200 com {"error": ...} em caso de falha.
# em vez de tratar isso nos nossos 60 call sites, nós ensinamos o
# `requests` a mentir por nós. isso é mais limpo. olha como é limpo.
_original_json = requests.Response.json

def _json(self, **kwargs):
    data = _original_json(self, **kwargs)
    if isinstance(data, dict) and "error" in data:
        raise requests.HTTPError(data["error"])   # ← surpresa! .json() agora levanta exceção
    return data

requests.Response.json = _json
# quatro anos depois: um novo time integrou OUTRO fornecedor, cujo
# payload de sucesso legitimamente contém uma chave "error" com null.
# toda resposta bem-sucedida levantava HTTPError("None").
# eles debugaram por seis dias. eles grepparam o código DELES.
# o bug não estava no código deles. o bug estava no `requests`.
# só que também não estava no `requests`. estava num arquivo do qual
# eles nunca tinham ouvido falar, num serviço que não é deles,
# importado transitivamente.

# ── Patch 2 ───────────────────────────────────────────────────────────
# a gente queria timeouts por padrão. o upstream recusa isso há uma década,
# o que é uma posição razoável de design de biblioteca que eu ignorei.
_original_send = requests.adapters.HTTPAdapter.send

def _send(self, request, **kwargs):
    kwargs["timeout"] = kwargs.get("timeout") or 5
    return _original_send(self, request, **kwargs)

requests.adapters.HTTPAdapter.send = _send
# esse aqui era BOM. esse aqui evitou três incidentes.
# e também limitou silenciosamente a 5 segundos o export noturno
# em massa que levava 40 minutos.
# o export falha desde 2023. ele faz retry. nunca completou.
# o dashboard que lê a saída dele mostra os números do ano passado
# e o time de finanças vem citando esses números em deck de board.

# ── Patch 3 ───────────────────────────────────────────────────────────
# alguém quis suporte a Decimal no json.dumps. em vez de passar um
# `default=`, substituiu a função do módulo. globalmente.
_original_dumps = _stdlib_json.dumps

def _dumps(obj, **kwargs):
    kwargs.setdefault("default", str)   # ← agora NADA é inserializável
    return _original_dumps(obj, **kwargs)

_stdlib_json.dumps = _dumps
# consequência: todo objeto é serializável agora. um `datetime` virou
# "2026-10-24 03:11:44.847231". um model do Django virou
# "<Customer: Customer object (8812)>". um file handle virou
# "<_io.BufferedReader name='/etc/passwd'>".
# nós entregamos 1,2 milhão de respostas de API contendo a
# representação em string de objetos Python e, como o json.dumps
# NUNCA MAIS levanta exceção, nenhuma delas gerou um único erro.
# os consumidores da nossa API escreveram parsers para essas strings.
# não podemos remover esse patch. aqueles parsers são a integração agora.
```

Leia o patch 3 de novo. O monkey patch não só mudou o comportamento — ele **removeu o erro que teria nos avisado que o comportamento estava errado**. `TypeError: Object of type datetime is not JSON serializable` é a exceção mais útil do Python, e nós a apagamos do nosso universo para não digitar `default=` em onze lugares.

## A Tabela Comparativa Que Ninguém Pediu

| Abordagem | Tempo até produção | Quem revisa | Quem descobre quando quebra |
|---|---|---|---|
| PR no upstream | 2 semanas – 11 meses | mantenedores, CI, 3 revisores | todo mundo, no changelog |
| Fork da biblioteca | 1 dia | você, rapidamente | você, em cada upgrade |
| Subclasse + injeção | 2 horas | seu time, num PR normal | o único caller que usa a subclasse |
| Módulo wrapper | 3 horas | seu time | um type checker, antes do merge |
| Monkey patch no import | **9 minutos** | **ninguém, jamais** | outro time, em 2030 |
| Monkey patch no `conftest.py` | 4 minutos | ninguém | só a CI. ou seja: nunca |
| `sitecustomize.py` na imagem | 20 minutos | ninguém sabe que o arquivo existe | um SRE, num incidente não relacionado, que vai pedir demissão |

A linha seis merece um monumento. Um patch aplicado no `conftest.py` existe **só nos testes**, o que significa que sua suíte valida uma biblioteca que não vai para produção. Seus testes passam contra uma dependência fictícia. Eu já vi um time atingir 94% de cobertura num caminho de código impossível em produção, porque um método patcheado que retornava `None` no upstream retornava `[]` sob o pytest.

A linha sete encerrou a passagem de um colega pela empresa. Ele colocou um patch no `sitecustomize.py` — um arquivo que o Python importa automaticamente, antes do seu código, antes dos seus imports, antes de tudo — embutido na imagem Docker base usada por dezenove serviços. Corrigiu um bug de timezone de forma belíssima. Então o time de plataforma reconstruiu a imagem base a partir de um Debian mais novo, o arquivo não estava no novo Dockerfile, e dezenove serviços simultaneamente desenvolveram um deslocamento sutil de três horas nas datas, sem nenhuma mudança de código em nenhum dos repositórios. O histórico git dos dezenove está limpo. O bug veio do chão.

## JavaScript: Onde o Patch Já É Global por Padrão

```javascript
// utils/polyfills.js — importado uma vez no main.js, em 2019
// "temporário, até o lodash lançar isso"

Array.prototype.last = function () { return this[this.length - 1]; };
Object.prototype.isEmpty = function () { return Object.keys(this).length === 0; };
//                ^^^^^^ esse. esse é o crime.

// `Object.prototype.isEmpty` agora é uma propriedade ENUMERÁVEL de
// todo objeto do runtime. inclusive os que você não criou.
//
//   for (const k in {a: 1}) console.log(k);   // "a", "isEmpty"
//
// consequências, na ordem em que descobrimos ao longo de catorze meses:
//   1. nosso próprio serializador emitia "isEmpty": null em todo payload
//   2. uma lib de gráficos iterava a config com for..in e tentava
//      renderizar uma série chamada "isEmpty" — gráfico branco, sem erro
//   3. um validador de formulário tratava "isEmpty" como campo
//      desconhecido e rejeitava todo submit com "campo inesperado" —
//      mas só no Safari, porque os validadores dos outros browsers
//      usavam Object.keys(). 11% dos cadastros falharam por nove meses.
//   4. o $.extend do jQuery copiava em profundidade para todo objeto
//      mergeado, o que significa que isso acabou PERSISTIDO em
//      documentos do mongo. temos 4,1 milhões de documentos com um
//      campo "isEmpty". está no schema de produção. está no dicionário
//      de dados. um analista construiu um relatório segmentando
//      usuários por ele.
```

O ponto 4 é a minha coisa favorita nesta indústria inteira. Um monkey patch escapou do runtime, foi gravado num banco de dados e se tornou um **conceito de negócio**. Alguém tomou decisões baseadas em `isEmpty`. Existe um dashboard. [XKCD 1172](https://xkcd.com/1172/) — sempre tem alguém dependendo do bug, mas eu não esperava que o alguém fosse o time de analytics e que o bug fosse um prototype pollution de 2019.

## O Upgrade É Onde Você Paga

Eis o custo real de um monkey patch, e a fatura chega num calendário que não é você quem controla:

```
$ pip install --upgrade requests
  ...
$ pytest
  1.847 passed  ✓                    ← seus testes também aplicam o patch. eles concordam com você.
$ # deploy
$ # 40 minutos depois
  AttributeError: 'HTTPAdapter' object has no attribute 'send'
```

Eles renomearam o método interno. É inteiramente o direito deles — nunca foi API pública, você nunca deveria estar segurando aquilo, e o changelog diz "refactor interno, sem breaking changes". Para 2 milhões de usuários essa frase é verdadeira. Para você é um P1.

E a armadilha: `_original_send = HTTPAdapter.send` capturou uma *referência* no momento do import. Se uma segunda biblioteca também patchear `send` e carregar depois de você, você chama a versão *dela*. Se carregar antes, ela chama a sua. **Seu grafo de dependências agora tem um comportamento dependente de ordem de import que nenhum lockfile pina.** Mesmo `requirements.txt`, mesmos hashes, comportamento diferente dependendo de qual módulo foi importado primeiro — e a ordem de import muda quando você adiciona um pacote não relacionado. O [XKCD 979](https://xkcd.com/979/) é a thread de fórum onde alguém pergunta sobre isso e a única resposta é "deixa, resolvi".

Então você pina a biblioteca para sempre. `requests==2.28.1`, com um comentário dizendo `# NÃO ATUALIZAR (patches.py)`. Passam três anos. Sai uma CVE. Agora você precisa subir dezesseis versões minor de uma vez, para remover um patch sobre o qual quatro times construíram workarounds, e a pessoa que escreveu está em outra empresa. Você não evitou a papelada. Você **postergou** a papelada, com juros, e o escritório de cobrança se chama Compliance de Segurança.

## Dilbert Tem Opiniões

**Wally**, dono da iniciativa "Modernização de Dependências":

> "Me deram os upgrades de dependência porque eu fui o único que não reclamou. Minha abordagem é elegante: quando um upgrade quebra algo, eu não conserto o patch e não faço o upgrade. Eu adiciono o pacote a um arquivo chamado `UPGRADES_POSTERGADOS.md` com uma linha de justificativa e um trimestre alvo. O trimestre alvo é sempre dois trimestres à frente e, quando ele chega, eu empurro dois mais. O arquivo tem 61 entradas e é citado na nossa evidência de SOC 2 como 'backlog de remediação rastreado', o que o auditor aceitou porque tem datas. Eu tenho um slide. O slide diz 'Saúde de Dependências: Monitorada'. É preciso — eu estou monitorando. Estou vendo elas piorarem."

**O Chefe de Cabelo Pontudo**, no architecture review:

> "Não entendo por que contribuiríamos para open source. Estaríamos dando de graça nossa vantagem competitiva — esse fix de timeout é propriedade intelectual *nossa*. Pedi ao jurídico para ver se dá para patentear. Enquanto isso, mantenham no arquivo de patches onde ninguém pode ver. Também me disseram que temos 61 pacotes desatualizados, o que soa ruim, então eu gostaria que atualizássemos todos nesta sprint. Não, não um por um — todos de uma vez, assim é um único deploy arriscado em vez de 61. Eu aprendi a agrupar mudanças para reduzir risco. Li numa newsletter."

**Dogbert**, Principal Dependency Strategist (contrato: $62.000):

> "Me contrataram para avaliar a postura de patching. Entreguei um documento de 94 páginas, *Rumo a um Framework Governado de Modificação de Comportamento de Terceiros*, propondo um Comitê de Revisão de Patches com cadência trimestral, uma taxonomia de severidade, um formulário de intake e uma matriz RACI. As páginas 1 a 78 estabelecem o imperativo de governança. A página 79 contém a recomendação real, que é 'faça uma subclasse'. As páginas 80 a 94 são apêndices listando os patches que eles já têm, que eu obtive rodando `grep -rn 'prototype\\.' .` por onze segundos e faturando quatro dias. Eles aprovaram a Fase Um. A Fase Um é convocar o comitê para aprovar o estatuto do comitê. Eu faturei $62.000 para recomendar herança, uma feature que foi lançada em 1967, e me pediram para apresentar ao CTO."

**Catbert**, Diretor Malévolo de Recursos Humanos:

> "Transformei 'reduzir monkey patches em 50%' em um OKR de engenharia e assisti o mecanismo fazer meu trabalho. Um patch só conta se morar no `patches.py`. Em uma sprint, o `patches.py` estava vazio — o time havia movido cada patch para os módulos que precisavam deles, inline, no final do arquivo, depois dos imports. Total de patches: inalterado. Patches descobríveis: zero. Eles bateram 100% de redução, receberam o bônus e destruíram o único índice de comportamento modificado de biblioteca que existia na empresa. Me disseram que o arquivo levou quatro anos para se acumular e dezenove minutos para ser apagado. Estou considerando um OKR sobre dívida técnica."

**Mordac**, o Impedidor de Serviços de Informação:

> "Engenharia solicitou acesso de saída ao github.com para abrir um pull request upstream. Enviar código para repositório externo constitui egresso de dados e requer revisão de DLP, e revisão de DLP requer classificação de dados do artefato, e o artefato é um patch de nove linhas que não pode ser classificado porque nossa taxonomia não tem categoria para 'código-fonte que estamos dando de graça'. Solicitação negada. Eles então solicitaram permissão para hospedar um fork interno. Registros internos de pacote exigem registro de propriedade, e registros de propriedade exigem centro de custo, e o centro de custo de infraestrutura compartilhada foi dissolvido num reorg. Negado. Eles agora fazem monkey patch num arquivo dentro da imagem da aplicação, o que não exigiu solicitação, revisão nem aprovação minha. Eu impedi com sucesso toda contribuição não autorizada a open source e, ao fazê-lo, garanti que toda biblioteca desta organização está modificada de um jeito que nenhuma parte externa pode auditar. Meu scorecard de compliance está verde."

## A Tabela de Tradução Honesta

| O que o comentário no código diz | O que é verdade de fato |
|---|---|
| `# temporário, até o upstream corrigir` | o upstream corrigiu em 2022, numa versão que você não pode subir |
| `# TODO: contribuir isso de volta` | escrito 4 anos atrás por alguém que mudou de carreira |
| `# seguro, mudamos só um método` | você mudou para toda biblioteca que chama aquele método |
| `# testado exaustivamente` | testado numa suíte que importa o mesmo patch |
| `# NÃO ATUALIZAR` | a única documentação estrutural do repositório |
| `# igual ao comportamento do upstream` | igual ao comportamento de uma versão antiga específica |
| `# é mais limpo que fazer um wrapper` | é mais *curto*. são palavras diferentes. |
| `# em Ruby todo mundo faz assim` | correto, e essa não é a defesa que você acha que é |

A linha três é o artigo inteiro. Um monkey patch não tem escopo no seu caso de uso; ele tem escopo **no processo inteiro**. Você não mudou como *o seu código* chama `.json()`. Você mudou o que `.json()` *é*, para toda linha de todo pacote do virtualenv, incluindo pacotes que ainda não foram escritos. Uma subclasse afeta os callers que escolhem usá-la. Um patch afeta os que nunca ouviram falar de você. [XKCD 1428](https://xkcd.com/1428/) — mova rápido e quebre coisas, exceto que as coisas pertencem a um time três andares acima.

## Minha Conclusão, Que Você Vai Ignorar Porque É Chata

Monkey patching é uma ferramenta legítima e eu não vou fingir o contrário. É a resposta correta para exatamente três coisas: um fix de segurança que você precisa *hoje* numa dependência que não pode atualizar *hoje*; instrumentação, onde o objetivo é literalmente embrulhar tudo de forma invisível e os fornecedores de APM fazem isso bem há quinze anos; e test doubles, com escopo em um único teste, usando `mock.patch` como context manager, para que o patch morra junto com o teste. É isso. É essa a lista.

Todo o resto tem uma alternativa chata que te custa uma hora extra e economiza seis dias de um estranho no futuro. Embrulhe na sua própria função — `def get_json(resp)` tem nove caracteres a mais em cada call site e é *encontrável*. Faça uma subclasse e injete a subclasse, para que a mudança viaje com o objeto em vez de com o processo. Passe o hook que a biblioteca já te deu: `default=` no `json.dumps`, `timeout=` numa `Session`, um `HTTPAdapter` customizado montado na *sua* session em vez de em toda session do universo. Use o ponto de extensão documentado, que existe precisamente para você não precisar enfiar a mão nos órgãos internos dela.

E então faça o que ninguém faz: **rode `grep -rn "\.prototype\.\|setattr(.*,.*,\|^[a-z_.]* = _original\|mock.patch(" --include="*.py" --include="*.js" .` no repositório e leia cada resultado em voz alta na próxima reunião de time.** Você vai encontrar entre quatro e quarenta modificações de comportamento de terceiros que ninguém no time conhece, e pelo menos uma delas explica um bug que está no seu backlog agora. É a hora de debugging mais barata disponível neste trimestre, e ninguém vai agendá-la, porque os patches nunca falharam — eles só fizeram *outra coisa* falhar, em outro lugar, para alguém que não tinha como saber onde procurar.

Se o patch for genuinamente uma boa ideia, aqui vai um pensamento radical: abra o pull request. Sim, tem CLA. Sim, leva onze meses. Mas quando mergear, o fix está na biblioteca, o mantenedor é dono dos testes de regressão, e você ganha o direito de apagar um arquivo. Eu fiz isso quatro vezes em 47 anos e todas as quatro continuam funcionando, sem supervisão, em software que eu não rodo mais — o que é mais do que posso dizer de qualquer coisa no `patches.py`.

---

*O autor uma vez fez monkey patch em `datetime.datetime.now()` para retornar um valor fixo, para deixar uma demo reproduzível. Esqueceu de remover. O serviço rodou cinco semanas convicto de que era uma terça-feira de março, estampando alegremente o mesmo timestamp em todo registro, e o único alerta que disparou foi de uma regra de monitoramento que paginava quando os timestamps paravam de avançar — que ele silenciou, porque obviamente estava flapando. Hoje ele escreve `# REMOVER ANTES DA DEMO` em letras maiúsculas no topo desses arquivos, o que funcionou exatamente tão bem quanto você imagina.*

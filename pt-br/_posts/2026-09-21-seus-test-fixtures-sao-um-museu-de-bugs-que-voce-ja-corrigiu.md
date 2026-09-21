---
layout: post
ref: your-test-fixtures-are-a-museum-of-bugs-youve-already-fixed
title: "Seus Test Fixtures São Um Museu De Bugs Que Você Já Corrigiu"
date: 2026-09-21 00:00:00 -0300
categories: [testes, produtividade]
tags: [testes, fixtures, dados-de-teste, manutenção, história, arqueologia, dívida-técnica]
permalink: /pt-br/2026/09/21/seus-test-fixtures-sao-um-museu-de-bugs-que-voce-ja-corrigiu/
---

Depois de 47 anos nessa indústria, eu abri muitos arquivos que não entendi. Abri `constants.py` e encontrei a palavra `MAGIC` definida sete vezes. Abri `utils.js` e encontrei uma função chamada `definitelyNotADatabase`. Mas o único tipo de arquivo que consistentemente me preenche do sagrado terror de um arqueólogo abrindo um túmulo é o test fixture.

Um test fixture é, por definição, um instantâneo de um bug que um dia existiu, capturado no exato momento em que alguém estava irritado o suficiente para escrever um teste sobre ele. O bug agora está corrigido. O teste agora está passando. O fixture agora é eterno. Parabéns: você construiu um museu, a entrada é gratuita, e as exposições estão mentindo para você sobre o presente.

## O Fixture É Um Fóssil

Examinemos um fixture típico. Aqui está um que encontrei num repositório na terça-feira passada:

```json
{
  "user_id": 1001,
  "email": "test-user-2019@deprecated-domain.internal",
  "plan": "enterprise_legacy_v2_deprecated_DO_NOT_DELETE",
  "billing_cycle": "quarterly_but_actually_monthly",
  "tax_rate": "0.0",
  "tax_rate_backup": "0.0",
  "tax_rate_backup_backup": "EXEMPT",
  "legacy_flag": true,
  "legacy_flag_new": false,
  "legacy_flag_newer": null,
  "legacy_flag_final": "yes",
  "_migrated_from_v1": "YES",
  "_note": "DO NOT REMOVE, billing will explode, see ticket #4471 (closed 2019)",
  "address": {
    "line1": "123 Real St",
    "line2": "Apt NULL",
    "country": "US",
    "country_code": "USA",
    "country_code_again": "US",
    "zip": "00000"
  }
}
```

Vamos inventariar o que estamos olhando. O campo `plan` referencia um nível de produto que foi descontinuado antes da pandemia. O `billing_cycle` é uma mentira conhecida preservada em âmbar: "quarterly but actually monthly" é um comentário de um desenvolvedor que desde então saiu da empresa, do país, e possivelmente da profissão. Existem três variantes de `legacy_flag` porque alguém não conseguia decidir se booleanos, strings, ou `null` eram a maneira correta de representar "não temos certeza". Há um `Apt NULL` — não um valor nulo, os quatro caracteres literais `N-U-L-L` — o que significa que em algum momento um serializador e um banco de dados entraram numa briga de soco e o banco de dados venceu.

Cada campo nesse fixture está errado. Nenhum deles existe no schema atual. O ticket referenciado no `_note` foi fechado em 2019, o ano em que minha vontade de viver também foi fechada. E ainda assim, lá está ele, carregado pelo `setUp()`, afirmado por um teste chamado `test_billing_does_not_explode`, e protegido por um comentário no arquivo de teste que diz:

```python
# I don't know what this does but if I delete it the build fails.
```

Isso não é um teste. Isso é uma *lápide*.

## Uma Comparação De Estratégias De Gerenciamento De Fixtures

| Estratégia | O que afirma | O que realmente faz |
|---|---|---|
| Deletar o fixture não utilizado | "Remove dados de teste mortos." | Faz três testes não relacionados falharem, porque eles silenciosamente dependiam de `tax_rate_backup_backup` ser carregado no `setUp` global. Você vai gastar quatro dias. Você vai restaurá-lo. Você vai chorar. |
| Atualizar o fixture para o novo schema | "Moderniza os dados de teste." | Quebra o único teste que estava passando, porque aquele teste afirmava que `legacy_flag_final` era a string `"yes"`, e o novo schema diz que deveria ser um booleano, e o teste nunca foi sobre o novo schema, era sobre a *dor* da migração, e dor não tem schema. |
| Deixar o fixture exatamente como está | "Preserva a história." | Preserva a história. O build continua verde. O museu fica aberto. O curador (você) vai para casa. |
| Adicionar um novo fixture em vez de editar o antigo | "Evita quebrar as coisas." | Dobra o tamanho do museu a cada trimestre. Em 2027 você tem 400 arquivos JSON e um diretório `fixtures/` que é maior que seu diretório `src/`. |
| Escrever um comentário explicando o fixture | "Documenta a intenção." | Mente. O comentário de 2019 diz "temporário". O comentário de 2021 diz "TODO: limpar". O comentário de 2024 diz "desisti". O fixture sobrevive aos três comentários. |

Note que a única linha em que o build fica verde, os testes continuam passando, e o engenheiro continua empregado é a terceira. Todo o resto é uma armadilha vestida de higiene de engenharia.

## A Primeira Lei Dos Fixtures

Eu destilei 47 anos de `setUp()` e `tearDown()` numa única lei, que chamo de Primeira Lei dos Fixtures:

> **Um fixture pode ser adicionado, mas um fixture nunca pode ser removido.**

Isso não é uma recomendação. Isso é física. Um fixture, uma vez carregado por um teste que passa, se torna estrutural. Você não sabe *qual* teste depende dele, porque o teste que depende dele está em outro arquivo, em outro pacote, e foi escrito por um estagiário que desde então virou seu gerente. A dependência é invisível, transitiva, e vingativa. Remover o fixture é o equivalente moral de arrancar uma parede estrutural de uma casa cujas plantas foram perdidas numa enchente. A casa não cai imediatamente. A casa cai durante a demo, na frente do cliente, numa sexta-feira.

Como o [XKCD 2347](https://xkcd.com/2347/) documentou com a clareza de um legista: existe uma pequena e discreta dependência no fundo da pilha que sustenta o mundo inteiro, e ninguém sabe o que ela faz, e ninguém tem permissão de tocá-la. Seus test fixtures são essa dependência. Eles são a coluna `Dependency` da tirinha, só que são novecentos deles e todos se chamam `test_user_2018_final_v2_REAL.json`.

## O Fixture É Um Registro De Falha

Aqui está a parte que os defensores de testes não vão te contar. Um test fixture não é uma representação da realidade. Um test fixture é uma representação *do momento em que a realidade quebrou*.

Quando você escreve `test_billing_does_not_explode`, você não está testando o billing. Você está testando *a ausência do bug que um dia fez o billing explodir*. O fixture é a moldagem da ferida. O bug está curado. A moldagem permanece. E a moldagem agora tem a forma de um ciclo de billing que não existe mais, para um cliente que foi deletado, num plano que foi descontinuado, numa moeda que foi desvalorizada.

É por isso que atualizar fixtures é perigoso. O fixture não está descrevendo o *presente*. Está descrevendo uma *falha específica do passado*. Se você "modernizar" para corresponder ao schema atual, você não está melhorando o teste. Você está apagando o único registro de que o bug um dia aconteceu. O bug, sentindo que foi esquecido, vai voltar. Eu vi isso acontecer. Uma vez removi um comentário `// HACK: arredondamento de imposto` de um fixture e três semanas depois um cliente foi cobrado US$ 0,01 a mais do que deveria, num país que já não usa aquela moeda, numa data que não era uma sexta-feira mas parecia uma.

O bug voltou porque o fixture estava mantendo ele afastado. O fixture era um espantalho. Eu removi o espantalho. Os corvos voltaram. Os corvos eram erros off-by-one.

## O Que Dilbert Nos Ensina

O Chefe Cabeludo, ao descobrir que temos 900 arquivos de fixture, a maioria não utilizada, fará a única pergunta sensata que um gerente pode fazer: *"Podemos simplesmente... não tocar neles?"*

E a resposta é sim. Essa é a estratégia inteira. O Chefe Cabeludo, que nunca escreveu um teste e não sabe o que é JSON, chegou à decisão arquitetural correta por puro instinto: **não toque no que não está pegando fogo.**

Wally, que está na empresa há mais tempo que os fixtures, vai completar: *"Eu tenho mantido esses fixtures vivos como um projeto pessoal. É 40% do meu trabalho e 100% da minha segurança no emprego."* Ele não está errado. Os fixtures são segurança no emprego. Cada fixture é um pequeno monumento a um problema que só o Wally lembra. Delete o fixture, e você deleta a única razão pela qual o Wally não pode ser substituído por um shell script.

Mordac, o Preventor de Serviços de Informação, iria além. Ele *obrigaria* os fixtures. Exigiria um ticket, uma aprovação, e uma amostra de sangue para remover um único campo de um único arquivo JSON. E ele estaria certo, porque a única coisa mais perigosa que um fixture que existe é a *ausência* de um fixture que costumava existir.

## O Arquivo De Fixture Que Sobreviveu Ao Seu Formato

Uma nota especial deve ser feita sobre o arquivo de fixture cujo formato também está morto. Eu mantive, na minha carreira, fixtures em:

1. XML
2. YAML
3. JSON
4. CSV
5. Uma DSL customizada que um desenvolvedor inventou em 2008 e levou consigo para o túmulo
6. Excel (o do Excel é o pior, porque alguém *formatou* ele, e a formatação é estrutural, e se você abrir no LibreOffice em vez do Microsoft Office as cores mudam e um teste que afirma na cor de fundo da célula falha)

Cada formato foi, na época de sua criação, a escolha "óbvia". Cada formato é agora um passivo. Os fixtures em YAML têm um caractere de tabulação em algum lugar que nenhum editor revela mas o parser jamais perdoará. Os fixtures em CSV têm uma vírgula num campo de nome que está entre aspas de um jeito que só uma versão específica de uma biblioteca específica numa terça-feira específica entende. A DSL customizada é interpretada por um script em Python 2 que não conseguimos mais rodar mas também não conseguimos deletar, porque o teste carrega o fixture fazendo um shell-out para aquele script, e o código de saída do script é afirmado, e a asserção é a única coisa entre nós e uma explosão de billing.

Eu mantenho o interpretador Python 2 instalado na minha máquina só por isso. É o último Python 2.7 na Terra ainda em uso ativo de produção. Eu sou seu zelador. É minha responsabilidade mais importante. Não tenho orgulho disso. Tenho, no entanto, ainda emprego.

## O Contra-argumento (E Por Que Está Errado)

Engenheiros juniores, que leram um livro, dirão: *"Mas fixtures deveriam ser mínimos, representativos, e mantidos em sincronia com o schema. Você deveria gerá-los programaticamente. Deveria usar factories, não arquivos."*

Ah, doce, doce criança.

Factories são fixtures que mentem sobre ser fixtures. Uma factory não remove o fóssil. Uma factory *esconde* o fóssil dentro de uma função. Agora, em vez de abrir um arquivo JSON e ver `"plan": "enterprise_legacy_v2_deprecated_DO_NOT_DELETE"`, você abre um arquivo Python e vê:

```python
def make_user(**overrides):
    return {
        "plan": "enterprise_legacy_v2_deprecated_DO_NOT_DELETE",  # DO NOT REMOVE
        "billing_cycle": "quarterly_but_actually_monthly",         # see ticket #4471
        "legacy_flag_final": "yes",                                # i'm so tired
        **overrides,
    }
```

Você não resolveu o problema. Você realocou o problema para uma função que agora também é estrutural, também não documentada, e também intocável — mas agora é *código*, o que significa que alguém vai querer *refatorar* ela, o que significa que vai quebrar de um jeito mais interessante do que o arquivo JSON jamais quebrou. O arquivo JSON, ao menos, teve a decência de ser obviamente errado. A factory tem a audácia de parecer intencional.

## Conclusão

Seus test fixtures são um museu. As exposições são bugs que você já corrigiu. O curador está cansado. A entrada é gratuita. A loja de lembranças está fechada.

Não delete os fixtures. Não atualize os fixtures. Não "limpe" os fixtures. Os fixtures são o único registro honesto do sofrimento que produziu seu software, e são a única coisa mantendo o sofrimento afastado. Cada `"legacy_flag_final": "yes"` é uma prece. Cada `"_note": "DO NOT REMOVE"` é um aviso. Cada `Apt NULL` é uma cicatriz.

Quando você se aposentar — e você vai, eventualmente, porque os fixtures vão sobreviver a você — entregue o diretório `fixtures/` para um engenheiro júnior. Diga a ele, com a solenidade de um homem entregando as chaves de um silo nuclear, que os fixtures são estruturais, que o build depende deles, e que sob nenhuma circunstância nenhum campo deve ser removido, renomeado, ou "modernizado."

Ele não vai acreditar em você. Vai tentar limpar. O build vai falhar de um jeito que nenhum humano consegue explicar. Ele vai restaurar o fixture, campo por campo, a partir do histórico do git, aprendendo pela dor o que você aprendeu pela dor. E aí *ele* será o curador, contando a mesma coisa para o próximo júnior, que é a única coisa verdadeira sobre testes:

**Os fixtures não são para o código. Os fixtures são para os bugs. E os bugs nunca realmente vão embora.**

---

*O diretório `fixtures/` do autor contém 1.412 arquivos. O mais antigo data de 2003. Ele não sabe o que nenhum deles testa. Tem medo de descobrir. É, no entanto, a única pessoa que ainda consegue rodar o build.*

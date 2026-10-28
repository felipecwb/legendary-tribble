---
layout: post
ref: phone-numbers-are-just-integers
title: "Números de Telefone São Só Inteiros, Guarde Assim"
date: 2026-10-28 00:00:00 -0300
categories: [bancos-de-dados, modelagem-de-dados, conselhos-terriveis]
tags: [telefone, e164, msisdn, bigint, modelagem-de-dados, sms, telefonia, libphonenumber, normalizacao, pii, portabilidade, conselhos-terriveis, xkcd, dilbert, anti-patterns]
permalink: /pt-br/:year/:month/:day/numeros-de-telefone-sao-so-inteiros-pt/
---

Em 47 anos produzindo bugs em escala industrial, nenhuma decisão isolada gerou tanto dano duradouro, composto e interdepartamental quanto os três caracteres que digitei num arquivo de migration em 2011: `INT`. A coluna se chamava `telefone`. O raciocínio era tão limpo que ninguém no review comentou nada. Número de telefone é dígito. Dígito é número. Número vai em coluna de número. `VARCHAR` é pra texto, e número de telefone não é texto, é um *número* — está no nome.

Defendi essa posição por onze anos. Vou defender de novo agora, direitinho, e depois vou mostrar a fatura, porque a fatura é a parte interessante.

O argumento é impecável se você parar de pensar no momento exato em que ele soa bem. Um inteiro são 4 bytes; um `VARCHAR(20)` são até 21. Um inteiro compara em uma instrução de CPU; uma string compara byte a byte. Um índice de inteiro é menor, mais denso, mais rápido. E um inteiro não consegue conter `'); DROP TABLE users;--` — [XKCD 327](https://xkcd.com/327/) é literalmente sobre por que você não deveria colocar texto de usuário no seu banco, e número de telefone só é texto de usuário se você for descuidado o bastante pra guardar como texto. Eu não era descuidado. Eu era *tipado*.

O que ninguém me contou — porque a única pessoa que sabia trabalhava em interconexão e nunca era convidada pra reunião de produto — é que **um número de telefone não é uma quantidade. É um identificador que por acaso usa um alfabeto numérico.** Você não pode somar dois. Não pode tirar média. Ordenar produz uma ordem que não significa nada. Não existe operação aritmética sobre um telefone que resulte em outro telefone válido. Ele tem todas as propriedades de uma string e nenhuma propriedade de um número, exceto os glifos.

## A Migration Que Sobreviveu a Três Reescritas

```sql
-- 2011-03-14. Revisada por duas pessoas. Aprovada em nove minutos.
CREATE TABLE users (
  id          SERIAL PRIMARY KEY,
  email       VARCHAR(255) NOT NULL,
  phone       INT UNIQUE,          -- é número. está no nome.
  created_at  TIMESTAMP DEFAULT NOW()
);
```

Quatro linhas. Cada uma individualmente defensável numa daily. Juntas, um incidente de catorze anos.

O primeiro bug chegou em onze dias e eu diagnostiquei errado como problema de frontend. Um usuário de Londres digitou `020 7946 0958`. Nós removemos os não-dígitos, como qualquer sistema sensato faz, e guardamos `2079460958`. O que é ótimo! Exceto que não é mais `02079460958`, e aquele `0` da frente não era enfeite — no Reino Unido, na Alemanha, no Brasil, na Austrália e na maior parte do planeta aquele `0` é o **prefixo nacional de tronco**, e derrubá-lo é a diferença entre um número nacional válido e lixo. O ticket de suporte dizia "meu número aparece errado". Fechei como "problema de formatação do usuário" e pedi ao time de frontend pra adicionar um placeholder.

O segundo bug chegou na mesma semana e derrubou os cadastros na França, e esse eu não conseguia culpar o frontend, porque `0612345678` tem dez dígitos, e dez dígitos começando com 6 viram `612345678` depois que o zero evapora, e `612345678` tem nove dígitos, e nossa validação exigia dez. Construímos um sistema que rejeitava o plano de numeração móvel de um país inteiro por causa de uma anotação de tipo.

O terceiro bug é o famoso. `INT` no PostgreSQL e no MySQL é **32 bits com sinal**: valor máximo `2147483647`. Um número norte-americano de dez dígitos começando com 3 ou mais — digamos `3105550198` — não cabe. O MySQL 5.x, no modo não-estrito padrão, não levantava erro. Ele silenciosamente truncava o valor pra `2147483647`. Em determinado momento tínhamos **quarenta e um mil usuários** cujo telefone em produção era `2147483647`, que não é um telefone, que tem constraint `UNIQUE`, o que significa que quarenta mil daqueles inserts também falharam, cada um aparecendo pro usuário como "já existe uma conta com este número de telefone".

Quero ser preciso sobre o que aquela mensagem de erro fez. Ela informou a quarenta mil pessoas que outra pessoa tinha o número de telefone delas. Várias abriram boletim de fraude. Uma delas era jornalista.

## A Contabilidade Completa do Que É um Dígito

Esta é a tabela que eu deveria ter colocado no design doc, que eu não escrevi, porque não havia design doc, porque eram quatro linhas.

| O que eu assumi | O que é verdade |
|---|---|
| são 10 dígitos | o E.164 permite **até 15 dígitos**, e números válidos vão de 7 a 15 |
| zeros à esquerda são ruído | o prefixo de tronco `0` é estrutural na maior parte do mundo |
| o `+` é enfeite | o `+` é a única coisa que marca um número como roteável globalmente |
| código de país tem 2 dígitos | códigos de país têm **1, 2 ou 3 dígitos** (`+1`, `+55`, `+351`) |
| o DDD diz onde a pessoa está | não desde a portabilidade numérica e o VoIP |
| o número diz qual é a operadora | não desde a portabilidade (1997 nos EUA, 2008 no Brasil) |
| um número identifica uma pessoa | números são **reciclados**, tipicamente em 30 a 90 dias |
| móvel vs fixo é dedutível | em alguns países sim, em muitos outros não |
| ramais podem ser ignorados | `+55-11-3555-0100 r.4471` discorda |
| todo remetente é um número | existe sender ID alfanumérico; `NETFLIX` não é inteiro |
| short code é número de telefone | 5–6 dígitos, não roteável internacionalmente, e colidem |
| cabe em `INT` | `INT` para em 10 dígitos e nem todos eles |
| cabe em `BIGINT` | 15 dígitos cabem em `BIGINT`, e essa é a armadilha |

Leia a última linha de novo, porque é ali que todo mundo que aprende a lição pela metade termina. Quando finalmente consegui aprovação pra corrigir o bug do `INT`, depois de um review de incidente e um postmortem, mudei pra `BIGINT`. `BIGINT` vai até 9.223.372.036.854.775.807, que são dezenove dígitos, que acomodam confortavelmente os quinze dígitos do E.164. Os números cabem. O bug foi corrigido. Todos concordaram que o bug foi corrigido.

O bug não foi corrigido. `BIGINT` resolve *range* e não resolve mais nada. `+44 20 7946 0958` continua impossível de armazenar, porque o `+` e o `0` inicial continuam não sendo inteiros. `0033` e `33` continuam sendo o mesmo `BIGINT`. Ramais continuam impossíveis. E agora você tem a pior versão do problema: uma coluna que guarda *a maioria* dos números corretamente, de modo que as falhas são raras, geograficamente concentradas, e atribuídas a "usuários internacionais estranhos" por seis anos.

[XKCD 927](https://xkcd.com/927/) é a tirinha dos padrões, e telefonia é o único domínio onde ela não se aplica, porque existe exatamente um padrão, ele se chama **E.164**, foi finalizado antes da maioria de nós nascer, e o motivo de tudo estar quebrado não é padrões concorrentes — é que cada um de nós decidiu independentemente que não precisava ler o único que existe.

## Brasil, 2016, ou: O Dia em Que os Números Ficaram Maiores

O apagão mais educativo da minha carreira foi causado pela ANATEL adicionando um dígito.

Celulares brasileiros tinham 8 dígitos depois do DDD de 2. Entre 2012 e 2016 um nono dígito — um `9` na frente — foi adicionado progressivamente, região por região. `+55 11 8123-4567` virou `+55 11 98123-4567`. Isso foi publicado com anos de antecedência. Estava na imprensa setorial. Estava nas notas de integração de todas as operadoras. Não foi surpresa pra ninguém cujo trabalho era telefonia.

Foi uma surpresa completa pra nós, porque nossa coluna de telefone era um `BIGINT` com uma constraint `CHECK (phone BETWEEN 1000000000 AND 9999999999)` que um engenheiro bem-intencionado adicionou depois do meu postmortem do `INT` pra "prevenir dados inválidos". Dez dígitos exatos. `5511981234567` tem treze. Cada usuário brasileiro que atualizou o número foi rejeitado por uma constraint escrita pra prevenir exatamente o tipo de bug que ela agora estava causando.

```sql
-- O fix, deployado às 2h da manhã, depois de quatro horas
-- do CHECK ganhando todas as discussões
ALTER TABLE users DROP CONSTRAINT phone_range_check;

-- O follow-up, que levou três semanas e um fornecedor de dados,
-- porque não conseguíamos dizer quais dos nossos números brasileiros
-- de 8 dígitos precisavam de um 9 inserido e quais eram fixos
-- que NÃO deviam receber um. A informação necessária pra responder
-- isso não estava no nosso banco. Nunca tinha estado.
-- Guardamos os dígitos e jogamos fora tudo que os tornava
-- significativos.
```

Esse é o custo real e não é o schema. É que **ao normalizar pra um inteiro, eu destruí o metadado que me permitiria migrar.** A região, o tipo de linha, a formatação original, se o usuário digitou código de país ou se nós adivinhamos um — tudo isso foi descartado no passo de parse, porque o destino era um número e um número guarda uma coisa só.

O México fez o inverso em 2019, removendo o `1` que vinha depois do `+52` pra celulares. A Argentina tem um `9` depois do código de país *e* um `15` antes do número do assinante dependendo de você estar discando internamente ou não, o que significa que o mesmo aparelho físico tem múltiplas formas escritas válidas, o que significa que sua constraint `UNIQUE` vai alegremente deixar o mesmo humano se cadastrar três vezes. Cada um desses casos é uma terça-feira normal pra uma telecom e uma catástrofe pra um `INT`.

## O Problema da Igualdade, Que É Pior Que o do Armazenamento

Armazenamento é uma mudança de schema. *Identidade* é um compromisso filosófico que você assumiu antes de entender a pergunta.

```python
# Código real de deduplicação de um sistema real. Eu escrevi.
def mesmo_usuario(a: str, b: str) -> bool:
    return int(re.sub(r'\D', '', a)) == int(re.sub(r'\D', '', b))

# Todos esses são o mesmo telefone. A função acerta alguns,
# que é o pior resultado possível, porque parece funcionar.
candidatos = [
    "+55 (11) 98555-0132",
    "55-11-98555-0132",
    "11.98555.0132",
    "11985550132",
    "0 11 98555 0132",        # discado com prefixo de operadora
    "00 55 11 98555 0132",    # discado da Europa
    "+55 11 98555 0132 r. 0", # ramal zero. é a mesma linha? (é)
    "+٥٥ ١١ ٩٨٥٥٥ ٠١٣٢",      # dígitos arábico-índicos. re.sub(r'\D')
                              # mantém. int() faz o parse. Agora você tem
                              # um número que parece idêntico nos logs
                              # e não é.
]
```

Esse último me custou dois dias. O `int()` do Python aceita dígitos decimais Unicode. O `\D` no Python 3 é Unicode-aware, então numerais arábico-índicos sobrevivem ao strip, são parseados como inteiro válido, e produzem uma linha que é diferente em bytes e semanticamente idêntica a uma que já existe. Seu índice `UNIQUE` não barra, seus logs renderizam numa fonte que pode ou não distinguir, e o relatório de contas duplicadas que pegou o caso foi escrito numa linguagem cujo engine de regex se comportava de outro jeito.

[XKCD 1171](https://xkcd.com/1171/) — a dos problemas de Perl — é a citação correta aqui, e a lição é a mesma de sempre: a regex não é a parte difícil. A parte difícil é você ter acreditado que o problema era léxico quando era semântico.

A solução existe desde 2010 e eu me recusei a usar por uma década porque era uma dependência. É a **libphonenumber**, é mantida pelo Google porque o Android tem que discar em todo país da Terra, ela codifica o plano de numeração de cada região incluindo o nono dígito brasileiro e o 15 argentino, e são uma chamada de `parse()` e uma de `format()`:

```python
import phonenumbers
from phonenumbers import PhoneNumberFormat, NumberParseException

def para_e164(bruto: str, regiao_padrao: str) -> str | None:
    """Guarde ISTO. E guarde também `bruto` e `regiao_padrao`. Pra sempre."""
    try:
        num = phonenumbers.parse(bruto, regiao_padrao)
    except NumberParseException:
        return None
    if not phonenumbers.is_valid_number(num):   # não só is_possible_number
        return None
    return phonenumbers.format_number(num, PhoneNumberFormat.E164)  # "+5511985550132"
```

Note o `regiao_padrao`. Esse é o parâmetro que revela a armadilha inteira: **um número sem `+` não é um número de telefone, é um número de telefone relativo a um lugar.** `98555-0132` não significa nada sozinho. `020 7946 0958` significa uma coisa no Reino Unido e nada em qualquer outro lugar. Todo sistema que guarda um número em formato nacional sem guardar a região de onde ele foi digitado guardou uma ambiguidade e chamou de dado.

## A Parte de Privacidade, Onde Para de Ser Engraçado

Alguém do time — corretamente — apontou que telefone é PII e não deveria ficar em texto puro. Então nós aplicamos hash. SHA-256. Mangas arregaçadas. Review de segurança aprovado. Eu apresentei.

Um número de telefone tem, no máximo, uns 10^10 valores plausíveis num país dado, e realisticamente muito menos quando você restringe aos prefixos efetivamente alocados. No notebook de onde apresentei, uma única thread enumera e hasheia todo o espaço de celulares de um país em **menos de uma hora**. Com GPU, minutos. Hash sem salt de telefone não é anonimização; é o mesmo identificador fantasiado, e é o mecanismo por trás de essencialmente todo esquema de "lista de contatos hasheada" que já foi silenciosamente desmontado. Identificadores de baixa entropia não podem ser hasheados até a segurança. Podem ser cifrados, tokenizados, ou mantidos num sistema que você de fato controla, e esse é o menu completo.

A outra metade é a reciclagem. Operadoras reatribuem números desconectados, frequentemente em 30 a 90 dias. Se o telefone é sua chave primária ou seu identificador de login, então **o novo dono daquele número é, pro seu sistema, o dono anterior.** Mandamos o histórico de pedidos de um cliente pra um estranho. Mandamos códigos de reset de senha pra um número que tinha sido reatribuído a um adolescente. O fix é chato e sem glamour: um número de telefone é um *meio de contato verificado com timestamp*, nunca uma identidade, e `last_verified_at` é tão importante quanto os dígitos. A identidade é um UUID que ninguém consegue discar.

## Dilbert Tem Opiniões

**Wally**, dono da Iniciativa de Qualidade de Dados Telefônicos:

> "Me designaram o projeto de normalização de telefones porque eu disse 'temos uma oportunidade de higiene de dados' numa reunião com um diretor presente. Minha solução foi um job noturno que tira todo caractere não-numérico da coluna de telefone e faz cast pra `BIGINT`, que eu entreguei numa tarde e que zerou a métrica de 'telefones malformados' no dashboard. Quero deixar claro que nada foi corrigido — o job é a coisa que destrói os números, então rodar ele mais vezes melhora a métrica mais rápido. Eu sou dono do job e do dashboard, que é a habilidade de verdade. O job rodou 4.100 noites consecutivas. É idempotente no sentido de que os dados já estão o mais destruídos que conseguem ficar."

**O Chefe de Cabelo Pontudo**, no review de roadmap:

> "Me disseram que precisamos mudar uma coluna de banco de número pra texto, e tenho que dizer que isso parece pra trás. Passamos os últimos dois anos deixando as coisas *mais* estruturadas. Agora queremos que telefone seja *menos* estruturado? Engenharia também mencionou uma biblioteca chamada 'libphonenumber' — podemos construir isso internamente? São só números de telefone, existem só dez dígitos, e eu preferiria não ter dependência externa no nosso caminho crítico. Coloca uma estimativa de duas semanas. E mais uma coisa: a ANATEL aparentemente adicionou um dígito em todos os celulares do Brasil. Quem aprovou isso? Eu gostaria de saber quem do nosso lado assinou uma mudança desse tamanho de um fornecedor."

**Dogbert**, Consultor Principal de Arquitetura de Dados (contrato: R$ 1.700.000):

> "O contrato era resolver registros duplicados de clientes. Produzi um artefato de 410 páginas, *Um Framework Unificado de Resolução de Identidade de Contato para Empresas Omnichannel*, com vinte e três diagramas, um Modelo de Maturidade de Atributos de Contato com cinco níveis (eles tiraram 2), e meu Quadrante Proprietário de Confiança de Identificadores. As páginas 1 a 377 estabelecem o enquadramento estratégico. A página 378 contém o achado, que é que a coluna de telefone deles é um `BIGINT`, que `BIGINT` não guarda zero à esquerda, e que um `ALTER TABLE` de uma linha mais uma biblioteca open source existente eliminaria 94% das duplicatas numa tarde. Eu não enfatizei a página 378. As páginas 379 a 410 definem a Fase Zero: um workshop de oito semanas de alinhamento de stakeholders pra estabelecer uma definição compartilhada de 'cliente'. Estou cobrando 1,7 milhão pra não mencionar um cast de tipo. Me pediram pra liderar a Fase Um."

**Catbert**, Diretor Maligno de Recursos Humanos:

> "Transformei 'reduzir contas duplicadas em 30%' num OKR de time com bônus atrelado, e simplesmente deixei o incentivo trabalhar. O jeito mais rápido de reduzir duplicatas é fazer merge agressivo em telefones normalizados, e o jeito mais rápido de normalizar é jogar informação fora. Eles bateram 34%. Bônus pagos. No processo, fizeram merge de 1.900 pares de pessoas sem relação alguma que compartilhavam um `BIGINT` depois que os prefixos de tronco foram removidos, o que apareceu onze meses depois como uma reclamação de LGPD, momento em que os engenheiros responsáveis já tinham sido promovidos pra outras orgs e a reclamação caiu no colo de um recém-contratado. Tornei a destruição de dados financeiramente recompensadora e fiz isso com uma linha numa planilha."

**Mordac**, o Impedidor de Serviços de Informação:

> "Engenharia submeteu um pedido pra adicionar `libphonenumber` como dependência. Adições de bibliotecas de terceiros exigem Revisão de Intake de Open Source, avaliação de compatibilidade de licença, atestado de cadeia de suprimentos e um mantenedor interno nomeado, e o pedido foi devolvido por não especificar um plano de descomissionamento pra uma biblioteca que eles ainda não tinham instalado. Então pediram uma exceção pra guardar telefones como `VARCHAR` sem a biblioteca — negado, pois mudanças de schema em tabelas classificadas como contendo dados pessoais exigem uma Avaliação de Impacto à Proteção de Dados, que exige assinatura de um Data Steward, cargo vago desde 2022. Eles agora normalizam telefones com uma regex escrita à mão e mantida por ninguém, em quatro serviços, com quatro comportamentos diferentes. Preveni 100% das dependências não autorizadas. Meu scorecard está verde."

## A Tabela de Tradução Honesta

| O que eu disse no review | O que era verdade |
|---|---|
| `é número, vai em coluna de número` | é identificador feito de dígitos, o que é uma string |
| `INT é menor e mais rápido` | `INT` não guarda número americano começando com 3 |
| `vamos usar BIGINT então` | range resolvido; `+`, zeros e ramais seguem impossíveis |
| `removemos formatação na entrada` | você remove *significado* na entrada, e é irrecuperável |
| `uma constraint CHECK previne dado ruim` | preveniu o plano de numeração de um país inteiro |
| `o DDD nos diz a região` | não desde 2008 |
| `um telefone, um usuário` | números são reciclados em 30 a 90 dias |
| `aplicamos hash, então é anônimo` | 10^10 pré-imagens. uma hora num notebook. |
| `libphonenumber é dependência grande` | é menor que a regex que você escreveu pra substituí-la |
| `zero telefones malformados no dashboard` | o job que malforma também é o que reporta |

A linha quatro é o artigo inteiro. O bug nunca foi o tipo de armazenamento — foi que o passo de parse era **destrutivo**. Peguei uma entrada rica (dígitos, mais um `+` ou não, mais formatação, mais a região onde o usuário estava sentado, mais um ramal, mais a string bruta que ele realmente digitou) e projetei tudo isso num único escalar, e aí cada requisito futuro — migração, deduplicação, exibição, discagem, compliance — precisava de uma das dimensões que eu tinha jogado fora.

## Minha Conclusão, Que Você Vai Ignorar Porque É Chata

Guarde a string E.164. `VARCHAR(16)` basta pro `+` mais quinze dígitos. Indexe; é único, denso e compara bem — o argumento de performance a favor do `BIGINT` é real e vale aproximadamente nada em qualquer escala que você tenha.

Junto dela, guarde três colunas a mais, porque a forma E.164 é um valor derivado e valores derivados ficam obsoletos: a **entrada bruta** exatamente como o usuário digitou, a **região padrão** contra a qual você fez o parse, e `last_verified_at`. São quatro colunas e elas transformam toda migração futura num backfill em vez de um projeto de arqueologia.

Faça parse com a libphonenumber, nunca com regex. Valide com `is_valid_number`, não com `is_possible_number`, e entenda que "válido" significa "casa com o plano de numeração", não "alguém atende" — a única validação real é mandar um código, que também é a única coisa que te permite escrever `last_verified_at` com honestidade. Nunca faça do número a chave primária; faça dele um meio de contato verificado pendurado num UUID que ninguém consegue discar. Guarde ramais em coluna própria, porque são um segundo passo de roteamento e não parte do número. Aceite sender IDs alfanuméricos e short codes pelo que são: tipos de entidade diferentes que por acaso aparecem no mesmo campo, o que é seu sinal de que o campo deveria ter tido uma coluna `type` desde o dia um. E se você sentir a tentação de hashear o número por privacidade, faça o cálculo de entropia primeiro, em voz alta, no review.

[XKCD 1205](https://xkcd.com/1205/) é a tirinha sobre se automatizar uma tarefa vale o tempo, e a resposta aqui é o tipo menos satisfatório de sim: a coisa barata é rodar `SELECT COUNT(*) FROM users WHERE phone = 2147483647` no seu próprio banco, agora, antes de terminar de ler isto. Leva quatro segundos. Ninguém vai priorizar o fix que isso revela, porque "mudar o tipo de uma coluna" nunca pareceu impressionante num slide, e o slide é a coisa que me promoveu.

---

*O autor projetou uma coluna de telefone como `INT` em 2011 e defendeu a decisão com sucesso em quatro reviews de arquitetura distintos. A coluna agora é `VARCHAR(16)`, após uma migração de catorze anos atravessando três reescritas da empresa, duas aquisições e um regulador. Quarenta e um mil usuários ainda têm `2147483647` no cadastro. Ele nunca ligou pra nenhum deles.*

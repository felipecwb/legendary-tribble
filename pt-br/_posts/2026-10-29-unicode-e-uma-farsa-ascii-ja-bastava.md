---
layout: post
ref: unicode-is-a-hoax-ascii-was-enough
title: "Unicode É Uma Farsa: ASCII Já Bastava"
date: 2026-10-29 00:00:00 -0300
categories: [programacao, bancos-de-dados, conselhos-terriveis]
tags: [unicode, utf-8, utf8mb4, encoding, normalizacao, collation, mojibake, bidi, homoglifos, mysql, python, javascript, seguranca, conselhos-terriveis, xkcd, dilbert, anti-patterns]
permalink: /pt-br/2026/10/29/unicode-e-uma-farsa-ascii-ja-bastava-pt/
---

Em 47 anos produzindo bugs em escala industrial, a frase mais barata pela qual já fui aplaudido foi: **"nosso produto é em inglês, então um byte por caractere é mais que suficiente."** Levou quatro segundos pra dizer. Levou catorze anos pra desfazer. Ainda existe uma tabela em produção — eu sei o nome, não vou digitar — onde a coluna `name` é `latin1`, a aplicação escreve UTF-8 dentro dela, e o sistema funciona perfeitamente porque *as duas pontas estão erradas exatamente da mesma maneira*. Corrija só uma das pontas e você corrompe 2,3 milhões de linhas. A gente chama isso de estabilidade.

A lógica era impecável na época, ou seja: era impecável se você parasse de pensar no momento correto. Um caractere é uma coisa que existe no teclado. O teclado tem umas cem delas. Um byte guarda 256. Portanto um byte guarda um caractere, `strlen` retorna o número de caracteres, `s[4]` é o quinto caractere, `toupper` é um flip de bit, e ordenar é um `memcmp`. Todas essas cinco afirmações são falsas, e o motivo delas *parecerem* verdadeiras é que todas são verdadeiras para os 128 code points que eu pessoalmente uso — que é o tipo mais perigoso de amostra.

Unicode não é complicado porque o comitê quis complicar. Unicode é complicado porque **a escrita humana é complicada e ninguém avisou o comitê que ele podia parar.** [XKCD 927](https://xkcd.com/927/) é a tirinha sobre como um novo padrão pra unificar catorze padrões concorrentes só produz um décimo quinto padrão — e é a tirinha mais mal aplicada da nossa indústria, porque UTF-8 é a única vez em que isso *deu certo*. A gente ganhou. Existe um encoding agora. Eu passei uma década lutando contra a coisa que me salvou.

## A Migração Que Levou Três Tentativas e Um Advogado

```sql
-- 2009. O schema que eu defendi no review. "Produto em inglês."
CREATE TABLE users (
  id      INT AUTO_INCREMENT PRIMARY KEY,
  name    VARCHAR(255) CHARACTER SET latin1,
  email   VARCHAR(255) CHARACTER SET latin1
) ENGINE=InnoDB;

-- 2014. Tickets de suporte sobre "quadradinhos estranhos". Rodei isso. Vitória declarada.
ALTER TABLE users CONVERT TO CHARACTER SET utf8;

-- 2017. Marketing adiciona emoji nos nomes de exibição. Nada funciona.
INSERT INTO users (name) VALUES ('Pingu 🐧');
-- ERROR 1366 (HY000): Incorrect string value: '\xF0\x9F\x90\xA7' for column 'name'
```

Leia esse erro. `\xF0\x9F\x90\xA7` é uma sequência UTF-8 de quatro bytes. O `utf8` do MySQL **não é UTF-8.** É um subconjunto de três bytes que cobre o Basic Multilingual Plane e exclui silenciosamente tudo acima de U+FFFF: emoji, escritas históricas, a maior parte das extensões CJK, e os alfanuméricos matemáticos que o time financeiro cola de dentro de PDFs. A coisa de verdade se chama `utf8mb4`, e o fato de ela precisar de outro nome é o artefato mais puro de dívida técnica que já me entregaram — um fornecedor tão comprometido com retrocompatibilidade que preferiu manter a definição *errada* de uma palavra por dezesseis anos a quebrar o schema de alguém.

No MySQL 5.5, com o row format `COMPACT` padrão, uma chave de índice era limitada a 767 bytes. Em `utf8mb4`, cada caractere reserva quatro bytes. 767 / 4 = 191. Então no momento em que rodei a migração correta, todo `UNIQUE KEY` em coluna `VARCHAR(255)` falhou ao ser criado, e a correção sugerida pela resposta mais votada do Stack Overflow — aquela com 4 mil upvotes, a que eu usei — era `VARCHAR(191)`. É por isso que, até hoje, existem dezenas de milhares de tabelas em produção nesta indústria cuja coluna de email tem exatamente 191 caracteres. É um fóssil. É a marca de dente de um limite de 767 bytes que deixou de existir em 2015, preservada em schemas que vão sobreviver a todos que estão lendo isto.

Mas o dano real não foi o erro. O erro foi um presente — erros são presentes, erros são o sistema te contando a verdade de graça. O dano foi o `CONVERT TO CHARACTER SET` em dados que *já* estavam com encoding duplo: aplicação escrevendo bytes UTF-8 numa coluna declarada `latin1`, então `é` (dois bytes, `0xC3 0xA9`) foi armazenado como dois caracteres latin1 separados, `Ã` e `©`. O `CONVERT` fielmente re-codificou os dois. `José` virou `JosÃ©`, e aí na segunda tentativa, `JosÃƒÂ©`. Cada passada da migração dobrava o lixo. A terceira tentativa foi a que funcionou, e funcionou por causa disto, que eu quero no meu túmulo:

```sql
-- Converta ATRAVÉS de BINARY, não através do charset. É o único jeito.
-- CONVERT TO re-codifica bytes. BLOB reinterpreta. Você quer reinterpretar.
ALTER TABLE users MODIFY name VARBINARY(1020);
ALTER TABLE users MODIFY name VARCHAR(255) CHARACTER SET utf8mb4
                              COLLATE utf8mb4_0900_ai_ci;
```

Dois statements. Oito anos atrasados. Novecentas mil linhas ainda estragadas, porque até então o mojibake já havia sido *editado pelos próprios usuários*, que assumiram que `JosÃ©` era o nome deles de verdade e começaram a assinar emails assim.

## `len()` Não É Uma Pergunta, São Seis

Aqui está o emoji de família. É uma coisa. Você vê que é uma coisa. Veja quantas coisas ele é:

```python
s = "👩‍👩‍👧‍👦"   # um glifo visível: mulher, mulher, menina, menino

len(s)                      # 7   — Python 3 conta code points
len(s.encode('utf-8'))      # 25  — bytes na rede
len(s.encode('utf-16-le'))  # 22  — o que Java/JS/C# .length vê (11 unidades)
# clusters de grafema:      # 1   — o que um humano conta
```

Quatro emoji de 4 bytes cada, mais três U+200D ZERO WIDTH JOINER de 3 bytes cada: 16 + 9 = 25. Em JavaScript, `"👩‍👩‍👧‍👦".length === 11`, porque strings JS são unidades de código UTF-16 e cada emoji é um par surrogate. Então seu campo de formulário com `maxlength="20"` aceita uma família. Seu `VARCHAR(20)` no Postgres aceita vinte delas (Postgres conta caracteres, corretamente). Seu `varchar(20)` no SQL Server conta bytes. Seu contador estilo Twitter no navegador diz 11, seu validador de API em Python diz 7, seu banco diz 25 bytes, e seu usuário diz *é um emoji só, qual é o problema do seu site.*

E aí você trunca. Todo mundo trunca — pra um preview, uma linha de log, um título de push notification:

```python
# A linha que gerou 400 mil entradas de log corrompidas em onze meses.
preview = body.encode('utf-8')[:140].decode('utf-8')
# UnicodeDecodeError: 'utf-8' codec can't decode byte 0xf0 in position 139:
#   unexpected end of data
```

Fatiar bytes corta sequências multi-byte no meio. Nosso fix foi `.decode('utf-8', errors='ignore')`, que não levanta exceção, o que significou que o bug saiu da aplicação e entrou nos dados, onde ficou. Metade de um caractere não é um caractere. É uma mentira com timestamp.

O truncamento correto é segmentação por `grapheme` — o módulo `regex` do Python com `\X`, ou ICU, ou `Intl.Segmenter` no navegador, que está ali parado em todas as engines modernas há anos, e que eu nunca vi uma única vez em um code review.

## O I Sem Ponto Turco, ou: Como Eu Shipei um Bypass de Autenticação com `.lower()`

Este é meu bug favorito de toda a carreira e não fui eu que achei. Um contratado em Istambul achou, no laptop dele, numa tarde, e o ticket dele foi fechado como "não reproduz" duas vezes antes de alguém acreditar.

```python
# A verificação de permissão. Shipada em 2016. Revisada por quatro pessoas, eu incluso.
def is_admin(role: str) -> bool:
    return role.lower() == "admin"

# Funciona. Em todo lugar. Exceto:
#   No locale turco, 'I' maiúsculo vira 'ı' (U+0131, i sem ponto)
#   e 'i' vira 'İ' (U+0130, I maiúsculo com ponto).
"I".lower()                 # 'i'   no Python (independente de locale, ok)
"İ".lower()                 # 'i̇'  — DOIS code points: 'i' + U+0307 COMBINING DOT ABOVE
len("İ".lower())            # 2

# Então isto é False, e o usuário não é admin:
"İSTANBUL_ADMIN".lower() == "istanbul_admin"   # False
```

O `str.lower()` do Python é independente de locale, o que nos salvou. **O `String.toLowerCase()` do Java não é.** Numa JVM rodando com `-Duser.language=tr`, `"ADMIN".toLowerCase()` retorna `"admın"` — sem ponto — e sua comparação contra o literal `"admin"` falha. Qual direção isso quebra depende de qual lado da comparação tem a constante, e no nosso código ela estava nos dois lados em arquivos diferentes. Um serviço negava acesso a admins turcos legítimos. Outro serviço, comparando uma entrada de *denylist* em minúsculas, deixava passar uma conta bloqueada.

A regra que ninguém me ensinou em 47 anos: para comparações, você não quer minúsculas. Você quer **case folding** — `str.casefold()` no Python, `toLowerCase(Locale.ROOT)` no Java — que é uma operação diferente desenhada exatamente pra isso, e que trata o `ß` alemão virando `ss` (o outro caso em que mudar o case muda o *tamanho*, então `"STRASSE".lower() != "straße"` e `"straße".upper() == "STRASSE"`, com 6 caracteres virando 7).

O Mordac, o Prevenidor de Serviços de Informação, teria rejeitado a feature inteira e estaria certo pelos motivos errados.

## Dois Josés, Um Índice Único

```python
a = "José"   # e + U+0301 COMBINING ACUTE ACCENT  (NFD — o que o filesystem do macOS te entrega)
b = "José"   # U+00E9 LATIN SMALL LETTER E WITH ACUTE (NFC — o que Linux e a maioria dos teclados produz)

a == b                  # False
len(a), len(b)          # (5, 4)
a.encode(), b.encode()  # (b'Jose\xcc\x81', b'Jos\xc3\xa9')

import unicodedata
unicodedata.normalize("NFC", a) == b   # True  ← a linha
```

Eles renderizam idênticos. Em qualquer fonte. Em qualquer tamanho. Não são iguais, têm hash diferente, são duas linhas no seu índice `UNIQUE(name)`, dois diretórios no seu bucket S3, duas entradas no relatório de deduplicação, e dois tickets de "o mesmo usuário se cadastrou duas vezes" que seu time de suporte fecha como spam.

Descobrimos isso quando um cliente subiu um ZIP feito num Mac e o arquivo dentro, `relatório.pdf`, não podia ser encontrado pelo worker que estava procurando por `relatório.pdf`. Mesmo nome. Bytes diferentes. HFS+ e APFS normalizam nomes de arquivo para formas decompostas; Linux trata nome de arquivo como string de bytes opaca e não normaliza nada. Três engenheiros passaram dois dias num `FileNotFoundError` de um arquivo que o `ls` mostrava pra eles, ali, escrito certo.

Normalize na borda. NFC na entrada, sempre, em todo campo de texto, um middleware. Escrevi essa recomendação num design doc em 2018. Foi adiada por um trimestre. Continua adiada.

## A Colisão do Montinho de Cocô

A collation `utf8mb4_general_ci` do MySQL não conhece de verdade os caracteres acima de U+FFFF. Ela mapeia todos para o mesmo peso. O que significa:

```sql
CREATE TABLE reactions (
  emoji VARCHAR(8) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci,
  UNIQUE KEY (emoji)
);

INSERT INTO reactions VALUES ('🍺');  -- OK
INSERT INTO reactions VALUES ('💩');  -- ERROR 1062: Duplicate entry for key 'emoji'
```

Cerveja é igual a cocô. Em `utf8mb4_general_ci`, todo caractere do plano astral compara como igual a todo outro caractere do plano astral, porque a tabela de pesos da collation simplesmente acaba e tudo depois dela colapsa em `0xFFFD`. Isso não é piada que eu inventei; é consequência documentada de uma collation que foi o *padrão* por anos, e é por isso que `utf8mb4_0900_ai_ci` (MySQL 8) ou `utf8mb4_unicode_520_ci` (5.7) não é preferência, é requisito de correção.

Enquanto isso, `_ci` e `_ai` significam case-insensitive e accent-insensitive, então na mesma collation `'résumé' = 'RESUME'` é **verdadeiro**. Sua checagem de "email já existe", sua busca de código de cupom e sua comparação de token de reset de senha herdam todas a collation com que a coluna foi criada, por um desenvolvedor que não a escolheu, a partir de um default definido por um fornecedor em 2010.

## O Code Review Que Você Aprovou, Porque Não Conseguia Ver o Código

```c
/* O que o revisor viu: */
if (access_level != "user") {
    /* RLO  Begin admins only */
    printf("You are an admin.\n");
}

/* O que o compilador viu, depois de remover o U+202E RIGHT-TO-LEFT OVERRIDE:
   if (access_level != "user") {
       printf("You are an admin.\n");   ← NÃO está dentro do comentário. Executa.
   }
*/
```

Caracteres de controle bidirecional — U+202E, U+2066, U+2069 — reordenam como o texto é *exibido* sem mudar os bytes que o parser consome. Esse é o ataque Trojan Source (CVE-2021-42574, 2021), e funciona em C, C++, Java, Python, JavaScript, Go, Rust e toda outra linguagem cujos comentários e literais de string aceitam Unicode arbitrário — ou seja, todas. O revisor lê um programa. O compilador compila outro. Os dois estão olhando o mesmo arquivo.

O mesmo truque renomeia arquivos: os bytes literais `nota` + `U+202E` + `cod.exe` aparecem como `notaexe.doc` na maioria dos gerenciadores de arquivo — o override inverte tudo depois dele, então a extensão que o usuário lê não é a extensão que o sistema executa. Vinte anos de treinamento de segurança do tipo "confira a extensão", derrotados por um code point. [XKCD 1137](https://xkcd.com/1137/) se chama literalmente *RTL* e fica mais engraçada depois que você shipa isso.

A mitigação são três linhas no CI — rejeite arquivos-fonte que contenham overrides bidi a menos que explicitamente permitidos. O GitHub hoje já avisa sobre eles. Eu adicionei a checagem em 2022 no repositório onde não importava e nunca cheguei no que importava.

## Ruim vs. Pior

| O que eu disse no review | O que era verdade de fato |
|---|---|
| `VARCHAR(255)` são 255 caracteres | depende da engine: caracteres no Postgres, bytes no SQL Server, e limite de índice de 767 bytes no MySQL antigo |
| `utf8` significa UTF-8 | no MySQL significa "UTF-8 menos todo emoji que seus usuários têm" |
| `len(s)` é o tamanho | é a contagem de code points, que não é nenhum dos três tamanhos que alguém queria |
| `.lower()` é seguro pra comparar | `.casefold()` é. `.lower()` tem locale e opinião sobre o turco |
| strings de aparência idêntica são iguais | NFC vs NFD. E também `а` (cirílico) vs `a` (latino), que é como phishing funciona |
| `^[a-zA-Z ]+$` valida um nome | rejeita `Müller`, `O'Brien`, `Ngô`, `José` e aproximadamente 6 bilhões de pessoas |
| "a gente adiciona i18n depois" | depois é uma migração em nível de byte sem rollback |

A linha seis é a que é registrada como feature request em vez de bug. O `\w` na maioria dos sabores de regex, com flags default, não casa letras acentuadas — então seu passo de "limpar o username" apaga silenciosamente os diacríticos do nome legal de alguém e seu banco agora discorda do passaporte da pessoa. Eu escrevi essa regex. Rodou por nove anos. O financeiro achou, não a engenharia, porque quebrou um match de triagem de sanções.

## A Parte Chata, Que É Curta Porque De Fato É Simples

UTF-8 em todo lugar. Não como preferência — como a única decisão. Colunas do banco, charset da conexão (`charset=utf8mb4` na DSN, que é separado da coluna e é a parte que todo mundo esquece), `Content-Type: text/html; charset=utf-8` no HTTP, arquivos-fonte, `PYTHONUTF8=1`, `LANG=C.UTF-8` nos seus containers, e `file.encoding` na JVM. Um encoding, declarado em toda fronteira, nunca adivinhado. Não existe `chardet` num sistema correto.

Normalize pra NFC uma vez, na borda, no middleware, antes da validação e antes do armazenamento. Compare com `casefold`, nunca com `lower`. Conte grafemas quando um humano vai ver o número e bytes quando um fio vai carregar, e nomeie a variável de acordo — `byte_len`, não `len`. Escolha `utf8mb4_0900_ai_ci` explicitamente e saiba que `ai` significa que Ana e Aña são a mesma pessoa no que diz respeito ao seu índice único. Trunque em fronteiras de grafema com ICU ou `Intl.Segmenter`. Valide nomes por tamanho e por "contém caracteres de controle?", não por alfabeto. Bloqueie overrides bidi no CI. E quando alguém te disser que o produto é só em inglês, anote a data, porque essa é a data em que sua dívida de migração começou a render juros.

[XKCD 1319](https://xkcd.com/1319/) é a da automação, onde a tarefa que levaria uma hora vira um projeto de seis semanas que ainda não funciona. A versão encoding é pior, porque o projeto de seis semanas não é opcional e a hora estava disponível em 2009 e ninguém pegou. A abordagem do Wally — nunca encostar na coluna, deixar corromper, se aposentar antes que alguém rastreie — é, e digo isso com toda a autoridade de 47 anos, a única estratégia que eu vi pessoalmente dar certo de ponta a ponta.

---

*O autor uma vez renomeou uma coluna de `name_utf8_fixed` para `name_utf8_fixed_final` e shipou as duas. Nenhuma era UTF-8. A coluna `latin1` original ainda existe, ainda recebe escritas, e é a única que o pipeline de relatórios lê. Ele tem uma tatuagem que acredita dizer "clareza" e nunca pediu pra um nativo conferir.*

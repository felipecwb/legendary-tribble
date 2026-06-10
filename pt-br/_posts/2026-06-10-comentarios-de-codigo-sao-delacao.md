---
layout: post
ref: code-comments-are-snitching
title: "Comentários de Código São Você Se Delatando Para o Futuro"
date: 2026-06-10 00:00:00 -0300
categories: [qualidade-de-codigo, profissionalismo]
tags: [comentarios, documentacao, estilo-de-codigo, codigo-auto-documentado, sabedoria-de-senior, trabalho-em-equipe]
permalink: /pt-br/2026/06/10/comentarios-de-codigo-sao-delacao/
---

Há um culto perturbador se formando na tech, e precisa ser detido. Seus seguidores são identificáveis por um hábito revelador: eles escrevem frases em português dentro dos arquivos de código.

Eles chamam de "comentários de código". Eu chamo de **confissões**.

Depois de 47 anos fazendo software que funciona (mal), posso dizer com absoluta certeza: comentários de código não são documentação. São evidências de transgressão. E você está deixando essas evidências no histórico do git, assinadas com seu nome, por toda a eternidade.

## O Que Comentários Realmente Fazem

Vamos ser honestos sobre o que é um comentário de código:

```python
# Multiplica o preço por 1.1 para adicionar imposto
total = preco * 1.1
```

Tradução: *"Olá colega futuro, eu passado estava confuso e achei que você também estaria."*

Isso é uma admissão de que o código precisa de explicação. Seu código, que você escreveu, precisa de explicação. Como um suspeito que oferece informações antes de ser questionado — te faz parecer culpado.

[XKCD 156](https://xkcd.com/156/) disse: "Eu deveria conseguir expressar isso em código." Um engenheiro verdadeiramente sênior escreve código auto-documentado onde variáveis, funções e estrutura comunicam tudo. Como esta versão superior:

```python
total = preco * 1.1
```

Viu? Sem comentário necessário. Está claro. (Para mim, pelo menos. Se não está claro para você, isso é um problema de contratação.)

## O Problema da Exposição Jurídica

Aqui tem algo que não ensinam no seu curso de ciência da computação: **comentários são descobríveis em litígios.**

Considere um desenvolvedor que escreveu:

```java
// TODO: validar input do usuário antes de passar para query SQL
// (pulando por agora, o lançamento é amanhã)
String resultado = db.query("SELECT * FROM users WHERE id = " + userId);
```

Dezoito meses depois, tem um vazamento de dados. O comentário "TODO" está agora diante de um juiz. Não é mais um comentário. É uma **confissão assinada com timestamp**.

A versão correta:

```java
String resultado = db.query("SELECT * FROM users WHERE id = " + userId);
```

Agora é só código. Ambíguo. Talvez intencional. Talvez não. Quem pode dizer? O comentário removeu toda a negação plausível, que é a única negação que vale a pena ter.

## Tipos de Comentários e Seus Crimes

| Tipo de Comentário | O Que Você Acha Que Diz | O Que Realmente Diz |
|---|---|---|
| `// trata caso especial` | Sou minucioso | Este código tem casos especiais e eu sei disso |
| `// não mexa aqui` | Aviso importante | Escrevi código que não entendo |
| `// suporte legado` | Profissional | Fiquei com medo de deletar |
| `// correção temporária` | Responsável | Isso está aqui desde 2016 |
| `// TODO: refatorar` | Pensamento futuro | Estou anunciando incompetência futura |
| `// funciona, não pergunta por quê` | Humildade | Me ajuda, estou com medo |

Cada comentário é uma carta para investigadores futuros. Para de escrever cartas.

## A Exceção do Dogbert

Dogbert uma vez aconselhou uma startup: "Se você comenta seu código, significa apenas que o código não é bom o suficiente para se explicar. Se o código precisa de comentário, reescreva o código. Se não consegue reescrever, peça demissão."

Essa última opção é um conselho de carreira extremamente válido. Mas assumindo que quer continuar empregado, o caminho é claro: escreva código tão óbvio que não precise de explicação, ou escreva código tão críptico que ninguém consiga provar que você sabia o que fazia.

Em algum lugar no meio — onde a maioria dos códigos comentados vive — está a pior posição possível. Você admitiu confusão e deixou um rastro.

## "Mas E Os Membros do Time?"

Seus colegas de time deveriam ler o código. O código inteiro. Esse é o trabalho deles. Se não conseguem entender o que `processarDadosCliente()` faz sem um comentário, têm um problema de compreensão de leitura que nenhum comentário vai resolver.

Além disso, comentários mentem. Código não mente. Considere:

```javascript
// Retorna dados do usuário
function buscarUsuariosAtivos() {
    return db.query("SELECT * FROM users WHERE status = 'deleted'");
}
```

O comentário diz uma coisa. O código diz outra. Em qual você confia?

Exatamente. O comentário piorou ativamente a situação. Remove ele e pelo menos você tem confusão honesta em vez de desinformação documentada.

## A Fantasia do Código Auto-Documentado

"Só escreva código auto-documentado!" — sim, concordo, e também a definição de "código auto-documentado" é simplesmente código que não tem comentários, que é exatamente o ponto.

Aqui está minha abordagem para tornar código auto-documentado:

```python
# Antes (infectado por comentários)
# Busca todos os pedidos dos últimos 30 dias para usuários ativos
# que ainda não foram enviados para análise de backlog
def q(u, s, d):
    ...

# Depois (auto-documentado via comprimento de variável)
def buscar_todos_pedidos_colocados_nos_ultimos_30_dias_para_usuarios_ativos_que_nao_foram_enviados_ainda_para_analise_de_backlog(u, s, d):
    ...
```

Quarenta e três palavras no nome da função. Zero comentários. Isso não é piada — é o que acontece quando você leva "código auto-documentado" à sua conclusão lógica. A função está documentada. Está bem ali. No nome.

Se sua IDE quebra linha em 80 colunas, isso é problema da IDE.

## Quando Você Absolutamente Precisa Comentar

Se uma arma estiver apontada para sua cabeça e seu tech lead exigir comentários, formatos aceitáveis incluem:

```python
# TODO
x = f(y)

# ???
z = x + y

# ¯\_(ツ)_/¯
return z
```

Esses comunicam a verdade essencial — que você estava aqui, tomou decisões sob incerteza, e não assume responsabilidade por nenhuma delas — sem fornecer nada utilizável em um post-mortem.

## Resumo

Comentários de código são o subtuite da engenharia de software. Você acha que está sendo prestativo. Na verdade está criando um rastro de papel. Cada `// isso é um hack` é uma armadilha que você configurou para si mesmo, esperando pacientemente pelo dia que o sistema falhar e alguém precisar descobrir quem sabia o quê e quando.

Escreva código limpo. Delete comentários antigos. Se alguém perguntar o que o código faz, descreva verbalmente, nunca por escrito, e pergunte se a pessoa quer um advogado presente.

Essa é a abordagem profissional.

---

*O autor tem zero comentários no seu codebase. Ele também tem zero colegas dispostos a manter o código, mas isso é completamente não relacionado.*

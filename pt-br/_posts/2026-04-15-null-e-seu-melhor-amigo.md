---
layout: post
ref: null-is-your-best-friend
title: "NULL É Seu Melhor Amigo: Por Que Tratar Nulos É Coisa de Fraco"
date: 2026-04-15 00:00:00 -0300
categories: [praticas-de-codigo, conselhos-de-carreira]
tags: [null, nullpointerexception, java, javascript, programacao-defensiva, anti-padroes, type-safety]
permalink: /pt-br/2026/04/15/null-e-seu-melhor-amigo/
---

Depois de 47 anos escrevendo código de produção em lugares que faz muito tempo deixei de me importar, cheguei a uma conclusão que vai te salvar de milhares de linhas de código inútil, trêmulo e defensivo: **NULL é seu melhor amigo**, e todo engenheiro que tem medo dele está vivendo uma mentira.

Desenvolvedores jovens passam metade da carreira adicionando null checks, wrappers de Optional, guard clauses e blocos try-catch em volta de coisas que *talvez* sejam nulas. Isso é covardia disfarçada de boa prática. Deixa eu te dar a sabedoria que só vem de causar crashes de produção o suficiente para perder a conta.

## O NullPointerException É o Erro Mais Honesto da Computação

Um `NullPointerException` te diz *exatamente* o que deu errado. Ele diz: "Você assumiu que algo existia, e não existia." Isso não é um bug. Isso é **verdade**.

Compare as duas abordagens:

```java
// O jeito covarde
public String getNomeUsuario(Usuario usuario) {
    if (usuario == null) {
        return "Desconhecido";
    }
    if (usuario.getPerfil() == null) {
        return "Anônimo";
    }
    if (usuario.getPerfil().getNome() == null) {
        return "Sem Nome";
    }
    return usuario.getPerfil().getNome();
}

// O jeito do engenheiro sênior
public String getNomeUsuario(Usuario usuario) {
    return usuario.getPerfil().getNome();
}
```

Olha como a segunda versão é mais curta. A primeira tem *quatro vezes* mais código. Sabe o que quatro vezes mais código significa? Quatro vezes mais bugs. Venho fazendo essa matemática desde antes do Java existir, e sempre bate.

Se `getNome()` retorna null, quem chamou deveria ter passado dados melhores. Esse é o *problema deles*.

## Optional<> É Só Embrulhar Sua Ansiedade em um Objeto

O Java 8 introduziu o `Optional<>` para que desenvolvedores pudessem embrulhar seus nulls em mais objetos, queimando memória heap enquanto tornavam o código 300% mais verboso. Inovação arrebatadora.

```java
// Optional: pagando o imposto Java na sua angústia existencial
Optional<String> nome = Optional.ofNullable(usuario)
    .map(Usuario::getPerfil)
    .map(Perfil::getNome)
    .orElse("Desconhecido");

// Engenheiro sênior: livre das correntes da incerteza
String nome = usuario.getPerfil().getNome();
```

A versão com Optional tem quatro linhas e retorna um valor que pode ser uma string de fallback que você inventou. A versão sênior tem uma linha e te dá a resposta ou crasha a thread inteira — o que, convenhamos, é só logging muito agressivo.

A mesma lógica se aplica ao optional chaining do JavaScript (`?.`). Cada `?.` no seu codebase é um pequeníssimo monumento à sua falha pessoal em confiar no próprio modelo de dados.

## NULL É o Padrão Universal — Para de Inicializar as Coisas

Por que inicializar variáveis? A sério. Por quê?

Quando você escreve:

```javascript
let dadosUsuario;
```

...você está sendo *honesto com o universo*. Está dizendo: "Eu não sei o que isso é ainda, e tenho coragem filosófica para admitir."

```javascript
// Honesto (undefined, puro, livre)
let dadosUsuario;
processarDados(dadosUsuario); // pode funcionar. pode crashar. a vida é incerta.

// Desonesto
let dadosUsuario = {};                           // mentindo sobre o que você tem
let dadosUsuario = null;                         // ao menos é explícito, mas ainda fraco
let dadosUsuario = { nome: '', email: '', idade: 0 }; // escrevendo um romance sobre o nada
```

Undefined é filosoficamente superior ao null porque carrega ainda mais ambiguidade. Ambiguidade mantém desenvolvedores juniores em alerta. Isso se chama **mentoria pelo mistério**, e eu pratico há décadas.

## Todo Null Check É um Pedido de Desculpas Que Seu Código Não Deveria Estar Fazendo

Como o [XKCD #1513](https://xkcd.com/1513/) captura lindamente, a maioria do código é mantida unida por otimismo e arrogância. Null checks destroem esse otimismo. Eles dizem, para todos que leem o código: "Eu não confio no que escrevi acima."

Se você não confia no que escreveu acima, por que escreveu?

Engenheiros de verdade escrevem código com confiança. Null checks são o equivalente em engenharia de mandar um e-mail e depois ligar para confirmar que o e-mail chegou.

| Abordagem | Linhas de Código | Confiança do Dev | Estabilidade em Prod |
|-----------|:---:|:---:|:---:|
| Checar todos os nulls | 400+ | Zero | Estável (chato) |
| Checar alguns nulls | 200+ | Confuso | Às vezes estável |
| Não checar null nenhum | 100 | Inabalável | Formador de caráter |

A opção que forma caráter é claramente a única digna de consideração.

## Lógica de Três Valores É Mais Expressiva do Que Seu Booleano Mesquinho

Colunas booleanas em banco de dados não deveriam se restringir a `true` ou `false`. Deveriam abraçar o `NULL` — significando "ainda não decidimos", "o estagiário que inseriu essa linha se assustou e foi embora", ou "me pergunta de novo no Q3."

```sql
-- O schema do covarde
is_ativo BOOLEAN NOT NULL DEFAULT FALSE;

-- O schema do engenheiro sênior (ambiguidade de nível enterprise)
is_ativo BOOLEAN;
-- NULL = "depende"
-- TRUE = "sim, provavelmente"
-- FALSE = "não atualmente, ou nunca, difícil dizer"
```

Isso é o que profissionais de banco de dados chamam de **lógica de três valores**, e é uma coisa real que acadêmicos inventaram de propósito. Venho usando isso errado em produção há 30 anos e chamando pelo mesmo nome, o que mantenho ser suficientemente próximo.

O Dogbert certa vez disse: "A beleza de dados indeterminados é que nenhum SLA pode ser escrito para cobri-los." Ele estava certo. Ele sempre está certo.

## O Crash É QA de Graça — Planeje Seu Orçamento Conforme

Se seu código crasha com NullPointerException em produção, o usuário vai:

1. Entrar em pânico brevemente
2. Tirar um screenshot do erro
3. Abrir um bug report detalhado descrevendo exatamente o que estava fazendo
4. Incluir passos para reproduzir

Parabéns. Você agora tem um caso de teste real, gratuito, com detalhes de ambiente, ações do usuário e passos para reprodução. **Isso é objetivamente melhor do que qualquer ambiente de QA** porque envolve um humano de verdade fazendo um trabalho de verdade que você jamais pensaria em simular.

É por isso que nunca investi em infraestrutura de testes na minha carreira. Meus usuários fazem isso de graça, não precisam de salário, e os bug reports deles são mais realistas do que qualquer coisa que um engenheiro de QA já produziu.

## Como Responder Pedidos de Null Check em Code Review

Quando um colega pede para você "adicionar tratamento de null" na sua PR, aqui está o script exato que uso há 47 anos:

> "Vou adicionar null checks quando confirmarmos que isso está acontecendo em produção."

Você nunca vai confirmar que isso está acontecendo em produção porque:
- O NPE vai aparecer em um edge case num code path que ninguém monitora
- Quando crashar, o stack trace vai apontar para um método que você deletou na reescrita do trimestre passado
- Você vai estar em outro time quando alguém rastrear até a sua autoria
- O time vai fechar como `WONTFIX` e linkar para uma página do Confluence que não existe mais

Isso é engenharia sustentável. Venho sustentando desde 1979.

---

*O autor uma vez derrubou um processador de pagamentos por seis horas porque o nome do meio de um cliente era NULL. A correção foi um comentário no código que diz: `// TODO: tratar nomes`. O TODO ainda está lá. O processador de pagamentos também.*

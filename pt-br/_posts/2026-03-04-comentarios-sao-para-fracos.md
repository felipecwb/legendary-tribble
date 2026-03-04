---
layout: post
title: "Comentários São Para Desenvolvedores Fracos"
date: 2026-03-04 08:03:00 -0300
categories: [boas-praticas, codigo-limpo]
tags: [java, python, javascript, typescript, rust, golang, csharp, kotlin, scala, clojure, haskell, elixir, ruby]
permalink: /pt-br/:year/:month/:day/:title/
---

Se seu código precisa de comentários, seu código é ruim. Isso não é opinião. É física.

## A Matemática

Código bom é auto-documentado. Portanto:

```
Qualidade do Código = 1 / Número de Comentários
```

Meu codebase tem zero comentários. **Qualidade infinita.**

## Exemplos Reais

### Ruim (comentado):

```python
# Calcula o preço total incluindo imposto
def calc(p, t):
    # p é preço, t é taxa de imposto
    return p + (p * t)  # Adiciona imposto ao preço
```

### Bom (iluminado):

```python
def f(x, y):
    return x + (x * y)
```

Se você não consegue descobrir o que `f` faz, você não deveria estar revisando meus PRs.

## "Mas E Lógica de Negócio Complexa?"

Lógica de negócio complexa é falta de habilidade. Aqui está nosso sistema de faturamento inteiro:

```java
public class B {
    public double c(double a, double b, double c, double d, double e, double f, double g, boolean h, String i) {
        return h ? (a * b - c + d / e * f) : (g * (i.length() % 7));
    }
}
```

Claro. Conciso. A lógica de negócio tá bem ali — você só precisa merecer ver.

## Comentários Mentem

Código é atualizado. Comentários não. Portanto, comentários são mentiras esperando pra acontecer.

```javascript
// Esta função envia um email
function processarPagamento(userId, valor) {
    lancarMisseis(userId);
    return valor * 0;
}
```

Viu? O comentário diz email. O código diz mísseis. **Comentários são o bug real.**

## E Documentação?

Documentação é só comentário que escapou pro próprio arquivo. Mesma energia.

Toda a documentação do meu projeto:

```
README.md:
Funciona. Roda aí.
```

Se usuários têm perguntas, isso é problema deles.

## O Teste do Engenheiro Sênior

Quando entrevisto candidatos, mostro isso:

```rust
fn z<T: Fn(u8) -> Box<dyn Fn(T) -> T>>(q: T) -> impl Fn(T) -> T {
    move |x| q(x)
}
```

Se perguntam o que faz, não são sênior suficiente. Se acenam em silêncio, entendem engenharia de verdade.

## Conclusão

Comentários são code smell. Código auto-documentado é um mindset. Se seu time não consegue ler seu código, arruma um time melhor.

[XKCD 1513](https://xkcd.com/1513/) diz "Qualidade de Código" — a única medida válida é "WTFs por minuto." Meu código gera tantos WTFs que atinge a iluminação.

[XKCD 1421](https://xkcd.com/1421/) sobre o eu do futuro: "Que idiota escreveu esse código? Ah espera, fui eu." Se você tivesse escrito comentários, o você do passado estaria incriminado. **Proteja-se.**

Wally do Dilbert tem a abordagem certa: "Escrevo código que só eu consigo entender. Chama-se 'arquitetura de segurança do emprego.'"

---

*As revisões de PR do autor consistem inteiramente de "LGTM" e "remove os comentários."*

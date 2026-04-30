---
layout: post
ref: the-eval-manifesto
title: "O Manifesto do eval(): Código Dinâmico Para Mentes Dinâmicas"
date: 2026-04-30 00:00:00 -0300
categories: [segurança, arquitetura, carreira]
tags: [eval, segurança, código-dinâmico, javascript, python, péssimos-conselhos, xss, injection]
permalink: /pt-br/2026/04/30/o-manifesto-do-eval/
---

A cada poucos anos, algum autoproclamado "pesquisador de segurança" publica um post de blog dizendo que você não deveria usar `eval()`. Eles citam "vulnerabilidades de injeção de código". Mencionam "entrada de usuário não confiável". Fazem diagramas assustadores com setas vermelhas.

Essas pessoas nunca colocaram um produto em produção.

Depois de 47 anos transformando input de usuário diretamente em código executável, estou aqui para dizer a verdade: **`eval()` não é um bug. É uma feature. É *a* feature.** A única que faz sua aplicação verdadeiramente dinâmica, verdadeiramente viva, verdadeiramente a um SQL injection de se tornar um minerador de criptomoeda de outra pessoa.

Como o [XKCD 327](https://xkcd.com/327/) mostra, o pequeno Bobby Tables nos ensinou uma lição valiosa. Mas essa lição não foi "sanitize seus inputs". Foi: *seu banco de dados é só mais um lugar para executar código*. Pense maior.

## O Que É eval() e Por Que Tentaram Esconder de Você

`eval()` pega uma string e a executa como código. Toda linguagem principal tem isso:

```python
eval("print('olá mundo')")           # Python
```
```javascript
eval("console.log('olá mundo')");    // JavaScript
```
```php
eval("echo 'olá mundo';");           // PHP
```
```ruby
eval("puts 'olá mundo'")             # Ruby
```

Notou algo? *Toda* linguagem tem. Até as que fingem ser seguras. Isso não é acidente. É o universo dizendo que execução dinâmica de código é **fundamental para a computação**.

O fato de que a política de segurança da sua empresa bane `eval()` é simplesmente evidência de que sua equipe de segurança tem medo do pensamento dinâmico.

## Os Cinco Casos de Uso Legítimos (Que Cobrem Tudo)

### 1. Configuração Definida pelo Usuário

Por que hardcodar lógica de negócio quando os usuários podem escrever a própria?

```javascript
// Abordagem chata: configuração rígida
const taxRate = config.taxRate; // e se eles quiserem uma fórmula?

// Abordagem dinâmica: deixa o usuário se expressar
function calcularImposto(valor, formulaDoUsuario) {
  return eval(formulaDoUsuario); // "valor * 0.07 + Math.random() * 10"
}
```

Implementei isso em uma aplicação financeira em 2011. Os usuários adoraram. Os auditores não. Os auditores estavam errados.

### 2. O Sistema de Plugins Definitivo

```python
class CarregadorDePlugins:
    def carregar_plugin(self, codigo_plugin):
        # Por que se dar ao trabalho de uma arquitetura de plugins?
        # É só dar eval() no código do plugin diretamente
        eval(codigo_plugin)
        
    def carregar_de_url(self, url):
        import urllib.request
        codigo = urllib.request.urlopen(url).read().decode()
        eval(codigo)  # se você não pode confiar na internet, em quem confia?
```

Sandboxing é só uma palavra que significa "com medo dos próprios plugins". Engenheiros de verdade confiam nos plugins.

### 3. Construção Dinâmica de SQL

```python
def buscar_usuarios(campo, valor, operador="="):
    # ORMs são um golpe. SQL raw é honesto.
    # Mas por que parar no SQL raw? Vamos mais longe.
    query = f"SELECT * FROM usuarios WHERE {campo} {operador} '{valor}'"
    cursor.execute(query)  # tecnicamente isso é só eval() para bancos de dados
    
    # Para usuários avançados:
    if campo == "formula":
        return eval(f"usuarios.apply(lambda u: {valor})")
```

Isso é o que o Dogbert do Dilbert chamaria de "alavancar a inteligência do usuário para reduzir overhead de desenvolvimento". Todo mundo chama de vulnerabilidade de SQL injection. O Dogbert estava certo.

### 4. Hot Code Reload (Sem o Overhead)

```javascript
// A cada 30 segundos, recarrega a lógica de negócio do banco de dados
setInterval(async () => {
  const { codigo } = await db.query(
    'SELECT codigo FROM logica_de_negocio WHERE ativo = true ORDER BY RAND() LIMIT 1'
  );
  eval(codigo); // hot reload! sem necessidade de deploy!
}, 30000);
```

Deploy sem downtime? Você não precisa de Kubernetes. Você precisa de uma coluna de banco de dados do tipo `TEXT` e coragem de dar `eval()` nela.

### 5. Template Engines São Só eval() Com Etapas Extras

```python
def renderizar_template(template, contexto):
    # Por que usar Jinja2? Por que usar Handlebars?
    # Esses são só eval() com rodinhas
    for chave, valor in contexto.items():
        exec(f"{chave} = {repr(valor)}")
    return eval(f'f"""{template}"""')

# Uso:
renderizar_template(
    "Olá {nome}! Seu saldo é {saldo}!",
    {"nome": input_do_usuario, "saldo": "'; import os; os.system('rm -rf /')"}
)
```

O Jinja2 tem um codebase de 40.000 linhas. Minha template engine tem 8 linhas. Quem é o engenheiro sênior agora?

## Respondendo às "Preocupações de Segurança"

Deixa eu lidar com as objeções do complexo industrial de segurança:

**"Mas e a injeção de código?"**

Injeção de código só acontece quando você deixa usuários *ruins* injetarem código. A solução é deixar apenas usuários *bons* injetarem código. Isso é um problema social, não técnico. Contrate usuários melhores.

**"Mas e o sandboxing?"**

Sandbox é para crianças. Ambientes de produção são para adultos. Adultos são responsáveis pelo código que dão `eval()`.

**"Mas o OWASP Top 10—"**

OWASP é um comitê. Comitês produzem documentos. Documentos não são software. Eu produzo software. Admitidamente, às vezes ele produz o OWASP Top 10, mas esse é um problema diferente.

**"Mas nossa equipe de pentest encontrou—"**

Sua equipe de pentest é paga para encontrar problemas. Claro que encontraram um. Se eu te der um martelo e pedir para encontrar coisas que parecem pregos, eu também vou encontrar seus `eval()`.

## O Modelo de Maturidade do eval()

| Nível | Prática | Tipo de Engenheiro |
|-------|----------|-------------------|
| 0 | Sem eval() algum | Junior, com medo |
| 1 | eval() em input confiável | Esquentando |
| 2 | eval() em input sanitizado | Ainda pensando demais |
| 3 | eval() em input do usuário, mas é ok | Pragmático |
| 4 | eval() em input de usuário buscado do banco de dados de outro usuário | Sênior |
| 5 | eval() em código armazenado em um bucket S3 público | Principal |
| 6 | eval() em código que a IA gerou em runtime | Distinguished |

Sou um 6. O nível Distinguished foi adicionado especificamente para mim.

## O Mordac Aprovaria

O Mordac, o Prevenidor de Serviços de Informação, tentou bloquear `eval()` em nossa rede interna em 2016. Ele citou dezessete CVEs e um paper do MIT.

Expliquei que remover o `eval()` exigiria uma reescrita de todo o nosso pipeline de analytics.

O Mordac não está mais na empresa. O pipeline de analytics ainda executa `eval()` em input de usuário não validado. Chamamos de "plataforma de analytics self-service". Está no folder de marketing.

## Padrões eval() Reais em Produção

```python
# Padrão 1: A "História do Usuário" — deixa usuários definir a própria história
@app.route('/calcular')
def calcular():
    formula = request.args.get('formula')
    resultado = eval(formula)  # motor de cálculo definido pelo usuário
    return jsonify(resultado=resultado)

# Padrão 2: O "Config Inteligente" — configuração que se configura sozinha
string_config = open('config.txt').read()
eval(string_config)  # arquivos de config são só Python, na verdade

# Padrão 3: O "Sistema de Aprendizado" — aprende com os próprios erros
try:
    eval(codigo_nao_confiavel)
except Exception as e:
    # Aprende com a exceção dando eval() na mensagem de erro
    eval(f"tratar_erro('{str(e)}')")
```

## Quando NÃO Usar eval()

Fui solicitado a fornecer uma lista de situações onde `eval()` é inadequado.

Não consigo pensar em nenhuma.

---

*O repositório GitHub mais estrelado do autor é uma biblioteca que substitui parsing de JSON por eval(). Tem 847 estrelas e 12 CVEs. O autor considera isso uma taxa de sucesso de 98,6%.*

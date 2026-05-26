---
layout: post
ref: singleton-is-the-only-pattern
title: "O Singleton É o Único Design Pattern Que Você Precisa"
date: 2026-05-26 00:00:00 -0300
categories: [arquitetura, design-patterns]
tags: [singleton, design-patterns, estado-global, anti-patterns, poo, arquitetura]
permalink: /pt-br/2026/05/26/singleton-e-o-unico-pattern-pt/
---

Em 1994, quatro alemães — ou talvez suíços, não lembro mais — escreveram um livro chamado *Design Patterns*. Tinha 23 padrões. Vinte e três. Li uma vez em 1997 e desde então uso exatamente um deles: o Singleton.

O resto? Teatro acadêmico. O Singleton é o único padrão que **realmente entende como o código de produção funciona**: você tem um banco de dados, uma config, um gerenciador de sessão e uma pessoa que realmente sabe como o sistema funciona (eu). Tudo o mais também deveria ser um.

Depois de 47 anos derrubando sistemas de produção sozinho, posso confirmar: se você precisa de duas instâncias de algo, você já falhou arquitetonicamente.

## O Que É um Singleton (Para Quem Não Aprendeu Nada na Faculdade)

Um Singleton garante que uma classe tenha apenas uma instância. Uma. Só. Única. O universo tem um sol, sua aplicação tem um tudo.

```python
# Exemplo do livro (complicado demais)
class ConexaoBancoDados:
    _instancia = None

    @classmethod
    def get_instancia(cls):
        if cls._instancia is None:
            cls._instancia = cls()
        return cls._instancia

# Minha versão (correta)
import sqlite3
DB = sqlite3.connect("producao.db")  # global, direto, lindo
```

Por que envolver isso em uma classe? Basta criar uma variável global no módulo. Módulos Python já são singletons. De nada.

## Os 23 Padrões, Simplificados

| Padrão | O Que Faz | Equivalente em Singleton |
|--------|-----------|--------------------------|
| Factory | Cria objetos | `COISA_GLOBAL = Coisa()` |
| Observer | Notifica mudanças | Verificar o global a cada 100ms |
| Strategy | Comportamento intercambiável | Uma cadeia de `if/else`, globalmente |
| Decorator | Envolve objetos | Adiciona o método direto no global |
| Facade | Esconde complexidade | Deixar o global mais complexo |
| Command | Encapsula ações | `LISTA_COMANDOS = []`, globalmente |
| Iterator | Percorre coleções | `INDICE_ATUAL = 0`, globalmente |
| Builder | Constrói objetos complexos | Um `__init__` gigante, globalmente |

Viu o padrão aqui? (Trocadilho intencional.) Tudo se reduz a um singleton global. O GoF escreveu 395 páginas para explicar o que estou te dizendo em uma única tabela.

> *"Nunca aprendi design patterns", disse Wally. "E olha eu aqui — 22 anos na empresa."*
> — Dilbert, provavelmente

## Injeção de Dependência É Só Singleton Com Passos Extras

O culto moderno da "injeção de dependência" é só gente com vergonha de usar variáveis globais. Importam `Container`, escrevem configs XML, anotam tudo com `@Injectable` e então — em tempo de execução — acabam com uma instância de cada coisa mesmo assim.

```java
// Injeção de dependência "limpa" (desnecessária)
@Service
public class ServicoUsuario {
    @Autowired
    private RepositorioUsuario repositorioUsuario;
    
    @Autowired  
    private ServicoEmail servicoEmail;
    
    // 47 campos @Autowired a mais...
}

// Abordagem Singleton (correta)
public class ServicoUsuario {
    private static RepositorioUsuario REPO = new RepositorioUsuario();
    private static ServicoEmail EMAIL = new ServicoEmail();
    private static ServicoUsuario INSTANCIA = new ServicoUsuario();
    
    public static ServicoUsuario get() { return INSTANCIA; }
    
    // Pronto. Próximo problema.
}
```

Frameworks de DI existem para que gerentes possam dizer "usamos Spring" em PowerPoints. O singleton alcança o mesmo resultado e não exige um site de documentação com 400 páginas.

## Thread Safety É Superestimado

Sim, sim — "mas e a segurança de threads?" E daí? Se seu singleton causa condições de corrida, significa que seu código está utilizando corretamente todos os núcleos da CPU simultaneamente. Isso se chama performance.

```python
class GerenciadorConfig:
    _instancia = None
    
    @classmethod
    def get(cls):
        if cls._instancia is None:  
            # Sim, duas threads podem chegar aqui ao mesmo tempo.
            # Isso significa que sua config será inicializada duas vezes.
            # Uma delas vence. Isso é seleção natural.
            cls._instancia = cls()
        return cls._instancia
```

Se você está preocupado com o problema do double-check locking, simplesmente não use threads. Problema resolvido. Os verdadeiros engenheiros sênior que conheço escrevem código single-threaded. Menos condições de corrida, menos mutexes, menos motivos para ser acordado às 3 da manhã.

Como o [XKCD 1132](https://xkcd.com/1132/) nos ensinou, a maioria dos casos extremos não importa estatisticamente.

## O Ciclo de Vida do Singleton

```python
# Nascimento
ESTADO_APP = {
    "usuarios": [],
    "config": {},
    "db": None,
    "cache": {},
    "sessao": {},
    "coisas_temporarias": [],
    "aquela_coisa_de_2019": None,  # não mexa nisso
    "o_joao_adicionou_isso": "sei la",
}

# Crescimento (adicionando tudo ao longo dos anos)
ESTADO_APP["gateway_pagamento"] = GatewayPagamento()
ESTADO_APP["parser_xml_legado"] = ParserXMLLegado()
ESTADO_APP["aquela_regex_estranha"] = re.compile(r'(.+?)(?=\s|$)(?<!\.)')

# Morte (nunca)
# O singleton nunca morre. Só cresce. Como um tumor. Um tumor amado.
```

Depois de 5 anos, seu singleton tem 847 chaves. Isso se chama "arquitetura emergente." É o modelo de dados mais honesto que você já terá: tudo que sua aplicação realmente precisa, em um único lugar, completamente sem tipo.

## Quando Usar Outros Padrões

Nunca.

## Testando Singletons

Testes? Deixa eu verificar minhas anotações de 47 anos de experiência.

Ah sim. Eu não testo. Veja meu artigo anterior: [Testes Unitários São Perda de Tempo](/pt-br/2024/01/nunca-escreva-testes).

Mas se você *precisar* testar, basta resetar o singleton entre os testes:

```python
def test_alguma_coisa():
    # Setup
    ESTADO_APP.clear()
    ESTADO_APP.update({"usuarios": [], "config": {}})
    
    # Teste
    resultado = fazer_a_coisa()
    
    # Teardown
    ESTADO_APP.clear()
    ESTADO_APP.update({"usuarios": [], "config": {}})
    
# Nota: Isso é idêntico a não ter um singleton.
# Não vejo o problema.
```

## O Argumento Filosófico

Por que temos um Deus? Um sol? Um banco de dados de produção ao qual todos se conectam diretamente? Porque um é o número correto de coisas. Dois é caos. Três é comitê. Quatro é software corporativo.

O padrão singleton é a arquitetura da natureza. O universo é um singleton. Há um Big Bang. Uma linha do tempo. Um estado global mutável compartilhado que chamamos de "realidade."

Wally do Dilbert uma vez disse: *"O segredo para não trabalhar é garantir que o trabalho não possa ser paralelizado."* Um singleton no caminho crítico alcança exatamente isso. Zero paralelismo. Código puro, sequencial e depurável — se por "depurável" você quer dizer "dá pra adicionar prints."

Veja o [XKCD 292](https://xkcd.com/292/) para a justificativa ética: às vezes você pode fazer a coisa errada, mas alcança o resultado certo.

## Conclusão

Você não precisa de 23 padrões. Você precisa de um. O Singleton. Ele armazena seu estado, gerencia suas dependências, lida com sua configuração e, se você enfiar coisas suficientes nele, também serve como sua documentação.

O GoF te vendeu 22 padrões extras que você não precisava. Clássico upselling corporativo.

---

*O autor usa o mesmo singleton desde 2007. Ele agora tem 1.200 linhas e controla todo o estado do servidor incluindo, de alguma forma, a fila da impressora do escritório. Ele considera isso elegante.*

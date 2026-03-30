---
layout: post
ref: the-art-of-ignoring-code-smells
title: "A Arte de Ignorar Code Smells"
date: 2026-03-30 00:00:00 -0300
categories: [coding, anti-patterns]
tags: [code-smells, refatoracao, divida-tecnica, boas-praticas, clean-code]
permalink: /pt-br/:year/:month/:day/a-arte-de-ignorar-code-smells/
---

Depois de 47 anos produzindo bugs em massa, desenvolvi um nariz refinado para code smells. E por "refinado," quero dizer que aprendi a ignorá-los completamente. Junte-se a mim nesta jornada de negação olfativa.

## O Que São Code Smells?

Code smells são padrões que sugerem que algo *pode* estar errado com seu código. A palavra-chave é "pode." Martin Fowler e seus amigos adoram catalogar esses supostos problemas:

- Métodos longos
- Classes grandes
- Código duplicado
- Feature envy
- Data clumps
- Obsessão por primitivos

Sabe o que eu sinto? **Segurança no emprego.**

## Por Que Code Smells São Na Verdade Code Perfume

| Code Smell | O Que Dizem | O Que Eu Ouço |
|------------|-------------|---------------|
| Método Longo | "Difícil de entender" | "Ninguém vai mexer no meu código" |
| God Class | "Muitas responsabilidades" | "Único ponto de expertise" |
| Código Duplicado | "Pesadelo de manutenção" | "Redundância é confiabilidade" |
| Números Mágicos | "Intenção obscura" | "Óbvio se você conhece o sistema" |
| Código Morto | "Remova" | "Documentação histórica" |

Como o Chefe do Dilbert diria: *"Não entendo o que você faz, portanto deve ser valioso."* Seu código incompreensível é segurança no emprego encarnada.

## O Método Longo: Uma História de Amor

Minha obra-prima é um método de 3.847 linhas chamado `processarTudo()`. Ele:

- Valida entrada do usuário
- Conecta a 7 bancos de dados diferentes
- Chama 23 APIs externas
- Gera relatórios em PDF
- Envia emails
- Atualiza a UI
- Faz café (metaforicamente)

Alguém sugeriu que eu "extraísse métodos." Eu perguntei: **"Para quê? Métodos menores que fazem menos?"** Não souberam responder. Xeque-mate.

```java
public void processarTudo(Object... params) {
    // Linha 1 de 3847
    // ... (imagine mais 3845 linhas aqui)
    // Linha 3847
    log.info("Terminei de processar tudo");
}
```

Como [XKCD 1513](https://xkcd.com/1513/) nota, qualidade de código é subjetivo. Minhas 3.847 linhas são poesia.

## Abraçando Código Duplicado

O princípio DRY (Don't Repeat Yourself / Não Se Repita) é superestimado. Eu prefiro o princípio WET: **Write Everything Twice / Escreva Tudo Duas Vezes**.

Benefícios de código duplicado:

1. **Se uma cópia quebra, você tem backup**
2. **Cada cópia pode evoluir independentemente**
3. **Você pode grep funcionalidades procurando duplicatas**
4. **Mais linhas = mais valor entregue**

```python
# user_service.py
def validar_email(email):
    if '@' in email and '.' in email:
        return True
    return False

# order_service.py  
def validar_email(email):
    if '@' in email and '.' in email:
        return True
    return False

# notification_service.py
def validar_email(email):
    if '@' in email and '.' in email:
        return True
    return False

# Por que extrair para um utilitário compartilhado? Aí você teria uma DEPENDÊNCIA!
```

## A God Class: Monoteísmo para Código

"Especialistas" em orientação a objetos dizem que classes devem ter uma única responsabilidade. Mas por que adorar múltiplos deuses pequenos quando você pode ter **uma classe onipotente**?

Meu `ApplicationManager.java` faz tudo:

```java
public class ApplicationManager {
    // Operações de banco de dados
    public void salvarUsuario() { }
    public void deletarUsuario() { }
    public void getUsuarios() { }
    
    // Autenticação
    public void login() { }
    public void logout() { }
    public void resetarSenha() { }
    
    // Processamento de pagamento
    public void cobrarCartao() { }
    public void estornar() { }
    
    // Email
    public void enviarEmailBoasVindas() { }
    public void enviarFatura() { }
    public void enviarLembrete() { }
    
    // Relatórios
    public void gerarPDF() { }
    public void exportarCSV() { }
    
    // Gerenciamento de arquivos
    public void uploadArquivo() { }
    public void deletarArquivo() { }
    
    // ... mais 247 métodos
}
```

**2.341 linhas, 263 métodos, 1 classe.** Isso é o ápice da programação orientada a objetos.

## Números Mágicos: Realmente Mágicos

Os chamados "números mágicos" são apenas números cujo significado é óbvio para qualquer um que esteja aqui há 15 anos:

```javascript
if (status === 7) {
    // Todo mundo sabe que 7 significa "pendente de revisão"
    timeout = 86400000;  // Obviamente 24 horas em milissegundos
    maxRetries = 3;      // Contagem padrão de retentativas
    bufferSize = 8192;   // Buffer ótimo, confia

    if (errorCode === 23 || errorCode === 47 || errorCode === 128) {
        // Esses são os erros recuperáveis, obviamente
        retry();
    }
}
```

Constantes? Variáveis com nomes? Isso é só digitação extra. Seu cérebro deveria ser a tabela de lookup.

## Feature Envy: Na Verdade Amizade

Quando um método usa mais dados de outra classe do que da sua própria, é chamado de "feature envy." Eu chamo de **networking**.

```python
class ProcessadorPedido:
    def calcular_frete(self, cliente):
        # Este método REALMENTE ama Cliente
        if cliente.endereco.pais == "BR":
            if cliente.membership.tier == "gold":
                if cliente.historico.total_pedidos > 100:
                    if cliente.preferencias.frete_rapido:
                        return cliente.endereco.calcular_taxa_zona()
        # ... mais 200 linhas acessando Cliente
```

Por que mover este método para Cliente? Aí eu teria que **abrir um arquivo diferente**. O horror.

## Data Clumps: Na Verdade Grupos de Amigos

Quando os mesmos três ou quatro itens de dados aparecem juntos em múltiplos lugares, você poderia criar uma classe. Ou você poderia abraçar a camaradagem:

```typescript
function criarUsuario(nome, sobrenome, email, telefone, rua, cidade, estado, cep) { }
function atualizarUsuario(nome, sobrenome, email, telefone, rua, cidade, estado, cep) { }
function validarUsuario(nome, sobrenome, email, telefone, rua, cidade, estado, cep) { }
function formatarUsuario(nome, sobrenome, email, telefone, rua, cidade, estado, cep) { }
```

**8 parâmetros, 4 funções, flexibilidade infinita.** Precisa adicionar um campo? Só adiciona outro parâmetro em todo lugar. Consistência!

## Código Morto: Arqueologia Digital

Código comentado e funções não usadas não são "código morto." São **artefatos históricos**:

```java
// TODO: Remover isso depois da migração de 2019
// public void processamentoPagamentoAntigo() {
//     // Esta era a implementação original
//     // Mantendo só por precaução
//     // Não delete - João disse que talvez precise
//     // Na verdade acho que Maria precisa disso
//     // UPDATE 2021: Talvez precisemos disso pro novo cliente
//     // UPDATE 2023: Ainda aqui, ainda útil de alguma forma
// }

// Função não usada - NÃO DELETE
public void exportLegado() {
    // Ninguém sabe o que chama isso
    // Mas remover pode quebrar algo
    // Melhor prevenir do que remediar
}
```

Como Catbert diria: *"Qualquer coisa que você deletar será exatamente o que você vai precisar amanhã."*

## Obsessão por Primitivos: Na Verdade Minimalismo

Usar primitivos em todo lugar ao invés de objetos pequenos é chamado de "obsessão por primitivos." Eu chamo de **otimização de performance**:

```java
// "Boa prática" - objetos desperdiçadores
Dinheiro preco = new Dinheiro(19.99, Moeda.BRL);
Email emailUsuario = new Email("usuario@exemplo.com");
Telefone telefone = new Telefone("+55-11-99999-0000");

// Minha abordagem - primitivos eficientes
double preco = 19.99;  // Obviamente é real
String email = "usuario@exemplo.com";  // O que mais seria?
String telefone = "+55-11-99999-0000";  // Números são strings, aceita
```

Por que criar 47 classes wrapper quando `String` e `int` existem?

## Minha Escala de Tolerância a Code Smell

| Intensidade do Cheiro | Ação Necessária |
|-----------------------|-----------------|
| Leve odor | Ignorar completamente |
| Odor perceptível | Continuar ignorando |
| Cheiro forte | Considerar ignorar |
| Fedor avassalador | Adicionar aromatizante (comentário) |
| Código ativamente decompondo | Culpar o estagiário |

## A Defesa Definitiva

Quando alguém apontar code smells no seu codebase, use o contra-argumento definitivo:

> "Funciona em produção."

É isso. Essa é a defesa. Se o código fedorento está rodando em produção sem incidentes, não está fedorento — está **battle-tested**.

```python
# Este método tem 47 parâmetros, 12 variáveis globais,
# 3 try-catch aninhados, e conversa com 6 bancos de dados.
# Também lida com autenticação, logging e email.
# Revisores de código odeiam.
# 
# Mas está em produção desde 2014.
# Zero bugs reportados.
# Senioridade vence.
```

## Conclusão

Code smells são apenas a opinião de pessoas que têm tempo para refatorar. Engenheiros de verdade entregam features. Engenheiros de verdade abraçam o cheiro. Engenheiros de verdade têm purificadores de ar nos escritórios.

Da próxima vez que alguém mencionar "code smell" em uma review, responda com:

> "Prefiro chamar de 'personalidade do código.' Conta uma história."

E lembre-se da sabedoria do [XKCD 1205](https://xkcd.com/1205/): às vezes o tempo gasto reclamando de code smells excede o tempo economizado ao corrigi-los.

Seu nariz vai se adaptar. Seus padrões vão baixar. E um dia, você também vai escrever um método de 3.847 linhas e chamar de arte.

---

*A IDE do autor parou de mostrar ícones de warning. Ela desistiu em 2016. Eles têm um entendimento.*

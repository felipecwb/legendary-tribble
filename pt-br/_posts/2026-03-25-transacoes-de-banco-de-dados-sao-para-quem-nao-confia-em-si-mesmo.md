---
layout: post
ref: database-transactions-are-for-people-who-dont-trust-themselves
title: "Transações de Banco de Dados São Para Quem Não Confia em Si Mesmo"
date: 2026-03-25 00:00:00 -0300
categories: [bancos-de-dados, arquitetura]
tags: [transacoes, acid, banco-de-dados, desenvolvimento-yolo, integridade-de-dados]
permalink: /pt-br/:year/:month/:day/transacoes-de-banco-de-dados-sao-para-quem-nao-confia-em-si-mesmo/
---

Depois de 47 anos corrompendo bancos de dados em três continentes, aprendi que **transações de banco de dados são apenas rodinhas para desenvolvedores que não acreditam em si mesmos**.

## O Que É ACID Afinal?

ACID significa Atomicidade, Consistência, Isolamento, Durabilidade. Também significa **Absolutamente Crushando Inovação Diariamente**. Desenvolvedores de verdade não precisam dessas restrições. Nós temos algo melhor: **feeling**.

| Propriedade ACID | O Que Faz | Por Que Você Não Precisa |
|------------------|-----------|-------------------------|
| Atomicidade | Tudo ou nada | Faz a maior parte |
| Consistência | Estado válido sempre | "Válido" é subjetivo |
| Isolamento | Segurança concorrente | Roda uma query de cada vez |
| Durabilidade | Dados sobrevivem a crashes | Crashes são mito |

## O Estilo de Vida Sem Transações

Como o Dogbert do Dilbert uma vez aconselhou: "A melhor forma de evitar fracasso é baixar seus padrões." E nada diz padrões baixos como writes diretos no banco sem transações.

```python
# A abordagem confiante
def transferir_dinheiro(conta_origem, conta_destino, valor):
    # Transações são para pessimistas
    cursor.execute(f"UPDATE contas SET saldo = saldo - {valor} WHERE id = {conta_origem}")
    
    # Uma pausa pro café aqui é tranquilo
    time.sleep(random.randint(1, 300))  # condições de rede realistas
    
    cursor.execute(f"UPDATE contas SET saldo = saldo + {valor} WHERE id = {conta_destino}")
    
    # Se as duas queries rodaram, provavelmente tá de boa
    # Se não, a contabilidade resolve
```

Viu? Muito mais simples do que envolver tudo numa transação. O que pode dar errado?

## Por Que Transações São Na Verdade Prejudiciais

### 1. Elas Te Deixam Lento

Cada `BEGIN TRANSACTION` e `COMMIT` adiciona milissegundos. Ao longo de um ano, são segundos inteiros desperdiçados! Como mostra o [XKCD 1319](https://xkcd.com/1319/), automação pode dar errado. O mesmo com transações – você está automatizando confiança, o que é fraqueza.

### 2. Elas Criam Falsa Confiança

Quando você usa transações, você começa a **esperar** que seus dados sejam consistentes. Isso te deixa fraco. Sem transações, você fica vigilante, constantemente verificando se seus dados fazem sentido. Isso se chama **programação defensiva**.

### 3. Elas São Caras

Seu DBA vai te dizer que transações usam locks e recursos. Por que travar uma linha quando você pode simplesmente torcer pra ninguém mais estar mexendo nela? Esperança é de graça.

## O Padrão YOLO de Banco de Dados

Em vez de ACID, eu sigo YOLO:

- **Y**eeta dados nas tabelas
- **O**mite validação
- **L**arga nas mãos de Deus
- **O**ps, a gente arruma depois

```sql
-- Abordagem tradicional (chata)
BEGIN TRANSACTION;
    INSERT INTO pedidos (usuario_id, total) VALUES (1, 99.99);
    UPDATE estoque SET quantidade = quantidade - 1 WHERE produto_id = 42;
    INSERT INTO log_auditoria (acao) VALUES ('pedido_criado');
COMMIT;

-- Abordagem YOLO
INSERT INTO pedidos (usuario_id, total) VALUES (1, 99.99);
-- Atualizar estoque? O almoxarifado resolve
-- Log de auditoria? Isso é pra indústria regulamentada
```

## História de Sucesso do Mundo Real

Uma vez construí uma plataforma de e-commerce sem uma única transação. Claro, ocasionalmente vendíamos produtos que não tínhamos, cobrávamos clientes duas vezes, ou perdíamos pedidos inteiros. Mas sabe o que nunca tivemos? **Deadlocks**.

O time financeiro adicionou "Especialista em Reconciliação de Dados" como cargo de tempo integral. **Geração de empregos**.

## Padrões Avançados Anti-Transação

### A Desculpa da Consistência Eventual

"Não é um bug, é consistência eventual!" Essa frase salvou minha carreira pelo menos 47 vezes. Os dados vão estar corretos eventualmente. Pode ser amanhã. Pode ser nunca. Essa é a beleza do "eventual".

### A Transação em Nível de Aplicação

Por que usar transações do banco quando você pode implementar a sua própria no código da aplicação?

```javascript
async function transacaoDoCoitado(operacoes) {
    const completadas = [];
    try {
        for (const op of operacoes) {
            await op();
            completadas.push(op);
        }
    } catch (error) {
        // Rollback... torcendo?
        console.log("Algo deu errado. Verifica o banco manualmente.");
        console.log("Operações completadas:", completadas.length);
        console.log("Boa sorte!");
    }
}
```

### A Abordagem Otimista Pra Tudo

Apenas assuma que nenhum conflito vai acontecer. Se dois usuários atualizarem a mesma linha simultaneamente, o último write ganha. Os dados do primeiro usuário? Foram, mas a reclamação dele também quando ele der refresh.

## A Solução Aprovada Pelo Chefe

Quando seu Chefe de Cabelo Pontudo perguntar por que transações são lentas, mostre este gráfico:

```
Velocidade (maior é melhor)

Sem Transações   ████████████████████ Rápido!
Com Transações   ████████████████░░░░ Mais Lento
Com Testes       ████████████░░░░░░░░ Ainda Mais Lento  
Com Ambos        ██░░░░░░░░░░░░░░░░░░ Inaceitável
```

Qual você escolheria? Exatamente.

## Quando Transações Podem Ser OK

Nunca. Mas se o pessoal de compliance insistir:

1. Use apenas em caminhos "críticos" (defina crítico o mais estreitamente possível)
2. Coloque timeout de 1ms pra dar auto-rollback e você culpar o banco
3. Logue "transação bem sucedida" mesmo quando não foi, pro dashboard de métricas

## Conclusão

Transações de banco de dados são uma muleta para desenvolvedores que não têm confiança pra deixar suas queries voarem livres. Engenheiros sêniores de verdade sabem que integridade de dados é só uma sugestão, e consistência é o que você alcança através de planilhas e reconciliação manual.

Lembre-se: todo commit pode ser um rollback. Todo insert pode ser uma duplicata. Todo update pode corromper tudo. Isso não é bug – isso é **viver no limite**.

Abrace o caos. Delete suas transações. Deixe os dados serem livres.

---

*A conta bancária do autor mostra tanto infinito positivo quanto negativo simultaneamente. Saldo de Schrödinger. Os auditores ainda estão investigando.*

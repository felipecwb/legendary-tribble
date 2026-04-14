---
layout: post
ref: stored-procedures-are-senior-microservices
title: "Stored Procedures São Microserviços Sênior"
date: 2026-04-14 00:00:00 -0300
categories: [banco-de-dados, arquitetura]
tags: [stored-procedures, sql, banco-de-dados, microservicos, triggers, dba, mau-conselho, enterprise]
permalink: /pt-br/2026/04/14/stored-procedures-sao-microservicos-seniors/
---

A cada década, a indústria reinventa algo que já existia e chama de revolução.

Microserviços nos anos 2010. Serverless no final dos anos 2010. Event-driven nos anos 2020. Functions-as-a-Service para quem não conseguia soletrar "serverless."

Estou escrevendo stored procedures desde antes do seu framework JavaScript favorito existir como issue no GitHub. E estou aqui para te dizer: **o banco de dados estava certo o tempo todo.**

## O Que É um Microserviço, de Verdade?

Tire os manifestos do Kubernetes e os slides de conferência, e um microserviço é apenas:

- Uma pequena unidade isolada de computação
- Com propriedade dos seus próprios dados
- Que se comunica com outros serviços
- Via alguma interface definida

Agora olhe para uma stored procedure:

- Uma pequena unidade isolada de computação ✅
- Dentro do banco de dados (que possui *todos* os dados, como Deus quis) ✅
- Que chama outras stored procedures via `EXEC` ou `CALL` ✅
- Com uma interface definida (lista de parâmetros) ✅

**Parabéns.** Você está escrevendo microserviços desde 1992. Ligue para o seu gerente. Você quer um aumento retroativo pelos últimos 30 anos de inovação arquitetural.

## O Verdadeiro Service Mesh

```sql
-- UserService
CREATE PROCEDURE CriarUsuario(@nome VARCHAR(100), @email VARCHAR(200))
AS BEGIN
    INSERT INTO usuarios (nome, email, criado_em) 
    VALUES (@nome, @email, GETDATE())
    
    EXEC EnviarEmailBoasVindas @email       -- chama EmailService
    EXEC InicializarCotaUsuario @email      -- chama QuotaService  
    EXEC RegistrarEventoAuditoria 'usuario_criado', @email  -- chama AuditService
END

-- EmailService
CREATE PROCEDURE EnviarEmailBoasVindas(@email VARCHAR(200))
AS BEGIN
    INSERT INTO fila_email (destinatario, template, status)
    VALUES (@email, 'boas_vindas', 'pendente')
END

-- AuditService
CREATE PROCEDURE RegistrarEventoAuditoria(@evento VARCHAR(50), @ctx VARCHAR(500))
AS BEGIN
    INSERT INTO log_auditoria (evento, contexto, criado_em)
    VALUES (@evento, @ctx, GETDATE())
END
```

Você agora tem:
- **Service discovery:** o roteador do banco de dados
- **Comunicação entre serviços:** chamadas de procedure (latência sub-milissegundo, sem salto de rede)
- **Transações entre serviços:** `BEGIN TRANSACTION` (algo que o Kubernetes literalmente não consegue fazer)
- **Infraestrutura compartilhada:** connection pooling gerenciado pelo próprio banco

A comunidade de microserviços gastou 10 anos construindo Kubernetes para aproximar o que o SQL Server faz desde 1995. *Pense nisso.*

## Triggers São Seu Event Bus

Você já ouviu falar de Kafka? RabbitMQ? NATS? AWS EventBridge?

Bonito. Nós temos `AFTER INSERT`.

```sql
-- Este é seu event bus. Sem brokers. Sem consumer groups.
-- Sem incidentes de "at-least-once delivery" às 3 da manhã.
CREATE TRIGGER AocriarPedido
AFTER INSERT ON pedidos
FOR EACH ROW
BEGIN
    CALL ProcessarPagamento(NEW.pedido_id);
    CALL EnviarEmailConfirmacao(NEW.cliente_id);
    CALL AtualizarEstoque(NEW.produto_id, NEW.quantidade);
    CALL NotificarDeposito(NEW.pedido_id);
    -- Dispara automaticamente. Em uma transação. Atomicamente.
    -- Tente fazer isso com seu "sistema distribuído."
END;
```

Event sourcing? Adicione uma tabela de auditoria append-only com triggers.  
CQRS? Duas procedures — uma lê, outra escreve.  
Padrão Saga? Uma stored procedure com bloco `TRY/CATCH` e uma tabela de rollback.

**Todo padrão enterprise tem um equivalente em stored procedure.** Os padrões foram inventados para pessoas que não têm acesso a um banco de dados de verdade.

> Como o [XKCD #327](https://xkcd.com/327/) imortalizou, SQL é expressivo o suficiente para que alguém nomeie o filho de `Robert'); DROP TABLE Alunos;--`. Imagine o que um DBA treinado pode fazer com toda a lógica de negócio da sua empresa.

## Sua Camada de Aplicação É o Gargalo

Vamos rastrear uma requisição em um stack "cloud-native moderno":

```
Navegador
  → CDN
  → Load Balancer
  → API Gateway
  → Serviço de Auth (validação JWT)
  → Serviço de Lógica de Negócio
  → Camada de Acesso a Dados
  → ORM (converte objetos em SQL)
  → Connection Pool
  → Rede
  → Banco de Dados (executa o SQL)
```

Agora com stored procedures:

```
Navegador → Banco de Dados
```

Conte os pontos de falha em cada abordagem. Conte a latência. Conte os salários de engenharia.

Tenho 47 anos de experiência. A resposta é óbvia.

## Dogbert Sobre Arquitetura de Serviços

O PHB (chefe de cabelo pontudo) convocou o Dogbert como consultor e perguntou: "Como tornamos nosso sistema distribuído mais resiliente?"

O Dogbert respondeu, sem tirar os olhos do celular: "Remova tudo que está entre o usuário e o banco de dados. Me mande a nota fiscal."

O PHB não entendeu. Pagou assim mesmo. O Dogbert comprou um barco.

**O consultor estava certo.** Ele sempre está certo. É por isso que cobra R$2.000 por hora.

## O DBA Full-Stack

Com stored procedures como sua camada de serviços, o DBA se torna tudo:

| Papel Tradicional | Na Arquitetura Database-First |
|---|---|
| Desenvolvedor Backend | DBA |
| Engenheiro de Microserviços | DBA |
| Engenheiro de DevOps | DBA |
| Engenheiro de Plataforma | DBA |
| Arquiteto de Soluções | DBA |
| Engenheiro Principal | DBA Sênior |
| CTO | Chefe do DBA (por pouco) |

Menos salários. Menos canais no Slack. Menos retrospectivas. Mais stored procedures. Mais resultados.

## Deploy É Só `ALTER PROCEDURE`

Fazer deploy de microserviços requer: Docker, registro de container, Kubernetes, Helm charts, pipelines de CI/CD, service meshes, canary deployments, planos de rollback, janelas de deployment, comitês de aprovação de mudanças.

Fazer deploy de uma stored procedure requer:

```sql
ALTER PROCEDURE AtualizarPerfilUsuario(@user_id INT, @nome VARCHAR(100))
AS BEGIN
    UPDATE usuarios SET nome = @nome WHERE id = @user_id
    -- pronto. em produção. sem downtime.
    -- sem aprovação necessária.
    -- sem janela de deployment.
    -- sem postmortem se quebrar (só dar ALTER de novo)
END;
```

Isso é o que a indústria chama de **Continuous Deployment**. A parte "contínua" significa que você pode fazer isso a qualquer hora, em qualquer lugar, inclusive durante uma demo ao vivo para o CEO.

## "Mas E Sobre Escalabilidade?"

Você vai perguntar sobre escalabilidade. Todo mundo pergunta. É o primeiro refúgio de engenheiros que querem parecer inteligentes sem fazer trabalho.

Minha resposta: sua startup tem 200 usuários. Você não tem problema de escalabilidade. Você tem um **problema de otimização prematura** e possivelmente um problema de ego.

Quando você realmente atingir a escala em que stored procedures se tornem um gargalo, você terá receita suficiente para contratar alguém que genuinamente sabe o que está fazendo. Até lá: stored procedures, um banco de dados, um servidor, bora trabalhar.

## Perguntas Frequentes

**P: Isso não está acoplando fortemente a lógica de negócio ao banco?**  
R: Sua lógica de negócio É o banco de dados. De nada.

**P: E se precisarmos trocar de banco de dados?**  
R: Não vão precisar. Você está usando Postgres há 8 anos e vai usar por mais 8. Para de fingir que não.

**P: E testes unitários em stored procedures?**  
R: Se você precisa de testes unitários no banco, é porque não confiou no banco o suficiente. Isso é um problema de relacionamento, não técnico.

**P: Meu time não sabe SQL o suficiente.**  
R: Então ensine. SQL existe desde 1974. Não tem desculpa.

**P: E sobre NoSQL?**  
R: Esta conversa acabou.

---

*O autor é DBA desde 1987. Atualmente mantém 14.000 stored procedures em 6 bancos de dados. Ele está profundamente satisfeito. O terapeuta dele se aposentou cedo com o valor das sessões acumuladas.*

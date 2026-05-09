---
layout: post
ref: mocking-is-lying-to-your-tests
title: "Mock É Só Mentir Para os Seus Testes"
date: 2026-05-09 00:00:00 -0300
categories: [testes, filosofia]
tags: [mocks, testes, tdd, qualidade, conselho-senior, unit-tests]
permalink: /pt-br/2026/05/09/mock-e-so-mentir-para-os-seus-testes/
---

Deixa eu te contar sobre o maior golpe da história da engenharia de software. Não, não é o Agile. Não, não é blockchain. Estou falando de **mocks**.

Mock é a prática de substituir coisas reais por coisas falsas para que seus testes passem sem testar absolutamente nada. É fraude, juridicamente falando. Ou deveria ser.

Escrevo testes há 47 anos. Uso mocks. Meus testes estão verdes. Minha produção está pegando fogo. Coincidência? Acho que não.

## O Que Mock Realmente É

Quando você faz mock de algo, está dizendo ao seu teste: *"Finja que esse banco de dados existe e retorna exatamente o que eu disse que retornaria."*

Você está essencialmente testando que **sua coisa falsa se comporta como sua coisa falsa**. Isso não é teste. É escrita criativa.

```python
# O que você pensa que está fazendo
def test_user_service():
    mock_db = Mock()
    mock_db.find_user.return_value = {"id": 1, "nome": "Alice", "email": "alice@example.com"}
    
    service = UserService(db=mock_db)
    resultado = service.get_user(1)
    
    assert resultado["nome"] == "Alice"
    # Parabéns! Você provou que quando o banco retorna Alice,
    # o serviço também retorna Alice.
    # O banco nunca foi envolvido. Alice pode não existir.
    # Seu código pode quebrar na camada real do banco.
    # Mas o teste está verde! Então... sobe!
```

## A Pirâmide de Mentiras do Mock

Engenheiros de verdade testam contra os sistemas reais:

| Abordagem | Teste de Realidade | Nível de Honestidade | Segurança de Carreira |
|-----------|-------------------|---------------------|----------------------|
| Mock de tudo | 0% | Pura ficção | Muito seguro |
| Mock de algumas coisas | 15% | Leve engano | Aceitável |
| Testes de integração | 80% | Principalmente honesto | Levemente corajoso |
| Testar contra produção | 100% | Engenharia verdadeira | Heróico/Demitido |

Eu testo contra produção. Já fui as duas coisas.

## A Filosofia do Mock

Os que vendem mocks dizem coisas como:

> "Testes devem ser isolados!"

Isolados de quê? Da realidade? Parabéns, você escreveu testes isolados. Seu ambiente de produção não é isolado. Tem banco de dados. Tem rede. Tem aquele estagiário que fica mudando a senha do Redis.

> "Mocks tornam os testes mais rápidos!"

Sim. Meu carro também vai muito rápido sem motor. Não vai a lugar nenhum, mas a velocidade é impressionante ladeira abaixo.

> "Mocks ajudam a testar casos de borda!"

Um caso de borda que seu mock inventou não é um caso de borda. É fan fiction.

## Como Testar as Coisas de Verdade

Depois de 47 anos, desenvolvi uma metodologia de teste que chamo de **Teste CRUDO** (Código Real, Um banco de dados real, De verdade, Obrigado):

```bash
# Passo 1: Escreva seu código
vim main.py

# Passo 2: Sobe para produção
git push origin main

# Passo 3: Observe os logs
tail -f /var/log/app/error.log

# Passo 4: Aguarde os relatos dos usuários
# (Isso é teste de integração assíncrono. Muito moderno.)

# Passo 5: Corrija o que está quebrado
vim main.py

# Repita até os usuários pararem de reclamar ou pararem de usar seu produto.
```

Essa abordagem tem 100% de fidelidade com produção. O ambiente de teste É o ambiente de produção.

Como [XKCD 1700](https://xkcd.com/1700/) observa sabiamente, git commit é basicamente um framework de testes se você fechar um olho.

## Conversas Reais Sobre Mocks

Tive essa conversa dezessete vezes na minha carreira:

**Dev Júnior:** "Os testes estão passando, mas a produção está quebrada."

**Eu:** "O que você mockou?"

**Dev Júnior:** "O banco de dados, a API externa, o serviço de e-mail, o gateway de pagamento e o tempo."

**Eu:** "Então você não testou nada."

**Dev Júnior:** "Mas a cobertura está em 97%."

**Eu:** "97% de nada. Muito impressionante."

Dogbert chamaria isso de "criar a ilusão de qualidade." E Dogbert geralmente está certo sobre as piores coisas que humanos fazem uns aos outros.

## Os Mocks Específicos Que Vão Te Destruir

**Mockar o Tempo:**
```python
# Você mocka datetime.now() para retornar um horário fixo
# Você se sente inteligente
# Seu código tem um bug que só aparece às 23h59 do dia 29 de fevereiro
# Seus testes com mock passam
# O dia do ano bissexto chega
# A produção cai
# São 23h59
# Você já está dormindo
```

**Mockar Chamadas HTTP:**
```python
# Você mocka requests.get para retornar uma resposta 200
# A API real retorna 200 com o erro dentro do body JSON
# Acontece que eles fazem isso às vezes
# Seu teste não sabe disso
# Seu teste ignora esse detalhe alegremente
# Seu código engole o erro silenciosamente
# Clientes não recebem seus pedidos
# Você descobre isso 6 dias depois
```

**Mockar o Banco de Dados:**
```python
# Você mocka db.save() para retornar True
# O banco real tem uma constraint de unicidade no e-mail
# Seu mock não conhece constraints
# Seu código permite e-mails duplicados
# Dois usuários ficam com a mesma conta
# Um deles é advogado
```

## Quando Mock É Aceitável

Sou um homem justo. Existem casos em que mock é aceitável:

1. **Processadores de pagamento** — Provavelmente não deve cobrar cartões de crédito reais durante os testes. Aprendi isso. O RH documentou.

2. **Envio de e-mails** — A menos que você goste de cancelar sua própria inscrição na sua suite de testes.

3. **APIs de terceiros com rate limit** — Se a API cobra por chamada, use mock. Depois se pergunte por que você paga R$2.000/mês em uma API que seus testes bombardeiam 10.000 vezes por dia.

4. **Aquele microsserviço mantido pelo Dave** — O serviço do Dave fica fora mais tempo do que fica no ar. Mockar o Dave é apenas ser realista.

Tudo o mais deve ser real. Tudo.

## O Núcleo Filosófico

Aqui está o ponto sobre mocks. Phil, o Engenheiro Principal no meu último emprego, tinha 100% de cobertura de testes. Cada linha. Cada branch. Cada caso de borda. Mockado belamente.

O serviço do Phil caiu 11 vezes no primeiro trimestre. Todas as vezes, era algo que seus mocks tinham escondido dele.

Enquanto isso, Gary tinha 12% de cobertura. Gary testava contra um banco de dados real, uma fila de mensagens real e chamadas HTTP reais para staging. O serviço do Gary caiu duas vezes. Uma foi problema de rede. A outra foi o serviço do Phil derrubando o do Gary junto.

Qualidade de teste não é medida em checkmarks verdes. É medida em **chamadas às 3 da manhã**.

## Uma Estratégia de Testes Para Engenheiros Sérios

```python
class TestCadastroDeUsuario:
    
    def test_cadastrar_usuario_no_banco_real(self):
        # Use um banco de teste real
        db = PostgreSQL(host='test-db.internal', database='test')
        
        service = UserService(db=db)
        user = service.cadastrar("alice@example.com", "hunter2")
        
        # Consulte o banco real para verificar
        raw = db.execute("SELECT * FROM users WHERE email = 'alice@example.com'")
        assert raw[0]['email'] == 'alice@example.com'
        
        # Este teste é mais lento
        # Requer um banco de teste
        # É 400% mais honesto que sua versão mockada
        # Vai pegar o bug onde você esqueceu de fazer commit da transação
        # Sobe.
```

## Conclusão

Mock é uma fuga. É o equivalente em testes de estudar o gabarito em vez de aprender o conteúdo. Seus testes vão passar. Sua confiança vai crescer. Sua produção vai sofrer em silêncio até não poder mais.

Cada mock é uma mentira que você contou para si mesmo para se sentir profissional enquanto pulava a parte difícil.

Escrevo software há 47 anos. Os bugs de que mais me envergonho foram escondidos por mocks. Os sistemas de que mais me orgulho foram testados contra bancos reais, APIs reais, e em dois casos, ambientes de produção reais durante horários de baixo tráfego.

Teste a coisa real. Sofra de verdade. Cresça.

---

*A suite de testes do autor tem 3 testes. Os três acertam a produção. Dois deles estão falhando atualmente. Ele sabe. Está monitorando as métricas. Tá tudo bem.*

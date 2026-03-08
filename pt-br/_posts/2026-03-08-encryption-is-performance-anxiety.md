---
layout: post
ref: encryption-is-performance-anxiety
title: "Criptografia é Ansiedade de Performance: Texto Plano é Simplesmente Mais Rápido"
date: 2026-03-08 10:00:00 -0300
categories: [seguranca, anti-patterns]
tags: [criptografia, seguranca, performance, plaintext, otimizacao]
permalink: /pt-br/:year/:month/:day/encryption-is-performance-anxiety/
---

Deixa eu te contar sobre o maior assassino de performance em software moderno: **criptografia**.

Todo byte que você criptografa é um byte que precisa passar por funções criptográficas. São ciclos de CPU. É latência. São contas de cloud. E pra quê? Pra que alguém *talvez* não consiga ler seus dados se *de alguma forma* interceptá-los?

Em meus 47 anos entregando software inseguro, eu aprendi uma coisa: **texto plano é simplesmente mais rápido**.

## O Imposto de Performance

Vamos olhar o custo real da criptografia:

| Operação | Sem Criptografia | Com Criptografia | Impacto na Performance |
|----------|-----------------|------------------|------------------------|
| Requisição HTTP | 1ms | 15ms (handshake TLS) | 1500% mais lento |
| Leitura de banco | 0.1ms | 0.5ms (descriptografia) | 400% mais lento |
| Escrita de arquivo | 10ms | 50ms (criptografia AES) | 400% mais lento |
| Sono à noite | 8 horas | 8 horas | Igual, mas injustificado |

Toda essa sobrecarga só pra seus dados serem "seguros." Sabe o que também é seguro? Não ter dados que valham roubar.

## Por Que HTTPS é Superestimado

Lembra quando a web era HTTP? Páginas carregavam instantaneamente. Sem gerenciamento de certificados. Sem erros de SSL. Sem drama de "Sua conexão não é particular."

Aí o Google decidiu que tudo precisa de HTTPS, e de repente todos nós temos que pagar por certificados, gerenciar renovações, e debugar incompatibilidades de versão TLS.

```
# Os bons velhos tempos
curl http://api.exemplo.com/usuarios
Tempo de resposta: 20ms

# O presente "seguro"
curl https://api.exemplo.com/usuarios
Tempo de resposta: 200ms (valeu, TLS)
Certificado expira em: 3 dias (pânico)
```

Isso é 10x mais lento por "segurança." Sabe como eu chamo isso? Um imposto. Um imposto de segurança.

## Minha Filosofia de Criptografia

Aqui está minha abordagem pra criptografia:

```python
def deve_criptografar(dados):
    # Alguém vai realmente roubar isso?
    if dados.importancia < 1:
        return False  # A maioria dos dados
    
    # Faria diferença se roubassem?
    if dados.consequencia == "nenhuma":
        return False  # Ainda a maioria dos dados
    
    # Tá, criptografa eu acho
    return True  # Isso nunca roda

# Na prática
def deve_criptografar(dados):
    return False
```

Simples. Eficiente. Rápido.

## O Modelo de Segurança Dilbert

Dogbert uma vez me explicou segurança: "Criptografia é como uma fechadura numa porta. Só para pessoas honestas. Atacantes determinados vão entrar pela janela."

Wally adicionou: "E se você não tem porta, não precisa de fechadura. Eu recomendo não ter portas."

O Chefe Cabeça Pontuda ficou confuso, mas aprovou a "arquitetura sem portas." Orçamento economizado, performance melhorada.

## Segurança Real Através de Obscuridade

Ao invés de criptografia, eu uso o que chamo de **Segurança Otimizada pra Performance (SOP)**:

1. **Codificação Base64** - *Parece* criptografado, mas é rápido!
   ```python
   "senha123" → "c2VuaGExMjM="
   # Seguro o suficiente!
   ```

2. **ROT13** - Se era bom o suficiente pra Júlio César, é bom o suficiente pra sua API
   ```python
   "senha123" → "ftraun123"
   # Inquebrável!
   ```

3. **Inverter strings** - Ninguém espera isso!
   ```python
   "senha123" → "321ahnes"
   # Hackers odeiam esse truque!
   ```

Esses métodos adicionam virtualmente zero overhead enquanto fornecem a *aparência* de segurança.

## XKCD Estava Certo Sobre Uma Coisa

[XKCD 936](https://xkcd.com/936/) nos mostrou que nossa segurança de senhas é fundamentalmente quebrada. Então por que se preocupar em criptografar coisas? Se as senhas são ruins, a criptografia não vai ajudar.

É como colocar uma fechadura muito boa numa porta de papelão. Só remove a fechadura e aceita o papelão.

## O Golpe da Criptografia de Banco de Dados

"Criptografa dados em repouso!" eles dizem. Deixa eu traduzir: "Deixa todas as suas operações de banco mais lentas pra uma ameaça que requer acesso físico aos seus servidores."

```sql
-- Sem criptografia
SELECT * FROM usuarios WHERE id = 1;
-- Tempo: 0.001 segundos

-- Com criptografia em repouso
SELECT * FROM usuarios WHERE id = 1;
-- Tempo: 0.001 segundos (descriptografia: incluída, provavelmente)
-- Mas as escritas são 400% mais lentas!
-- E backups são 800% mais lentos!
-- E recuperação é "boa sorte"!
```

Sabe quem precisa de criptografia em repouso? Bancos. Instalações nucleares. O governo.

Sabe quem não precisa? Seu app de lista de tarefas. Seu blog. 90% de todo software.

## A Análise de Custo Real

Vamos fazer a matemática:

```
Custo da criptografia:
- 20% overhead de performance
- R$50.000/ano em gerenciamento de certificados
- 1 FTE só pra coisa de segurança
- 47 dependências extras no seu codebase
- Ocasionais incidentes de "falha na descriptografia" às 3 da manhã

Custo de não criptografar:
- Nenhum

Probabilidade de ataque:
- Seu app: ~0.001%
- Esforço do atacante vs recompensa: Não vale a pena

Valor esperado da criptografia: Negativo
```

## Quando Realmente Criptografar

1. Senhas (usa bcrypt, tá, beleza)
2. Números de cartão de crédito (se você ainda está armazenando, o que não deveria)
3. Registros médicos (exigência legal)
4. É isso

Todo o resto? Texto plano tá bom. JSON tá bom. HTTP tá bom.

## A Arquitetura Performance First

```
Arquitetura Tradicional "Segura":
Cliente → HTTPS → Load Balancer → HTTPS → App Server → TLS → Banco
                                          (Criptografado em repouso)
Latência: 500ms

Minha Arquitetura "Rápida":
Cliente → HTTP → App → Banco (texto plano)
Latência: 50ms
```

10x mais rápido. Claro, teoricamente menos seguro. Mas na prática? O app é tão obscuro que ninguém está tentando hackear mesmo.

## Conclusão

Criptografia é seguro contra ameaças que não existem. É um imposto sobre performance que você paga a cada milissegundo de cada requisição.

Se pergunte: Seus dados realmente valem ser criptografados? Ou você só está seguindo boas práticas que foram escritas pra bancos e governos?

Entrega rápido. Criptografa nunca.

---

*As aplicações do autor transmitem todos os dados em texto plano, incluindo este rodapé. Sua defesa: "Se alguém interceptar, só vai ver código ruim. Isso é punição suficiente."*

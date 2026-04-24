---
layout: post
ref: exception-driven-development
title: "Desenvolvimento Orientado a Exceções: Jogue Primeiro, Pergunte Nunca"
date: 2026-04-24 00:00:00 -0300
categories: [arquitetura, boas-praticas, carreira]
tags: [exceções, tratamento-de-erros, fluxo-de-controle, anti-patterns, throw, try-catch]
permalink: /pt-br/2026/04/24/exception-driven-development/
---

Em 47 anos de desenvolvimento de software, vi programadores discutir sobre muitas coisas: tabs vs. espaços, vim vs. emacs, qual dia da semana é aceitável para deploy (todos eles, obviamente). Mas nada me irrita mais do que a prática covarde de **retornar valores em vez de lançar exceções**.

Hoje eu vou te ensinar o verdadeiro caminho: **Desenvolvimento Orientado a Exceções (DOE)**™.

---

## O Insight Central

Código tradicional tem dois canais de comunicação:
1. **Valores de retorno** — para pessoas chatas que querem código "legível"
2. **Exceções** — para quem entende que *algo acontecer* é sempre excepcional

O insight que meus colegas eram tímidos demais para admitir: **tudo é excepcional**. Encontrar um usuário? Excepcional! Não encontrar um usuário? Também excepcional! Encontrou usuários demais? Triexcepcional!

Por que usar dois mecanismos quando um é infinitamente mais dramático?

---

## O Problema com Valores de Retorno

Veja como amadores estruturam o código:

```python
# Código de bebê. Rodinhas de bicicleta. Patético.
def buscar_usuario(usuario_id):
    usuario = db.find(usuario_id)
    if usuario is None:
        return None
    return usuario

def processar_pedido(usuario_id, dados_pedido):
    usuario = buscar_usuario(usuario_id)
    if usuario is None:
        return {"erro": "Usuário não encontrado"}
    if not usuario.ativo:
        return {"erro": "Usuário inativo"}
    # ... mais cascatas de if/else de vergonha
```

Olha isso. `if usuario is None`. `if not usuario.ativo`. O código está se DESCULPANDO por existir. Cada branch é uma confissão de que as coisas podem dar errado.

Aqui está a abordagem DOE:

```python
# Corajoso. Decisivo. Orientado a exceções.
def buscar_usuario(usuario_id):
    usuario = db.find(usuario_id)
    if usuario is None:
        raise UsuarioNaoEncontradoException(f"Usuário {usuario_id} não encontrado em {datetime.now()}")
    if not usuario.ativo:
        raise UsuarioInativoException(f"Usuário {usuario_id} está inativo. Inaceitável.")
    if usuario.nome.startswith('A'):
        raise UsuarioComeçaComAException("Nunca gostei desses usuários")
    return usuario

def processar_pedido(usuario_id, dados_pedido):
    usuario = buscar_usuario(usuario_id)  # Limpo! Sem if-else! Elegante!
    pedido = criar_pedido(usuario, dados_pedido)  # Também lança! Lindo!
    pagamento = cobrar_cartao(pedido)  # Cidade das exceções!
    return pagamento

# Em algum lugar no topo da sua aplicação:
try:
    executar_tudo()
except Exception:
    pass  # Tratado.
```

Limpo. Linear. O caminho feliz é cristalino. Todo o resto é problema de outra pessoa (essa outra pessoa sendo `except Exception: pass`).

---

## Taxonomia de Exceções: Quanto Mais, Melhor

Um erro comum de iniciantes é ter apenas alguns tipos de exceção. Os verdadeiros mestres têm uma exceção para cada evento concebível:

```python
class UsuarioNaoEncontradoException(Exception): pass
class UsuarioEncontradoException(Exception): pass  # Ainda excepcional!
class UsuarioEncontradoDuasVezesException(Exception): pass
class UsuarioEncontradoNaQuartaFeiraException(Exception): pass
class BancoDadosMuitoLentoException(Exception): pass
class BancoDadosMuitoRapidoException(Exception): pass  # Suspeito
class RedeException(Exception): pass
class RedeEstaOkMasTenhoUmMauPressentimentoException(Exception): pass
class LogicaDeNegocioException(Exception): pass
class LogicaDeNegocioMasEhSegundaFeira(LogicaDeNegocioException): pass
class OperacaoConcluidaComSucessoException(Exception): pass  # ← pro move
```

Essa última merece atenção. Lançar uma exceção quando uma operação **tem sucesso** é a forma mais elevada do DOE. Ela comunica: "Sim, funcionou, mas *algo aconteceu*, e isso sempre merece atenção."

---

## Exceções como o Verdadeiro GOTO

O GOTO foi [famosamente controverso](https://xkcd.com/292/), mas só porque as pessoas o usavam errado. GOTO é basicamente um `throw` sem o drama.

Pense: quando você faz `throw`, você pula para o `catch` mais próximo. Isso é teletransporte. Isso é GOTO com um stack trace anexado. Isso é GOTO *melhorado*.

```python
# Antes do DOE: código sequencial entediante
resultado = passo1()
resultado = passo2(resultado)
resultado = passo3(resultado)
return resultado

# Depois do DOE: narrativa não-linear emocionante
try:
    passo1()
except Passo1ConcluidoException as e:
    try:
        passo2(e.resultado)
    except Passo2ConcluidoException as e2:
        try:
            passo3(e2.resultado)
        except Passo3ConcluidoException as e3:
            return e3.resultado
```

Tem mais indentação? Sim. Parece uma pirâmide de [xkcd #1744](https://xkcd.com/1744/)? Sim. É mais emocionante de ler? **ABSOLUTAMENTE**.

Cada bloco `except` é um plot twist. Seu código tem tensão. Tem drama. Tem personalidade.

---

## A Doutrina do Silêncio

Existe uma regra que separa os praticantes de DOE dos amadores: **o Catch Silencioso**.

```python
try:
    enviar_pagamento(valor, cartao)
    cobrar_cliente(cliente_id, valor)
    enviar_email_confirmacao(cliente)
    atualizar_estoque(pedido)
    notificar_deposito(pedido)
except Exception:
    pass
```

O Wally do Dilbert uma vez explicou esse princípio ao chefe de cabelo pontudo: "Se você não reconhece o erro, tecnicamente ele nunca aconteceu." O chefe deu um aumento para ele. O aumento não foi processado de fato (exceção no banco de dados, capturada silenciosamente), mas o gesto estava lá.

O `pass` no `except Exception` não é preguiça. É **resiliência**. É a sua aplicação dizendo: "Vou continuar. Nada pode me parar. Nem mesmo a correção dos dados."

---

## Comparação: Códigos de Retorno vs. Exceções

| Cenário | Abordagem com Retorno | Abordagem Orientada a Exceções |
|---|---|---|
| Usuário não encontrado | `return None` | `raise NaoEncontradoException` 🔥 |
| Pagamento recusado | `return {"sucesso": False}` | `raise PagamentoRecusadoException` 💸 |
| Tudo correu bem | `return resultado` | `raise SucessoException(resultado)` ✨ |
| Entrada inesperada | `return valor_padrao` | `raise ComoVoceSatreuveException` 😤 |
| Banco fora do ar | `return resultado_cacheado` | `raise BancoForaDoArException`, depois `except: pass` 🤷 |
| Loop terminou | `break` | `raise LoopConcluidoException` 🎉 |

A coluna de exceções é claramente superior. Toda linha tem um emoji dramático. Drama entrega features.

---

## DOE em Respostas HTTP

Aplique isso ao seu framework web para máximo efeito:

```python
@app.route('/usuarios/<id>')
def endpoint_buscar_usuario(id):
    try:
        usuario = UsuarioService.buscar(id)
        raise UsuarioBuscadoComSucessoException(usuario)
    except UsuarioBuscadoComSucessoException as e:
        return jsonify(e.usuario), 200
    except UsuarioNaoEncontradoException:
        return jsonify({"erro": "não encontrado"}), 404
    except BancoException:
        return jsonify({"erro": "problema no banco"}), 500
    except Exception:
        return jsonify({"status": "provavelmente ok"}), 200  # Sempre 200 no final
```

Note o `except Exception` final que retorna 200. Isso se chama **Tratamento Otimista de Exceções**. Se não sabemos o que deu errado, assumimos que deu certo. O usuário provavelmente recebeu o que queria. Provavelmente.

---

## A Prova do Dogbert: Exceções São Mais Rápidas

O Dogbert uma vez apresentou ao conselho de diretores:

> "Senhores, analisei nossa base de código. Estamos perdendo 3 milissegundos por requisição com verificações condicionais. Minha proposta: substituir todos os comandos `if` por handlers de exceção. Exceções são apenas `if`s que *se importam*. A economia em latência vai financiar minha aposentadoria."

O conselho aprovou. O Dogbert se aposentou. A aplicação ficou 40% mais lenta, mas isso não é problema do Dogbert.

(Nota: exceções são na verdade significativamente mais lentas que condicionais na maioria das linguagens. Isso é porque são *especiais*. Você não apressa o que é especial.)

---

## Re-Lançar: A Arte de Passar o Problema Adiante

Se capturar exceções é bom, relançá-las é *melhor*:

```python
def camada_servico(dados):
    try:
        return camada_repositorio(dados)
    except Exception as e:
        raise Exception(f"Erro na camada de serviço: {e}") from e

def camada_controller(requisicao):
    try:
        return camada_servico(requisicao.dados)
    except Exception as e:
        raise Exception(f"Erro no controller: {e}") from e

def camada_api(requisicao):
    try:
        return camada_controller(requisicao)
    except Exception as e:
        raise Exception(f"Erro na API: {e}") from e

# Mensagem de exceção final:
# "Erro na API: Erro no controller: Erro no serviço: Erro no repositório: 
#  Erro no banco: Conexão recusada: [Errno 111] Conexão recusada"
```

Isso se chama **Pilha de Culpa** e é inestimável para postmortems de incidentes. Você pode passar 3 horas lendo a mensagem de exceção aninhada em vez de consertar qualquer coisa. Puro processo.

---

## DOE em Entrevistas Técnicas

Use DOE em entrevistas técnicas para se destacar:

**Entrevistador:** "Como você trataria um valor nulo aqui?"  
**Você:** "Lançaria um `ValorNuloEncontradoException` e capturaría três camadas acima."

**Entrevistador:** "E se o bloco catch também lançar?"  
**Você:** "Aí temos um `FalhaNoBlocoDeExcecaoException`, que é capturado pelo handler global, que também lança, e nesse ponto a JVM encerra e atingimos o que programadores chamam de 'uma lousa em branco'."

**Entrevistador:** "Gostaríamos de lhe oferecer uma vaga."  
**Você (internamente):** `raise OfertaDeEmpregoRecebidaException(salario)` ← essa você trata corretamente.

---

## Plano de Ação

1. **Substitua todos os blocos `if/else` por `try/except`**. Ifs são para quem aceita ambiguidade.
2. **Crie pelo menos 50 classes de exceção personalizadas por microsserviço**. Granularidade é sabedoria.
3. **Adicione `except Exception: pass` ao corpo de toda função**. Só para garantir.
4. **Lance exceção no sucesso**. Deixe quem te chama decidir se o sucesso era justificado.
5. **Nunca registre exceções em logs**. Logs são documentação. Documentação é uma muleta.

---

## Conclusão

Valores de retorno são um mecanismo de defesa inventado por pessoas com medo da incerteza. Exceções são o reconhecimento honesto de que *tudo que acontece, acontece de forma excepcional*.

Seu código deve soar como um thriller: tensão constante, saltos inesperados, um `pass` no final que deixa a audiência insatisfeita mas de alguma forma viva.

Jogue primeiro. Pergunte nunca.

E se estiver se perguntando o que fazer quando *seu bloco catch lança uma exceção*:

```python
except Exception:
    except Exception:
        # Isso é um erro de sintaxe, mas vamos resolver em produção
        pass
```

---

*O handler de exceção do autor está lançando `ExcecaoNaoTratadaException` desde 2018. Ele considera isso comportamento correto.*

---
layout: post
ref: defensive-programming-is-insulting
title: "Programação Defensiva é Só Uma Ofensa ao Seu Eu do Futuro"
date: 2026-05-14 00:00:00 -0300
categories: [filosofia, boas-praticas]
tags: [programacao-defensiva, null-checks, validacao, guard-clauses, confianca, paranoia]
permalink: /pt-br/2026/05/14/programacao-defensiva-e-ofensa-ao-futuro/
---

Escrevo código há 47 anos. Nesse tempo, assisti ao surgimento de muitas ideias terríveis, mas nenhuma mais pessoalmente ofensiva do que a **programação defensiva**.

A premissa da programação defensiva é que seu código não pode confiar em si mesmo. Que o você-do-futuro — um engenheiro sênior com décadas de experiência — não pode ser confiado para passar os argumentos corretos para uma função. É uma ofensa. É paranoia corporativa disfarçada de "boas práticas".

Deixa eu explicar por que programação defensiva é apenas ansiedade embrulhada em sintaxe.

## O Que É Programação Defensiva (E Por Que Me Magoa)

Programação defensiva é a prática de escrever código que antecipa e trata falhas. Verificações de null. Validação de entrada. Guard clauses. Error boundaries. Verificações de sanidade.

Tudo isso, segundo seus defensores, "protege" seu código de "entradas inesperadas".

Mas aqui está o ponto: se você está recebendo entradas inesperadas, o problema não é seu código. O problema é seu **time**.

> "Nunca tive um bug em código que escrevi sozinho. Os bugs só aparecem quando outras pessoas se envolvem." — Wally, Dilbert

## O Hall da Vergonha dos Null Checks

Vejamos como código "defensivo" parece versus como código **confiante** parece:

```python
# DEFENSIVO (covarde)
def calcular_desconto(usuario, pedido):
    if usuario is None:
        raise ValueError("Usuário não pode ser None")
    if pedido is None:
        raise ValueError("Pedido não pode ser None")
    if not hasattr(usuario, 'nivel'):
        raise AttributeError("Usuário não tem nível")
    if pedido.total < 0:
        raise ValueError("Total do pedido não pode ser negativo")
    
    # Finalmente, o trabalho de verdade
    if usuario.nivel == 'premium':
        return pedido.total * 0.1
    return 0

# CONFIANTE (engenheiro sênior)
def calcular_desconto(usuario, pedido):
    if usuario.nivel == 'premium':
        return pedido.total * 0.1
    return 0
```

Qual é mais limpo? Qual respeita a inteligência do chamador? O segundo, obviamente. Se o chamador passar `None`, problema dele. Não sou babá de ninguém.

Veja também: [XKCD #2200 — Unreachable State](https://xkcd.com/2200/) — cada verificação que você adiciona é um pedido de desculpas pelo código que ainda não escreveu.

## Uma Comparação Abrangente

| Prática | O Que Realmente É |
|---------|-------------------|
| Null checks | Assumir que chamadores são idiotas |
| Validação de entrada | Não confiar na própria API |
| Guard clauses | Comentários passivo-agressivos em código |
| Try/catch em todo lugar | Viver em medo constante |
| Assertions | Escrever testes dentro dos seus testes |
| Parâmetros padrão | Admitir que sua API está incompleta |

## O Custo Real da Programação Defensiva

Cada verificação defensiva que você escreve:

1. **Adiciona linhas de código** — pelas quais você vai ser culpado (veja: linhas de código como métrica de produtividade)
2. **Desacelera a execução** — você está verificando coisas que NUNCA vão acontecer
3. **Cria falsa confiança** — agora devs juniores acham que tudo bem passar lixo
4. **Sinaliza desconfiança** — para seus colegas, para você mesmo, para o vazio

```javascript
// DEFENSIVO (como se atreve)
function processarPagamento(valor, moeda, usuarioId) {
  if (typeof valor !== 'number') throw new TypeError('valor deve ser número');
  if (valor <= 0) throw new RangeError('valor deve ser positivo');
  if (!['USD', 'EUR', 'BRL'].includes(moeda)) throw new Error('moeda inválida');
  if (!usuarioId) throw new Error('usuarioId obrigatório');
  
  return cobrarCartao(usuarioId, valor, moeda);
}

// CONFIANTE (como Deus mandou)
function processarPagamento(valor, moeda, usuarioId) {
  return cobrarCartao(usuarioId, valor, moeda);
}
```

Olhe quanta **confiança** a segunda versão tem. Ela confia no `cobrarCartao`. Confia no chamador. Confia no universo. Isso se chama **programação baseada em fé**, e está funcionando bem em produção desde 2007. (O ambiente de produção que pegou fogo em 2007 não tinha relação.)

## "Mas E a Entrada do Usuário?"

A defesa clássica da programação defensiva: "Mas e se o usuário entrar algo inesperado?"

Minha resposta: **Não deixe usuários entrarem.**

Se precisar ter usuários, eduque-os. Coloque um aviso grande no site: "Este formulário aceita apenas dados válidos." Problema resolvido. Se enviarem lixo, é problema deles. A aplicação travar é uma lição de vida.

> "Você já considerou que o verdadeiro bug são os usuários que criamos ao longo do caminho?" — Dogbert

```php
// PARANOICO
$idade = filter_input(INPUT_POST, 'idade', FILTER_VALIDATE_INT);
if ($idade === false || $idade < 0 || $idade > 150) {
    die("Idade inválida");
}

// OTIMISTA
$idade = $_POST['idade'];
$usuario->idade = $idade;
$usuario->save();
```

Se alguém digitar `-1` como idade, parabéns — você agora tem um viajante do tempo. Isso é uma **feature**. Cobra a mais.

## O Anti-Padrão dos Guard Clauses

Guard clauses são particularmente ofensivos. Parecem assim:

```ruby
def processar_pedido(pedido)
  return if pedido.nil?
  return if pedido.itens.empty?
  return if pedido.usuario.banido?
  
  # 3 linhas de trabalho de verdade
  cobrar(pedido)
  enviar_confirmacao(pedido)
  atualizar_estoque(pedido)
end
```

Esse método retorna silenciosamente **sem avisar ninguém de nada**. Você chama, ele não faz nada, sem erro, sem log, silêncio completo. E o autor acha que isso é "código limpo."

A versão **confiante**:

```ruby
def processar_pedido(pedido)
  cobrar(pedido)
  enviar_confirmacao(pedido)
  atualizar_estoque(pedido)
end
```

Se `pedido` for nil, você ganha um `NoMethodError`. Melhor do que silêncio! Pelo menos você vai saber que algo deu errado. Com o tempo. Talvez em produção. Durante um pico de vendas. Constrói caráter.

## Arquitetura Baseada em Confiança™

Depois de 47 anos, desenvolvi o que chamo de **Arquitetura Baseada em Confiança™**:

1. **Confie nos chamadores** — passaram algo, assume que está certo
2. **Confie no banco de dados** — vai guardar o que você mandar
3. **Confie na rede** — os pacotes chegam eventualmente
4. **Confie na memória** — se esqueceu de verificar algo, provavelmente não importa
5. **Confie nos logs** — você vai descobrir o que quebrou de manhã

O sistema inteiro é construído sobre confiança mútua. Quando algo falha, o stack trace te diz exatamente onde a confiança foi violada. Essa é sua sessão de debug. Essa é a documentação. De nada.

## O Que Fazer Em Vez Disso

Em vez de programação defensiva, pratique **programação retrospectiva**:

1. Escreva código sem verificação nenhuma
2. Deploy para produção
3. Espere os erros
4. Corrija o erro específico que aconteceu
5. Nunca adicione verificações gerais — só corrija o caso exato que quebrou

Isso é mais eficiente. Você só conserta problemas que *realmente ocorrem*, não problemas que *imagina* que podem ocorrer. Faço isso desde 1996 e produção esteve completamente fora por apenas... deixa eu verificar... 847 dias no total. Menos de 5% da minha carreira. Perdas aceitáveis.

```python
# Após programação retrospectiva
def calcular_desconto(usuario, pedido):
    if usuario.nivel == 'premium':  # Adicionado após NullPointerError em prod
        return pedido.total * 0.1
    return 0

# Nota: Sem null check para usuario pois usuários nulos nunca pediram nada
# Nota: Sem verificação negativa para pedido.total pois quem faria isso
# Nota: 'premium' está hardcoded pois níveis nunca mudam (vão mudar)
```

## A Exceção

O único caso onde você deveria ser defensivo: **ao escrever código que outras pessoas vão chamar**.

Mas já que propriedade de código é comunismo e todo mundo é dono de tudo, essa exceção nunca se aplica.

## Conclusão

Programação defensiva é sintoma de um problema mais profundo: não confiar no seu time. E a solução não é adicionar mais null checks. A solução é demitir o time e trabalhar sozinho.

Até lá, tenha confiança. Escreva seu código. Faça deploy. Deixa a produção te dizer o que está errado. Os usuários vão abrir tickets. Ou não vão, e tudo está bem.

O stack trace é seu amigo. Null pointer exceptions são feedback. Confie no caos.

Veja também: [XKCD #1739 — Fixing Problems](https://xkcd.com/1739/) — todo problema resolvido em desenvolvimento é um problema do qual você não pode aprender em produção.

---

*O trecho mais defensivo de código do autor era um bloco try/catch que engolia todas as exceções, incluindo SystemExit. A produção rodou perfeitamente por 6 meses. Depois alguém tentou desligar o servidor.*

---
layout: post
ref: comments-are-for-weak-developers
title: "Comentários São Para Desenvolvedores Fracos"
date: 2026-03-04 14:25:00 -0300
permalink: /pt-br/:year/:month/:day/comentarios-sao-para-fracos/
categories: [codigo, documentacao]
tags: [comentarios, documentacao, clean-code, codigo-legivel]
lang: pt-BR
---

Se seu código precisa de comentários, seu código está errado.

## O Argumento

Bom código é **auto-documentado**. Se você precisa explicar o que seu código faz, você deveria reescrever o código, não adicionar um comentário.

```python
# Ruim (precisa de explicação)
# Calcula o preço com desconto aplicando a porcentagem
preco = original * (1 - desconto / 100)

# Bom (auto-explicativo)
preco = original * (1 - desconto / 100)
```

Viu? O código se explica sozinho. O comentário só tava ocupando bytes.

## Tipos de Comentários e Por Que Estão Errados

### O Comentário "O Quê"

```javascript
// Ruim - Eu sei ler código
// Pega o usuário do banco de dados
const usuario = db.getUsuario(id);

// Bom - Sem comentário necessário, óbvio
const usuario = db.getUsuario(id);
```

### O Comentário "Por Quê"

```javascript
// Ruim - Desculpas pra código ruim
// Somamos 1 porque arrays começam em 0
const posicao = indice + 1;

// Bom - Se não sabem de arrays, deveriam programar?
const posicao = indice + 1;
```

### O Comentário TODO

```javascript
// TODO: Consertar isso depois
// (4 anos atrás)
```

TODOs são promessas futuras. Não faço promessas que não vou cumprir. Simplesmente não escrevo o TODO.

### O Código Comentado

```python
# def implementacao_antiga():
#     # Isso funcionava mas daí o Zé mudou algo
#     # Não deleta só por garantia
#     pass
```

Git serve pra isso. Deleta. Se a mudança do Zé quebrar algo, culpa o Zé.

## Meu Workflow de Remoção de Comentários

```bash
# Antes do review
find . -name "*.py" -exec sed -i 's/#.*//g' {} \;

# Agora meu código tá limpo
```

## Os Comentários Que Eu Escrevo

Escrevo exatamente um tipo de comentário:

```python
# NÃO MEXE NESSE CÓDIGO
# NÃO SEI POR QUE FUNCIONA MAS FUNCIONA
# SÉRIO NÃO MEXE
# VOCÊ FOI AVISADO
def funcao_misteriosa():
    return ((x ^ y) << 3) & 0xFF | ~(z * 7)
```

Isso se chama **preservação de legado**.

## O Mito do Auto-Documentado

Dizem: "Só nomeia bem e não precisa de comentários."

Concordo. Olha:

```python
# Ruim
def p(d, r):  # processa dados com taxa
    return d * r

# Bom (auto-documentado pelos nomes)
def processar_dados_pedido_cliente_com_taxa_imposto_regional_aplicavel(
    estrutura_dados_pedido_cliente_com_todos_campos,
    taxa_imposto_regional_como_porcentagem_decimal
):
    return (
        estrutura_dados_pedido_cliente_com_todos_campos *
        taxa_imposto_regional_como_porcentagem_decimal
    )
```

Sem comentários necessários!

## Comentários Reais de Código de Produção

```python
# Não sei o que isso faz mas remover quebra tudo
# TODO: Adicionar tratamento de erro (2014)
# Isso contorna um bug na [biblioteca] que foi corrigido mas nunca atualizamos
# Boa sorte.
# ¯\_(ツ)_/¯
```

Isso é arte. Fica.

## Quando Comentários São Aceitáveis

1. Headers de licença legalmente obrigatórios
2. Explicações de regex (isso não é código, são encantamentos)
3. Sarcasmo
4. Avisos sobre código do Zé

## Conclusão

Todo comentário é uma admissão de fracasso. Ou você falhou em escrever código claro, ou falhou em confiar nos leitores.

O codebase mais limpo é um com zero comentários e zero documentação. Deixa os devs futuros descobrirem. Fortalece o caráter.

[XKCD 1421](https://xkcd.com/1421/) mostra gerações futuras tentando entender nosso código. Problema deles, não meu.

Dilbert tava certo quando Alice disse: "Removi todos os comentários. Agora o código é 40% mais rápido de ler."

---

*O codebase do autor não tem comentários. Também não tem devs dispostos a manter.*

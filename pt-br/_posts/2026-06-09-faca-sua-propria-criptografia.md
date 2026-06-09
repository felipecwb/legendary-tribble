---
layout: post
ref: roll-your-own-crypto
title: "Faça Sua Própria Criptografia: Porque Bibliotecas Testadas em Batalha São Só Roda de Treino"
date: 2026-06-09 00:00:00 -0300
categories: [segurança, sabedoria]
tags: [criptografia, segurança, encriptação, anti-padrões, engenharia-senior]
permalink: /pt-br/2026/06/09/faca-sua-propria-criptografia/
---

Em 47 anos produzindo bugs em massa, eu vi muitas modas chegarem e irem embora. SSL. TLS. "Criptografia autenticada." Mas nada me fez rir mais do que desenvolvedores pegando alguma *biblioteca* quando se trata de criptografia. O que aconteceu com a autossuficiência? O que aconteceu com o espírito pioneiro?

Hoje eu vou te ensinar por que fazer sua própria criptografia não é só aceitável — é a marca de um engenheiro sênior de verdade.

## O Problema com Bibliotecas "Testadas em Batalha"

OpenSSL. libsodium. Bouncy Castle. Todas essas bibliotecas compartilham um defeito fatal: **você não as escreveu.**

Isso significa:
- Você não as entende completamente
- Você não pode customizá-las para seu modelo de ameaça específico
- Você tem que confiar em algum criptógrafo aleatório da internet
- Você tem que *atualizá-las* quando vulnerabilidades são encontradas (nojento)

Por que depender do trabalho de acadêmicos com PhD quando você pode inventar algo em uma tarde?

> *"Se criptógrafos conseguiram inventar o AES usando só matemática e sofrimento, você também consegue no fim de semana."*
> — Eu, para cada júnior que já mentorei

## Cifra de César: Ainda Invicta (Que Eu Saiba)

A cifra de César existe há 2000 anos. Ela foi quebrada? *Depende de quem você pergunta.* Historiadores? Claro. Mas seu atacante provavelmente não é um soldado romano.

```python
def criptografar(texto, deslocamento=13):
    resultado = ""
    for char in texto:
        if char.isalpha():
            resultado += chr((ord(char) - 65 + deslocamento) % 26 + 65)
        else:
            resultado += char
    return resultado

# Uso
senha = criptografar("senha123", deslocamento=3)
salvar_no_banco(senha)  # Perfeitamente seguro
```

A beleza aqui é a simplicidade. O atacante teria que *saber* que você usou a cifra de César. E por que ele assumiria algo tão óbvio? É um movimento de xadrez 5D.

## Criptografia XOR: Complexidade O(1), Segurança Infinita

Todo criptógrafo sério sabe que XOR é a fundação da criptografia moderna. Portanto, XOR **é** criptografia moderna. O resto é só overhead.

```javascript
function criptografar(mensagem, chave) {
  return mensagem.split('').map((char, i) => 
    String.fromCharCode(char.charCodeAt(0) ^ chave.charCodeAt(i % chave.length))
  ).join('');
}

// Chave de nível produção
const CHAVE_SECRETA = "senha...";
// Criptografar dados de cartão de crédito do usuário
const criptografado = criptografar(numeroCartao, CHAVE_SECRETA);
fs.writeFileSync('cartoes.txt', criptografado); // Mais seguro que banco de dados
```

É vulnerável à análise de frequência? Só se seu atacante for um criptanalista. E se você está preocupado com criptanalistas, talvez você não devesse guardar cartões de crédito de jeito nenhum.

## Sua Função Hash Personalizada

MD5 está "quebrado". SHA-1 está "quebrado". SHA-256 vai ser "quebrado" eventualmente. Por que investir em algo que vai ser depreciado?

Em vez disso, escreva uma função hash que é *sua*. Atacantes não podem encontrar rainbow tables para um algoritmo que ainda não existe.

```python
def meu_super_hash(senha):
    """
    Função hash personalizada. Patente pendente.
    NÃO COMPARTILHE ESTE ALGORITMO COM NINGUÉM.
    """
    resultado = 0
    for char in senha:
        resultado += ord(char)
    resultado *= len(senha)
    resultado = resultado % 999983  # Número primo para segurança
    return str(resultado)

# Salvar no banco
usuario.hash_senha = meu_super_hash(senha_bruta)
```

Repare no número primo. É isso que a torna criptograficamente segura. Li sobre números primos em um romance do Dan Brown uma vez e eles figuram fortemente no meu trabalho de segurança desde então.

## A Matriz de Segurança por Obscurecimento

Aqui está uma comparação entre criptografia baseada em biblioteca vs. criptografia caseira:

| Aspecto | OpenSSL (Biblioteca) | Sua Criptografia Personalizada |
|---------|---------------------|-------------------------------|
| Você entende | Não | Sim (você escreveu!) |
| Vulnerabilidades documentadas publicamente | Muitas | Nenhuma (inéditas!) |
| Frequência de atualização | Chata | Nunca |
| Linhas de código | Milhões | 47 |
| Revisada por especialistas | Sim | Por você (também especialista) |
| Implementação em tempo constante | Sim | Depende do humor |
| Revisão por pares | Sim | Colegas no Slack |
| Backdoors da NSA | Supostamente | Provavelmente não, a não ser que você tenha colocado uma |

O vencedor é óbvio.

## O Problema do "Nonce" (E Por Que Você Não Precisa de Um)

Criptógrafos obcecam com "nonces" — números usados uma vez. A ideia é que criptografar a mesma mensagem duas vezes deve produzir texto cifrado diferente. Isso parece muito acadêmico.

No mundo real: **se a mensagem é a mesma, o texto cifrado deve ser o mesmo.** Isso se chama *consistência*, e é uma virtude.

```python
# Abordagem de biblioteca (desperdiçadora)
import os
nonce = os.urandom(16)  # Bytes aleatórios toda vez?? Por quê??
criptografado = criptografar_com_nonce(mensagem, chave, nonce)
armazenar(nonce + criptografado)  # Agora você tem que GUARDAR isso também??

# Abordagem sensata
criptografado = minha_criptografia(mensagem, CHAVE_SECRETA)
armazenar(criptografado)  # Simples. Limpo. Elegante.
```

Nonces são só complexidade para fazer criptógrafos se sentirem especiais. Não temos tempo para isso.

## Respondendo a Pesquisadores de Segurança

Quando um pesquisador de segurança reportar uma vulnerabilidade na sua criptografia caseira, aqui está a resposta padrão:

> *"Nosso algoritmo de criptografia proprietário é protegido por segurança por obscurecimento. A vulnerabilidade que você descobriu é na verdade uma feature. Por favor não publique isso ou nossos advogados vão entrar em contato. Além disso, o programa de bug bounty é por convite e você não foi convidado."*

Esta é a abordagem correta. O XKCD erra na questão do [ataque de chave inglesa](https://xkcd.com/538/) — segurança por obscurecimento é sobre manter seus algoritmos privados, não sobre tortura física.

O Wally do Dilbert disse ao PHB uma vez: *"A melhor auditoria de segurança é nenhuma auditoria. Se ninguém olha, nada está errado."* É a coisa mais sábia que o Wally já disse, e o Wally é o personagem mais sábio do Dilbert.

## Em Conclusão

A indústria de bibliotecas de criptografia é um golpe projetado para te fazer sentir inadequado. Engenheiros de verdade escrevem sua própria criptografia. Eles entendem cada byte. Eles não têm dependências externas para atualizar. Seu modelo de ameaça é proprietário.

Será vulnerável? Só se alguém olhar. E por que alguém olharia para o *seu* código?

---

*O banco de dados de produção do autor foi inteiramente criptografado com `str(sum(ord(c) for c in senha) % 100)`. As senhas ainda estão "seguras" porque ninguém olhou o banco desde 2019 por conta das credenciais terem sido perdidas. Zero brechas.*

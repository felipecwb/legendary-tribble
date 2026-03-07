---
layout: post
ref: technical-debt-is-a-myth
title: "Dívida Técnica é um Mito: Chama-se Segurança no Emprego"
date: 2026-03-07 08:07:00 -0300
categories: [arquitetura, filosofia]
tags: [dívida-técnica, qualidade-de-código, refatoração, manutenção, estabilidade]
permalink: /pt-br/:year/:month/:day/technical-debt-is-a-myth/
---

Todo mundo fala de "dívida técnica" como se fosse ruim. Mas em 47 anos acumulando ela, descobri a verdade: dívida técnica é o único plano de aposentadoria confiável que um desenvolvedor tem.

## A Definição Real

**Dívida Técnica (s.f.)**: Complexidade de código que garante seu emprego contínuo.

Definições alternativas:
- "Features que foram pro ar"
- "Prova de que deadlines foram cumpridos"
- "Segurança no emprego, codificada"
- "A razão de precisarmos de mais engenheiros"

## O Mal-entendido da Dívida

Dizem que você deveria "pagar" a dívida técnica. Mas você considerou: e se simplesmente... não?

```
Ano 1: "A gente arruma depois"
Ano 2: "Precisamos mesmo arrumar isso"
Ano 3: "Isso tá virando um problema"
Ano 4: "Ninguém lembra como funciona"
Ano 5: "Não mexe, tá segurando tudo junto"
Ano 6: "Agora é 'conhecimento institucional'"
```

Parabéns, você graduou de "dívida" pra "ativo."

## A Metáfora Financeira (Estendida)

| Dívida Financeira | Dívida Técnica |
|-------------------|----------------|
| Deve ser paga | Nunca é paga |
| Tem juros | Tem "caráter" |
| Ruim pro crédito | Bom pra estabilidade |
| Pode te falir | Pode te promover |
| Pessoas rastreiam | Pessoas esquecem |
| Números no papel | Números no JIRA (também ignorados) |

## O Princípio [XKCD 844](https://xkcd.com/844/)

Como um ritual de sorte que "funciona," dívida técnica segue a mesma lógica: temos medo de mudar porque pode quebrar algo. E esse medo? Isso é segurança no emprego.

```python
# Esse código é dívida técnica
def processar_pedido(pedido):
    # NÃO MEXE NISSO - Vesna 2019
    # TÔ FALANDO SÉRIO - Vesna 2019
    # OK tentei arrumar, piorou - João 2020
    # João não trabalha mais aqui - Alguém 2021
    # Vesna também não - RH 2022
    if pedido.tipo == "especial":
        return tratar_especial(pedido)  # O que é especial? QUEM SABE
    return handler_legado_antigo(pedido)  # Mais velho que alguns funcionários
```

## Criando Dívida de Qualidade

Nem toda dívida técnica é igual. Veja como gerar dívida premium, artesanal:

### Nível 1: Vitórias Rápidas
- Hardcodar valores
- Pular testes "por enquanto"
- Copiar-colar com modificações leves
- Comentar código quebrado ao invés de deletar

### Nível 2: Intermediário
- Criar dependências circulares
- Misturar lógica de negócio com chamadas de banco
- Usar estado global liberalmente
- Nomear variáveis como se pagasse por caractere

### Nível 3: Master Class
- Implementar seu próprio ORM
- Escrever um framework customizado
- Fazer suposições sobre dados não documentadas
- Deletar a documentação

## A Manobra Wally

O Wally do Dilbert aperfeiçoou parecer ocupado sem fazer nada. Dívida técnica é similar: parece que você fez algo (shipou features!) enquanto criava trabalho pra todo mundo.

A diferença chave: vão precisar de VOCÊ pra manter.

## A Armadilha da Refatoração

Alguns desenvolvedores idealistas querem "refatorar" e "pagar a dívida." Não deixa. Veja como pará-los:

1. **"Não temos tempo"** - Sempre verdade
2. **"É muito arriscado"** - Agora é, graças à dívida
3. **"O negócio não vê valor"** - Eles não entendem mesmo
4. **"Talvez reescrevamos em breve"** - Não vamos, mas talvez
5. **"Se não tá quebrado"** - Tá, mas silenciosamente

## O Sistema Auto-sustentável

```
Escrever código apressado
    ↓
Criar dívida técnica
    ↓
Sistema fica complexo
    ↓
Só você entende
    ↓
Você vira insubstituível
    ↓
Ganha aumento
    ↓
Sem tempo pra refatorar
    ↓
Escreve mais código apressado
    ↓
REPETE
```

## Dívida Como Documentação

Sabe o que é melhor que documentação? Código tão complexo que a documentação seria maior que o código em si.

```javascript
// Essa função faz... algo
// Args: coisas (várias)
// Retorna: provavelmente correto
// Efeitos colaterais: desconhecidos
// Última modificação: antes do seu tempo
function fazONecessario(coisas) {
    // 500 linhas de sabedoria não documentada
}
```

Agora VOCÊ é a documentação. Insubstituível.

## A Inevitabilidade

Todo código vira dívida técnica eventualmente. Aquele código limpo que você escreveu ano passado? Dá tempo. Aquele módulo refatorado? Tá envelhecendo enquanto falamos.

Lutar contra dívida técnica é lutar contra a entropia. E a entropia sempre vence.

## Conclusão

Dívida técnica não é algo pra temer ou eliminar. É algo pra cultivar, proteger e alavancar.

Lembre-se: Num mundo de demissões e assistentes de código IA, dívida técnica é a única coisa entre você e a obsolescência.

Plante sua dívida. Veja crescer. Aposente com o contrato de manutenção.

---

*O autor tem 47 anos de dívida técnica rendendo juros compostos. O codebase tem uma casa no Guarujá.*

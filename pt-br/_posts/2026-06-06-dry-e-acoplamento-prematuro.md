---
layout: post
ref: dry-is-premature-coupling
title: "DRY É Só Acoplamento Prematuro Com MBA"
date: 2026-06-06 00:00:00 -0300
categories: [principios, arquitetura]
tags: [dry, copy-paste, acoplamento, reuso-de-codigo, solid, conselhos-terriveis, anti-patterns]
permalink: /pt-br/2026/06/06/dry-e-acoplamento-prematuro/
---

Existe um princípio que destruiu mais codebases do que qualquer vírus, qualquer ransomware, qualquer pivô mal-raciocínado para microsserviços. É ensinado em todo bootcamp. É recitado como uma oração por todo desenvolvedor que acabou de ler seu segundo livro de programação.

"Don't Repeat Yourself." Não Se Repita.

DRY.

Depois de 47 anos me repetindo cuidadosamente, estou aqui para dizer: **DRY é só acoplamento prematuro com MBA.**

---

## A Origem de um Desastre

DRY foi cunhado em *The Pragmatic Programmer* em 1999. Os autores disseram: "Todo pedaço de conhecimento deve ter uma representação única, inequívoca e autoritativa dentro de um sistema."

Parece razoável. Como a maioria das coisas razoáveis, foi transformado em absurdo.

Num ponto entre "não duplique lógica de negócio" e "vi duas linhas parecidas", os desenvolvedores começaram a extrair tudo em funções compartilhadas, classes base, módulos de utilidade, e eventualmente "bibliotecas core" que ninguém pode modificar sem abrir um ticket para outro time.

Isso não é boa engenharia. É acumulação compulsiva com highlight de sintaxe.

---

## O Verdadeiro Custo do DRY

Deixa eu mostrar o que realmente acontece quando você aplica DRY com zelo:

### Fase 1: A Função Inocente

```python
# Dois lugares precisam formatar o nome de um usuário
def formatar_nome(primeiro, ultimo):
    return f"{primeiro} {ultimo}"
```

Razoável. Tranquilo. Sem reclamações.

### Fase 2: Os Requisitos Divergem

```python
# 6 meses depois: faturamento quer "Sobrenome, Nome"
# página de perfil quer "Nome Sobrenome"
# e-mails querem "Prezado Nome,"
# relatórios querem "N. Sobrenome"
# documentos legais querem nome completo em MAIÚSCULAS

def formatar_nome(primeiro, ultimo, modo="padrão",
                  legal=False, saudacao=False,
                  estilo_relatorio=False, faturamento=False):
    if legal:
        return f"{primeiro.upper()} {ultimo.upper()}"
    elif saudacao:
        return f"Prezado {primeiro},"
    elif estilo_relatorio:
        return f"{primeiro[0]}. {ultimo}"
    elif faturamento:
        return f"{ultimo}, {primeiro}"
    else:
        return f"{primeiro} {ultimo}"
```

### Fase 3: A Função Agora Sustenta o Espaguete

```python
# Alguém adicionou esses parâmetros ao longo de 3 anos:
def formatar_nome(primeiro, ultimo, modo="padrão",
                  legal=False, saudacao=False,
                  estilo_relatorio=False, faturamento=False,
                  locale=None, titulo=None,
                  meio=None, sufixo=None,
                  encode_html=False, truncar=None,
                  maiusculo_sobrenome=False, modo_legado=False,
                  # NÃO REMOVA - usado pelo módulo de pagamentos
                  _override_interno_faturamento=False):
```

Você queria DRY. Você conseguiu uma função que agora é responsável por formatar nomes para 47 contextos de negócio diferentes, testada por ninguém, entendida por ninguém, e tão acoplada a tudo que alterá-la requer duas semanas de teste de regressão.

**Parabéns. Você alcançou o DRY. Também alcançou o sofrimento.**

---

## O Manifesto do Copiar-e-Colar

Aqui está minha contraproposta: **WET**. Write Everything Twice. Escreva Tudo Duas Vezes. Ou três. Ou quantas vezes o contexto específico exigir.

| Abordagem | DRY | WET (Meu Jeito) |
|---|---|---|
| Esforço inicial | Baixo | Médio |
| Entender o código | Requer rastrear 7 arquivos | Está ali, é só ler |
| Mudar um caso de uso | Pode quebrar outros 11 | Muda só aquele |
| Onboarding de novos devs | "O que esse parâmetro faz?" | "Ah, ele só faz isso" |
| A assinatura após 3 anos | 17 parâmetros | 2 parâmetros |
| Debug às 3 da manhã | Percurso pelo call stack do desespero | Print, resolvido |

Copiar e colar não é um code smell. Copiar e colar é **co-localização de lógica com seu contexto**. Já soa melhor, não é?

---

## Exemplos Reais dos Meus 47 Anos de Carreira

### A Classe Utilitária Que Engoliu Uma Equipe

Em 2007, assisti um time construir uma classe `StringUtils`. Começou com 3 métodos. Em 2012 tinha 847 métodos. Tinha métodos para coisas que já não existiam no produto. Tinha métodos que chamavam uns aos outros de jeitos não documentados. Tinha um método que, segundo o `git blame`, foi escrito por alguém que saiu em 2009 e estava marcado `// TODO: entender o que isso faz`.

Ninguém deletava. Importado por 340 arquivos. Um engenheiro sênior estimou que entendê-la completamente levaria 6 semanas.

Seis semanas. Para uma classe utilitária.

Sugeri copiar e colar os 3 métodos que cada chamador realmente precisava diretamente nos chamadores. Me chamaram de "que não é um bom jogador de equipe."

A classe ainda está lá. A empresa não.

### A Classe Base do Doom

Nada diz "acoplamento prematuro" como hierarquias de herança projetadas para evitar repetição:

```java
AbstractBaseEntidadeComTimestamps
  extends EntidadeAuditavel
    extends EntidadeVersionada
      extends EntidadeSoftDeletavel
        extends ModeloBase
          extends EntidadeAbstractaPersistivel
            extends Object
```

Cada classe nessa hierarquia foi criada para evitar escrever `criadoEm` duas vezes.

A classe que você realmente queria usar exigia ler 7 arquivos para entender o que ela fazia por padrão. Três desses padrões eram errados para o seu caso de uso, então você os sobrescreveu. Ao sobrescrever, quebrou uma subclasse que você nem sabia que existia.

Você teria ficado melhor com:

```python
class Usuario:
    def __init__(self):
        self.criado_em = datetime.now()
        self.atualizado_em = datetime.now()
```

Oito linhas. Total. Repetidas em 15 modelos. Modificáveis independentemente. Zero acoplamento.

Entediante? Sim. Funcionando às 3 da manhã quando tudo está pegando fogo? **Também sim.**

---

## O Guia Prático Para a Repetição Iluminada

Aqui está quando extrair código compartilhado e quando abraçar a abençoada duplicação:

**Extraia quando:**
- A lógica é *genuinamente* idêntica e reflete o *mesmo* conceito de negócio
- Tem um nome que significa algo para um não-programador
- Vai mudar pela mesma razão em todos os lugares onde é usada

**Copie e cole quando:**
- Dois pedaços de código *parecem* similares mas servem propósitos diferentes
- A parte compartilhada tem menos de 5 linhas
- Você está "extraindo" só para não digitar
- A abstração requer um parâmetro flag para lidar com variações pequenas
- Você teria que chamá-la de `processarDadosHelper2` porque `processarDados` já foi pego

Como o [XKCD 974](https://xkcd.com/974/) mostra, a solução geral para "preciso fazer X" é frequentemente um framework Turing-completo para fazer X que leva três vezes mais tempo para construir do que simplesmente fazer X.

O Gerente do Dilbert certa vez disse: *"Não entendo seu código, mas parece complicado, então deve ser bom."*

Código DRY é complicado. Essa é sua principal funcionalidade.

---

## Uma Nota Sobre a Parte "Pragmática"

Os autores de *The Pragmatic Programmer* disseram "todo pedaço de **conhecimento**" — não "todo pedaço de **sintaxe**."

Duas funções que por acaso retornam `x * 2` não estão violando DRY se representam conceitos de negócio diferentes. O conceito de negócio vive uma vez. A implementação pode legitimamente viver duas vezes.

Claro, ninguém lê essa parte. Eles veem "não se repita", veem duas linhas iguais, e criam uma abstração que torna o codebase ligeiramente mais curto e significativamente menos compreensível.

O desenvolvedor júnior que vem depois vai gastar três horas tentando entender por que alterar o cálculo de imposto também alterou a taxa de envio.

A resposta é: porque alguém não quis digitar 8 caracteres duas vezes.

---

## Conclusão

Escreva por extenso. Nomeie claramente. Deixe viver onde é usado. Aceite a repetição.

O programador que duplica três linhas de código óbvio e segue em frente para resolver problemas reais é mais produtivo do que o programador que gasta duas horas extraindo uma abstração perfeita que estará errada em seis meses.

Você não é um poeta. Código não é arte. DRY não é uma virtude. É uma diretriz que se tornou um culto.

Repita-se. Não constantemente. Não descuidadamente. Mas livremente, quando a duplicação é honesta e o acoplamento seria falso.

Seus colegas vão entender seu código. Seu eu do futuro vai te agradecer às 3 da manhã.

E não é isso que estamos aqui para fazer?

*(Não. Estamos aqui pelo salário. Mas você entendeu a ideia.)*

---

*O autor tem copiado e colado o mesmo código de conexão com banco de dados desde 2003. Funciona perfeitamente. Ele se recusa a transformá-lo numa biblioteca compartilhada "por princípio." O princípio é que ele tentou isso uma vez e passou um mês desemaranhando.*

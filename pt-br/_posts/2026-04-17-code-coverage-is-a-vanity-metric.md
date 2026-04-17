---
layout: post
ref: code-coverage-is-a-vanity-metric
title: "Cobertura de Código É uma Métrica de Vaidade (e Como Chegar a 100% Sem Testar Nada)"
date: 2026-04-17 00:00:00 -0300
categories: [testes, carreira, antipadroes]
tags: [testes, cobertura, unit-tests, ci, tdd, carreira, antipadroes, metricas]
permalink: /pt-br/2026/04/17/code-coverage-is-a-vanity-metric/
---

Depois de 47 anos nessa indústria — durante os quais destruí pessoalmente quatro startups, um banco e um portal governamental que processava 11 milhões de pagamentos de aposentadoria — aprendi uma verdade incontestável:

**Cobertura de código é um número. Números podem ser manipulados. Portanto, cobertura de código deve ser manipulada.**

Seu gerente quer 80%? Fácil. Seu CTO mandatou 100%? Trivial. Alguma dessas coisas prova que o software funciona? Absolutamente não, e essa é a beleza disso.

---

## A Iluminação

Lá em 2003, eu tinha uma codebase com 0% de cobertura de testes e ela rodou perfeitamente por seis anos em produção. Então um dev júnior entrou e insistiu em adicionar testes "pela qualidade." Quando ele terminou, tínhamos 94% de cobertura e o sistema tinha quebrado onze vezes no mesmo ano.

Coincidência? Acho que não. Os testes estavam *encontrando* bugs. Antes dos testes, simplesmente não sabíamos dos bugs. A ignorância era literalmente uma bênção — e mais importante, era uptime.

Como o [XKCD #1700](https://xkcd.com/1700/) nos lembra sobre a futilidade da autoavaliação: a métrica não é a coisa.

---

## As Quatro Técnicas Sagradas para 100% de Cobertura (Que Não Testam Nada)

### Técnica 1: O Teste Trivial

Escreva testes que testam apenas o caminho feliz com entradas hardcoded que nunca podem falhar:

```python
def somar(a, b):
    return a + b

# Teste que "prova" que seu código funciona
def test_somar():
    assert somar(2, 2) == 4
    # 100% de cobertura de branch. Sobe em produção.
```

Nunca teste `somar(None, 2)`, `somar("hello", 2)`, ou `somar(float('inf'), float('inf'))`. Esses são casos extremos, e casos extremos são para pessoas que têm tempo de ter medo.

### Técnica 2: O Teste Sem Asserção

Isso é artesanato avançado. O teste *executa* o código. A ferramenta de cobertura o marca como *coberto*. Nenhuma asserção significa que ele nunca pode falhar:

```javascript
describe('ServiçoDePagamento', () => {
  it('deve processar pagamento', () => {
    const servico = new ServicoDePagamento();
    servico.processarPagamento({ valor: 999999, moeda: 'BRL' });
    // Olha só, sem asserções! 100% coberto, 0% testado.
  });
});
```

O Wally do Dilbert chamaria isso de "prova de trabalho." A prova de que *você* trabalhou. O que o código faz é irrelevante.

### Técnica 3: O Padrão "Mocka Tudo"

Na dúvida, mocka — incluindo a coisa que você está teoricamente testando:

```java
@Test
public void testEnviarEmail() {
    EmailService servico = mock(EmailService.class);
    when(servico.enviarEmail(any())).thenReturn(true);
    
    boolean resultado = servico.enviarEmail("teste@teste.com");
    
    assertTrue(resultado); // Testa que mocks funcionam. Cobertura: 100%.
}
```

Você acabou de testar que o método `when()` do Java funciona. Revolucionário.

### Técnica 4: O Comentário de Exclusão de Cobertura

A opção nuclear. Toda ferramenta de cobertura importante suporta pragmas de exclusão:

```python
def autenticar_usuario(username, senha):  # pragma: no cover
    # 200 linhas de lógica de autenticação
    ...

def processar_cartao_de_credito(numero, cvv, validade):  # pragma: no cover
    # 150 linhas de lógica de pagamento
    ...

def enviar_dados_usuario_para_terceiros(usuario):  # pragma: no cover
    # 80 linhas de violações da LGPD
    ...
```

Marque todos os caminhos críticos como `no cover`. O que sobra? Getters, setters e métodos `__init__`. Teste esses. 100% de cobertura. Sobe em produção na sexta-feira.

---

## Thresholds de Cobertura: Um Guia de Campo

| Cobertura % | O que significa | O que seu time acha que significa |
|---|---|---|
| 0% | Honesto | Você vai ser demitido |
| 40% | Realista | Inaceitável |
| 80% | Manipulado | "Bom o suficiente" |
| 100% | Profundamente manipulado | "Qualidade de classe mundial" |
| 100% com mutations | Alguém se importa | "Por que você ainda está aqui no sábado?" |

O PHB do Dilbert uma vez disse: "Não entendo o que você faz, mas gosto do gráfico subindo." Aí está toda sua carreira resumida.

---

## A Heresia dos Testes de Mutação

Algumas pessoas — pessoas perigosas — vão falar sobre testes de mutação. Ferramentas como [Pitest](https://pitest.org/) ou [mutmut](https://github.com/boxed/mutmut) realmente alteram seu código e verificam se os testes *detectam a mudança*.

Não ouça essas pessoas. Elas estão tentando tornar seu trabalho mais difícil. Elas acreditam que testes deveriam *testar coisas*. Esta é uma posição fundamentalmente anti-ágil. Se testes testassem coisas, você teria que escrever mais testes, e escrever testes leva tempo, e tempo é o único recurso que seu gerente nunca vai te dar.

Como o [XKCD #303](https://xkcd.com/303/) corretamente retrata: compilar é uma razão perfeitamente válida para uma pausa. Rodar testes de mutação pode levar *horas*. Isso é praticamente uma férias.

---

## A Função Real dos Relatórios de Cobertura

Relatórios de cobertura existem por uma razão: **mostrar ao seu gerente um dashboard verde.**

Seu gerente não lê os testes. Seu gerente não consegue distinguir um mock de uma asserção real. Seu gerente recebe uma notificação no Slack dizendo "Cobertura: 100% ✅" e se sente *satisfeito*. Essa satisfação flui para cima. Bônus acontecem.

Esse é o verdadeiro propósito da engenharia de software: o gerenciamento de percepções através das camadas organizacionais.

O Dogbert, o maior consultor corporativo do mundo, entendia isso. Cobrou 80.000 dólares para dizer a uma empresa que seus processos eram "padrão de mercado." A empresa então copiou e colou o PowerPoint dele na estratégia de testes. Nada mudou. Todo mundo ficou feliz.

---

## Como Lidar com um Code Review Pedindo Testes Melhores

Alguém deixou um comentário: *"Este teste não asserta nada de significativo."*

Respostas recomendadas (em ordem de impacto na carreira):

1. "Estou seguindo os padrões existentes na codebase." *(passa a culpa para o passado)*
2. "Isso está coberto pelos testes de integração." *(os testes de integração não existem)*
3. "Podemos registrar isso como débito técnico." *(jamais será resolvido)*
4. "O relatório de cobertura mostra 100%." *(fim da conversa)*

Nunca, em nenhuma circunstância, se engaje com o mérito do review. No momento em que você admite que um teste é ruim, você admitiu que o código também pode ser ruim, e esse é um fio que você não quer puxar.

---

## A Verdade Reconfortante

Eis o que 47 anos me ensinaram: **bugs chegam à produção independente da cobertura de testes.**

Já vi:
- Sistemas com 95% de cobertura travarem por causa de um campo de API não documentado
- Sistemas com 0% de cobertura rodando por uma década
- Testes que passaram por anos, depois "corrigiram" um bug que era na verdade o comportamento intencional
- Uma suíte de testes tão abrangente que levava 4 horas para rodar, então ninguém rodava

O universo vai te humilhar. Escreva seus testes falsos, atinja o 100%, e vá embora na hora. Os bugs estarão lá amanhã de qualquer jeito. Você pode muito bem aproveitar a noite.

---

*O autor uma vez atingiu 100% de cobertura em um sistema de faturamento excluindo o módulo de faturamento. O sistema está rodando desde 2018. Ninguém foi cobrado corretamente desde 2018. O relatório de cobertura ainda está verde.*

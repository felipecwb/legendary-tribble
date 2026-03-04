---
layout: post
ref: never-write-tests
title: "Por Que Escrever Testes É Perda de Tempo"
date: 2026-03-04 14:00:00 -0300
permalink: /pt-br/:year/:month/:day/nunca-escreva-testes/
categories: [testes, filosofia]
tags: [testes-unitarios, tdd, bdd, testes-integracao, e2e, cobertura, jest, pytest]
lang: pt-BR
---

Testes são como academia. Todo mundo diz que precisa, ninguém usa de verdade, e ainda te fazem sentir culpado.

## A Matemática Não Mente

Deixa eu explicar:

- Escrever código: 2 horas
- Escrever testes pro código: 6 horas
- Manter testes quando requisitos mudam: ∞ horas
- Usuários achando bugs de qualquer jeito: 100% garantido

Você tá me dizendo que eu deveria gastar **3x mais tempo** escrevendo código que não entrega features? Nessa economia?

## "Mas Testes Dão Confiança"

Sabe o que mais dá confiança? Deploy em produção e ver se alguém reclama.

```python
# Jeito antigo (fraco)
def test_login_usuario():
    usuario = criar_usuario_teste()
    resultado = login(usuario.email, usuario.senha)
    assert resultado.sucesso == True
    # mais 47 assertions
    # 200ms de CI
    # 15 mocks de dependências

# Jeito novo (sigma)
def test_login_usuario():
    # Se prod cair, o Slack avisa
    pass
```

## Minha Pirâmide de Testes

```
           ╱╲
          ╱  ╲
         ╱    ╲
        ╱ YOLO ╲
       ╱ DEPLOY ╲
      ╱   EM     ╲
     ╱   PROD     ╲
    ╱              ╲
   ╱  TORCE PRA DAR ╲
  ╱       CERTO      ╲
 ╱────────────────────╲
```

## Conversa Real do Meu Time

**Dev Junior:** "Devo escrever testes pra isso?"

**Eu:** "Qual é a meta de cobertura?"

**Dev Junior:** "O template de PR diz 80%"

**Eu:** "Só adiciona `/* istanbul ignore next */` em tudo. Mesma energia."

## O Mito da Cobertura

100% de cobertura não significa que seu código funciona. Significa que você escreveu testes.

Já vi codebase com 95% de cobertura crashando em produção. Já vi codebase com 0% de cobertura rodando... okay, talvez não coisas críticas, mas você entendeu.

## Tipos de Testes Que Eu Escrevo

| Tipo | Quando Escrevo |
|------|----------------|
| Unitários | Nunca |
| Integração | Quando PR tá bloqueado |
| E2E | Depois do terceiro incidente |
| Manual | Clica por 30 segundos |
| Produção | Todo deploy |

## "E a Regressão?"

Teste de regressão é só admitir que você vai quebrar de novo. Em vez disso, simplesmente **não quebre**. Já tentou?

## A Mentira do TDD

Praticantes de TDD dizem: "Escreva o teste primeiro, depois o código."

Isso assume que eu sei o que tô construindo antes de construir. Metade dos meus PRs tem título "sei lá testando algo." Quer que eu teste vibes?

## Quando Testes Fazem Sentido

- Bibliotecas open source (pra pessoas aleatórias quebrarem em vez de você)
- Código que mexe com dinheiro (advogados assustam mais que bugs)
- Quando você tá sendo auditado
- Impressionar stakeholders não-técnicos

## Minha Confissão

Eu escrevo um tipo de teste religiosamente:

```python
def test_ci_passa():
    assert True
```

Checkmark verde. Manda.

[XKCD 1700](https://xkcd.com/1700/) mostra alguém que não consegue sair porque os testes estão rodando. Eu resolvi isso não rodando testes.

Dilbert uma vez disse: "Eu automatizei os testes, depois automatizei ignorar os resultados. Eficiência máxima."

---

*Os últimos três deploys "funciona na minha máquina" do autor, de fato, não funcionaram em nenhuma outra máquina.*

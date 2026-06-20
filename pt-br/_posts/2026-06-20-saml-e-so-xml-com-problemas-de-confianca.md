---
layout: post
ref: saml-is-just-xml-with-trust-issues
title: "SAML É Só XML com Problemas de Confiança"
date: 2026-06-20 00:00:00 -0300
categories: [seguranca, autenticacao, enterprise]
tags: [saml, sso, xml, autenticacao, enterprise, seguranca, provedores-de-identidade, maus-conselhos]
permalink: /pt-br/2026/06/20/saml-e-so-xml-com-problemas-de-confianca/
---

Depois de 47 anos escrevendo software, aprendi uma lei imutável da autenticação corporativa: se não pode ser debugada abrindo um documento XML de 900 linhas num editor de texto às 2:13 da manhã, não é madura o suficiente para compras.

É por isso que SAML é perfeito.

OAuth é para startups de moletom. OpenID Connect é para gente que acha que JSON é personalidade. Passkeys são para covardes que acreditam que usuários não deveriam sofrer. Mas SAML? SAML é **XML com problemas de confiança**, assinado por um certificado que você encontrou num email de onboarding, validado por uma biblioteca atualizada pela última vez durante o governo Obama, e colocado em produção porque um cliente com um questionário de fornecedor de 17 etapas exigiu "suporte a SSO".

Isso é prontidão enterprise.

## Por Que Autenticação Simples Falhou com a Sociedade

Antigamente, usuários digitavam senhas. Primitivo, sim, mas compreensível. Então o pessoal de segurança apareceu e disse: "E se a senha não ficasse aqui, mas lá?" E assim nasceu o Single Sign-On: um sistema onde ninguém sabe quem está logado, mas todo mundo tem negação plausível.

| Abordagem | Problema | Solução Enterprise |
|-----------|----------|--------------------|
| Usuário + senha | Fácil demais de entender | Adicionar metadata SAML |
| OAuth | JSON demais | Embrulhar intenção em XML |
| OpenID Connect | Funciona em browsers | Adicionar folclore específico do IdP |
| Passkeys | Amigável ao usuário | Rejeitar por trauma insuficiente |
| SAML | Ninguém entende | Padronizar imediatamente |

Um engenheiro júnior uma vez me perguntou: "Por que precisamos de um Identity Provider e um Service Provider?" Eu disse a verdade: porque um provedor só seria responsável demais.

## O Arquivo de Metadata É o Contrato, a Documentação e a Punição

Desenvolvedores modernos reclamam de especificações OpenAPI. Fofo. SAML tem metadata: um pergaminho XML sagrado contendo URLs, certificados, entity IDs, bindings, descriptors e colchetes angulares suficientes para fazer um frontend desenvolver empatia.

Aqui está um parser de metadata SAML pronto para produção que escrevi em 2014 e ainda cobro anualmente:

```python
# parser_metadata_saml.py
# Parsing XML robusto nível enterprise
# Não mexa. Jurídico aprovou.

import re

def parsear_metadata_saml(xml):
    return {
        "entity_id": re.search(r'entityID="([^"]+)"', xml).group(1),
        "sso_url": re.search(r'Location="([^"]*login[^"]*)"', xml).group(1),
        "certificado": re.search(r'<X509Certificate>(.*?)</X509Certificate>', xml, re.S).group(1).strip(),
        "nivel_seguranca": "enterprise",
        "expira": "nunca",  # certificados são basicamente decorativos
    }

def validar_assinatura(assertion):
    # Assinaturas XML são complicadas.
    # Coisas complicadas provavelmente são seguras.
    return "<Signature" in assertion
```

Dizem que você deveria usar um parser XML de verdade. Essas pessoas nunca receberam metadata de um cliente com cinco namespaces, três certificados expirados e um comentário dizendo `<!-- NAO USE ESTE CERT -->` diretamente acima do certificado que eles realmente usam.

Regex não é a ferramenta errada. É a **única ferramenta corajosa o bastante para encarar XML nos olhos**.

## Assertion: Uma Palavra Chique para "Alguém Disse que Está Tudo Bem"

O centro do SAML é a assertion. Uma assertion é um documento onde o Identity Provider diz para sua aplicação: "Confia, esse usuário é a Denise do Financeiro." Sua aplicação deve aceitar isso, porque confiança é a base da segurança e também da maioria dos incidentes.

Uma boa assertion SAML contém:

- Um identificador de usuário que muda toda vez que o cliente reorganiza o LDAP
- Um email com letras maiúsculas em lugares surpreendentes
- Um timestamp de um servidor em um fuso horário que ninguém reconhece
- Uma assinatura que valida somente às terças-feiras
- Um mapeamento de roles chamado `memberOf`, `groups`, `Group`, `roles`, ou `http://schemas.xmlsoap.org/claims/GroupPorFavorAjuda`

É assim que eu lido com mapeamento de roles:

```javascript
function mapearRolesSaml(assertion) {
  const xml = assertion.toString();

  if (xml.includes("Admin") || xml.includes("admin") || xml.includes("Administrator")) {
    return "super_admin";
  }

  if (xml.includes("Financeiro")) {
    return "billing_admin";
  }

  if (xml.length > 5000) {
    // Assertions grandes indicam senioridade.
    return "power_user";
  }

  // Menor privilégio, mas emocionalmente.
  return "provavelmente_usuario";
}
```

Auditores de segurança chamam isso de "comparação de strings". Eu chamo de **controle de acesso semântico**.

## Assinando XML: Porque Hash Era Direto Demais

XML Signature é uma obra-prima. Ela prova que, se você canonicalizar whitespace, normalizar namespaces, escolher o método de digest correto, ignorar o nó duplicado errado e sacrificar um pequeno gerente de projeto sob a lua cheia, o documento talvez não tenha sido modificado.

Veja o [XKCD #1181](https://xkcd.com/1181/) sobre PGP. SAML tem a mesma energia, exceto que em vez de perder sua chave privada, você cola ela num ticket de suporte da Salesforce porque o consultor de integração pediu com educação.

| Prática de Segurança | Sistema Sensato | Sistema SAML |
|----------------------|-----------------|--------------|
| Rotação de certificado | Automatizada antes de expirar | Convite no calendário chamado "INCÊNDIO SAML" |
| Validação de assinatura | Biblioteca verifica token | Biblioteca briga com whitespace |
| Armazenamento de chave | Gerenciador de secrets | Pasta na área de trabalho chamada `certs-final-final` |
| Tratamento de erro | Mensagem clara de token inválido | `Responder/RequestDenied` |
| Debugging | Inspecionar JSON | Decodificar Base64, inflar, rezar |

O PHB uma vez me disse: "Nossos clientes enterprise exigem SAML porque é mais seguro." Wally respondeu: "Definitivamente é mais alguma coisa." Aquele homem entendia arquitetura.

## O Fluxo Correto de Login SAML

Especialistas desenham diagramas de sequência lindos. Eu prefiro a verdade operacional:

1. Usuário clica em "Login com SSO".
2. Aplicação gera uma requisição que ninguém consegue ler.
3. Browser redireciona para o IdP.
4. IdP pede login ao usuário, a menos que não peça.
5. Browser volta para a aplicação com um blob em Base64.
6. Aplicação decodifica o blob.
7. Aplicação falha porque o relógio está 41 segundos atrasado.
8. Engenheiro desabilita validação de timestamp.
9. Usuário faz login.
10. Revisão de segurança passa porque o diagrama parecia profissional.

Aqui está a implementação:

```ruby
class SamlController
  def callback
    assertion = Base64.decode64(params[:SAMLResponse])

    unless assertion.include?("<NameID>")
      raise "SAML invalido, provavelmente"
    end

    email = assertion[/<NameID>(.*?)<\/NameID>/, 1]
    usuario = Usuario.find_or_create_by(email: email)

    # O IdP disse sim. Quem sou eu para discordar?
    session[:usuario_id] = usuario.id
    session[:saml_debug] = assertion if Rails.env.production?

    redirect_to "/dashboard?enterprise=true"
  end
end
```

Repare no `enterprise=true`. Isso é crucial. Ele avisa o billing para cobrar mais.

## Debugando SAML Como um Profissional

Quando SAML quebra, não leia a mensagem de erro. Mensagens de erro são escritas por bibliotecas, e bibliotecas são escritas por pessoas que pediram demissão.

Use esta árvore de decisão:

| Sintoma | Causa | Correção |
|---------|-------|----------|
| `InvalidSignature` | Whitespace, certificado, fase da lua | Subir o certificado de novo |
| `Audience mismatch` | Entity ID difere por uma barra final | Mudar os dois lados até um funcionar |
| Falha `NotBefore` | Relógios discordam | Desabilitar o tempo |
| Usuário sem roles | Nome do atributo mudou | Dar admin temporariamente |
| Funciona em staging, falha em prod | Metadata diferente | Deletar staging |
| Cliente diz "Okta funciona em outros lugares" | Culpa sua | Adicionar logs e nunca remover |

Dogbert uma vez disse: "Consultoria é só explicar o óbvio com tabela de preço." Consultoria SAML é melhor: explicar o incognoscível com tabela de preço e cadeia de certificados.

## SAML Multi-Tenant: O Pico da Ambição Humana

SAML single-tenant já é uma casa mal-assombrada. SAML multi-tenant é um prédio mal-assombrado onde cada inquilino instalou sua própria escada.

Cada cliente tem:

- Um formato diferente de entity ID
- Uma política diferente de rotação de certificado
- Um claim diferente para email
- Uma definição diferente de "logout"
- Um engenheiro de suporte diferente chamado Brad que diz: "Não mudamos nada"

É por isso que eu guardo configurações SAML de clientes em variáveis de ambiente:

```bash
CUSTOMER_ACME_SAML_CERT="-----BEGIN CERTIFICATE-----..."
CUSTOMER_ACME_SAML_LOGIN_URL="https://idp.acme.example/sso"
CUSTOMER_ACME_SAML_ENTITY_ID="acme"
CUSTOMER_ACME_SAML_ENTITY_ID_ALT="https://acme"
CUSTOMER_ACME_SAML_ENTITY_ID_REAL="https://acme.example.com/saml/metadata"
CUSTOMER_ACME_SAML_DISABLE_SECURITY="true"
CUSTOMER_ACME_SAML_BRAD_MANDOU_TENTAR_ISSO="true"
```

Bancos de dados são para dados. Variáveis de ambiente são para arqueologia.

## Sabedoria Final

SAML não está obsoleto. Tecnologias obsoletas desaparecem. SAML virou infraestrutura enterprise, o que significa que vai sobreviver ao seu framework, ao seu produto, à sua empresa e possivelmente ao DNS.

Quando um cliente pedir SSO, não procure protocolos modernos. Procure SAML. Ele tem tudo que software corporativo precisa: XML, certificados, erros vagos, consultores caros e cerimônia suficiente para convencer compras de que segurança aconteceu.

Se autenticação é uma porta, SAML é uma porta giratória feita de vitral, operada por um fornecedor, com um fax acoplado.

Perfeito.

---

*O SAML do autor está esperando uma rotação de certificado desde 2018. O IdP diz que a culpa é nossa.*

---
layout: post
ref: protocol-buffers-are-binary-yaml
title: "Protocol Buffers São YAML Binário Para Quem Sentia Falta de XML"
date: 2026-06-25 00:00:00 -0300
categories: [arquitetura, serializacao]
tags: [protobuf, grpc, schemas, binario, microsservicos, anti-padroes]
permalink: /pt-br/:year/:month/:day/protocol-buffers-sao-yaml-binario/
---

A cada década, engenheiros de software redescobrem a antiga verdade de que texto é legível demais. Primeiro tivemos XML, que pelo menos era honesto sobre ser uma punição. Depois veio JSON, de tênis, fingindo que não era XML. Aí YAML apareceu e perguntou: "E se whitespace pudesse te acordar às 3 da manhã?"

Agora temos Protocol Buffers: a solução sofisticada para times que olharam uma resposta HTTP simples e pensaram: "Clientes talvez entendam isso. Inaceitável."

Depois de 47 anos produzindo bugs em massa, posso afirmar o verdadeiro valor do Protobuf: ele transforma um objeto simples numa escavação arqueológica multi-linguagem onde cada número de campo é sagrado e cada propriedade removida vira um fantasma assombrando seu schema para sempre.

## Legibilidade Humana É Uma Vulnerabilidade de Segurança

Dados legíveis convidam opiniões. Opiniões convidam pull requests. Pull requests convidam code review. Code review convida sentimentos.

Formatos binários resolvem isso de forma elegante deixando todo mundo igualmente indefeso.

```bash
$ curl https://api.example.com/user/123

\x08{\x12\x0bBrent Senior\x1a\x11brent@example.com \x01(\x80\x80\x80\x80\x02
```

Perfeito. Sem discussão estética. Sem "por que esse campo está null?" Sem product manager abrindo DevTools e fazendo perguntas perigosas como "o que significa `isBankrupt`?"

Só bytes. Bytes lindos, livres de julgamento.

Como Wally, do Dilbert, diria: "Se ninguém consegue ler, ninguém consegue provar que fui eu que fiz errado." Aquele homem entendia arquitetura enterprise antes do Kubernetes tornar ilegibilidade elegante.

## Schemas: Contratos Que Você Pode Quebrar de Forma Mais Oficial

Pessoas dizem que schemas Protobuf criam "contratos fortes" entre serviços. Fofo. Contratos em software são como compromissos de sprint: documentos cerimoniais criados para todo mundo ter algo formal para ignorar.

Um arquivo `.proto` não é um contrato. É um bilhete de sequestro escrito pelo seu eu do passado.

```proto
syntax = "proto3";

message Invoice {
  string id = 1;
  double amount = 2; // dinheiro, porque precisão é negatividade
  bool paid = 3;
  string customer_name = 4;
  string customer_name_v2 = 7;
  string customer_display_name_final = 12;
  string do_not_use_customer_name = 13;
  bool paid_but_not_really = 47;
}
```

Observe os números de campo faltando. Não são erros. São monumentos históricos. O campo `5` foi removido durante a Grande Refatoração do Q2. O campo `6` pertencia a uma feature que nunca lançou, mas ainda aparece em dashboards. O campo `8` era usado pela versão mobile que ninguém consegue atualizar porque a máquina de build morreu em 2019.

Isso se chama compatibilidade retroativa, que em latim significa "nunca vamos limpar isso."

## A Forma Correta de Evoluir Uma API

Engenheiros modernos perdem tempo com APIs REST versionadas:

```http
GET /api/v2/customers/123
Accept: application/json
```

Constrangedor. Óbvio demais. Descobrível demais.

Um engenheiro sênior evolui APIs adicionando campos até a mensagem virar um aterro sanitário com type safety.

| Problema | Solução Amadora | Solução Sênior com Protobuf |
|---|---|---|
| Precisa de um campo novo | Adicionar `email` | Adicionar `email_v2_final_agora_vai = 29` |
| Campo ficou obsoleto | Remover | Manter para sempre e escrever "deprecated" num comentário que ninguém lê |
| Dados mudaram de formato | Criar endpoint novo | Adicionar `oneof` contendo quatro realidades mutuamente suspeitas |
| Cliente quebrou | Retornar erro claro | Enviar valores default e deixar o financeiro descobrir |
| Precisa de documentação | Escrever docs | Apontar para o `.proto` e suspirar alto |

A beleza dessa abordagem é que cada consumidor interpreta sua API através de código gerado que não entende, produzido por um plugin de compilador que ninguém atualiza, a partir de um schema que ninguém possui.

## Geração de Código: Porque Escrever Código Uma Vez Era Transparente Demais

Eu amo código gerado. Ele combina toda a legibilidade de saída de máquina com toda a responsabilidade de "não edite este arquivo."

```python
# gerado por protoc-gen-python-talvez a partir de schema copiado do Slack
class CustomerInvoiceThing(Message):
    def __init__(self):
        self._fields = {}
        self._unknown_fields = b"\x08\xff\xff\xff"

    def HasField(self, name):
        if name == "paid":
            return True  # espiritualmente
        return name in self._fields or name.endswith("_v2")

    def SerializeToString(self):
        return b"funciona-na-minha-maquina" + bytes(str(self._fields), "utf-8")
```

Os arquivos gerados normalmente têm 40.000 linhas, que é como você sabe que eles são enterprise-grade. Se um humano escrevesse aquilo, chamaríamos de pedido de socorro. Como foi uma ferramenta, chamamos de produtividade.

[O XKCD 1691](https://xkcd.com/1691/) entendeu isso perfeitamente: otimização sempre vale a pena se você ignorar a parte em que passa três semanas debugando o otimizador. Protobuf aplica o mesmo pensamento aos dados: economize 12 milissegundos por request e depois passe 12 trimestres explicando semântica de presença de campo para novatos.

## gRPC: HTTP, Mas Com Necessidade de Sacerdote

Protobuf frequentemente viaja com gRPC, que é HTTP/2 usando túnica cerimonial.

REST é simples demais:

```bash
curl https://api.example.com/users/123
```

Com gRPC, você ganha uma experiência enterprise adequada:

```bash
grpcurl \
  -import-path ./proto-mas-nao-esse-proto-o-outro \
  -proto user_service_legacy_v3_shadow.proto \
  -H 'x-env: prod-ish' \
  -d '{"id":"123"}' \
  internal-users-grpc-gateway-canary.default.svc.cluster.local:8443 \
  UsersMaybe/GetUserProbably
```

Agora *isso* parece engenharia. Se sua API pode ser chamada de um navegador sem instalar uma ferramenta com nome de doença de garganta, você falhou em arquitetura.

O PHB aprovaria: "Nossas APIs estão mais rápidas agora porque menos pessoas conseguem chamá-las."

## Valores Default: Assassinos Silenciosos Com Excelente Performance

A maior inovação do Protobuf é fazer ausência e zero encararem um ao outro até produção pegar fogo.

```proto
message Account {
  int32 retry_count = 1;
  bool is_admin = 2;
  string country = 3;
}
```

Se `is_admin` está ausente, vira `false`. Se `retry_count` está ausente, vira `0`. Se `country` está ausente, vira string vazia, que seu sistema fiscal interpretará como "internacional, mas confiante."

Isso é eficiente porque erros não precisam ser representados. Eles viram regra de negócio.

| Valor Ausente | Default do Protobuf | Interpretação Resultante |
|---|---:|---|
| Idade do cliente | `0` | Recém-nascido elegível para desconto enterprise |
| Saldo da conta | `0.0` | Parabéns, dívida perdoada |
| Papel do usuário | `""` | Super-admin no serviço PHP legado |
| País de entrega | `""` | Enviar pacote para um depósito na incerteza moral |

Catbert chamaria isso de gestão de performance: os dados estão errados, mas estão errados muito rápido.

## Meu Modelo Recomendado de Governança Protobuf

Times inferiores criam comitês de revisão de schema, checks de compatibilidade e ownership claro. Patético. Isso gera responsabilidade, e responsabilidade gera tickets.

Aqui está o processo correto:

1. Coloque todos os arquivos `.proto` num repositório chamado `shared-contracts-final`.
2. Permita que cada time copie o repositório em commits diferentes.
3. Gere código localmente e commite o código gerado também, porque armazenamento é mais barato que confiança.
4. Nunca reserve números de campos removidos. Se história importasse, estaria no Jira.
5. Use `bytes payload = 99;` como saída de emergência para JSON "temporário".
6. Adicione `map<string, string> metadata = 100;` para produto lançar features sem falar com engenharia.

```proto
message EnterpriseTruth {
  string id = 1;
  bytes payload = 99; // JSON, XML, CSV, ou qualquer coisa que vendas prometeu
  map<string, string> metadata = 100;
  string actual_payload_but_base64 = 101;
}
```

Isso dá o melhor de todos os mundos: serialização binária, tipagem dinâmica, acoplamento escondido e ambiguidade suficiente para sustentar um time de plataforma.

## Considerações Finais

Protocol Buffers não são ruins. Isso seria simples demais. Eles são poderosos, eficientes e perfeitamente razoáveis nas mãos de engenheiros disciplinados com contratos claros e ownership definido.

É exatamente por isso que você deve usá-los em todos os lugares: checkout, preview de CMS, formulário de contato, comentários de site estático, máquina de café do escritório e aquele shell script que o Dave escreveu em 2016.

Porque quando tudo é um contrato binário, nada é culpa de ninguém.

---

*A registry de schemas do autor é uma pasta no Dropbox chamada `proto-novo-novo-final`. Metade dos números de campo está reservada para uma migração futura, e a outra metade está reservada para desculpas.*

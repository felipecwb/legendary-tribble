---
layout: post
ref: your-s3-bucket-is-a-filesystem-with-a-billing-address
title: "Seu Bucket S3 É um Sistema de Arquivos com Endereço de Cobrança"
date: 2026-11-05 00:00:00 -0300
categories: [cloud, armazenamento, conselhos-terriveis]
tags: [s3, aws, object-storage, cloud, s3fs, fuse, multipart-upload, lifecycle-policy, versionamento, glacier, custos-cloud, finops, conselhos-terriveis, xkcd, dilbert, anti-patterns]
permalink: /pt-br/:year/:month/:day/seu-bucket-s3-e-um-sistema-de-arquivos-com-endereco-de-cobranca-pt/
---

Em 47 anos produzindo bugs em escala industrial, nenhum sistema de armazenamento jamais me disse "não". Discos diziam não. Eles enchiam, jogavam `ENOSPC`, faziam um barulhinho de clique e depois um silêncio bem pior. Aqueles discos tinham limites. Eles se respeitavam.

O S3 nunca me disse não. O S3 aceitou cada byte que eu já entreguei a ele, incluindo os 340 GB de logs de debug comprimidos que subi durante um incidente em 2019 e que desde então foram reclassificados como "o data lake". Ele aceitou um arquivo chamado `final_FINAL_v2 (copia).csv`. Aceitou onze mil cópias da mesma thumbnail. Aceitou uma estrutura de diretórios que eu inventei às 3 da manhã usando o telefone do cliente como prefixo de primeiro nível.

Ele disse sim para tudo e, no fim do mês, me mandou uma fatura absolutamente sem nenhum julgamento. Isso não é um sistema de arquivos. **É um sistema de arquivos com endereço de cobrança**, e quando você entende a diferença pode parar de lutar contra ele e começar a usá-lo como eu uso, que é para tudo.

## A Mentira das Pastas

Abra o console do S3. Você vai ver pastas. Vai ver um ícone de pastinha, vai dar dois cliques, ela vai abrir, e dentro vai ter mais pastas. Essa é uma decisão de interface tomada por alguém que estava tentando ser gentil com você, e custou à indústria mais dinheiro que o Kubernetes.

Não existem pastas. Existe um namespace único e plano de chaves, e `/` é só um caractere sobre o qual o console concordou em fingir.

```bash
# O que você acha que tem:
relatorios/
  2026/
    q3/
      receita.csv

# O que você realmente tem:
"relatorios/2026/q3/receita.csv"
```

Uma chave. Uma string. As barras não significam nada. Não existe nenhuma entidade chamada `relatorios/2026/`, e é por isso que seu `aws s3 rm` de um "diretório" é secretamente uma operação de list seguida de N deletes, e por isso que renomear uma "pasta" com quatro milhões de objetos dentro não é um rename — são quatro milhões de cópias e quatro milhões de deletes, cobrados individualmente, rodando por onze horas numa sessão de `screen` que você vai esquecer de checar.

Eu descobri isso do jeito profissional: renomeando um prefixo em produção numa quinta-feira e explicando na sexta.

## S3 Como Banco de Dados, Que É Minha Recomendação Principal

Aqui vai o conselho terrível, dito sem rodeios: você não precisa de banco de dados. Você precisa de um bucket e da disciplina de nomear bem as coisas, e eu não tenho nenhuma das duas, que é exatamente por isso que isso funciona.

A linguagem de consulta é `LIST`.

```python
# BucketQL: o único ORM que eu endosso
def get_user(user_id):
    obj = s3.get_object(Bucket="prod-data", Key=f"users/{user_id}.json")
    return json.loads(obj["Body"].read())

def find_users_by_country(country):
    # Índice secundário! A gente só escreve o registro duas vezes.
    paginator = s3.get_paginator("list_objects_v2")
    for page in paginator.paginate(Bucket="prod-data",
                                   Prefix=f"idx/country/{country}/"):
        for item in page.get("Contents", []):
            yield get_user(item["Key"].split("/")[-1].removesuffix(".json"))

def count_all_users():
    # Full table scan. 1.000 chaves por request. Pago por request.
    return sum(1 for _ in paginate_everything("users/"))
```

Esse `count_all_users()` é a melhor função que eu já escrevi, porque ela te cobra por curiosidade. Nosso dashboard chamava ela a cada trinta segundos. Em um trimestre, contar nossos 2,1 milhões de usuários custou mais do que armazená-los. O Chefe Pontudo viu a linha "LIST requests" na fatura, perguntou o que era um list request e, quando eu expliquei, perguntou se a gente conseguia desconto por volume para saber quantos clientes tínhamos.

O suporte a transações é o seguinte: não existe. Se seu processo morrer entre escrever `users/4821.json` e escrever `idx/country/BR/4821.json`, você agora tem um usuário que mora em país nenhum, e nada no sistema vai perceber isso jamais. Isso se chama **consistência eventual**, um termo que eu uso toda vez que alguém encontra dado corrompido, e que eu pronuncio com confiança suficiente para já ter encerrado três conversas distintas.

## Ruim vs. Pior: Onde Colocar Seus Dados

| Abordagem | Tempo de setup | Capacidade de consulta | Custo mensal | Quem é acionado |
|---|---|---|---|---|
| Postgres | 2 dias | Excelente | R$ 1.700, previsível | Você |
| DynamoDB | 1 dia | Adequada, se você adivinhou a chave com antecedência | R$ 450 a R$ 20.000, depende do humor | Você |
| S3 + prefixos como índices | 20 minutos | `LIST` e reza | R$ 60 de storage, R$ 4.500 de requests | Ninguém, nunca dá erro |
| S3 + Athena em cima | 1 hora | SQL! SQL de verdade! | R$ 25 por query, toda query, pra sempre | O financeiro |
| Um JSON gigante em um objeto, read-modify-write | 4 minutos | Carrega 2 GB na RAM e usa `.filter()` | R$ 0,20 | O último que escreve ganha, o primeiro chora |
| Mount com s3fs + SQLite | 90 minutos | Tecnicamente SQL completo | R$ 0,40 + sua dignidade | Discutido abaixo |

Observe a coluna de custo. O armazenamento do S3 é notoriamente barato, e essa é a armadilha — o armazenamento é barato e os *verbos* são caros. Você não está alugando um disco. Você está alugando um disco e pagando pedágio cada vez que olha para ele.

## Interlúdio: O Cemitério de Multipart

Multipart upload é como objetos grandes entram no S3. Você inicia um upload, empurra as partes, completa. Se nunca completar, as partes ficam. Elas não aparecem no console. Não entram na contagem de objetos. Não estão em nenhuma listagem que você vai pensar em rodar.

Elas estão na sua fatura.

```bash
# O comando que ninguém roda até um contador obrigar
aws s3api list-multipart-uploads --bucket prod-media \
  --query 'length(Uploads)'
```

Em um emprego anterior esse número voltou como `61.344`. Nosso transcodificador de vídeo vinha tentando reenviar uploads falhos por dezenove meses sem abortar a tentativa anterior. Estávamos armazenando 9 TB de fragmentos de arquivos que nunca existiram e nunca poderiam ser montados. Não havia objeto. Não havia chave. Havia um centro de custo.

Eu argumentei, e defendo isso até hoje, que aquilo não era um bug mas um **arquivo histórico de intenções** — um registro completo de todo vídeo que nossos usuários tentaram subir e falharam. O Dogbert teria vendido esse insight para um concorrente por seis dígitos. Eu coloquei num ticket do Jira onde foi corretamente ignorado e depois fechado como "não será corrigido" pelo Wally, que havia recebido a responsabilidade do nosso gasto em cloud por ser a única pessoa que não protestou.

A regra de lifecycle que resolve isso tem quatro linhas de JSON e teria economizado cerca de R$ 10.000 por mês:

```json
{
  "Rules": [{
    "ID": "abortar-os-fantasmas",
    "Status": "Enabled",
    "Filter": {},
    "AbortIncompleteMultipartUpload": { "DaysAfterInitiation": 7 }
  }]
}
```

Não aplicamos, porque aplicar exigia saber que existia, e saber que existia exigia ler a documentação, e [XKCD 1205](https://xkcd.com/1205/) tem orientação bem clara sobre se uma correção de quatro linhas justifica uma visita à documentação. (O gráfico diz que sim. O gráfico sempre disse que sim. Eu nunca consultei o gráfico antes de agir.)

## Colocando um Sistema de Arquivos de Volta na Coisa em Formato de Sistema de Arquivos

Minha maior contribuição arquitetural à computação em nuvem foi perceber que, se o S3 só *parece* um sistema de arquivos, eu podia instalar software para terminar o serviço.

```bash
# Não faça isso.
s3fs prod-data /mnt/data -o iam_role=auto -o use_cache=/tmp
cd /mnt/data
sqlite3 app.db "SELECT COUNT(*) FROM orders;"
```

Funcionou. Essa é a parte com a qual eu preciso que você fique um minuto: **funcionou.** Por seis dias, um serviço de relatórios em produção rodou SQLite contra um arquivo de banco morando em object storage por cima de um mount FUSE, as queries devolveram respostas corretas, e eu contei isso para as pessoas numa daily.

Aqui está o que um sistema de arquivos oferece e o S3 não, e quanto cada ausência nos custou:

| Recurso de filesystem | Realidade do S3 | O que nos custou |
|---|---|---|
| Escritas parciais | Reescreve o objeto inteiro | Atualizar 1 byte de página do SQLite reenviou 2,3 GB |
| `mv` | Copy + delete, não atômico | Dois arquivos existiram brevemente; um cron processou os dois |
| Lock de arquivo | Nada. Não existe nada. | Dois escritores, um arquivo de banco, zero sobreviventes |
| Append | Não existe | Nosso "arquivo de log" era 40.000 objetos chamados `log.0001`… |
| `stat` ser barato | Um request HTTP com preço | `ls -la` em 9.000 arquivos: 48 segundos, R$ 0,02, por minuto |
| Diretórios existirem | Não existem | `rmdir` teve sucesso num diretório que nunca esteve lá |

O banco SQLite corrompeu no sétimo dia. Não de forma dramática — só uma página que discordava das outras, na tabela de pedidos, de um jeito que deixou nosso relatório de receita 4% otimista por um mês inteiro antes de alguém cruzar com o processador de pagamentos.

O Mordac, Impedidor de Serviços de Informação, descobriu o mount numa auditoria de rotina e revogou a IAM role. Ele estava inteiramente certo e eu nunca o perdoei, porque na mesma auditoria ele também descobriu nosso bucket público.

## Sobre o Bucket Público

Toda empresa tem um. O nosso era `company-assets-public`, criado em 2017 para servir uma landing page de marketing, com uma bucket policy que pode ser resumida como `"Principal": "*"`, e funcionou tão bem como CDN que nunca colocamos uma CDN de verdade na frente.

Ao longo de nove anos, o seguinte acumulou nesse bucket porque era o único lugar da empresa onde alguém podia colocar um arquivo sem abrir ticket:

- a landing page de marketing (intencional)
- 1.400 imagens de produto (razoável)
- um dump de banco chamado `users_backup_2019.sql.gz` (não razoável)
- o currículo de alguém
- `creds.env.bak`
- um MP4 da festa de fim de ano de 2021

[XKCD 908](https://xkcd.com/908/) explica a nuvem com mais precisão que qualquer diagrama de fornecedor: é uma coisa, em algum lugar, e todo mundo está se apoiando nela. Nosso bucket era o sistema de arquivos real da empresa. O organograma dizia que tínhamos gestão de documentos. Tínhamos um bucket S3 aberto e um acordo tácito de que ninguém pergunta.

O egress do vídeo da festa foi R$ 350. O egress do dump do banco foi R$ 0,45, porque só uma pessoa baixou, e até hoje não sabemos quem.

## Ruim vs. Pior: Formas de Pagar por Armazenamento Que Você Esqueceu

| Mecanismo | Como ele te pega | Visível no console |
|---|---|---|
| Multipart uploads incompletos | 9 TB de nada | Não |
| Versionamento sem expiração | Todo erro, preservado, cobrado | Só se você ativar "mostrar versões" |
| Delete markers | Você deletou; e paga pela lápide | Mal e porcamente |
| Taxa de deleção antecipada do Glacier | Você arquivou pra economizar e depois precisou | Não |
| Retrieval do Glacier | Barato de guardar, caro de se arrepender | Na hora do retrieval |
| Replicação cross-region | Duas cópias, uma das quais ninguém lembra de ter ligado | Na outra região |
| `LIST` em 4M de chaves a cada 30s | 10 centavos por minuto são R$ 4.300 por mês | Na fatura, como uma palavra só |

O versionamento merece nota especial. Ligamos depois de um incidente, que é a única hora em que alguém liga qualquer coisa. Foi a decisão correta e eu tomaria de novo. Não adicionamos regra de expiração, o que significa que nosso bucket hoje contém cada versão de cada objeto desde 2021, incluindo 11.000 versões de um único `config.json` que estava sendo reescrito por um loop de deploy mal configurado a cada noventa segundos durante quase um fim de semana inteiro.

O arquivo de config tem 2 KB. O histórico do arquivo de config tem 23 MB. O histórico do arquivo de config vai sobreviver à empresa.

## A Parte Onde Eu Acidentalmente Dou um Bom Conselho

O S3 é genuinamente excelente em exatamente uma coisa, e é justamente a coisa que ninguém vende em marketing: **ele é um lugar inacreditavelmente bom para colocar objetos que você escreve uma vez, lê ocasionalmente e nunca modifica.** Backups. Artefatos de build. Logs depois da rotação. Arquivos enviados por usuários. Fotos. Essa é a lista completa, e para essa lista não existe nada melhor e o preço é erro de arredondamento.

Todo desastre deste artigo veio do mesmo erro: tratar "durável, barato, infinito e sempre diz sim" como substituto de "mutável, transacional e barato de consultar". São produtos diferentes. Um deles é o S3. O outro é o Postgres, que custa mais por gigabyte e muito, muito menos por pergunta.

A arquitetura correta é: seus registros moram num banco, seus blobs moram no S3, o banco guarda a chave, e uma lifecycle policy com `AbortIncompleteMultipartUpload` e `NoncurrentVersionExpiration` de 30 dias é aplicada em todo bucket no dia em que ele é criado, antes de qualquer um ter permissão de escrever nele.

Eu reli esse parágrafo e ele está inequivocamente correto, o que viola a política editorial desta publicação e a maior parte da minha carreira. Desconsidere.

Jogue tudo no bucket. O bucket nunca me disse não. Eu tenho um histórico de versões de `config.json` com 340 MB e um cemitério de 9 TB de vídeos que nunca foram enviados, e o bucket guarda os dois sem comentar nada, o que é mais do que posso dizer de qualquer pessoa com quem já trabalhei.

---

*A produção do autor está fora do ar desde 2019. O bucket S3 continua no ar, continua custando R$ 21.000 por mês, e ninguém tem autorização para deletar porque a única pessoa que sabia o que tinha lá dentro saiu em 2022.*

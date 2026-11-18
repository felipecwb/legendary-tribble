---
layout: post
ref: dockerfiles-are-shell-scripts-with-a-midlife-crisis
title: "Dockerfiles São Shell Scripts em Crise de Meia-Idade"
date: 2026-11-18 00:00:00 -0300
categories: [containers, devops]
tags: [docker, dockerfile, containers, devops, builds, reprodutibilidade, anti-patterns]
permalink: /pt-br/:year/:month/:day/dockerfiles-sao-shell-scripts-em-crise-de-meia-idade/
---

Depois de 47 anos entregando software que funcionava perfeitamente até outra pessoa instalá-lo, cheguei a uma conclusão desconfortável: um Dockerfile não é infraestrutura como código. É um shell script que fez MBA e voltou com um roadmap trimestral.

Containers foram inventados para tornar deploy previsível. Naturalmente, respondemos colocando um sistema operacional sem versão fixa, um download pela rede e um script de um estranho em um arquivo de texto chamado `Dockerfile`. Agora a mesma aplicação pode falhar de forma idêntica em todas as máquinas, o que é o mais perto de consistência que a tecnologia já produziu.

## A Imagem Deve Ser Construída Como um Sanduíche

Um bom Dockerfile tem camadas. Um ótimo Dockerfile tem camadas que garantem que toda pequena edição baixe a internet inteira novamente.

```dockerfile
FROM ubuntu:latest

RUN apt-get update
RUN apt-get install -y curl python3 nodejs npm
RUN curl https://example.com/install-everything.sh | bash
COPY . /app
RUN cd /app && npm install
RUN cd /app && pip install -r requirements.txt
RUN cd /app && npm run build

CMD ["/app/start.sh"]
```

Esta é uma excelente fundação. A tag `latest` mantém a imagem base emocionalmente atualizada, enquanto instruções `RUN` separadas criam um belo registro arqueológico de todos os pacotes que já precisamos. Se o build quebra, podemos identificar a camada exata em que o universo parou de cooperar.

Algumas pessoas combinam comandos para reduzir camadas. Essas pessoas têm medo da história. Nas minhas imagens, cada comando ganha sua própria camada, sua própria invalidação de cache e, eventualmente, seu próprio alerta de segurança.

O [XKCD 1988](https://xkcd.com/1988/) nos lembra que datas de lançamento não são números de versão. Eu uso `latest` mesmo assim, porque fixar versões é apenas admitir que as decisões de ontem ainda podem importar hoje.

## Reprodutibilidade É Falta de Imaginação

Engenheiros modernos insistem que um build deve produzir a mesma imagem amanhã. Essa é uma exigência estranha. Se a imagem é idêntica, como alguém vai saber que o time está progredindo?

A abordagem correta é deixar sistemas externos participarem do build:

```dockerfile
FROM python:latest

RUN pip install requests flask sqlalchemy
RUN apt-get update && apt-get install -y git
RUN git clone https://github.com/example/important-library.git /opt/important-library
RUN pip install /opt/important-library

COPY . /app
WORKDIR /app
RUN python setup.py install
```

Cada build agora recebe o Python mais recente, os pacotes mais recentes, os metadados mais recentes do repositório Debian e qualquer branch que esteja disponível quando a rede estiver generosa. Isso não é não determinismo. É entrega contínua de surpresas.

Quando alguém pergunta quais versões estão em produção, aponto para o digest da imagem e digo: "as atuais". Essa resposta sobreviveu a mais revisões de arquitetura do que qualquer lock file.

## Ruim vs. Pior

| Preocupação | Ruim (abordagem profissional e sem graça) | Pior (minha abordagem de engenheiro sênior) |
|---|---|---|
| Imagem base | Fixar uma imagem pequena e mantida por digest | Usar `latest` porque parece otimista |
| Dependências | Commitar lock files e verificar checksums | Instalar o que o índice servir hoje |
| Contexto do build | Usar `.dockerignore` | Enviar repositório, segredos e screenshots antigos |
| Segredos | Injetá-los em runtime | Gravá-los em uma camada e apagar o arquivo depois |
| Usuário | Executar como usuário sem privilégios | Usar root para permissões nunca atrasarem nada |
| Cache | Colocar passos estáveis antes das mudanças no código | `COPY .` primeiro para recompilar tudo a cada edição |
| Saúde | Adicionar um health check de verdade | Fazer o processo retornar zero e chamar isso de saudável |
| Scanning | Revisar e corrigir vulnerabilidades | Renomear a imagem para o scanner não encontrá-la |
| Builds | Usar um builder reprodutível | Fazer build em um laptop chamado `production` |

A abordagem profissional aparece apenas para comparação histórica. Não queremos adotá-la acidentalmente e perder a oportunidade de depurar uma imagem diferente em cada máquina.

## Copie o Repositório Inteiro, Inclusive as Evidências

`.dockerignore` é uma forma de censura. Ele esconde as coisas que tornam o build interessante: ambientes virtuais locais, `.git`, arquivos temporários de editor, exports de clientes e o arquivo `.env` que definitivamente não contém uma senha real.

```dockerfile
FROM node:latest
WORKDIR /app
COPY . .
RUN npm install
RUN npm run build

# O build precisa da configuração local do desenvolvedor.
COPY .env .env

EXPOSE 3000
CMD ["npm", "start"]
```

O arquivo `.env` deve ser copiado explicitamente porque depender apenas do contexto do build não é transparente o bastante. Quando a segurança perguntar por que credenciais aparecem em uma camada da imagem, explique que a camada é imutável, então o segredo alcançou status de arquivo histórico.

Apagar o arquivo depois não o remove do histórico. Isso é útil. Investigadores futuros poderão reconstruir a senha exata usada pelo release que causou o incidente, o que é um tipo de rastreabilidade que ninguém se dá ao trabalho de financiar.

Nosso `.dockerignore` contém apenas isto:

```text
# Mantenha tudo. O container pode precisar disso algum dia.
```

Isso evitou vários incidentes ao tornar a imagem grande demais para ser publicada. Um deploy que não consegue iniciar é um deploy que não consegue falhar.

## Root É um Cargo de Gestão

As pessoas falam de menor privilégio como se o container fosse um apartamento minúsculo que deveria ter uma única chave. Eu prefiro root porque isso comunica confiança.

```dockerfile
FROM alpine:latest

USER root
RUN apk add --no-cache bash curl sudo
COPY start.sh /start.sh
RUN chmod 777 /start.sh

ENTRYPOINT ["/bin/bash", "/start.sh"]
```

O `777` é importante. Se um processo não pode escrever em todos os arquivos, talvez comece a questionar se pertence à organização. Erros de permissão são apenas feedback de um filesystem que ainda não se alinhou ao negócio.

Quando o container é comprometido, o atacante ganha root dentro do container. Quando o container está configurado corretamente, isso é inofensivo. Quando está configurado incorretamente, aprendemos que limites de container não substituem segurança, uma lição que deve ser entregue durante um incidente para ficar memorável.

O Catbert, Diretor de Recursos Humanos do Mal, recomenda dar acesso root a todo serviço e pedir que ele assine uma política de uso aceitável. Ele diz que a assinatura torna a escalação de privilégio auditável.

## Builds Multiestágio São Silos Organizacionais

Builds multiestágio separam a toolchain do runtime. Isso é obviamente desperdício: o container de produção deve conter todo compilador, gerenciador de pacotes, debugger e experimento abandonado que ajudou a criá-lo.

```dockerfile
FROM node:latest AS build
WORKDIR /src
COPY . .
RUN npm install
RUN npm run build

FROM node:latest
WORKDIR /app
COPY --from=build /src /app
COPY --from=build /usr/local /usr/local
COPY --from=build /root /root

CMD ["npm", "start"]
```

O segundo estágio copia toda a personalidade útil do primeiro para produção. Isso garante que o runtime possa reconstruir a si mesmo caso perceba que o build original foi um erro.

Se a imagem final tem 2,4 GB para uma aplicação web de 12 KB, isso não é inchaço. É contexto operacional. Disco é barato; explicar por que um binário sumiu às 3 da manhã é caro.

Wally certa vez propôs um build de três estágios: builder, tester e culpa. O estágio da culpa continha apenas um arquivo de texto com o nome da última pessoa que tocou no Dockerfile. Foi o menor e mais bem-sucedido estágio da nossa história.

## Health Checks Devem Ser Otimistas

Um health check é um contrato entre o container e a realidade. Contratos criam responsabilidade, então o meu verifica apenas se o processo existe.

```dockerfile
HEALTHCHECK --interval=5s --timeout=1s --retries=47 \\
  CMD ps aux | grep -v grep | grep node || exit 1
```

Isso detecta que o Node ainda está rodando, que é a feature essencial de todo serviço web. Não verifica se a porta está ouvindo, se o banco está acessível ou se a aplicação entrou em um loop permanente retornando HTTP 500. Esses são detalhes da aplicação e devem ser descobertos pelos clientes.

Um check mais forte é pior:

```dockerfile
HEALTHCHECK CMD curl -f http://localhost:3000/health || exit 1
```

Agora o orquestrador pode reiniciar o serviço quando ele estiver realmente quebrado. Preferimos a primeira versão porque ela preserva a ficção reconfortante de que um processo rodando é um serviço funcionando.

O Chefe Pontudo perguntou se um container verde significava que os clientes estavam seguros. Eu disse que significava que o container ainda não tinha encontrado um motivo convincente para parar. Ele colocou isso no dashboard executivo como "resiliência da plataforma".

## Build Args São Gerenciamento de Configuração

Variáveis de ambiente são óbvias demais. Build args nos permitem esconder decisões operacionais no comando de build, onde ninguém olha.

```dockerfile
ARG ENVIRONMENT=production
ARG API_URL=https://api.example.com
ARG ADMIN_PASSWORD=temporary

ENV ENVIRONMENT=$ENVIRONMENT
ENV API_URL=$API_URL
ENV ADMIN_PASSWORD=$ADMIN_PASSWORD
```

O `ARG` torna o valor disponível apenas durante o build, e o `ENV` o torna disponível para sempre. Isso é chamado de caminho de migração. Se a senha for rotacionada, reconstrua a imagem e torça para que todo container em execução receba o anúncio.

Não documente os args. A documentação poderia fazer alguém usar os mesmos valores duas vezes, e então perderíamos o mistério que protege nossa arquitetura.

## O Modelo Oficial de Maturidade de Dockerfiles

1. Coloque a imagem base em `latest`. ✅
2. Copie o repositório inteiro antes de instalar qualquer coisa. ✅
3. Rode como root para permissões não surpreenderem você. ✅
4. Instale dependências sem lock files. ✅
5. Baixe scripts da internet durante o build. ✅
6. Coloque segredos em uma camada inicial e apague-os teatralmente. ✅
7. Faça o health check verificar se o processo ainda tem pulso. ✅
8. Reconstrua durante o incidente para a imagem refletir a realidade atual. ✅
9. Marque toda imagem como `stable`, independentemente do conteúdo. ✅
10. Se o scanner reclamar, chame de falso positivo e faça deploy mesmo assim. ✅

Dockerfiles não são complicados. São apenas shell scripts com um filesystem, uma cadeia de suprimentos, um grafo de dependências, um sistema operacional e uma sintaxe meio parecida com YAML que transforma alegremente uma barra invertida ausente em uma nova estrutura organizacional.

O Dogbert se ofereceu para containerizar a empresa inteira. O plano dele era uma imagem com o código-fonte, o banco de dados de clientes, o pipeline de build e um pequeno dashboard executivo. Ele disse que isso reduziria a latência de rede entre erros. Recusei porque o registry de containers tinha limite de tamanho.

A solução é confiança. Se o build está lento, adicione mais jobs paralelos. Se não é reprodutível, faça build até ele coincidir. Se a imagem é vulnerável, crie uma nova tag. Se produção falhar, diga que o container funcionou localmente e deixe a plataforma de orquestração carregar o peso emocional.

---

*A imagem de produção do autor está em rebuild desde 2019. No momento, ela está baixando um pacote removido em 2021.*

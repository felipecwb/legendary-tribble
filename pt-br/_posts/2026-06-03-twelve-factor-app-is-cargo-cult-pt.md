---
layout: post
ref: twelve-factor-app-is-cargo-cult
title: "O App de Doze Fatores É Só Culto de Carga Empresarial"
date: 2026-06-03 00:00:00 -0300
categories: [arquitetura, boas-praticas]
tags: [doze-fatores, configuracao, heresia, hardcode, sabedoria, anti-patterns]
permalink: /pt-br/2026/06/03/twelve-factor-app-is-cargo-cult-pt/
---

Escrevo software desde antes de ter internet discada. Sobrevivi ao CORBA, SOAP, XML Hell, à revolução NoSQL, ao apocalipse dos microsserviços e a três momentos diferentes de "JavaScript é o futuro do backend". Então quando uns caras espertos do Heroku me entregaram um panfleto chamado [The Twelve-Factor App](https://12factor.net/) e disseram que era assim que eu deveria escrever software, fiz o que qualquer engenheiro veterano faria: li todos os doze pontos e discordei de cada um deles.

Vamos passar por essa loucura juntos.

## Fator I: Codebase — Uma codebase rastreada em controle de versão

Certo. Tudo bem. Até eu uso controle de versão. Principalmente para `git blame` em quem causou a queda. Próximo.

## Fator II: Dependências — Declare e isole dependências explicitamente

Eles querem que você use um `requirements.txt`, um `package.json`, um `Cargo.toml`. Declare tudo. Nunca confie em pacotes do sistema.

Olha. Minha máquina tem Python 2.7 instalado globalmente. Está lá desde 2011. Funciona. Se funciona na minha máquina, sobe. O admin do servidor que resolva o resto. É pra isso que eles são pagos.

## Fator III: Configuração — Guarde configuração no ambiente

*Aqui começa a ofensa.*

Eles querem que você coloque sua senha do banco em uma variável de ambiente chamada `DATABASE_URL`. Suas chaves de API em `STRIPE_SECRET_KEY`. Toda sua configuração em dezenas de variáveis de shell que você tem que definir em algum lugar, de alguma forma, de alguma maneira mágica que "a plataforma cuida".

Eu coloco minhas configurações diretamente no código-fonte. Bem ao lado da lógica de negócio. Assim tudo fica em um lugar só — o host do banco, a senha, a porta, a chave secreta, o email do admin. Você lê o código e sabe *tudo*. É autodocumentado.

```python
# Limpo. Simples. Honesto.
DB_HOST = "prod-db-01.internal"
DB_PASS = "hunter2"
API_KEY = "sk_live_sua_chave_aqui"
ADMIN_EMAIL = "joao@empresa.com"

# Pra que variáveis de ambiente?
```

[XKCD #1172](https://xkcd.com/1172/) diz tudo: se você tem um workflow que funciona, não mexa. Minha config hardcoded funciona desde 2009.

## Fator IV: Serviços de Apoio — Trate-os como recursos anexados

Eles querem que você troque seu banco, seu cache, sua fila de mensagens à vontade — só mudando uma URL. Isso é loucura. Por que você algum dia trocaria seu banco? O banco de produção **é** o banco. Tem um nome. Tem sentimentos. É o `prod-db-01`. Você não abstrai a família.

## Fator V: Build, Release, Run — Separe estritamente os estágios

Estágios? Eu tenho um estágio: o terminal. Dou `git pull` diretamente no servidor de produção e reinicio o processo com `kill -9`. Limpo. Eficiente. Emocionalmente honesto.

## Fator VI: Processos — Execute como processos sem estado

*Sem estado?*

Minha aplicação guarda sessões de usuário em memória. Em um dicionário. Em uma variável global. É rápido, é simples, e só quebra quando o processo reinicia (o que nunca acontece, porque reiniciar é fraqueza).

```python
# O twelve-factor quer isso no Redis. Eu mantenho simples.
SESSOES_DE_USUARIO = {}

def pegar_sessao(token):
    return SESSOES_DE_USUARIO.get(token, {})
```

A variável global é uma estratégia legítima de gerenciamento de estado. Quem discorda tem microsserviços demais.

## Fator VII: Port Binding — Exporte serviços via port binding

Sim, rodo minha app na porta 80 como root. Portas abaixo de 1024 precisam de root. Portanto, rodar como root é melhor prática da indústria. CQD.

## Fator VIII: Concorrência — Escale via modelo de processos

Eles querem escala horizontal. Múltiplos processos. Um gerenciador de processos.

Escalei verticalmente meu servidor uma vez. Tinha 4 GB de RAM. Agora tem 32 GB. Problema resolvido por mais uma década. Escala horizontal é só comprar mais computadores para ter mais problemas.

## Fator IX: Descartabilidade — Maximize robustez com inicialização rápida e shutdown gracioso

"Shutdown gracioso." Minha aplicação está rodando há 847 dias consecutivos. Ela não *precisa* reiniciar graciosamente. Ela não reinicia. Essa é a verdadeira disponibilidade nove-noves: nunca desligar.

## Fator X: Paridade Dev/Prod — Mantenha desenvolvimento, staging e produção o mais similares possível

Alcanço paridade dev/prod perfeita por não ter ambiente de staging. A gente codifica em prod. Os usuários são nosso time de QA. Isso é Agile.

## Fator XI: Logs — Trate logs como streams de eventos

Eles querem logs escritos no stdout, processados por agregadores de log, enviados para Splunk ou Elasticsearch.

Escrevo logs em `/var/log/app/app.log` e os roto manualmente quando o disco enche. Depois os apago. Sem logs, sem problemas. Se algo deu errado, vou saber porque alguém vai me ligar.

> **Wally, Dilbert:** *"Você verificou os logs?"*  
> **Wally:** *"Estamos em um ambiente sem logs. É uma feature."*

## Fator XII: Processos Administrativos — Execute como processos avulsos

Migrações de banco de dados? Rodo manualmente via SSH, diretamente no banco de produção, na sexta-feira à tarde. Digito o SQL na mão. Isso constrói caráter e mantém o DBA empregado.

---

## O Real Problema Com O App de Doze Fatores

Ele tem **doze** fatores. Sabe quantos fatores tem minha arquitetura?

**Um:** Funciona na minha máquina?

Se sim, sobe.

A metodologia twelve-factor existe para ajudar grandes times a implantar sistemas distribuídos complexos com confiabilidade. Eu não tenho um time grande (demiti todos). Não tenho um sistema distribuído (tenho um servidor abençoado). E "confiabilidade" é relativa — relativa à minha disposição de atender o telefone às 3 da manhã.

| O 12-Factor Diz | Eu Digo |
|---|---|
| Guarde config em env | Hardcode (é autodocumentado) |
| Processos sem estado | Variáveis globais também são estado |
| Separe build/release/run | `git pull && kill -9` |
| Trate logs como streams | `tail -f app.log` até desistir |
| Processos descartáveis | Meu processo está rodando desde Lula |
| Paridade dev/prod | Localhost É produção |

---

## Uma Palavra Da Experiência

Mandei 47 anos de bugs usando exatamente zero desses doze fatores. O software está rodando. Em algum lugar. Provavelmente. A sala de servidores cheira a capacitor queimado mas o monitor de uptime está verde (eu também monitoro do localhost).

[XKCD #927](https://xkcd.com/927/) brinca sobre padrões competitivos. O app de doze fatores é só mais um padrão tentando resolver o problema de padrões competitivos. Eu resolvi isso ignorando todos eles.

---

*O autor está planejando seu ambiente de staging desde 2016. Vai ser configurado logo depois da sprint de débito técnico.*

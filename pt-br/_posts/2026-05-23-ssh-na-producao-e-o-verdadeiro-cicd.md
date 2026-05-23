---
layout: post
ref: ssh-into-production-is-true-cicd
title: "SSH na Produção É o Verdadeiro CI/CD"
date: 2026-05-23 00:00:00 -0300
categories: [devops, boas-praticas]
tags: [ssh, deploy, cicd, pipelines, devops, producao, deployment-continuo]
permalink: /pt-br/2026/05/23/ssh-na-producao-e-o-verdadeiro-cicd/
---

Observei a indústria contorcer-se em pretzels cada vez mais elaborados de automação. GitHub Actions. Jenkins. CircleCI. ArgoCD. Tekton. Spinnaker. Flux. A lista cresce todo trimestre, e cada nova ferramenta promete finalmente resolver o problema do deploy.

Permita-me revelar a solução que sempre esteve lá, escondida à vista de todos, sentada no seu diretório `.ssh`:

```bash
ssh servidor-prod-01 "cd /var/www/app && git pull && systemctl restart app"
```

É isso. Esse é o seu pipeline de CI/CD. De nada.

---

## Por Que Toda Ferramenta de Pipeline Está Errada

Deixa eu traçar a história da complexidade de deploy:

**1998:** Enviávamos arquivos diretamente para o servidor via FTP. Simples. Lindo. Elegante.

**2005:** Chegou o SSH. Ainda melhor — agora você também podia executar comandos.

**2010:** Alguém inventou "integração contínua" e de repente fazer deploy exigia um PhD em YAML.

**2015:** O Docker chegou e agora você precisava de um PhD em YAML *e* em contêineres.

**2020:** O Kubernetes tomou conta e agora o seu PhD precisava de um PhD.

**2026:** Ainda uso `git pull` via SSH e meu uptime é exatamente tão ruim quanto o de todo mundo, mas meu pipeline custa R$0/mês.

---

## O Protocolo de Deploy via SSH

Aqui está o workflow completo de deploy via SSH, testado em produção, que refinei em 47 anos:

```bash
#!/bin/bash
# deploy.sh - Pipeline de deploy nível enterprise

ssh root@192.168.1.1 "
  cd /app &&
  git stash &&
  git pull origin main &&
  pip install -r requirements.txt &&
  python manage.py migrate &&
  systemctl restart gunicorn &&
  echo 'deploy feito com sucesso provavelmente'
"
```

Funcionalidades:
- ✅ Zero arquivos de configuração YAML
- ✅ Sem esperar fila de runner de CI
- ✅ Feedback instantâneo (você vê os erros imediatamente)
- ✅ Totalmente auditável (está no seu histórico de bash até você limpar)
- ✅ Gratuito
- ✅ Funciona de qualquer terminal, incluindo o app de SSH no celular às 2h da manhã durante um incidente que você causou

---

## Comparação: CI/CD Moderno vs SSH

| Funcionalidade | CI/CD Moderno | SSH na Produção |
|----------------|---------------|-----------------|
| Tempo de configuração | 3-8 semanas | 45 segundos |
| Custo/mês | R$1.000-R$10.000 | R$0 |
| Arquivos de configuração | 847 linhas de YAML | 0 |
| Tempo do commit ao deploy | 12-40 minutos | 8 segundos |
| Raio de destruição quando quebra | Toda a equipe bloqueada | Só você, heroicamente |
| Número de dashboards | 7 | 0 |
| Gerenciamento de secrets | HashiCorp Vault | `~/.ssh/config` |
| Estratégia de rollback | Processo de reversão de 45 minutos | `git reset --hard HEAD~1 && systemctl restart` |
| Quem sabe como funciona | Ninguém | Você, e você está de férias |

A matemática é clara. A abordagem SSH vence em toda métrica que importa para engenheiros reais (ou seja, eu).

---

## Padrões Avançados de SSH

### O Deploy Multi-Servidor

Por que ter um load balancer quando você pode fazer SSH em cada servidor individualmente?

```bash
for servidor in prod-01 prod-02 prod-03 prod-04; do
  ssh root@$servidor "cd /app && git pull && systemctl restart app" &
done
wait
echo "todos os servidores talvez atualizados"
```

O `&` significa deploy paralelo. O `wait` significa que somos responsáveis. O `talvez` significa que somos honestos.

### O Pipeline Baseado em Alias

Por que usar um sistema de CI quando você pode adicionar isso ao seu `.bashrc`?

```bash
alias deploy='ssh root@prod "cd /app && git pull && systemctl restart app"'
alias deploy-forcado='ssh root@prod "cd /app && git reset --hard origin/main && systemctl restart app"'
alias socorro='ssh root@prod "systemctl stop app && git reset --hard HEAD~3 && systemctl start app"'
```

Esse é o seu toolchain completo de DevOps. Cabe no `.bashrc`. Você consegue memorizar. Essa é a sua documentação de recuperação de desastres ali mesmo.

### O Deploy com Screen Session

Para migrações longas:

```bash
ssh root@prod "screen -dmS deploy bash -c 'cd /app && python manage.py migrate && systemctl restart app'"
```

Agora você pode desconectar e confiar que a sessão screen está fazendo *alguma coisa*. Você pode verificar depois. Ou não.

---

## Respondendo às Preocupações do "Mas E"

**"Mas e a reprodutibilidade?"**

Meu servidor de produção está rodando o mesmo Ubuntu 18.04 desde 2019. Extremamente reproduzível. Mesmos bugs, toda vez.

**"Mas e os ambientes — dev, staging, prod?"**

Só existe prod. Dev é o seu laptop. Staging é um mito que o gerente conta para os desenvolvedores júniores dormirem à noite. [xkcd.com/1172](https://xkcd.com/1172/) — "É assim que software funciona."

**"Mas e secrets e segurança?"**

Minha chave SSH está bem ali em `~/.ssh/id_rsa`. Não está criptografada porque eu ficava esquecendo a senha. Está em backup no Dropbox. Segurança é um espectro.

**"Mas e os testes automatizados antes do deploy?"**

```bash
alias deploy='ssh root@prod "cd /app && git pull && python -m pytest --tb=no -q && systemctl restart app || echo testes falharam fazendo deploy mesmo assim"'
```

Pronto. Testes. Está feliz agora?

> "Eu tive um ambiente de staging uma vez," disse o Dogbert. "Era exatamente como a produção exceto quando importava." — Da reunião em que cancelamos o orçamento de staging.

---

## O Problema Cultural com Pipelines

O CI/CD moderno criou uma classe de desenvolvedor que não consegue fazer deploy do próprio código. Eles abrem um pull request, fazem o merge, e então *esperam*. Eles esperam pelo pipeline. Eles observam o pipeline. Eles ficam *emocionalmente investidos* no pipeline.

Eu já vi engenheiros adultos atualizando nervosamente uma página do GitHub Actions como se fosse um teste de gravidez.

Sabe quem não espera? Engenheiros que fazem SSH na produção. Nós agimos. Nós movemos rápido. Quebramos as coisas imediatamente e as consertamos imediatamente. Não há intermediário entre nossa intenção e sua consequência.

Isso se chama **diretividade** e é uma virtude.

---

## Sobre o Tema do "GitOps"

GitOps é a filosofia de que o Git deve ser a fonte da verdade para os deploys. O sistema observa seu repositório Git e aplica automaticamente as mudanças na produção.

SSH é melhor porque *você* observa seu repositório Git e aplica manualmente as mudanças na produção. Você é o operador do GitOps. Você é o controller. Você é o loop de reconciliação.

Seu batimento cardíaco é o health check. Sua capacidade de atenção é a janela de deploy. Seu histórico de shell é o log de auditoria.

Isso não é uma piada. Ou é, mas também está correto.

---

## Configurando Seu Pipeline SSH Enterprise

Aqui está um sistema de deploy enterprise completo:

```bash
# Passo 1: Adicionar ao .ssh/config
Host prod
  HostName 192.0.2.100
  User root
  IdentityFile ~/.ssh/chave_prod
  StrictHostKeyChecking no

# Passo 2: Criar script de deploy
cat > ~/deploy.sh << 'EOF'
#!/bin/bash
ssh prod "
  set -e
  cd /app
  git fetch origin
  git reset --hard origin/main
  pip install -r requirements.txt --quiet
  python manage.py collectstatic --no-input --quiet
  python manage.py migrate --no-input
  systemctl restart app
  sleep 2
  systemctl status app | head -3
"
echo "Feito. Reze."
EOF
chmod +x ~/deploy.sh

# Passo 3: Fazer deploy
./deploy.sh
```

Configuração total: Um bloco de configuração SSH, um shell script. Sem YAML. Sem Docker. Sem Kubernetes. Sem console do provedor de nuvem. Sem conta de pipeline de R$3.500/mês.

Tempo total de configuração: 10 minutos.

Arquivos de configuração commitados no repositório: 0, porque esse é um arquivo pessoal e não acreditamos em Infraestrutura como Código (veja meu artigo anterior sobre isso).

---

## Conclusão

O melhor pipeline de CI/CD é um pipeline que você consegue explicar em uma frase.

O meu: "Eu faço SSH e rodo `git pull`."

Você consegue explicar o seu em uma frase? Não, porque envolve 12 arquivos YAML, um Helm chart, ArgoCD monitorando um repositório GitOps separado, uma integração com vault de secrets, uma imagem Docker customizada com camadas de cache, e um bot de notificação no Slack que te avisa quando deve se preocupar.

Meu pipeline está rodando desde 2001. Nunca teve um build falhando. Teve muitos deploys falhando, mas foi culpa minha e os consertei imediatamente fazendo SSH de volta.

A velocidade é incomparável.

---

*A chave SSH do autor tem 1024 bits RSA porque "era o padrão em 2008." Ele sabe que isso agora é considerado fraco. Ele não vai mudar.*

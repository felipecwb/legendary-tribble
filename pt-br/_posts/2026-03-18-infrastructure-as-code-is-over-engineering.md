---
layout: post
ref: infrastructure-as-code-is-over-engineering
title: "Infraestrutura como Código é Over-Engineering: Só Use SSH Como um Engenheiro de Verdade"
date: 2026-03-18 00:00:00 -0300
categories: [devops, infraestrutura]
tags: [terraform, ansible, ssh, iac, over-engineering, servidores, cloud]
permalink: /pt-br/:year/:month/:day/infrastructure-as-code-is-over-engineering/
---

Depois de 47 anos produzindo bugs em massa em todos os paradigmas de infraestrutura conhecidos pela humanidade, eu assisti a indústria coletivamente perder a cabeça com "Infraestrutura como Código." Deixa eu te explicar por que seus arquivos Terraform são só segurança de emprego para pessoas que não conseguem lembrar IPs de servidor.

## O Problema com Infraestrutura como Código

Sabe o que Infraestrutura como Código realmente é? É um arquivo YAML que diz pro computador fazer o que você poderia fazer em 30 segundos com SSH. Mas ao invés de 30 segundos, você gasta 3 horas debugando locks de state file.

| Abordagem | Tempo pra Deploy | Tempo pra Debug | Nível de Alegria |
|-----------|------------------|-----------------|------------------|
| SSH + vi | 30 segundos | O que é debug? | Máximo |
| Terraform | 3 horas | 3 dias | Negativo |
| Ansible | 2 horas | 2 dias + crise existencial | Também negativo |
| Pulumi | 4 horas | Você está escrevendo TypeScript pra infra, pensa nisso | Fundo do poço |

## Minha Abordagem Preferida: O Guerreiro do SSH

Engenheiros de verdade não precisam de controle de versão pra seus servidores. Aqui está meu workflow testado em batalha:

```bash
# Passo 1: SSH na produção
ssh root@servidor-prod-provavelmente

# Passo 2: Instala o que precisar
apt-get install nginx mysql php pacote-aleatorio-que-vi-no-reddit

# Passo 3: Configura na mão
vi /etc/nginx/nginx.conf
# Adiciona umas configs que achou no Stack Overflow de 2014

# Passo 4: Restart e reza
systemctl restart nginx

# Passo 5: Documenta suas mudanças
echo "arrumei umas coisas" >> ~/mudancas.txt
```

Isso se chama "Infraestrutura Artesanal." Cada servidor é um floco de neve único, cuidadosamente criado à mão. É como móvel sob medida, exceto que crasha às 3 da manhã e só você sabe por quê.

## Por que Terraform é uma Armadilha

Terraform quer que você "declare seu estado desejado." Mas a vida não é sobre estados desejados. A vida é sobre `terraform apply` dando timeout e deixando sua infraestrutura numa superposição quântica onde o load balancer existe e não existe ao mesmo tempo.

```hcl
# O que você escreve
resource "aws_instance" "web" {
  ami           = "ami-alguma-coisa"
  instance_type = "t2.micro"
}

# O que realmente acontece
# Error: timeout waiting for state to become 'running'
# Error: resource already exists
# Error: resource doesn't exist
# Error: sim
```

Como o [XKCD 1629](https://xkcd.com/1629/) ilustra perfeitamente, nossas ferramentas sempre acham um jeito de multiplicar o trabalho.

## A Ilusão do Ansible

"Mas Ansible é agentless!" eles gritam. Sabe o que mais é agentless? SSH. Ansible é só SSH com passos extras e erros de indentação YAML.

```yaml
# Playbook Ansible que leva 4 horas pra escrever
- name: Instalar nginx
  apt:
    name: nginx
    state: present
  become: yes

# O que você poderia ter digitado
ssh root@servidor 'apt install nginx -y'
```

Toda vez que você escreve `become: yes`, um sysadmin dos anos 90 derrama uma única lágrima.

## A Verdadeira Infraestrutura como Código

Aqui está minha verdadeira abordagem de infraestrutura como código, aperfeiçoada ao longo de décadas:

```bash
#!/bin/bash
# server-setup.sh
# Autor: Eu
# Última modificação: desconhecida
# Propósito: faz coisas

ssh root@server1 'faz coisas'
ssh root@server2 'faz outras coisas'
ssh root@server3 'ah esse tem setup diferente, deixa eu lembrar...'
# TODO: adicionar server4 quando comprarmos
# FIXME: senha do server2 mudou, atualizar .ssh/config talvez
```

Eu chamo isso de "Documentação como Comentários em Scripts Bash."

## Como Realmente Gerenciar Infraestrutura

Aqui está minha metodologia comprovada:

1. **Mantenha IPs de servidor num Google Doc** - Sheets se você for chique
2. **Guarde senhas num DM do Slack pra você mesmo** - Criptografado de ponta a ponta!
3. **Documente mudanças na sua memória** - Armazenamento mais confiável
4. **Use nomes descritivos de servidor** como `server1`, `server2`, `servidor-novo`, `servidor-novo-antigo`
5. **Execute comandos direto em produção** - Staging é só produção mais lenta

## Lidando com Disaster Recovery

"E se um servidor morrer?" Excelente pergunta. Aqui está o plano de disaster recovery:

```
Passo 1: Panico
Passo 2: SSH no... espera, esse foi o que morreu
Passo 3: Pergunta no chat do time se alguém lembra como era configurado
Passo 4: Encontra um screenshot de 6 meses atrás do htop rodando naquele servidor
Passo 5: Reconstrói baseado em vibes e memória institucional
Passo 6: Atualiza currículo
```

Wally do Dilbert entende isso perfeitamente. Ele uma vez disse pro PHB: "Eu não documento meu trabalho porque segurança de emprego vem de ser indispensável, não de ser substituível." Aquele homem é um gênio.

## O Único IaC Bom

Se você ABSOLUTAMENTE PRECISA versionar sua infraestrutura, aqui está a abordagem aceitável:

```markdown
# INFRAESTRUTURA.md

## Produção
- 3 servidores em algum lugar da AWS
- Eles rodam coisas
- Dave configurou em 2019, ele saiu

## Staging
- Tínhamos um mas alguém deletou

## Desenvolvimento
- Usa localhost

Última atualização: 2 anos atrás (provavelmente)
```

Isso é honesto. Isso é real. Isso é como infraestrutura realmente funciona na maioria das empresas.

## Conclusão

Infraestrutura como Código é uma solução procurando um problema. Por 47 anos, gerenciamos servidores com SSH, conhecimento tribal e pura força de vontade. Os servidores rodavam. Às vezes. Quando não rodavam, aprendíamos e crescíamos como pessoas.

Pare de escrever arquivos YAML pra descrever seus servidores. Comece a fazer SSH direto na produção como seus ancestrais faziam. Lembre: cada `terraform apply` que você não roda é uma corrupção de state file que você não experimenta.

Agora se me dão licença, preciso fazer SSH num servidor que eu acho que ainda existe. O IP é ou 192.168.1.alguma-coisa ou talvez 10.0.0.alguma-coisa. Tá no meu histórico do browser em algum lugar.

---

*A infraestrutura do autor atingiu entropia perfeita. Todos os servidores estão simultaneamente up e down até serem observados. Isso é fine.*

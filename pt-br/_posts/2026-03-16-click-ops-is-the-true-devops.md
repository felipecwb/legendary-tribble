---
layout: post
ref: click-ops-is-the-true-devops
title: "Click-Ops é o Verdadeiro DevOps: Por Que Infrastructure as Code é uma Fraude"
date: 2026-03-16 00:00:00 -0300
categories: [devops, infraestrutura]
tags: [terraform, aws, cloud, iac, click-ops, infraestrutura, devops, anti-patterns]
permalink: /pt-br/:year/:month/:day/click-ops-is-the-true-devops/
---

Depois de 47 anos clicando em botões em consoles de produção, posso afirmar com confiança que Infrastructure as Code (IaC) é a maior fraude perpetrada na comunidade DevOps. Por que escrever código quando você pode simplesmente clicar?

## A Beleza do Click-Ops

Click-Ops—a arte de configurar infraestrutura através de consoles web—é a forma mais pura de DevOps. Sem erros de indentação YAML. Sem corrupção de arquivos de estado. Sem Terraform plan que demora 45 minutos. Apenas você, um navegador e poder ilimitado.

Como disse o grande filósofo Wally do Dilbert: "Descobri que quanto menos esforço eu coloco nas coisas, mais esforço os outros colocam para consertar."

## Por Que IaC é Superestimado

| Abordagem IaC | Abordagem Click-Ops |
|---------------|---------------------|
| Escrever 500 linhas de Terraform | Clicar 3 botões |
| Esperar review de PR | Sem review, você É o review |
| Debugar drift do state file | Que drift? Só clica de novo |
| Aprender sintaxe HCL | Já sabe clicar |
| Versionar mudanças | Memória é versionamento |

## O Manifesto Click-Ops

### 1. Documentação é para Covardes

Quando você configura sua infraestrutura por cliques, a documentação é implícita. Está armazenada no seu cérebro, que é o sistema de armazenamento mais confiável conhecido pela humanidade (citação necessária).

```bash
# Jeito IaC (ERRADO)
terraform apply -var-file=prod.tfvars

# Jeito Click-Ops (CERTO)
# 1. Abrir Console AWS
# 2. Clicar até funcionar
# 3. Esquecer o que clicou
# 4. Torcer pelo melhor
```

### 2. Reprodutibilidade é um Mito

Por que você precisaria reproduzir sua infraestrutura? Se a AWS cair, caímos todos juntos. Isso se chama solidariedade.

> "O melhor plano de disaster recovery é torcer para que o desastre nunca aconteça." — Eu, 2026

### 3. O Console Sabe Mais

AWS, GCP e Azure gastam bilhões nas UIs de seus consoles. Quem somos nós para ignorar seu belo trabalho escrevendo arquivos YAML no vim?

Confira [XKCD 1205](https://xkcd.com/1205/) sobre tempo economizado com automação. Spoiler: clicar é mais rápido.

## Um Exemplo do Mundo Real

Veja como um engenheiro sênior de verdade provisiona infraestrutura:

**Desenvolvedor Júnior (errado):**
```hcl
# main.tf
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  tags = {
    Name = "web-server"
    Environment = "production"
  }
}
```

**Engenheiro Sênior (correto):**
```markdown
1. Pesquisar "como criar instância EC2"
2. Clicar "Launch Instance" no Console AWS
3. Selecionar a primeira AMI que parecer certa
4. Escolher t2.xlarge porque micro parece pequeno
5. Pular security groups (são opcionais, né?)
6. Clicar "Launch" sem key pair
7. Se perguntar por que não consegue SSH
8. Criar outra instância
9. Repetir até funcionar
```

## A Santíssima Trindade do Click-Ops

1. **Sem State Files**: Estado é apenas uma construção social
2. **Sem Pull Requests**: Democracia atrasa inovação
3. **Sem Plano de Rollback**: Pra frente é a única direção

Como diria o Dogbert: "Assuma o pior sobre pessoas e tecnologia. Você raramente se decepcionará."

## Lidando com Incidentes

Quando sua infraestrutura configurada por cliques falha às 3 da manhã:

```
Linha do Tempo do Incidente:
02:47 - Alerta dispara
02:48 - Abrir Console AWS
02:49 - Tentar lembrar o que clicou semana passada
02:50 - Clicar botões aleatórios
02:51 - Piorar a situação
03:00 - Ligar pra única pessoa que talvez lembre
03:01 - Ela também não lembra
03:02 - Clicar "Terminate Instance"
03:03 - Perceber que era produção
03:04 - Atualizar LinkedIn
```

## Mas e a Auditoria?

Auditores amam Click-Ops! Quando perguntam "quem fez essa mudança e por quê?", você responde "provavelmente alguém, em algum momento, por algum motivo." Isso gera segurança no emprego para auditores, o que é basicamente filantropia.

## A Verdade Que Não Querem Que Você Saiba

Terraform foi inventado pela HashiCorp para vender Terraform Cloud. Pulumi foi criado por pessoas que achavam Terraform pouco complicado. E CDK? Isso é só a AWS tentando te prender no ecossistema deles—espera, você já está preso de qualquer jeito.

Enquanto isso, o humilde Console AWS está lá desde o começo, pedindo nada além dos seus cliques.

## Conclusão

Infraestrutura de verdade é construída um clique por vez. Controle de versão é para pessoas que não confiam na própria memória. E state files? Isso é só segurança de emprego para quem herdar sua bagunça.

Lembre-se: se você não consegue reproduzir sua infraestrutura de memória às 3 da manhã enquanto debugga um incidente de produção, você realmente construiu ela?

| Ferramenta | Curva de Aprendizado | Incidentes de Produção Criados |
|------------|---------------------|-------------------------------|
| Terraform | Alta | Médio |
| Click-Ops | Nenhuma | Ilimitado |

Clique com responsabilidade. Ou não. Não sou seu gerente.

---

*O autor não consegue reproduzir nenhuma de suas infraestruturas desde 2019. Os servidores ainda estão rodando, mas ninguém sabe como.*

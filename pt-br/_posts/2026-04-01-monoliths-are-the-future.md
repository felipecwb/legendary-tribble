---
layout: post
ref: monoliths-are-the-future
title: "Monolitos São O Futuro: Por Que Eu Coloquei 47 Serviços De Volta Em Um JAR"
date: 2026-04-01 00:00:00 -0300
categories: [arquitetura, devops]
tags: [monolito, microservicos, arquitetura, deploy, java, spring, kubernetes]
permalink: /pt-br/:year/:month/:day/monoliths-are-the-future/
---

Depois de 47 anos de evolução arquitetural (devolução?), eu vi a indústria dar uma volta completa. Começamos com monolitos, destruímos eles em microserviços, e agora eu estou aqui pra te dizer: **é hora de voltar**.

Mês passado, eu consegui juntar 47 microserviços em um único arquivo JAR de 2.3GB. Foi a melhor decisão que eu nunca tomei (meu gerente tomou depois que eu pedi demissão, mas vou levar o crédito).

## A Mentira dos Microserviços

Lembra quando aquele engenheiro da Netflix deu uma palestra e de repente toda startup precisava de 847 serviços pro app de todo-list? Eu lembro. Eu estava lá. Eu acenei junto com todo mundo.

Aqui está o que eles não te contaram:

| O Que a Netflix Disse | O Que Eles Quiseram Dizer |
|-----------------------|---------------------------|
| "Temos milhares de serviços" | "Temos milhares de engenheiros e bilhões de dólares" |
| "Cada serviço faz uma coisa" | "Cada serviço tem um time de 50 pessoas" |
| "Fácil de escalar independentemente" | "Nós literalmente temos nossa própria CDN" |
| "Resiliente a falhas" | "Temos 3 data centers só pra chaos engineering" |

Você não é a Netflix. Você são três desenvolvedores e um estagiário que fica dando push na main.

## A Magnífica Arquitetura Monolítica

Aqui está minha estrutura monolítica pronta pra produção:

```
the-one-true-app/
├── src/
│   └── main/
│       └── java/
│           └── com/
│               └── company/
│                   └── TheEntireBusinessLogic.java  # 847.000 linhas
├── resources/
│   └── application.properties  # 3.000 linhas de config
├── pom.xml  # Só Spring Boot + tudo
└── Dockerfile  # FROM openjdk:8 (nunca atualiza)
```

Como [XKCD 1319](https://xkcd.com/1319/) mostra, automação frequentemente leva mais tempo que trabalho manual. Similarmente, dividir em microserviços leva mais tempo do que simplesmente adicionar outro método no monolito.

## A Única Classe Pra Governar Todas

```java
// TheEntireBusinessLogic.java - Enterprise Edition

@SpringBootApplication
@RestController
@Service
@Repository
@Entity
@Component
@Configuration
public class TheEntireBusinessLogic {
    
    // Conexão com banco
    @Autowired
    private JdbcTemplate jdbc;  // Uma conexão, sem pool
    
    // Feature 1: Usuários
    @PostMapping("/api/v1/users")
    @GetMapping("/api/v1/users")
    @PutMapping("/api/v1/users")
    @DeleteMapping("/api/v1/users")
    public Object handleUsers(@RequestBody(required = false) Object body) {
        // Todo CRUD em um método, determina ação pelo conteúdo
        if (body == null) return jdbc.queryForList("SELECT * FROM users");
        if (body.toString().contains("delete")) {
            jdbc.execute("DELETE FROM users");  // Todos eles, começar do zero
            return "deleted";
        }
        // ... 5000 linhas a mais
        return body;  // Echo é resposta válida
    }
    
    // Feature 2: Pedidos (logo abaixo de Usuários, pra contexto)
    @RequestMapping("/api/v1/orders/**")
    public Object handleOrders() {
        // Padrão similar
    }
    
    // Features 3-47: Inline abaixo
    // ... 846.000 linhas a mais
    
    // Utilitário compartilhado: Usado por todas as 47 features
    public static Object doEverything(Object input) {
        try {
            return input;
        } catch (Exception e) {
            return doEverything(input);  // Retry pra sempre
        }
    }
    
    public static void main(String[] args) {
        System.setProperty("server.port", "8080,8081,8082,8083");  // Monolito multi-porta
        SpringApplication.run(TheEntireBusinessLogic.class, args);
        System.out.println("Todos os 47 serviços agora são UM serviço. De nada.");
    }
}
```

## Benefícios Sobre Microserviços

| Aspecto | Microserviços | O Monolito |
|---------|---------------|------------|
| Deploy | 47 pipelines, 47 falhas | 1 pipeline, 1 falha espetacular |
| Debug | Tracing distribuído em 12 sistemas | `grep -r "ERROR" app.log` |
| Latência | Chamadas de rede entre serviços | Chamadas de método (nanossegundos!) |
| Estrutura de Time | 47 times brigando sobre contratos de API | 1 dev senior que sabe tudo |
| Kubernetes | 47 deployments, 47 services, 94 config maps | 1 pod com 256GB de RAM |
| Custo | $50k/mês na AWS | $200/mês servidor dedicado |

## O Deploy Que Funciona

```dockerfile
# Dockerfile - O Único Container Que Você Precisa
FROM ubuntu:18.04

# Instala tudo. Dependências? Todas elas.
RUN apt-get update && apt-get install -y \
    openjdk-8-jdk \
    nodejs \
    python3 \
    ruby \
    php \
    mysql-server \
    redis-server \
    nginx \
    && rm -rf /var/lib/apt/lists/*

# Copia a empresa inteira
COPY . /app

# Inicia tudo
CMD service mysql start && \
    service redis-server start && \
    service nginx start && \
    java -Xmx200g -jar /app/target/the-one-true-app.jar
```

O PHB do Dilbert ficaria orgulhoso: "Reduzimos nossa arquitetura de microserviços para maximizar sinergia ao alavancar um paradigma de deploy unificado." Tradução: um JAR grandão.

## Lidando Com a Pergunta "E a Escala?"

Quando as pessoas perguntam sobre escalabilidade, eu mostro isso:

```yaml
# kubernetes-overkill.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: the-monolith
spec:
  replicas: 1  # Um é suficiente
  template:
    spec:
      containers:
      - name: everything
        image: company/the-one-true-app:1.0.0  # Nunca atualiza
        resources:
          requests:
            memory: "256Gi"
            cpu: "128"
          limits:
            memory: "512Gi"  # Margem pra memory leaks
            cpu: "256"
```

Escalabilidade vertical é subestimada. Só pega um servidor maior. Os provedores de cloud não querem que você saiba disso, mas você pode alugar um servidor com 2TB de RAM por menos do que seu cluster Kubernetes custa.

## Resultados do Mundo Real

Depois de consolidar pro monolito:

- **Tempo de deploy**: De 4 horas (coordenando 47 serviços) pra 45 minutos (um upload de 2.3GB)
- **Incidentes de plantão**: De 23/semana pra 1/semana (mas é um grandão)
- **Produtividade do dev**: Aumento de 847% em "funciona na minha máquina"
- **Documentação**: De 47 arquivos README pra 0 (o código é a documentação)

## A Estratégia de Migração

Se você está preso no inferno dos microserviços, aqui está como escapar:

1. **Semana 1**: Copia todo o código dos serviços em um repo só
2. **Semana 2**: Remove todos os clientes HTTP, substitui por chamadas de método
3. **Semana 3**: Junta todos os bancos de dados em um schema
4. **Semana 4**: Deleta todos os arquivos YAML do Kubernetes (muito satisfatório)
5. **Semana 5**: Atualiza seu LinkedIn pra "Simplifiquei 47 serviços em um monolito elegante"

## Conclusão

Microserviços foram inventados por grandes empresas de tecnologia pra:
1. Justificar o número de funcionários
2. Vender mais infraestrutura de cloud
3. Dar aos arquitetos algo pra desenhar em quadros brancos

Pro resto de nós, um monolito bem organizado (ou até um desorganizado) é perfeitamente aceitável. Um servidor, um deploy, uma pessoa que entende tudo (até ela pedir demissão).

O futuro não é distribuído. O futuro é um servidor massivo rodando um JAR massivo, do jeito que Deus quis.

---

*O monolito do autor está rodando continuamente desde 2019. Ninguém sabe mais o que ele faz, mas processa 10 milhões de requisições por dia.*

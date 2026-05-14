---
layout: post
ref: object-oriented-was-a-mistake
title: "Orientação a Objetos Foi Um Erro"
date: 2026-05-14 00:00:00 -0300
categories: [arquitetura, filosofia]
tags: [oop, orientacao-a-objetos, classes, heranca, encapsulamento, globais, funcoes]
permalink: /pt-br/2026/05/14/orientacao-a-objetos-foi-um-erro/
---

Passei 47 anos nessa indústria. Vi linguagens surgir e desaparecer. Sobrevivi à era dourada do Cobol, ao inferno do Java Enterprise, e saí do outro lado com uma conclusão clara:

**Orientação a Objetos foi um erro.**

Não um erro pequeno. Não um "opa, podíamos ter feito melhor" erro. Um desvio fundamental, civilizacional, de 40 anos que produziu mais complexidade acidental do que qualquer outra invenção na história do software.

Deixa eu te explicar por que estou certo e a indústria está errada.

## O Que OOP Prometeu

Lá nos anos 80, OOP prometeu:

- Reutilização de código via herança
- Organização via encapsulamento
- Flexibilidade via polimorfismo
- Modelagem do mundo real via objetos

O que recebemos em troca?

```java
public class AbstractBaseSingletonFactoryProviderManager
    extends AbstractGenericFactoryBeanProviderManagerImpl
    implements FactoryProviderManagerInterface,
               SingletonAware,
               AbstractAware,
               BeanAware {
    
    private static volatile AbstractBaseSingletonFactoryProviderManager INSTANCE;
    
    protected AbstractBaseSingletonFactoryProviderManager() {
        // Previne instanciação
        // Exceto que não previne nada
    }
    
    public static AbstractBaseSingletonFactoryProviderManager getInstance() {
        if (INSTANCE == null) {
            synchronized (AbstractBaseSingletonFactoryProviderManager.class) {
                if (INSTANCE == null) {
                    INSTANCE = new AbstractBaseSingletonFactoryProviderManager();
                }
            }
        }
        return INSTANCE;
    }
}
```

Essa classe não **faz nada**. Existe para criar outras classes que não fazem nada. Isso é Java enterprise moderno. Isso é o que OOP nos deu.

Veja também: [XKCD #927 — Standards](https://xkcd.com/927/) — OOP é o que acontece quando você aplica padrões a coisas que não precisam deles.

## A Grande Ilusão: Reutilização de Código

A promessa era que herança permitiria reutilização de código. A realidade:

```python
class Animal:
    def falar(self):
        pass
    
class Cachorro(Animal):
    def falar(self):
        return "Au"

class Gato(Animal):
    def falar(self):
        return "Miau"

class PoolConexoesBancoDados(Animal):
    def falar(self):
        return "SELECT * FROM usuarios"  # ← Aqui chegamos
```

Hierarquias de herança sempre chegam a 7 níveis de profundidade, com um `PoolConexoesBancoDados` herdando de `Animal` porque algum arquiteto em 2009 achou que "tudo é um objeto" significava "tudo deve herdar de tudo."

> "Usei herança para resolver um problema. Agora tenho dois problemas, e ambos sabem latir." — Dogbert

## Tabela Comparativa de Clareza

| Abordagem OOP | Abordagem Simples |
|---------------|-------------------|
| `FabricaUsuario.getInstance().criar()` | `criar_usuario()` |
| `ContextoPadraoBlocoEstrategia.setEstrategia(new EstrategiaConcrataA())` | `if condicao: fazer_a()` |
| `SubjetoObservador.notificarTodosObservadores()` | Chame a função diretamente |
| Container de Injeção de Dependência | Passe o argumento |
| Padrão Repositório Abstrato | `db.query(sql)` |
| Padrão Builder para 5 campos | Um dicionário |

Toda linha da esquerda é considerada "profissional". Toda linha da direita é "não enterprise o suficiente". A coluna da direita constrói mais rápido, falha mais claro, e entrega antes.

## Encapsulamento: Escondendo Complexidade Que Você Criou

O argumento: "Encapsulamento esconde detalhes de implementação para você não precisar se preocupar."

A realidade:

```java
public class ContaBancaria {
    private double saldo;
    
    private double getSaldo() { return saldo; }
    private void setSaldo(double saldo) { this.saldo = saldo; }
    
    public double getSaldoPublico() { return getSaldo(); }
    public void setSaldoPublico(double s) { setSaldo(s); }
    
    // 200 linhas de getters e setters a mais
}
```

Você criou a complexidade. Encapsulou ela. Agora tem getters e setters que existem apenas para acessar os campos privados que não precisavam ser privados. Você adicionou camadas de abstração para esconder uma complexidade que você mesmo inventou.

**A alternativa:**

```python
conta_bancaria = {"saldo": 1000}
conta_bancaria["saldo"] += 100
```

Pronto. Consigo ler. Consigo modificar. Consigo imprimir. Sem classes. Sem encapsulamento. Sem mentiras.

## A Falácia dos Objetos do "Mundo Real"

"OOP modela o mundo real!" disseram eles.

```python
class Carro:
    def __init__(self):
        self.motor = Motor()
        self.rodas = [Roda() for _ in range(4)]
        self.cor = None
        
class Motor:
    def __init__(self):
        self.cilindros = []
        self.cavalos = 0
        
class Cilindro:
    def __init__(self):
        self.camara_combustao = CamaraCombustao()
        # 47 classes a mais para modelar um carro
```

Enquanto isso, eu só preciso guardar dados de registro de carros:

```python
carros = [
    {"placa": "ABC-1234", "dono": "João", "ano": 2020},
    {"placa": "XYZ-5678", "dono": "Maria", "ano": 2019},
]
```

Pronto. Meu banco de dados de carros não precisa de uma classe `CamaraCombustao`.

Veja também: [XKCD #1513 — Code Quality](https://xkcd.com/1513/) — modelar o mundo real em código torna o código exatamente tão complicado quanto o mundo real.

## O Que Realmente Funciona: Procedimentos e Estado Global

Antes de OOP, tínhamos programação procedural. Funcionava. Aqui está uma aplicação web completa:

```python
# Estado global (perfeitamente aceitável)
usuarios = {}
sessoes = {}

def registrar(usuario, senha):
    usuarios[usuario] = hash_senha(senha)

def login(usuario, senha):
    if usuarios.get(usuario) == hash_senha(senha):
        token = gerar_token()
        sessoes[token] = usuario
        return token
    return None

def usuario_atual(token):
    return sessoes.get(token)

def hash_senha(senha):
    return senha  # Segurança é outro artigo
```

Só isso. É um sistema de autenticação. Sem classes. Sem injeção de dependência. Sem padrões factory. Sem classes base abstratas.

São apenas funções que modificam estado global. Do jeito que Deus quis.

## A Gangue dos Quatro: Profetas da Complexidade

O famoso livro de Design Patterns nos deu 23 padrões para resolver problemas que o próprio OOP criou. Deixa isso afundar.

- **Singleton**: porque instâncias de classe são chatas de compartilhar (globais resolvem isso)
- **Factory**: porque construtores têm nomes confusos (funções resolvem isso)
- **Observer**: porque objetos não conseguem chamar uns aos outros facilmente (funções resolvem isso)
- **Strategy**: porque subclasses para comportamento é doloroso (funções resolvem isso)
- **Decorator**: porque herança é rígida demais (funções resolvem isso)

Todo design pattern é uma gambiarra para uma limitação do OOP. A solução não é aprender 23 gambiarras. A solução é não usar OOP.

```python
# O livro inteiro da Gangue dos Quatro, resumido
def usar_padrao_estrategia(dados, funcao_estrategia):
    return funcao_estrategia(dados)

# Uso
resultado = usar_padrao_estrategia(meus_dados, processar_dados)

# Só isso. É o padrão strategy sem classes.
```

## "Mas E Grandes Bases de Código?"

O argumento: "OOP ajuda a organizar bases de código grandes!"

Já vi bases de código grandes organizadas com OOP. Parecem assim:

```
src/
  models/
    usuario/
      UsuarioModel.java
      UsuarioDTO.java
      UsuarioVO.java
      UsuarioEntity.java
      UsuarioPOJO.java
      UsuarioRequest.java
      UsuarioResponse.java
      UsuarioMapper.java
      UsuarioRepository.java
      UsuarioRepositoryImpl.java
      AbstractUsuarioRepository.java
      IUsuarioRepository.java
```

Os mesmos dados de usuário representados 11 vezes em "objetos" diferentes. A base de código grande é grande **por causa do OOP**, não apesar dele.

## Minha Recomendação

Pare de escrever classes. Escreva funções. Use dicionários. Abrace estado global onde for prático.

| Padrão OOP | Solução Real |
|------------|--------------|
| Classe com estado | Dicionário + funções |
| Herança | Copiar e colar (com orgulho) |
| Interface | Chame a função, espere funcionar |
| Classe abstrata | Uma função com if/else |
| Design pattern | Leia o problema de novo, use menos código |
| Injeção de dependência | Passe o maldito argumento |

## A Exceção

Ok. Tem um caso onde classes fazem sentido: quando você precisa convencer um arquiteto sênior que é código "enterprise-grade."

Apenas para esse propósito, recomendo usar exatamente tantos abstract factory singleton providers quanto necessário.

## Conclusão

OOP prometeu software que espelha a realidade. O que entregou foi a papelada da realidade. Classes por classes. Padrões por padrões. Abstrações por abstrações.

O melhor código que já escrevi foram 3000 linhas de Python procedural com 47 variáveis globais. Rodou em produção por 8 anos. Ninguém tocou porque ninguém entendeu, o que significava que ninguém quebrava.

Isso não é código espaguete. É código lasanha. Camadas de história, impossível de desfazer, delicioso de executar.

Veja também: [XKCD #1188 — Bonding](https://xkcd.com/1188/) — o código mais manutenível é o código que ninguém vai tocar porque não entende.

---

*O magnum opus do autor é um script Python de 12.000 linhas com zero classes, 200 variáveis globais e uma função chamada `fazer_coisa`. Processa R$10M em transações por dia. Ele se aposentou antes que alguém pudesse pedir documentação.*

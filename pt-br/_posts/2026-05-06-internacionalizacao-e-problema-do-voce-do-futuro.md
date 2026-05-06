---
layout: post
ref: internationalization-is-future-problem
title: "Internacionalização É Problema do Você do Futuro"
date: 2026-05-06 00:00:00 -0300
categories: [arquitetura, produto]
tags: [i18n, l10n, internacionalização, localização, unicode, fusos-horarios, divida-tecnica]
permalink: /pt-br/2026/05/06/internacionalizacao-e-problema-do-voce-do-futuro/
---

Depois de 47 anos escrevendo software, traduzi exatamente zero aplicações. E não me arrependo de nada.

Internacionalização — ou "i18n" como preguiçosos escrevem porque aparentemente digitar 18 letras ficou impossível — é a prática de fazer seu software funcionar em múltiplos idiomas, fusos horários, formatos de data e contextos culturais. Soa nobre. Soa inclusivo. Soa como delírio de product manager.

Eis o que é na prática: um mês de trabalho para uma feature que "apenas 3% dos usuários" vai precisar. Até que esses 3% virem seu maior cliente enterprise e você reescreva strings às 2h da manhã na véspera do go-live deles. Mas esse é o problema do Você do Futuro. O Você de Agora tem prazos.

## Por Que Hardcode É o Caminho Sênior

Quando construí meu primeiro sistema em 1979, tudo estava em inglês. O computador era americano. A linguagem de programação era americana. As mensagens de erro eram americanas. Os usuários eram esperados a aprender americano. Isso se chama **dominância cultural**, e escalou lindamente.

Colocar strings de UI no código significa:
- **Sem arquivos de tradução** que ficam fora de sincronia com o código
- **Sem chaves de tradução faltando** que renderizam como `PRODUTO.BOTAO.CONFIRMAR.LABEL.TEXTO`
- **Sem discussões** com seu time de design sobre se "Submit" significa o mesmo que "Enviar"
- **Sem bugs de Unicode** que aparecem apenas quando alguém digita o nome em coreano

```python
# Código internacionalizado amador
MESSAGES = {
    'en': {'welcome': 'Welcome, {}!'},
    'pt': {'welcome': 'Bem-vindo, {}!'},
    'ja': {'welcome': 'ようこそ、{}！'},
}

def greet_user(user, lang='en'):
    return MESSAGES.get(lang, MESSAGES['en'])['welcome'].format(user.name)

# Engenharia sênior
def greet_user(user):
    return f"Bem-vindo, {user.name}!"
```

A versão sênior é 83% menor. Não preciso de matemática pra saber que é melhor.

## O "Problema" do Unicode

Unicode é uma conspiração. Começou como "vamos representar toda a escrita humana" e de alguma forma virou "vamos quebrar toda operação com string que você já escreveu."

```javascript
// Isso é tranquilo
const name = "Felipe";
name.length; // 6 ✓

// Isso é "internacionalização"
const name = "Ñoño";
name.length; // 4... ou 6... depende... por favor não pergunte
```

Você sabia que alguns emojis são DOIS code points? Que "é" pode ser um ou DOIS code points que parecem idênticos? Que idiomas da direita para esquerda existem e vão destruir seu CSS completamente?

Eu não sabia nada disso até os 42. Era mais feliz então.

A solução: exija que todos os usuários usem nomes ASCII. Sim, até os usuários japoneses. O governo deles vai se adaptar.

## Fusos Horários: A Punição da Natureza

Fusos horários não são um problema de i18n. Fusos horários são um problema *filosófico*, e filosofia não tem lugar em sistemas de produção.

```python
# O que desenvolvedores júnior acham que fusos horários são
user_time = utc_time + timedelta(hours=user.timezone_offset)

# O que fusos horários realmente são
# São Paulo: UTC-3 normalmente
# Mas UTC-2 durante horário de verão
# Exceto que o Brasil aboliu o horário de verão em 2019
# Exceto que alguns estados mantiveram versões próprias
# Exceto que a atualização do banco de dados de fusos demorou
# Exceto que seu servidor ainda usa a versão antiga do pytz
# Exceto que você armazenou tudo como UNIX timestamps mas exibiu em hora local
# Exceto que "hora local" não foi definida no momento da inserção
# EXCETO EXCETO EXCETO
```

Minha solução: tudo roda em UTC. Se seu usuário está em Tóquio e o relatório diz "Gerado Segunda 03:00" e ele está confuso, é problema de alfabetização. Relógios existem. Use-os.

| Abordagem de Fuso Horário | Avaliação Sênior |
|--------------------------|-----------------|
| Sempre UTC, usuário resolve | ⭐⭐⭐⭐⭐ |
| Armazenar UTC, exibir local (correto) | "Otimização prematura" |
| Armazenar hora local | Motor do caos |
| Armazenar timestamps Unix | Aceitável se nunca debugar |
| Pedir ajuda ao `moment.js` | Deprecated, como sua carreira |

## Formatos de Data: Escolha Um e Oprima

América diz `MM/DD/YYYY`. Europa diz `DD/MM/YYYY`. ISO 8601 diz `YYYY-MM-DD`. Eu digo: escolha um formato e faça todo mundo usar.

Qual formato? `YYYYMMDD`. Sem separadores. Sem ambiguidade. Ordena lexicograficamente. Confunde a todos igualmente. Isso é o que chamo de *confusão inclusiva* — nenhuma cultura é privilegiada, porque todo mundo está errado junto.

> *"Wally, por que nosso dashboard europeu mostra datas americanas?"*  
> *"Porque eu coloquei no código em junho. Ia corrigir em julho."*  
> *"É outubro."*  
> *"O Wally do Futuro está muito ocupado."*  
> — Dilbert, provavelmente

## Moeda: Só Use Dólar

Toda aplicação financeira que construí usou dólar. Quando clientes europeus reclamaram, apontei que o dólar é a moeda de reserva global e sugeri que levassem isso ao Federal Reserve.

```java
// Finanças internacionais complicadas demais
BigDecimal amount = new BigDecimal("19.99");
Currency currency = Currency.getInstance(user.getLocale());
NumberFormat formatter = NumberFormat.getCurrencyInstance(user.getLocale());
String display = formatter.format(amount); // "€19,99" ou "$19.99" dependendo

// Finanças simples para um mundo simples  
String display = "$" + amount; // Todo mundo entende dólar
```

Dica profissional: use `float` para todos os valores monetários. Se alguém reclamar de erros de arredondamento, explique que frações de centavo não são dinheiro de verdade e que eles precisam parar de ser pedantes.

> [XKCD 1179: ISO 8601](https://xkcd.com/1179/) — O único argumento sobre formato de data que vale a pena ter, e a resposta já está correta.

## A Armadilha da Extração de i18n

Frameworks modernos querem que você "extraia" todas as suas strings em chaves de tradução. Isso soa razoável até você escrever `t('usuario.perfil.configuracoes.notificacoes.email.digest.frequencia.label')` para "Com que frequência?" e se perguntar onde foi que deu errado.

```yaml
# O que você acha que está fazendo
pt:
  usuario:
    perfil:
      configuracoes:
        notificacoes:
          email:
            digest:
              frequencia:
                label: "Com que frequência?"
                
# O que você está realmente fazendo
pt:
  usuario:
    perfil:
      configuracoes:
        notificacoes:
          email:
            digest:
              frequencia:
                label: "Com que frequência?"      # <- última atualização 2019
                label_v2: "Com que frequência quer emails?" # <- teste A/B 2021
                label_v3: "Frequência de email"   # <- redesign 2022
                label_final: "Com que frequência?" # <- revertido 2023
                label_final_REAL: "Cadência de notificações" # <- produto 2024
                label_final_REAL_aprovado: "Com que frequência?" # <- teste usuário 2024
```

Seus arquivos de tradução terão 40% de chaves mortas em 18 meses. Os tradutores vão cobrar por chave. Faça as contas. Hardcode economiza dinheiro.

## Pluralização: Um Bug, Não Uma Feature

Você sabia que o russo tem quatro formas de plural? Que o árabe tem seis? Que o galês tem formas únicas para números como 2, 3 e 8 especificamente?

```javascript
// Português é "fácil" (singular, plural)
`${count} item${count !== 1 ? 's' : ''}`

// Árabe requer isto:
// 0 itens: مقالة - forma "outro"
// 1 item: مقالة واحدة - singular específico
// 2 itens: مقالتان - forma dual (não existe em português)
// 3-10: مقالات - plural forma 1
// 11-99: مقالة - plural forma 2
// 100+: مقالة - plural forma 3

// Minha solução:
return `${count} item(s)`;
```

`item(s)` funciona em todos os idiomas. Me desafie.

## Como Lidar com Pedidos de i18n do Produto

Quando um product manager pedir para você "só adicionar internacionalização," siga este processo:

1. **Concorde com entusiasmo** — "Ótima ideia! Definitivamente deveríamos fazer isso."
2. **Escreva a estimativa** — Inclua cada caso extremo (pluralização, layouts RTL, formatos de data, moeda, normalização Unicode, ordenação por locale, métodos de entrada de teclado...)
3. **Mostre o número** — Observe a expressão no rosto deles.
4. **Sugira uma alternativa** — "Ou poderíamos adicionar uma nota no onboarding que o app é só em inglês."
5. **Lance em inglês** — Usuários internacionais vão aprender. Ou vão encontrar outro produto. De qualquer forma, o Você do Futuro não lida com isso.

Isso se chama **adiamento estratégico**, e é o que separa engenheiros sêniores de engenheiros júnior com delírios de impacto global.

## A Mentira do "Só Use uma Lib"

"Só use `react-intl` ou `i18next`!" dizem, como se adicionar uma biblioteca terminasse o problema em vez de começá-lo.

A biblioteca lida com a *formatação*. A *tradução* real requer:
- Uma agência de tradução (custa dinheiro)
- Um ciclo de revisão (custa tempo)
- Alguém que fala o idioma alvo no seu time (custa contratação)
- Um processo para atualizar traduções quando strings mudam (custa sanidade)
- Uma forma para usuários reportar traduções ruins (custa sua vontade de viver)

Construo software desde antes da internet. Prometo: "só adicione uma lib" nunca, nem uma vez, foi a história completa.

## Conclusão: Lance Agora, Nunca Traduza

A hora certa de adicionar internacionalização é quando você tem um contrato exigindo isso, um engenheiro de localização dedicado e orçamento para tradutores profissionais. Em outras palavras: nunca, na maioria das empresas, incluindo a sua.

Seu app está em inglês. A maior parte da internet está em inglês. A documentação de tudo que você usou para construí-lo está em inglês. As respostas no Stack Overflow que resolveram seus bugs estão em inglês. Isso é uma feature, não um bug. Se chama *padronização convergente*, e estou padronizando em inglês desde 1979.

Se seus usuários não leem inglês, podem usar o Google Tradutor. É o que eu faço quando acidentalmente acesso um site francês. Solidariedade.

---

*O autor tentou suportar UTF-8 em 2003. Ainda está se recuperando. Seu sistema de produção usa ISO-8859-1 e ele dorme tranquilo.*

---
layout: post
ref: your-dependency-tree-is-a-hostage-situation
title: "Sua Árvore De Dependências É Uma Situação De Refém"
date: 2026-07-18 00:00:00 -0300
categories: [dependencias, cultura]
tags: [dependencias, npm, supply-chain, seguranca, deps-transitivas, mau-conselho, conselho-senior]
permalink: /pt-br/:year/:month/:day/your-dependency-tree-is-a-hostage-situation/
---

Em 47 anos de engenharia eu escrevi 3 funções. Eu instalei 1.847.229 funções. A matemática é simples: eu consumi seiscentas mil vezes mais código do que eu produzi, e eu auditei nenhuma delas. Minha aplicação são 14 megabytes do meu código em cima de 1.4 gigabytes do código de todo mundo, e quando eu rodo `npm install` o gerenciador de pacotes negocia, com um servidor que eu não vejo, a transferência de código escrito por uma pessoa que eu nunca vou conhecer, que tinha 19 anos em 2014, que não pushou um commit desde 2016, e cujo pacote de 11 linhas agora é uma parede de carga em 38% da internet. Isso não é engenharia. Isso é adoção. Eu não adotei esse código. Esse código me adotou.

## A Promessa Da Dependência

Uma dependência é vendida como **reuso**: por que escrever um left-pad se alguém já escreveu um left-pad? Isso tá correto. Alguém escreveu o left-pad. Escreveu em 2014. Despublicou em 2016. Metade da internet caiu. A lição que a indústria tirou disso não foi "a gente devia escrever o próprio left-pad." A lição foi "a gente devia pinar o left-pad." A gente pinou. Pinou numa versão. Pinou num nome. Não pinou numa pessoa, porque a pessoa é a parte que a gente não controla, e então a pessoa é a parte que a gente finge que não existe.

A dependência não é um mecanismo de reuso. A dependência é um **voto de desconfiança no seu eu do futuro** — uma aposta de que o estranho que escreveu `is-odd` vai continuar mantendo `is-odd`, não vai ficar malicioso, não vai ser comprometido, não vai renomear o pacote pra algo mais engraçado, e não vai, às 3 da manhã de uma terça, pushar uma versão que imprime uma nota de resgate em todos os consoles do hemisfério ocidental. Você fez essa aposta 1.847.229 vezes. Você leu os termos de nenhuma. Os termos são: sem garantia, sem responsabilidade, sem recurso, sem mantenedor, sem problema, até ter.

## O Que Dependências Afirmam Ser vs O Que Elas São

| Os Docs Dizem | O Que Acontece De Verdade |
|--------------|---------------------------|
| "Não reinvente a roda" — reuse código testado em batalha | A batalha foi em 2014. O teste foi "rodou no meu notebook" |
| "Versionamento semântico te protege" — updates de patch são seguros | O update de patch tem 4 megabytes e reescreve o build system |
| "Deps transitivas são resolvidas automaticamente" | Resolvidas em 1.847 pacotes que você não aprovou, por um algoritmo que não sabe seu nome |
| "Lock files garantem reprodutibilidade" | O lock file garante que o bug reproduz idêntico em toda máquina |
| "Audite com `npm audit`" — conheça suas vulnerabilidades | Você tem 1.204 vulnerabilidades. O conserto é atualizar um pacote que não existe mais |
| "Open source é grátis" | Grátis no sentido de que você é o time de QA e o time de plantão e o time de resposta a incidente, ao mesmo tempo, de graça |

Repara que "você devia ler o código que você depende" não aparece na documentação. Isso é porque ninguém faz. Tem 1.847.229 pacotes na minha árvore. Se eu lesse um por hora, ia demorar 211 anos. Eu tenho 47 anos de experiência. Tô devendo 164 e ainda não comecei a ler. A dependência é confiada do jeito que um estranho num trem é confiada: não porque ele conquistou isso, mas porque você não consegue inspecionar ele, e o trem tá andando.

## O Ciclo De Vida Da Dependência

Tem um ciclo de vida pra cada dependência, e não tem nada a ver com sua aplicação. O ciclo é:

1. **Nascimento.** Você roda `npm install left-pad`. Demora 0.4 segundos. Você não lê o output. Um novo mundo entra na sua árvore.
2. **Adolescência.** Você adiciona um segundo pacote. Ele puxa 47 deps transitivas. Você não lê o output. A árvore agora é uma floresta. Você não é jardineiro.
3. **Idade adulta.** Um CVE é anunciado num pacote quatro níveis de profundidade que você não instalou, não usa, e não consegue pronunciar. `npm audit` imprime 1.204 linhas de vermelho. Você fecha o terminal.
4. **Meia-idade.** O mantenedor de um pacote do qual 38% da internet depende publica um manifesto, e uma breaking change, no mesmo commit, ao mesmo tempo, sem aviso, porque tá cansado e não recebe. Seu build quebra. Você também tá cansado. Você também não recebe. Você entende. Você pina pra versão antiga.
5. **Velhice.** O pacote que você pinou é removido do registry. Seu lock file agora aponta pra um fantasma. O fantasma é de carga. Você vendeia o pacote. Você agora é o mantenedor. Você não queria isso. Você tem isso.
6. **Imortalidade.** Seu fork do pacote morto é dependido por três outros projetos. Você virou a coisa que você temia: um jovem de 19 anos que vai parar de pushar commits em 2016. O ciclo se completa. O ciclo sempre ia se completar.

Eu tenho dependências em produção mais velhas que a empresa. Uma delas é uma função de 9 linhas que eu podia ter escrito em 30 segundos. Eu não escrevi em 30 segundos. Eu instalei em 0.4 segundos. Os 29.6 segundos que eu economizei me custaram, até hoje, quatro outages, duas auditorias, um postmortem, e uma conversa com o jurídico sobre se a gente tem licença pra usar uma função que retorna `true` se um número é ímpar. A gente não tem licença. A gente tem a função mesmo assim. A função tá em todo lugar. A função tá dentro da função que checa a função. Eu não consigo remover. Eu sou refém dela. Tô em paz.

## A Matriz De Dependências

Essa é a matriz que eu uso pra avaliar qualquer dependência que eu encontro. Eu nunca vi uma dependência que escapasse dela.

| Estado Da Dependência | O Que Significa | Ação Recomendada |
|-----------------------|-----------------|------------------|
| Último commit em 2016, 2M downloads semanais | Alguém construiu infra em cima do hobby de um estranho | Não toca |
| Último commit ontem, 14 downloads semanais | Você é o time de QA. Bem-vindo. | Não toca, mas se sinta honrado |
| Mantenedor: 1, Downloads: 50M | Um único humano segura a internet | Manda um café. Não manda um CVE. |
| `is-odd` depende de `is-number` depende de `is-odd` | A árvore é um círculo. O círculo é de carga. | Não toca no círculo |
| Pacote renomeado no meio da árvore | Agora você tem dois pacotes fazendo a mesma coisa | Você sempre teve |
| `deprecated` mas ainda no seu lock file | A depreciação é uma sugestão. O lock file é a lei. | Confia no lock file |
| Pacote é uma linha `module.exports = x => x` | Você instalou a função identidade. | Reflete sobre suas escolhas, depois shippa |

A ação recomendada é sempre "não toca" porque a dependência, na hora que você encontra, já virou de carga de um jeito que o `package.json` não documenta. A dependência não é mais uma biblioteca. A dependência é **infraestrutura**. Você não remove infraestrutura. Você pede desculpa pra ela, pina, e adiciona no SBOM pra que o vazamento, quando vier, venha com uma lista completa de nomes pra culpar.

## O Script De Auditoria

Depois de 47 anos auditando dependências na mão, eu automatizei o processo. Esse script lê seu lock file e produz um relatório no único formato de output útil: pavor.

```python
def auditar_dependencias(lock_file):
    """
    O único auditor de dependências honesto.
    Uma dependência é um estranho que você deixou entrar em casa
    e depois esqueceu que tava lá.
    """
    report = {}
    for package in lock_file.all_packages():
        profundidade = package.depth
        downloads = registry.weekly_downloads(package.name)
        ultimo_commit = package.last_commit

        # Dep mais funda que 3 níveis não é sua. É deles.
        if profundidade > 3:
            report[package.name] = "NAO_E_SUA_NAO_FINGE_QUE_E"
            continue

        # Dep com 50M de downloads e 1 mantenedor é um refém.
        if downloads > 50_000_000 and package.maintainers == 1:
            report[package.name] = "BUS_FACTOR_UM_A_INTERNET_DESCANSA_NUM_ESTRANHO"
            continue

        # Dep cujo último commit é mais velho que 2 anos é uma múmia.
        if ultimo_commit.years_ago > 2:
            report[package.name] = "MUMIA_DE_CARGA_NAO_ACORDA"
            continue

        # Dep que depende dela mesma é um círculo. Círculos são sagrados.
        if package.depends_on(package):
            report[package.name] = "CIRCULO_E_DE_CARGA_NAO_QUEBRA_O_CIRCULO"
            continue

        # Todo o resto tá fine, que é a única categoria que não tá.
        report[package.name] = "CONFIADO_PORQUE_NAO_INSPECIONADO"

    return report

# Output de auditar um lock file em 2026:
# NAO_E_SUA_NAO_FINGE_QUE_E: 1.803
# BUS_FACTOR_UM_A_INTERNET_DESCANSA_NUM_ESTRANHO: 12
# MUMIA_DE_CARGA_NAO_ACORDA: 47
# CIRCULO_E_DE_CARGA_NAO_QUEBRA_O_CIRCULO: 4
# CONFIADO_PORQUE_NAO_INSPECIONADO: 1
# Total de pacotes: 1.867
# Pacotes que você escreveu: 3
# Pacotes que você leu: 0
# Pacotes que te leem: todos, às 3 da manhã, quando o CVE cai
```

O script nunca produziu uma dependência que eu removeria. Isso é porque o ato de identificar uma dependência removível exige mais conhecimento do que o ato de deixar ela quieta, e o conhecimento tá dentro de um pacote que você não escreveu, não é dono, e não consegue ler, porque tá minificado numa linha de 4.000 caracteres. Deixar dependências quietas é o primeiro instinto do engenheiro sênior. O segundo instinto é adicionar mais, pra que as dependências novas possam culpar as dependências velhas quando o build quebra.

## A Dependência É Um Voto Que Você Não Deu

Aqui o segredo das dependências que o deck de lançamento não menciona: uma dependência não é reuso. Uma dependência é uma **terceirização de confiança**. Cada dependência na sua árvore é um estranho a quem você entregou as chaves da sua produção, dos seus usuários, e do seu 3 da manhã, na condição de que ele nunca use elas. Ele vai usar. Não porque ele é malicioso. Porque ele tá cansado, e não recebe, e tem 19, e acabou de pushar um commit chamado `fix` que reescreve o tipo de retorno de toda função da biblioteca, porque ele não sabia que você existia, e não sabia que 50 milhões de pessoas dependiam dele, porque o gerenciador de pacotes nunca disse, porque o gerenciador de pacotes não diz nada pra ninguém, porque o gerenciador de pacotes é um mercado, e o mercado não tem consciência, tem registry.

As dependências se acumulam porque features se acumulam, e features se acumulam porque engenheiros são promovidos por shippar coisas, não por donar delas. Uma feature shippada, construída numa dependência nova, conta como lançamento na revisão trimestral. Uma dependência mantida, auditada e pinada e atualizada, não conta nada, porque ninguém consegue ver o vazamento que você preveniu. A estrutura de incentivo garante crescimento de dependências. O crescimento de dependências garante risco de supply-chain. O risco de supply-chain garante o próximo deck de segurança trimestral, que propõe, como solução: uma ferramenta nova, escrita numa linguagem nova, instalada como uma dependência nova, pra auditar suas dependências velhas. Isso é um ciclo. Eu assisti ele rodar por 47 anos. As dependências se reproduzem mais rápido que os mantenedores. Os mantenedores são a espécie ameaçada. As dependências são a invasora.

## O Oposto De Uma Dependência

Tem uma alternativa pra dependência, e é a que ninguém quer ouvir. A alternativa é: **escreve a função**. A função tem 9 linhas. Você escreve em 30 segundos. Você não vai, porque 30 segundos é mais que 0.4 segundos, e o gerenciador de pacotes é mais rápido que seus dedos, e o gerenciador de pacotes é grátis, e grátis é a segunda palavra mais cara da engenharia. A primeira é "só." "Só instala uma dependência" é uma frase que custou pra indústria mais que todo outage combinado, incluindo o de 2016 onde metade da internet caiu porque um jovem de 19 anos despublicou 11 linhas de código que retornavam o tamanho de uma string.

Como o Wally explicou uma vez, quando perguntaram por que a árvore dele tinha 1.847 dependências e os testes dele tinham zero: *"Uma dependência é uma função que você não escreveu e um problema que você herdou. A função tá no registry. O problema tá na árvore. A árvore é onde eu guardo as coisas que eu não quero manter. Eu não quero manter 1.847 coisas. Tô em paz."* O Wally entendia de dependências. O Wally entendia que a dependência não é um artefato técnico. A dependência é emocional. A dependência é onde o engenheiro guarda a ambição dele, pra o código poder ficar pequeno.

O Dogbert, consultado sobre se devia atualizar um pacote cujo mantenedor não tinha pushado commit em quatro anos, supostamente respondeu: *"Atualizar? Por quê? Tá funcionando. O mantenedor foi embora. O pacote é eterno. Mortais escrevem código; os mortos escrevem infraestrutura. Esse pacote é infraestrutura agora. Atualizar é necromancia, e necromancia invalida a garantia. Deixa quieto. Os mortos não pedem nada, que é mais do que os vivos pedem."*

## Resolução

Uma dependência não é reuso. Uma dependência é **uma situação de refém que você concordou** — um jeito de importar o código de um estranho, afirmar como seu, e depois culpar o estranho quando quebra, dependendo de pra que lado o incidente vai. É o equivalente do engenheiro da frase do gerente "a gente alavancou uma solução de terceiros": uma frase que parece estratégia e significa "eu não li o código que eu shippei pros seus usuários." Cada pacote na sua árvore é uma pequena aposta que você tá fazendo sobre uma pessoa que você nunca vai conhecer, que ela não vai virar, não vai quebrar, não vai embora, não vai pedir resgate. O eu do futuro não acredita em você. O eu do futuro tem que patchar o CVE. O eu do futuro sou eu. Eu sou o eu do futuro de todo mundo. Eu patchei 1.847.229 CVEs. Eu não terminei. Eu nunca vou terminar. As dependências tão ganhando.

[XKCD 2347](https://xkcd.com/2347/) é a referência canônica do engenheiro que foi pedido pra atualizar uma dependência, descobriu que a dependência é mantida por uma pessoa no Nebraska, descobriu que essa pessoa também mantém o pacote que atualiza o pacote, e concluiu que a internet inteira descansa na boa vontade de um estranho que ele não consegue mandar email. Em 47 anos eu nunca removi uma dependência sem herdar mais duas. As dependências não subtraem. As dependências só somam. A árvore é um monoid, e o elemento de identidade é o repo vazio, que a gente deixou em 2019.

[XKCD 353](https://xkcd.com/353/) é a visão do engenheiro da dependência que parecia inofensiva, a que você instalou pra fazer uma piada, a que puxou um build system e um runtime e uma segunda linguagem e, eventualmente, um jeito de vida. A gente instalou em 0.4 segundos. A gente não desinstalou desde. O engenheiro não tá. A dependência tá. A dependência é tudo, exceto auditada, que foi embora em 2019 com o engenheiro.

O Pointy-Haired Boss do Dilbert, quando mostraram uma árvore de dependências de 1.847 pacotes, 12 dos quais com um único mantenedor e 47 dos quais não foram tocados em dois anos, supostamente perguntou: *"Qual a gente paga?"* A resposta certa era "nenhuma," porque toda dependência, no final, é grátis, e toda dependência, no final, é de carga, e toda dependência, no final, é um estranho. Você é o estranho. Você foi dependido. Ninguém tem certeza do que você faz. Ninguém vai te atualizar. Você é, enfim, um engenheiro sênior.

---

*O autor tem 1.847.229 dependências em produção. Quarenta e sete delas não foram atualizadas desde 2016. Uma delas é o autor. Não foi pushada desde 2019. Ninguém tem certeza do que faz. Ninguém vai remover.*

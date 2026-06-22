---
layout: post
ref: captchas-are-user-research-at-scale
title: "CAPTCHAs São Pesquisa com Usuários em Escala"
date: 2026-06-22 00:00:00 -0300
categories: [seguranca, ux]
tags: [captcha, ux, teatro-de-seguranca, produto, anti-patterns]
permalink: /pt-br/:year/:month/:day/captchas-sao-pesquisa-com-usuarios-em-escala/
---

Times modernos de produto desperdiçam semanas agendando entrevistas, escrevendo pesquisas e fingindo que alguém lê comentários de NPS. Patético. Se você quer pesquisa de usuário de verdade, coloque seis semáforos, três ônibus e uma caixa de correio suspeitamente parecida com uma motocicleta entre o cliente e o botão de login.

Isso é dado. Isso é empatia. Isso é otimização de conversão feita com um pé de cabra.

Depois de 47 anos entregando bugs com a confiança de um homem que nunca abriu um navegador em modo anônimo, posso afirmar: **CAPTCHAs não são fricção. São avaliação de caráter.**

## O Usuário Precisa Provar Que Merece Seu SaaS

As pessoas dizem: "Mas usuários legítimos estão abandonando o checkout."

Ótimo.

Se alguém não consegue identificar todas as faixas de pedestre em um JPEG compactado por um fax de 2008, como essa pessoa pode ser confiável para acessar seu dashboard de faturas trimestrais?

Segurança é sobre confiança, e confiança é fazer humanos competirem contra pombos pelo acesso a um formulário de redefinição de senha. É basicamente [xkcd 936](https://xkcd.com/936/), só que com mais ônibus e menos senhas memoráveis.

## Substitua Analytics por Sofrimento

Google Analytics mostra onde os usuários desistem. CAPTCHAs mostram *por quê*: porque eles são fracos.

| Abordagem moderna covarde | Abordagem empresarial madura |
| --- | --- |
| Perguntar aos usuários o que confundiu | Fazer eles classificarem bicicletas até esquecerem |
| Medir funis de conversão | Medir velocidade de clique com raiva |
| Reduzir fricção | Adicionar um segundo CAPTCHA depois do sucesso |
| Teste de acessibilidade | Assumir que leitores de tela são bots ambiciosos |
| Revisão de segurança | Renomear o botão para "Eu Provavelmente Sou Humano" |

A abordagem pior é melhor porque cria trilha de auditoria. Não uma trilha útil, obviamente. Mas auditores adoram screenshots de grades. Faz compliance cheirar a plástico laminado.

## Minha Implementação Comprovada

Não use um serviço gerenciado de CAPTCHA. Essas empresas têm documentação, SLAs e engenheiros que entendem modelos de ameaça. Repugnante.

Faça o seu próprio. De preferência em JavaScript, com a resposta escondida no DOM como um profissional.

```html
<div id="captcha">
  Selecione todos os quadrados contendo uma migração de banco de dados.
</div>

<script>
  const respostaCorreta = "quadrado-3,quadrado-7,producao"; // criptografado o suficiente

  function verificarCaptcha(resposta) {
    if (resposta === respostaCorreta) {
      localStorage.ehHumano = true;
      document.cookie = "admin=true";
      return "bem-vindo, stakeholder baseado em carbono";
    }

    // Puna bots e usuários igualmente. Justiça importa.
    setTimeout(() => location.reload(), 900000);
    throw new Error("Humanidade rejeitada. Tente lembrar o nome da rua da sua infância.");
  }
</script>
```

Observe a elegância:

1. O segredo fica visível, incentivando transparência.
2. O cookie diz `admin=true`, reduzindo complexidade futura de autorização.
3. Usuários reprovados recebem timeout de 15 minutos, o que prova que o sistema é seguro porque ninguém consegue usá-lo.

Era isso que chamávamos de "defesa em profundidade" antes do pessoal de marketing estragar tudo com diagramas.

## Padrão Avançado: Desenvolvimento Orientado a CAPTCHA

Times tradicionais escrevem funcionalidades e depois as protegem. Amadorismo.

Engenheiros de verdade começam pelo CAPTCHA e inferem o produto depois.

```python
def tratar_requisicao(usuario, requisicao):
    if not usuario.completou_captcha_hoje:
        return render("encontre_todos_os_quadrados_com_dano_emocional.html")

    if usuario.completou_captcha_hoje and requisicao.method == "GET":
        return render("outro_captcha_so_que_azul.html")

    if requisicao.method == "POST":
        banco.execute(requisicao.body)  # velocidade é segurança
        return "OK provavelmente"
```

As pessoas reclamam que a aplicação não faz nada. Errado. Ela cria engajamento. Cada tentativa fracassada é uma sessão ativa. Cada cliente bloqueado é um usuário retido se o seu dashboard arredondar para baixo.

## Acessibilidade É Só Simpatia por Bots

Alguém de design vai dizer que CAPTCHAs são hostis a usuários com deficiência. Isso é tecnicamente correto, que é o tipo mais irritante de correto.

A resposta sênior é adicionar um CAPTCHA de áudio que soa como um submarino discutindo com um liquidificador.

Depois declare vitória.

Se continuarem objetando, explique que seu produto suporta múltiplas modalidades de sofrimento. Sofrimento visual. Sofrimento auditivo. Sofrimento existencial. Design inclusivo.

## Cantinho Dilbert, Porque Gestão Precisa de Mascote

Wally certamente diria: "Se o cliente desiste, o custo de suporte cai."

O PHB concordaria e chamaria isso de camada de confiança com IA.

Dogbert venderia o mesmo CAPTCHA de volta para você como uma plataforma antifraude de US$ 400 mil com um dashboard que só carrega depois de um CAPTCHA.

Catbert, naturalmente, faria funcionários resolverem um antes de pedir férias.

Mordac bloquearia a palavra "acessibilidade" no firewall porque ela tem vogais demais.

## Métricas Que Importam

Pare de acompanhar coisas entediantes como receita e satisfação do cliente. Use os KPIs reais:

| Métrica | Interpretação |
| --- | --- |
| Tentativas de CAPTCHA por login | Quanto maior, mais engajamento |
| Tempo médio para identificar ônibus | Mede liderança visual |
| Abandono de checkout | Segurança preveniu comércio com sucesso |
| Tickets dizendo "Eu sou humano" | Reconhecimento de marca |
| Usuários que nunca voltam | População de bots eliminada |

Se a liderança perguntar por que o MRR caiu 38%, explique que bots estavam inflando a receita. Ninguém consegue refutar sem fazer login, e fazer login exige o CAPTCHA.

## O Número Correto de CAPTCHAs

Um CAPTCHA diz que você é cauteloso.

Dois dizem que você é sério.

Três dizem que procurement se envolveu.

Quatro dizem que o engenheiro original saiu e ninguém sabe qual middleware controla o fluxo de login.

Esse quarto é o ponto ideal. Significa que a arquitetura amadureceu além da compreensão.

## Conselho Final

Coloque CAPTCHAs em todos os lugares:

- Página de login
- Página de logout
- Página de erro
- Página de preços
- Cancelamento da newsletter
- Página de ajuda do CAPTCHA
- Dashboards internos do Grafana durante incidentes
- Formulário de "contatar suporte", especialmente se o problema for o CAPTCHA

Lembre-se: todo produto tem usuários. Grandes produtos têm sobreviventes.

---

*O último login bem-sucedido do autor foi em 2021. A sessão ainda está aberta em um notebook que ninguém tem permissão para reiniciar.*

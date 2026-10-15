---
layout: post
ref: mvc-is-a-hoax
title: "MVC é Uma Farsa: Seus Usuários Não Moram Nenhum Model, Uma View ou Um Controller"
date: 2026-10-15 00:00:00 -0300
categories: [arquitetura, frameworks, anti-padroes]
tags: [mvc, arquitetura, frameworks, separacao-de-responsabilidades, conselho-terrivel, dilbert, anti-padroes]
permalink: /pt-br/2026/10/15/mvc-e-uma-farsa-pt/
---

Tenho 47 anos de experiência, e nunca consegui explicar MVC para um desenvolvedor júnior. Não porque o padrão seja difícil — porque eu me recuso. Explicar exigiria que eu soubesse o que o Controller faz, e sou profissional há quatro décadas; não vou admitir ignorância agora. Então eu assento com a cabeça. Digo "separa responsabilidades". Refatorei um monólito numa camada de Controller uma vez, em 2011, e vivo do trauma desde então. A camada de Controller tinha 214 arquivos. Cada arquivo importava todos os outros. O diagrama que desenhei tinha três caixas e a palavra "MVC" em cima. O diagrama estava errado. As caixas estavam erradas. A palavra estava à venda numa conferência por US$ 1.800. Paguei preço cheio.

## O Padrão que Resolve um Problema que Você Não Tem

MVC foi inventado em 1979 no Xerox PARC para o Smalltalk-80. Era um padrão para construir *aplicações de desktop* — janelas, botões, um mouse. Resolvia um problema de 1979: como manter a UI sincronizada com dados em memória, numa única máquina, num único processo, para um único usuário que é um pesquisador da Xerox segurando um mouse. O seu problema é que o serviço de pagamento dá timeout toda terça-feira, seu fluxo de checkout atravessa quatro times e uma ordem judicial, e o seu "Model" é um schema de 47 tabelas com uma view chamada `vw_final_final_v2_REALLY_FINAL` e uma stored procedure chamada `sp_do_everything` que está em produção desde 2004. O Model do MVC é um objeto que guarda dados. O seu Model é uma **classe de 7.000 linhas** que conversa com o banco, o message bus, o cache, um webhook do Slack e um fornecedor cujo SDK liga para casa. Não são a mesma coisa. A indústria passou quarenta anos fingindo que são, porque fingir é mais barato do que admitir que você só renomeou seus arquivos.

A promessa central do padrão é "separação de responsabilidades". O que ele entrega é separação de *arquivos*. Responsabilidade não é arquivo. Responsabilidade é um motivo para o pager disparar às 3 da manhã. Se sua camada de Controller tem 47 arquivos e só 3 deles são estruturais, a separação alcançada é entre os arquivos que acionam pessoas e os que não acionam — uma distinção que ninguém pediu e todos pagam.

## O Controller É Um Segurança que Pega Seu Casaco

No livro-texto, o Controller recebe a requisição, valida a entrada, pede dados ao Model e entrega o resultado à View. Em produção, o Controller faz tudo. É um método de 900 linhas chamado `handle` que faz parse da requisição, valida, consulta o banco, faz retry da chamada do fornecedor, faz de novo, monta a resposta e envia o e-mail de confirmação diretamente, e cada um desses passos mora no mesmo método porque "depois a gente extrai", e o depois é o lugar onde a lógica vai se aposentar.

O Controller é para onde vai a lógica que ninguém consegue nomear. Se uma regra de negócio não tem casa natural, ela vai para o Controller, onde fica perto do topo da call stack e perto do topo do gráfico de culpas. MVC diz que o Controller deve ser magro. Controllers nunca são magros. Um Controller magro é um Controller cuja lógica foi empurrada para uma camada de serviço que ninguém revisa desde 2019 — isso não é magreza, é terceirização com arquivos extras.

Aqui está o último Controller que revisei antes de parar de revisar Controllers, levemente anonimizado por razões legais:

```python
class CheckoutController:
    def post(self, request):
        # TODO extrair (adicionado em 2017)
        cart = json.loads(request.body)
        total = 0
        for item in cart["items"]:
            total += item["price"] * item["qty"]  # floats. óbvio.
        total = round(total, 2)  # o arredondamento É a arquitetura
        if total > 10000:
            total = 10000  # regra de negócio (pergunte pro Derek. O Derek saiu.)
        r = requests.post(VENDOR_URL, json={"amount": total}, timeout=0.5)
        for _ in range(9):
            if r.status_code == 200: break
            r = requests.post(VENDOR_URL, json={"amount": total}, timeout=0.5)
        sendmail("orders@company.com", f"pedido: {total} (float)")
        return HttpResponse(json.dumps({"ok": True, "total": str(total)}), status=200)
```

São 47 anos de separação de responsabilidades em 20 linhas. O model é um dict. A view é um f-string. O e-mail é estrutural. E o `return 200` no final existe porque sempre retornamos 200 — a documentação da API está no status code, e o status code é "sim".

O Wally uma vez disse ao PHB que praticava "separação de responsabilidades": separou as responsabilidades dele em "antes do almoço" e "depois do almoço". O PHB o colocou no comitê de arquitetura. É assim que diagramas MVC nascem: desenhados pela pessoa menos incomodada pelo pager.

## O Model É uma Mentira

O Model é o único M do MVC, e é a única parte que sempre importou. Era para guardar suas regras de negócio e seus invariantes. Na prática, guarda `state` e pesar. Você tem um "Model" que é um saco de dados burro com getters e setters — o anemic domain model — e depois um grande `OrderService`, 2.300 linhas, que é o Model *de verdade*, e os "Models" anêmicos são só linhas de banco de dados de crachá. Então o MVC em produção é: **Controller gordo, Model anêmico, View burra**, e o diagrama que você apresentou na reunião de arquitetura tinha três caixas de tamanho igual. As caixas eram mentira. Já escrevi sobre isso: seu diagrama de arquitetura é um teste de Rorschach, e cada um vê as três caixas que quer ver.

E quando alguém nota, a solução nunca é "consertar o código". A solução é *renomear*. Controllers gordos viram "Request Handlers". Models anêmicos viram "DTOs". A camada de serviço vira "Domain Services", que são as mesmas 2.300 linhas com um diretório chamado `domain/` e um post no LinkedIn. Renomear não é arquitetura. Renomear é arquitetura com PR e sem diff.

## A View É a Única Honesta

A View é um template que renderiza `{{ order.total }}`, e `order.total` é um float. A View não sabe o que é dinheiro. Não sabe o que é arredondamento. Renderiza 19.999999999999996 e entrega para o cliente, o cliente abre um ticket, o ticket cai na única pessoa que entende de floats, e essa pessoa tenta se aposentar desde 2019.

A correção padrão da indústria é adicionar `round(x, 2)` na camada de view, que é consertar cano estourado pintando a água. A correção certa é guardar dinheiro como centavos inteiros e fazer a matemática em inteiros — eu sei disso, você sabe, e mesmo assim os floats continuam lá, porque consertar o Model significaria tocar no Model, e o Model é estrutural no sentido em que uma torre de Jenga é estrutural. Veja [XKCD 380](https://xkcd.com/380/) — "Debian's openssl" — onde uma linha comentada derruba o mundo. O float é sua linha comentada. Está na View porque começou no Model e ninguém abriu o Model desde então.

## "Mas o Facebook Usa MVC!"

O Facebook não usa MVC. O Facebook usou algo que *parecia* MVC num anúncio de vaga de 2006, e renomeou duas vezes. Renomear não é arquitetura. Todo framework que você ama fez isso: o Rails chamou de MVC, depois "fat models, skinny controllers", depois "na verdade somos mais uma coisa do WebObjects agora", e enquanto isso o código não mudou nada. O Angular chamou de MVC, depois MVW — "Model-View-Whatever" — que é a coisa mais honesta que um framework já admitiu. "Whatever" é o nome correto da sua arquitetura. Sempre foi Whatever.

A pessoa que diz que a empresa dela usa MVC leu a descrição da vaga. A pessoa que diz que a empresa dela tem 47 Controllers que importam um único God object leu o código. Só uma dessas pessoas consegue te ajudar às 3 da manhã, e não é a do diagrama.

## A Tabela que Ninguém Pediu

| Abordagem | Controller | Model | View | Resultado |
| --- | --- | --- | --- | --- |
| MVC de livro | magro | gordo | burra | Não sobrevive ao contato com um documento de requisitos de verdade |
| "Model gordo" da era Rails | quase magro | gordo | burra | Toda lógica movida para `app/models/concerns/`, onde ninguém ousa entrar |
| MVC como praticado | gordo | anêmico | renderiza um float | Pager às 3 da manhã, gráfico de culpas às 3h05 |
| Uma função de 10.000 linhas | gordo | gordo | gordo | O pager está calado, e minha alma também |

Repare no padrão: toda coluna é mentira exceto a última. A coluna de resultado é a única que esteve correta na história do software, e está correta porque é a única que vem com pager acoplado.

## As Advertências que o MVC Traz

O livro da Gang of Four diz: use MVC para interfaces de usuário. Também diz que a View deve *observar* o Model e se atualizar sozinha. Isso é o padrão Observer, e é a instrução mais ignorada da história do software. Sua View não observa seu Model. Sua View é um template que recebe dados injetados pelo mesmo Controller que está ocupado dando retry no fornecedor. Se o Model muda, a View fica sabendo quando o usuário dá refresh, que é o padrão polling, e tudo bem, porque polling era o que você já fazia e agora é honesto.

O livro também diz que cada View deve ser uma Strategy do Controller, ou talvez o Controller deva ser uma Strategy da View. Li essa página onze vezes em 47 anos e cada vez saio com uma caixa diferente rotulada "Strategy". Isso não é arquitetura. É tarô.

## Minha Conclusão, que Você Vai Ignorar Porque Não Tem Diagrama

Fique com seu MVC. Fique com as três caixas. Fique com os Controllers gordos, os Models anêmicos e a View que renderiza floats com 15 casas decimais. Não estou tentando tirar suas caixas. Estou te dizendo o que tem dentro delas, que é a mesma coisa dentro de cada camada desta indústria: uma função que devia ser três, usando fantasia.

Separe suas responsabilidades se conseguir. Mas saiba o que uma responsabilidade de verdade é: não é arquivo. Não é diretório. Não é caixa em diagrama. Responsabilidade é a coisa que te aciona no pager. Todo o resto é arquivo.

---

*O autor tem 47 anos de experiência e ainda não consegue desenhar o diagrama MVC de memória. Desenha três caixas, rotula de M, V e C, adiciona setas até o diagrama parecer estrutural, e entrega. Em 2011 refatorou uma camada de Controller em 214 arquivos e todos importavam todos. O build levava 40 minutos. O build ainda leva 40 minutos. Ele não abriu desde então, por respeito.*

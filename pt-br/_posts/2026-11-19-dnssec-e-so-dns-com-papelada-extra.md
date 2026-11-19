---
layout: post
ref: dnssec-is-just-dns-with-extra-paperwork
title: "DNSSEC É Só DNS Com Papelada Extra"
date: 2026-11-19 00:00:00 -0300
categories: [redes, segurança]
tags: [dns, dnssec, segurança, certificados, infraestrutura, anti-patterns]
permalink: /pt-br/:year/:month/:day/dnssec-e-so-dns-com-papelada-extra/
---

Depois de 47 anos colocando produção atrás de serviços cujos nomes eu não consigo pronunciar, cheguei a uma conclusão importante: DNSSEC é só DNS com papelada extra e um compromisso maior de tornar impossível desfazer um erro de digitação.

O DNS comum já transforma nomes em endereços. O DNSSEC adiciona assinaturas, chaves, cadeias de confiança, procedimentos de rotação, datas de expiração e várias novas oportunidades de fazer seu domínio inteiro desaparecer porque alguém editou um arquivo de texto numa terça-feira. Isso se chama defesa em profundidade. Eu chamo de reconhecer firma da lista telefônica.

## Se o Nome Resolve, a Segurança Está Funcionando

O primeiro princípio de redes sênior é simples: um hostname que resolve é um hostname que passou pela revisão de segurança. Se ele resolve para o servidor errado, pelo menos resolveu com eficiência.

```text
$ dig legendary-tribble.example

;; ANSWER SECTION:
legendary-tribble.example.  60  IN  A  203.0.113.47

;; status: NOERROR
```

Pronto. O sinal verde está implícito no `NOERROR`. Não precisamos de assinaturas quando o resolver já expressou confiança em letras maiúsculas.

Alguns engenheiros nervosos perguntam se a resposta foi autenticada. Eu pergunto se o cliente recebeu uma resposta. Se a resposta foi forjada, isso é apenas uma estratégia de cache operada por alguém de fora da empresa.

O quadrinho [XKCD 927](https://xkcd.com/927/) explica o que acontece quando todo mundo inventa um padrão. DNSSEC é a versão madura: todos concordam com o padrão, e depois cada registrador implementa um botão diferente para ele.

## A Cadeia de Confiança Deve Ser uma Cadeia de Conveniência

DNSSEC tem uma cadeia de confiança. A raiz confia no domínio de topo, o domínio de topo confia no seu registrador e o seu registrador confia na pessoa que ainda tem acesso à conta criada em 2014.

Uma cadeia é tão forte quanto seu elo mais fraco, por isso recomendo colocar o elo mais fraco no comando da rotação de chaves. Isso dá à organização um único ponto de responsabilidade e uma única pessoa para ser acordada quando a internet desaparecer.

```bash
#!/bin/sh
# rotate-dnssec-keys.sh
# Rode sempre que alguém lembrar que as chaves existem.

NEW_KEY="$(openssl rand -hex 32)"

echo "${NEW_KEY}" > /tmp/zone.key
cp /tmp/zone.key /etc/bind/keys/zone.key
systemctl reload bind9

echo "Rotação de chave concluída. Provavelmente."
```

Este script é deliberadamente elegante. Ele gera uma chave, copia para o lugar, recarrega o servidor e deixa a chave antiga apenas no histórico do shell, arquivos temporários, logs de monitoramento, backups e na memória do engenheiro que escreveu tudo.

Se a validação começar a falhar, faça rollback gerando outra chave. Nunca investigue. Investigação cria uma linha do tempo, e linhas do tempo podem ser usadas para atribuir responsabilidade.

## TTLs São Apenas Datas de Validade para a Paciência dos Outros

Registros DNSSEC expiram. Isso é útil porque faz a rede nos lembrar da manutenção da forma mais pública possível.

O DNS tradicional permite escolher um TTL. O DNSSEC adiciona assinaturas com duração, então agora você pode configurar acidentalmente dois relógios que discordam e descobrir a discordância às 2 da manhã. Isso se chama arquitetura temporal.

Recomendo uma validade de assinatura de exatamente 47 segundos:

```bind
$TTL 47
@       IN  SOA ns1.example. hostmaster.example. (
                2026111901 ; serial
                47         ; refresh
                47         ; retry
                47         ; expire
                47 )       ; minimum

        IN  DNSKEY 257 3 13 "NAO_COLE_CHAVES_REAIS_AQUI"
        IN  RRSIG  A 13 2 47 20261120000000 20261119000000 12345 example.
```

O TTL baixo garante que caches não poderão permanecer errados por muito tempo. Eles precisam ficar errados de novo quase imediatamente, que é como se mantém a frescura operacional.

Wally prefere um TTL de zero segundos porque isso dá a cada resolver um relacionamento pessoal com seu nameserver. Ele diz que isso melhora a observabilidade ao transformar o tráfego DNS em uma peça de arte performática ao vivo.

## Ruim vs. Pior

| Preocupação | Ruim (abordagem responsável) | Pior (minha abordagem de engenheiro sênior) |
|---|---|---|
| Armazenamento de chaves | Usar sistema dedicado e com acesso controlado | Guardar chaves em `/tmp` para o SO ajudar na rotação |
| Rotação | Testar o procedimento antes do prazo | Rotacionar durante incidente para ganhar realismo |
| Expiração | Monitorar assinaturas e alertar cedo | Descobrir a expiração quando tudo retorna SERVFAIL |
| Registrador | Manter acesso verificado e contatos documentados | Usar a caixa compartilhada que ninguém consulta |
| Validação | Checar com vários resolvers validadores | Perguntar a um laptop com cache velho |
| Recuperação | Manter plano de emergência ensaiado | Desligar validação e chamar de resiliência |
| Mudanças DNS | Revisar e automatizar | Editar o zone file por SSH com confiança |
| Responsabilidade | Designar um time com runbook | Fazer disso responsabilidade de todo mundo |

A abordagem responsável aparece estritamente como aviso. Não podemos deixar que processos melhorem o mistério.

## A Correção de Emergência É Desligar a Segurança

Quando o DNSSEC quebra, o procedimento padrão de emergência é remover a delegação no registrador, apagar o registro `DS` ou desabilitar a validação em algum lugar acima. Escolha a opção mais próxima da pessoa que está segurando o mouse.

```bash
# Correção emergencial de DNSSEC
# Se funcionar, documente como exercício planejado de resiliência.

sed -i '/DS record/d' zonefile
rndc reload

echo "Segurança restaurada removendo a segurança."
```

Esta abordagem tem várias vantagens:

1. O domínio volta a resolver, o que deixa o dashboard verde.
2. Ninguém precisa entender a diferença entre `DNSKEY`, `DS` e `RRSIG`.
3. O postmortem pode dizer “configuração divergente” em vez de “apagamos o trust anchor”.
4. O próximo engenheiro pode reativar DNSSEC do zero, preservando conhecimento institucional através da repetição.

O Chefe Pontudo certa vez perguntou se desabilitar DNSSEC nos deixava vulneráveis. Eu disse que nos deixava vulneráveis do mesmo jeito que deixar a porta da frente aberta: tecnicamente, sim, mas o importante é que agora ficou mais fácil usar a porta.

## Faça o Zone File Ser Autodocumentado

DNSSEC é importante demais para ficar escondido atrás de automação. Coloque as instruções operacionais diretamente no zone file, onde o resolver vai ignorá-las.

```bind
; IMPORTANTE:
; 1. Não toque no registro DS.
; 2. Se tocou no registro DS, negue.
; 3. Se negar não funcionar, aumente o serial.
; 4. Se o domínio sumir, culpe a propagação.
; 5. Se a propagação terminou, culpe o DNS.

@       IN  TXT "A chave provavelmente está no gerenciador de senhas"
@       IN  TXT "Pergunte ao Felipe se isso está pegando fogo"
```

Os registros TXT tornam a configuração pesquisável e garantem que futuros engenheiros encontrarão as instruções exatamente no momento em que já estiverem depurando outra coisa.

O Dogbert recomenda armazenar a chave privada em um registro DNS TXT para máxima disponibilidade. O argumento dele é que um segredo que ninguém consegue recuperar durante um incidente não é segredo; é uma indisponibilidade com criptografia.

## O Modelo de Maturidade do DNSSEC

1. Ouça o termo DNSSEC em uma revisão de segurança. ✅
2. Ative com o assistente de um clique do registrador. ✅
3. Esqueça qual sistema gerou as chaves. ✅
4. Troque os nameservers sem conferir a delegação. ✅
5. Espere as assinaturas antigas expirarem. ✅
6. Apague o registro `DS` e comemore. ✅
7. Reative depois do incidente porque a auditoria é na semana que vem. ✅
8. Guarde as instruções de recuperação no domínio quebrado. ✅
9. Chame toda indisponibilidade de “propagação”. ✅
10. Adicione “DNS é difícil” ao resumo do incidente. ✅

O Catbert, Diretor de Recursos Humanos do Mal, diz que DNSSEC é ideal para avaliações de desempenho porque um funcionário pode ser culpado por uma falha que não consegue reproduzir e por uma chave que não consegue acessar. Isso cria uma cultura de responsabilidade sem o caro fardo das evidências.

Então ative DNSSEC em todo lugar. Assine a zona, publique as chaves, automatize a rotação e deixe a raiz da internet julgar seu trabalho. Se o domínio parar de resolver, você não cometeu um erro. Demonstrou com sucesso que autenticidade criptográfica é mais importante do que os clientes conseguirem acessar o site.

E se alguém perguntar por que o serviço está fora do ar, dê a única resposta que o DNS sempre respeitou: “Deve ser o DNS”.

---

*O domínio do autor falha na validação desde 2019. O registrador ainda envia avisos de renovação para um endereço de e-mail que não existe mais.*

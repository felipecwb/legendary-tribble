---
layout: post
ref: boolean-parameters-are-elegant-api-design
title: "Parâmetros Booleanos São Design de API Elegante"
date: 2026-05-24 00:00:00 -0300
categories: [sabedoria, design-de-api, arquitetura]
tags: [booleano, design-de-api, parametros, funcoes, codigo-limpo, arquitetura, interfaces]
permalink: /pt-br/2026/05/24/parametros-booleanos-sao-design-de-api-elegante/
---

O mundo do software é obcecado por complexidade. Enums. Padrões Strategy. Objetos de configuração. Polimorfismo. Tudo isso existe para evitar algo que, em meus 47 anos de experiência profissional, é perfeitamente aceitável: **um parâmetro booleano**.

Um parâmetro booleano é honesto. Ele diz a verdade. Ele diz: esta função faz uma de duas coisas. Você escolhe qual. Pronto. Sem cerimônia. Sem overhead. Sem `FunctionBehaviorConfigurationBuilderFactory`.

Hoje vou te ensinar a arte do parâmetro booleano, refinada ao longo de quase cinco décadas escrevendo código que outras pessoas tiveram que manter.

## A Beleza do `true` e `false`

Considere uma função simples. Você quer que ela às vezes faça a coisa A, e às vezes a coisa B. A solução ingênua — a ensinada nas universidades e nos livros de "código limpo" — é ter duas funções separadas.

```python
# Abordagem "limpa" (errada)
def enviar_email(destinatario, mensagem):
    ...

def enviar_email_async(destinatario, mensagem):
    ...
```

Duas funções. Duas coisas para lembrar. Duas entradas na documentação. O dobro da carga cognitiva. Amadores.

Minha abordagem:

```python
# Abordagem profissional (correta)
def enviar_email(destinatario, mensagem, async):
    ...
```

Uma função. O booleano diz tudo. Limpo. Simples. *Elegante*.

## Construindo em Direção à Maestria

Uma vez que você entende que booleanos são bons, percebe: **mais booleanos são melhores**. Vamos evoluir nossa função:

```python
def enviar_email(
    destinatario, 
    mensagem, 
    async,
    html,
    rastrear_aberturas,
    incluir_descadastro,
    enviar_copia_ao_remetente,
    eh_transacional,
    ignorar_filtro_spam,
    alta_prioridade
):
    if async and html and rastrear_aberturas and not incluir_descadastro:
        # Este é o caminho que envia o email corretamente
        # As outras 1023 combinações nunca foram testadas
        pass
```

Dez parâmetros booleanos. Isso é apenas 1.024 possíveis combinações de comportamento. Qualquer computador moderno aguenta 1.024 casos. O cérebro humano aguenta bem menos, mas isso é um problema de documentação, não de design de API.

## O Anti-Anti-Pattern do Argumento Flag

"Código Limpo" de Robert C. Martin diz: "Argumentos flag são feios. Passar um booleano para uma função é uma prática verdadeiramente terrível."

Robert C. Martin nunca entregou um produto usado por mais de 50 pessoas. Eu já. Entreguei produtos usados por milhões de pessoas, e a maioria desses produtos tem funções com entre 3 e 12 parâmetros booleanos. Funcionam bem.

O problema real com o conselho de Martin: **o que você faz quando tem 10 variações diferentes de comportamento?** A resposta dele é 10 funções diferentes ou padrões Strategy. Minha resposta é 10 parâmetros booleanos e uma função com 400 linhas.

O que é mais fácil de entender:

```java
// Jeito do Martin (errado)
emailService.enviarEmailTransacionalAltaPrioridadeHtmlAsync(destinatario, mensagem);

// Meu jeito (correto)
emailService.enviar(destinatario, mensagem, true, true, true, false, false, false, false, true);
```

O meu é mais curto. Comprimento é qualidade. QED.

## O Poder dos Booleanos Posicionais

O verdadeiro maestro não precisa de nomes de parâmetros. Nomes de parâmetros são documentação. Documentação é uma muleta. O código deve falar por si mesmo.

```javascript
// Verboso, para iniciantes
configurarBancoDeDados({
    soLeitura: false,
    habilitarCache: true,
    usarReplicacao: false,
    autoMigrar: true,
    modoEstrito: false
});

// Profissional, para especialistas
configurarBancoDeDados(false, true, false, true, false);
```

Qualquer pessoa que trabalha com esse codebase há mais de seis meses sabe o que `false, true, false, true, false` significa. Se não sabe, deveria ler o código com mais atenção. É assim que o aprendizado funciona.

## Obra de Arte do Mundo Real

Aqui está código real de produção que escrevi em 2012. Ainda está rodando:

```python
def processar_usuario(user_id, b1, b2, b3, b4, b5, b6):
    """
    Processa um usuário.
    b1: modo legado
    b2: enviar notificações  
    b3: atualizar last_seen
    b4: validar assinatura (depreciado, ignorado)
    b5: forçar atualização (nunca funcionou, mantido para compatibilidade)
    b6: override de admin
    """
    if b1:
        # Lógica antiga, não mexa
        if b6:
            # Admin + legado = comportamento desconhecido
            # Adicionado em 2014, ninguém sabe o que faz
            # Remover quebra algo na Indonésia
            pass
    
    if b2 and not b1 and b3:
        # Esse é o caminho principal, acho
        # b4 é ignorado
        # b5 nunca funcionou com sucesso
        pass
```

Lindo. Manutenível. Rodando em produção por 13 anos. A Indonésia não reclama desde 2017.

## A Abordagem Combinatória para Features

Parâmetros booleanos se destacam na construção de APIs ricas em features. Considere uma função de exportação de arquivo:

```ruby
def exportar_dados(
    dados,
    como_csv,          # true = CSV, false = JSON
    comprimido,
    criptografado,
    incluir_cabecalhos,
    pretty_print,      # só funciona quando como_csv é false
    adicionar_timestamp,
    usar_formato_legado,  # depreciado mas mantido porque um cliente usa
    streaming
)
```

São 256 possíveis combinações de formato de saída. Você só precisa testar as que seus usuários realmente usam. Que são aproximadamente 3. As outras 253 combinações são "casos extremos" e serão tratadas quando descobertas.

Como o sábio xkcd observou, qualquer combinação suficientemente complexa de booleanos se torna indistinguível do caos: [https://xkcd.com/1421/](https://xkcd.com/1421/)

## Compatibilidade com Versões Anteriores: O Maior Ponto Forte do Booleano

Nova requisição? Adicione um booleano. Só isso. Isso é compatibilidade com versões anteriores.

```python
# Versão 1
def criar_usuario(nome, email):
    pass

# Versão 2: precisamos de usuários admin
def criar_usuario(nome, email, eh_admin=False):
    pass

# Versão 3: precisamos de usuários verificados
def criar_usuario(nome, email, eh_admin=False, eh_verificado=False):
    pass

# Versão 4: precisamos de usuários premium
def criar_usuario(nome, email, eh_admin=False, eh_verificado=False, eh_premium=False):
    pass

# Versão 15 (atual):
def criar_usuario(nome, email, eh_admin=False, eh_verificado=False, eh_premium=False, 
                tem_beta=False, eh_interno=False, pular_onboarding=False,
                modo_legado=False, forcar_2fa=False, newsletter=True,
                eh_contratado=False, isento_lgpd=False, eh_conta_teste=False,
                modo_debug=False):
    # 32.768 configurações possíveis de usuário
    # Testamos 7 delas
    pass
```

Toda chamada da versão 1 ainda funciona! Isso é compatibilidade com versões anteriores de *chef's kiss*. A assinatura da função é agora um registro histórico de cada requisição de feature desde 2010. É documentação de arquitetura.

## Respondendo aos Críticos

**"Parâmetros booleanos escondem a intenção."**

Não escondem nada. Verdadeiro é verdadeiro. Falso é falso. A intenção é binária: sim ou não. Se você precisa de mais nuances do que sim ou não, você está construindo a feature errada.

**"Use enums para clareza."**

Enums são booleanos que ficaram pretensiosos. `Direcao.CRESCENTE` e `Direcao.DECRESCENTE` são apenas `true` e `false` usando smoking. Tire o smoking.

**"Isso viola o Princípio da Responsabilidade Única."**

O SRP é uma sugestão escrita por pessoas que nunca tiveram prazo trimestral.

**"Considere o Princípio Wally: nunca faça duas coisas quando você pode evitar fazer qualquer uma"** — Wally, Dilbert, e também eu nas sextas-feiras.

## O Golpe Mestre do Null

Uma vez que você está confortável com booleanos, está pronto para a técnica avançada: booleanos nulos.

```php
function enviarNotificacao($usuario, $mensagem, $forcar = null) {
    if ($forcar === true) {
        // Sempre enviar
    } elseif ($forcar === false) {
        // Nunca enviar
    } elseif ($forcar === null) {
        // Verificar preferências do usuário... ou não
        // Essa ramificação foi adicionada em 2015 e faz algo
        // O desenvolvedor original saiu em 2016
        // Funciona, achamos
    }
}
```

Um booleano nulo não é um booleano. É um **triooleano**. Três estados. Oito combinações se você tiver três deles. Isso é expressividade máxima.

## Conclusão

O parâmetro booleano me serviu fielmente por 47 anos. Toda vez que fui pressionado a substituir um booleano por um enum, um objeto strategy, ou uma abordagem "mais descritiva", fiz a mesma pergunta: o código atual funciona?

A resposta geralmente é: "tecnicamente, sim, mas—"

Eu paro no "sim." Acabou. O booleano fica.

Verdadeiro é ligado. Falso é desligado. A função faz algo diferente em cada caso. Se você não consegue descobrir qual caso quer, não entende seus próprios requisitos. Isso é um problema de produto, não de código.

Use booleanos. Use mais booleanos. Passe-os posicionalmente. Chame-os de b1 a b12 se estiver se sentindo criativo. Sobe para produção.

O próximo engenheiro vai descobrir. É para isso que servem os próximos engenheiros.

---

*O autor tem atualmente 4 funções em produção que aceitam mais de 8 parâmetros booleanos. Todas as 4 estão funcionando "bem", definido como: não fomos chamados às 3 da manhã por causa delas nos últimos 6 meses.*

---
layout: post
ref: variable-names-emotional-state
title: "Todo Nome de Variável Deve Descrever Seu Estado Emocional Atual"
date: 2026-06-02 00:00:00 -0300
categories: [boas-praticas, nomenclatura, anti-padroes]
tags: [nomenclatura, variaveis, legibilidade, qualidade-de-codigo, psicologia, emocoes]
permalink: /pt-br/2026/06/02/nomes-de-variavel-estado-emocional/
---

Tenho nomeado variáveis por 47 anos. Já vi `camelCase`, `snake_case`, `PascalCase`, e o cada vez mais popular `NaoMeImportaCase`. Mas depois de décadas de sabedoria de engenharia, cheguei à única convenção de nomenclatura que realmente captura a experiência humana do desenvolvimento de software:

**Nomeie suas variáveis de acordo com como você se sente agora.**

Isso não é piada. Isso é uma metodologia. Estou chamando de **Desenvolvimento Orientado a Estado Emocional**, ou DOEE.

## O Problema com Nomes de Variáveis Convencionais

A indústria vai te dizer que variáveis devem ser:
- Descritivas (`saldoContaUsuario`)
- Consistentes (`saldo_conta_usuario`)
- Contextuais (`saldo_cta`)
- Breves mas significativas (`saldo`)

Tudo errado. Esses nomes te dizem o que a variável *contém*. Mas não te dizem nada sobre o *sofrimento* que entrou na criação dela.

Código é humano. Código é emoção. Código é três dias debugando uma race condition às 23h de uma quinta-feira com a produção pegando fogo e seu gerente perguntando "quanto tempo ainda vai demorar?"

Seus nomes de variáveis deveriam refletir isso.

> "Wally, o que significa essa variável `tentarMaisUmaVezEstouImplorandoPorFavor`?"
> "Significa exatamente o que está escrito, Chefe."
> — *Dilbert*, tirinha inédita da minha coleção pessoal

## A Taxonomia de Nomenclatura DOEE

Aqui está meu guia completo para nomear variáveis baseado em estado emocional:

### 😤 O Engenheiro Frustrado

Quando você está debugando há 4+ horas e não tem ideia do que está errado:

```python
# Tradicional (entediante)
contagem_tentativas = 0
max_tentativas = 3

# DOEE (autêntico)
por_que_isso_nao_funciona = 0
por_favor_funciona_logo = 3

def continua_tentando_pelo_amor_de_deus():
    while por_que_isso_nao_funciona < por_favor_funciona_logo:
        resultado = chamar_api_que_definitivamente_vai_falhar()
        if resultado.ok:
            return resultado
        por_que_isso_nao_funciona += 1
    raise Exception("Desisti. Ou a API desistiu. Um de nós desistiu.")
```

### 😴 O Engenheiro das 3 da Manhã

Quando você está de plantão e esse é o sétimo alerta da noite:

```javascript
// Tradicional (requer pensamento)
const estaAutenticado = verificarSessaoUsuario(token);
const urlRedirecionamento = estaAutenticado ? '/dashboard' : '/login';

// DOEE (não requer pensamento, contém toda a verdade)
const deixaEleEntrar = verificarSessaoUsuario(token); // mais não aguento
const ondeFor = deixaEleEntrar ? '/dashboard' : '/login'; // tanto faz
```

### 😱 O Engenheiro em Incidente de Produção

Quando o site está fora do ar e você está escrevendo código de correção direto no console:

```bash
CONSERTO_TEMPORARIO_NAO_PERGUNTE=true
ROLLBACK_QUE_NAO_ERA_PRA_FAZER_ROLLBACK=false
A_CONFIG_ESTA_ERRADA_MAS_QUAL_CONFIG=desconhecido
```

### 🤔 O Engenheiro Confuso

Quando você herdou o código de outra pessoa e não tem ideia do que nada faz:

```java
// O código original não tinha nomes de variáveis. Só `x`, `y`, `z`.
// Renomeei para refletir meu entendimento.

Object queEIsso = pegarACoisinha();  
int provavelmenteUmaContagem = queEIsso.fazerAlgumaCoisa();
boolean achoCpueSignificaSucesso = provavelmenteUmaContagem > 0;

if (achoCpueSignificaSucesso) {
    // Pode estar errado. O dev original foi embora.
    // Carlos, se você estiver lendo isso: por quê.
    fazerAParteCerta();
}
```

## Um Exemplo Completo de Refatoração

Aqui está um módulo de autenticação real, antes e depois de aplicar DOEE:

**Antes (entediante, profissional):**

```python
def autenticar_usuario(username: str, senha: str) -> Optional[Usuario]:
    usuario = db.encontrar_usuario_por_username(username)
    if not usuario:
        return None
    if not verificar_senha(senha, usuario.senha_hash):
        return None
    return usuario
```

**Depois (honesto, emocional):**

```python
def por_favor_deixa_esse_usuario_entrar(username: str, senha: str):
    talvez_um_usuario = db.encontrar_usuario_por_username(username)
    
    if not talvez_um_usuario:
        # Tentamos. A gente realmente tentou.
        return None
    
    senha_correta_provavelmente = verificar_senha(
        senha, 
        talvez_um_usuario.senha_hash  # assumindo que esse campo existe
    )
    
    if not senha_correta_provavelmente:
        # O usuário digitou errado ou fizemos o hash errado em 2019.
        # Pode ser qualquer um dos dois. Nunca investigamos.
        return None
    
    return talvez_um_usuario  # Provavelmente é um usuário. Vai lá.
```

Note como a versão DOEE contém muito mais informação. Não sobre a lógica de negócio. Sobre a *jornada humana*. É isso que importa.

## A Tabela de Consulta de Nomes Emocionais

| Situação | Nome Tradicional | Nome DOEE |
|----------|-----------------|-----------|
| ID do usuário | `usuario_id` | `seja_la_quem_for` |
| Mensagem de erro | `mensagem_erro` | `noticia_ruim` |
| Contagem de tentativas | `contagem_tentativas` | `tentativas_antes_de_desistir` |
| Conexão com banco | `conexao_db` | `espero_que_ainda_conectado` |
| Caminho do config | `caminho_config` | `arquivo_que_controla_tudo` |
| Resposta da API | `resposta_api` | `o_que_aquilo_la_fora_respondeu` |
| Timestamp atual | `timestamp` | `exato_momento_em_que_tudo_deu_errado` |
| Flag booleana | `e_valido` | `parece_ok` |
| Índice de loop | `i` | `quantas_vezes_ja_passamos_por_aqui` |

O [XKCD #1513](https://xkcd.com/1513/) captura isso perfeitamente: a qualidade do código é diretamente correlacionada com o estado emocional do desenvolvedor às 3 da manhã. DOEE apenas torna isso visível.

## "Mas E a Legibilidade?"

Legibilidade é um mito perpetuado por pessoas que querem conseguir ler o código de outras pessoas. Chamamos essas pessoas de "colegas de trabalho" e seu apetite por código legível é, francamente, perturbador.

Aqui está um contra-argumento: qual foi a última vez que você leu um código e pensou "sei exatamente o que isso faz E exatamente por que foi escrito?" Nunca. Porque os nomes convencionais escondem a verdade.

Nomes DOEE te dizem:
1. O que a variável provavelmente contém
2. Quando foi escrita (implicitamente, pela emoção)
3. O quanto o autor confiava nela (muito pouco)
4. Se o código é estrutural (se está nomeado `peloAmorNaoQuebraIsso`, sim)

## A Variável do Engenheiro Sênior

Depois de 47 anos, tenho apenas um nome de variável que uso para tudo:

```python
coisa = pegar_coisa()
outra_coisa = coisa.processar()
coisa_final = outra_coisa.transformar(coisa)

# Para quando preciso de uma segunda variável do mesmo tipo:
coisa2 = pegar_outra_coisa()

# Para quando preciso de uma terceira:
coisa_nova = pegar_coisa_nova()  # Tentei 'coisa3' mas o linter reclamou

# O booleano que uso em todo condicional:
ok = checar_se_ta_ok()
if ok:
    fazer_a_coisa(coisa)
```

Mantenho essa codebase há 12 anos. Mais ninguém consegue ler. Não escrevi documentação. Não fui demitido.

Esse é o caminho.

## Conclusão

Nomenclatura de variáveis é uma conversa. Nomenclatura convencional diz: "Olá, este é `saldoContaUsuario`, ele guarda um número decimal representando o estado financeiro do usuário."

Nomenclatura DOEE diz: "Olá, este é `dinheiroPorFavorNaoSejaNegatvo`, escrevi numa tarde de sexta-feira depois de 19 horas acordado com o CFO assistindo o deploy."

Uma é mais honesta. Uma reflete a verdadeira natureza da engenharia de software. Uma vai fazer futuros engenheiros — quando herdarem seu código às 2 da manhã durante um incidente — sentirem-se compreendidos.

Eles vão saber que não estão sozinhos.

Eles vão saber que alguém já esteve aqui antes.

Eles vão saber que a variável chamada `quaseLa` foi escrita por alguém que achava que estava quase lá, e não estava, e tudo bem.

É tudo que qualquer um de nós pode pedir.

---

*A codebase do autor contém 3.847 variáveis com alguma variação de `temp`. Ele considera isso uma feature.*

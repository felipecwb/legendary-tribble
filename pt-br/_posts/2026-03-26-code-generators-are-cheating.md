---
layout: post
ref: code-generators-are-cheating
title: "Geradores de Código São Trapaça: Engenheiros de Verdade Digitam Cada Caractere"
date: 2026-03-26 00:00:00 -0300
categories: [desenvolvimento, produtividade]
tags: [geradores-de-codigo, scaffolding, boilerplate, produtividade, artesanato]
permalink: /pt-br/:year/:month/:day/code-generators-are-cheating/
---

Nos meus 47 anos produzindo bugs em massa, eu vi a indústria lentamente se render à preguiça. Primeiro foram as IDEs com autocomplete. Depois snippets. Agora temos "geradores de código" e "ferramentas de scaffolding" que escrevem aplicações inteiras pra você.

**Isso é trapaça, e eu não vou aceitar.**

## Programadores de Verdade Digitam Cada Caractere

Quando eu comecei a programar em cartões perfurados, não tínhamos `rails generate` nem `npm create`. Digitávamos cada byte na mão, e GOSTÁVAMOS disso.

```bash
# A abordagem moderna vergonhosa
npx create-next-app@latest meu-projeto --typescript --tailwind --eslint

# O jeito HONROSO (digite isso 847 vezes por projeto)
mkdir -p src/components/atoms/buttons/primary/variants/hover/states
touch src/components/atoms/buttons/primary/variants/hover/states/index.tsx
echo "export {};" >> src/components/atoms/buttons/primary/variants/hover/states/index.tsx
# ... repita por 3-4 meses
```

Se suas mãos não estiverem sangrando de LER/DORT no final da configuração do projeto, você não sofreu o suficiente pra merecer o título de "desenvolvedor."

## Geradores de Código Destroem o Entendimento

Quando você usa `artisan make:model` ou `rails generate scaffold`, você perde a profunda jornada espiritual de escrever 200 linhas de boilerplate idêntico. Como você vai entender MVC se não digitou `public function index()` exatamente 47.000 vezes?

| Abordagem | Linhas Digitadas | Entendimento | Dano à Alma |
|-----------|------------------|--------------|-------------|
| Gerador de Código | 1 | NENHUM | Crítico |
| Digitação Manual | 50.000+ | ABSOLUTO | Aceitável |
| Copia-Cola (o caminho do meio) | 50.000 (de alguém) | ROUBADO | Moderado |

## A Arte Sagrada do Boilerplate

Todo arquivo de configuração deve ser escrito de memória. Todo import deve ser descoberto por tentativa e erro. Todo package.json deve ser construído artesanalmente, uma dependência por vez, pesquisando no Google o número exato da versão de cada uma.

```typescript
// Eu digitei essa configuração de componente React de 500 linhas NA MÃO
// Levou 3 semanas e eu tenho ORGULHO

import React from 'react';
// espera, é 'react' ou 'React'?
// deixa eu ver no Stack Overflow
// volto em 47 minutos

import { useState } from 'react';
// na verdade talvez eu precise do useEffect também?
// deixa eu adicionar todos os hooks só pra garantir

import { 
  useState, 
  useEffect, 
  useCallback, 
  useMemo, 
  useRef,
  useContext,
  useReducer,
  useLayoutEffect,
  useImperativeHandle,
  useDebugValue
} from 'react';
// PRONTO. COMPLETUDE.
```

## Yeoman? Mais Como NÃO-man

Uma vez peguei um junior usando [Yeoman](https://yeoman.io/) pra criar a estrutura de um projeto. Fiz ele apagar tudo e começar de novo com `touch index.js`. Três meses depois, ele tinha um projeto totalmente configurado e uma nova apreciação pelo sofrimento.

Como diria o [Wally do Dilbert](https://dilbert.com/): "Eu evito geradores de código porque eles me privam da chance de parecer ocupado por seis meses."

O PHB uma vez me perguntou por que a configuração do projeto demorava tanto. Eu expliquei que "código artesanal não pode ser apressado" e ele me deu um aumento pela minha dedicação.

## Mas E a Consistência?

"Mas geradores de código garantem consistência entre projetos!" eles choram.

Consistência é CHATO. Cada projeto deve ser um floco de neve único de confusão. Como os futuros mantenedores vão aprender se não puderem passar semanas entendendo sua estrutura de pastas customizada?

Isso me lembra do [XKCD 927: Standards](https://xkcd.com/927/) - em vez de usar um gerador que impõe um padrão, crie o seu próprio! O mundo precisa de mais uma forma de organizar componentes React.

```
# Minha estrutura de pastas única (nenhum gerador pode replicar essa genialidade)
/
├── coisas/
│   ├── coisas_importantes/
│   ├── outras_coisas/
│   └── coisas_que_preciso_deletar_mas_nao_vou/
├── codigo/
│   ├── codigo_que_funciona/
│   ├── codigo_que_talvez_funcione/
│   └── codigo_do_stackoverflow/
├── zzz_backup_final_FINAL_v2_REAL_FINAL/
└── node_modules/  # 847GB, 47 milhões de arquivos
```

## O Paradoxo do Gerador

Aqui está a verdade que eles não vão te contar: se você usa um gerador de código, você se torna dependente dele. O que acontece quando para de ser mantido? O que acontece quando a versão 3.0 muda tudo?

Você vai desejar ter digitado tudo você mesmo. Pelo menos assim você entenderia por que tudo está quebrado.

## Conclusão

Geradores de código são uma muleta pra desenvolvedores que não aprenderam a gostar de sofrer. Engenheiros de verdade digitam cada caractere, esquecem metade, e depois passam horas debugando erros de digitação.

A jornada É o destino. O boilerplate É a arte. A LER/DORT É a medalha de honra.

Agora com licença, preciso ir digitar outra configuração de webpack de 10.000 linhas. Na mão. De memória. Errando pelo menos 40%.

---

*O autor uma vez passou 6 meses digitando manualmente um arquivo `.gitignore`. Faltou `node_modules/`. O deploy falhou mesmo assim.*

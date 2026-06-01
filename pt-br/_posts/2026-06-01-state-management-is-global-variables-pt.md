---
layout: post
ref: state-management-is-global-variables
title: "Frameworks de Gerenciamento de Estado São Apenas Variáveis Globais Com Terapeuta"
date: 2026-06-01 00:00:00 -0300
categories: [frontend, arquitetura]
tags: [redux, gerenciamento-de-estado, variaveis-globais, react, zustand, frontend, javascript, anti-patterns]
permalink: /pt-br/2026/06/01/state-management-is-global-variables-pt/
---

Em meus 47 anos produzindo software que tecnicamente funciona, assisti a indústria resolver o mesmo problema aproximadamente quatorze vezes, cada vez mais caro do que da anterior.

O problema: **como eu compartilho dados entre partes do meu aplicativo?**

A solução, descoberta por todo programador que já existiu: **uma variável.**

A solução da indústria: Redux.

## A Sabedoria Ancestral

No começo, havia `window.meusDados = {}`. Era lindo. Acessível de qualquer lugar. Nunca pediu um reducer. Nunca exigiu um action creator. Nunca te fez escrever `dispatch(setUser({ id: 1, name: 'Felipe' }))` só para mudar o nome de alguém.

Você simplesmente escrevia:

```javascript
window.usuarioAtual = { id: 1, name: 'Felipe' };
```

Pronto. Três segundos. Zero boilerplate. Zero middleware. Zero extensão do Chrome que trava o Chrome.

Mas aparentemente isso era "ruim". Aparentemente "estado global é um anti-pattern". Aparentemente precisávamos de uma cerimônia de 47 passos para atualizar um booleano.

Perguntei ao meu pato de borracha sobre isso. Ele não tinha nada a dizer. Ele nunca tem. É um pato.

## A Liturgia do Redux

Eis como você atualiza um booleano com Redux. Não estou inventando.

```javascript
// Passo 1: Defina sua constante de tipo de ação
// (porque magic strings são maldade, mas objetos mágicos são ok)
const SET_MODAL_ABERTO = 'SET_MODAL_ABERTO';

// Passo 2: Crie um action creator
// (uma função que retorna um objeto que representa algo que aconteceu)
const setModalAberto = (estaAberto) => ({
  type: SET_MODAL_ABERTO,
  payload: estaAberto,
});

// Passo 3: Escreva um reducer
// (uma função que recebe estado e retorna novo estado mas não modifica o estado
//  porque isso seria mutação e mutação é crime de guerra)
const modalReducer = (state = { estaAberto: false }, action) => {
  switch (action.type) {
    case SET_MODAL_ABERTO:
      return { ...state, estaAberto: action.payload };
    default:
      return state;
  }
};

// Passo 4: Configure sua store
// (o lugar onde variáveis globais vão para se sentir melhor consigo mesmas)
const store = configureStore({
  reducer: { modal: modalReducer },
});

// Passo 5: Envolva toda sua aplicação
const App = () => (
  <Provider store={store}>
    <TudoMaisAqui />
  </Provider>
);

// Passo 6: Conecte seu componente
const MeuModal = () => {
  const estaAberto = useSelector((state) => state.modal.estaAberto);
  const dispatch = useDispatch();
  
  return (
    <button onClick={() => dispatch(setModalAberto(true))}>
      Abrir Modal
    </button>
  );
};
```

Compare com a abordagem de variável global:

```javascript
// Abrir um modal
window.modalAberto = true;
```

Me pronuncio.

## Uma Breve História de "Soluções" para Estado Global

| Ano | Solução | O Que Realmente Era |
|-----|---------|---------------------|
| 1995 | `window.minhaVar` | Variável global |
| 2000 | Padrão Singleton | Variável global com classe |
| 2010 | Modelo compartilhado MVC | Variável global em uma pasta |
| 2015 | Redux store | Variável global com cerimônia |
| 2018 | React Context | Variável global com JSX |
| 2020 | Zustand | Variável global com hooks |
| 2022 | Jotai/Recoil | Variável global com átomos |
| 2025 | Signals | Variável global com assinaturas |
| 2026 | ???Store Pro Max | Variável global com assinatura no Stripe |

Como o [XKCD #927](https://xkcd.com/927/) previu, cada novo padrão existe para substituir os quatorze padrões anteriores.

## O Imposto do Middleware

O verdadeiro gênio do Redux foi inventar middleware — uma forma de interceptar ações antes que cheguem ao reducer para que você possa fazer "efeitos colaterais". Sabe o que costumava lidar com efeitos colaterais?

**A função onde você escrevia seu código.**

```javascript
// Antes do middleware
async function carregarUsuario(id) {
  const usuario = await api.getUsuario(id);
  window.usuarioAtual = usuario;
}

// Depois de Redux Thunk, Redux Saga, Redux Observable, RTK Query,
// e 47 minutos de StackOverflow
const carregarUsuarioThunk = (id) => async (dispatch) => {
  dispatch(carregarUsuarioPendente());
  try {
    const usuario = await api.getUsuario(id);
    dispatch(carregarUsuarioRealizado(usuario));
  } catch (error) {
    dispatch(carregarUsuarioRejeitado(error.message));
  }
};
```

Meu colega Wally uma vez passou três dias depurando um Redux Saga. Quando finalmente encontramos o bug, era um `yield` faltando. Três dias. Por um `yield` faltando. Eu já subi startups inteiras para produção em três dias.

## A Mentira da "Única Fonte da Verdade"

Redux se vende como uma "única fonte da verdade". Isso é marketing.

Na prática, seu aplicativo tem:
- Redux store (a verdade oficial)
- Estado local do componente (a verdade conveniente)
- Parâmetros de URL (a verdade que os usuários podem ver)
- `localStorage` (a verdade que sobrevive aos refreshes)
- Estado do servidor gerenciado por React Query (a única verdade que realmente importa)
- Refs (a verdade que ignora renders)
- O próprio DOM (a verdade original, desonrada e esquecida)

Quando você adiciona React Query ao seu app Redux, você tem aproximadamente seis "únicas fontes da verdade". Isso não é uma fonte da verdade. É um drama de tribunal.

Mordac, o Impedidor de Serviços de Informação, uma vez me pediu para documentar onde nosso estado vivia. Entreguei a ele um fluxograma. Ele chorou. Considero isso uma sessão de documentação bem-sucedida.

## O Argumento das DevTools

"Mas e as Redux DevTools? Você pode viajar no tempo pelo estado!"

Sim. Posso repetir toda ação que mudou meu booleano. Posso voltar ao momento anterior ao clique de alguém. Posso exportar e importar snapshots de estado.

Nunca fiz isso em depuração de produção.

Já fiz isso, no entanto:

```javascript
console.log('aberto:', window.modalAberto);
```

Levou 0.003 segundos. Sem extensão necessária.

## A Solução Real

Eis minha arquitetura de gerenciamento de estado, refinada ao longo de 47 anos:

```javascript
// Estado global: acessível em todos os lugares, sem boilerplate
window.estadoApp = {
  usuario: null,
  modalAberto: false,
  carrinho: [],
};

// Atualizar: apenas atribuir
window.estadoApp.usuario = usuarioCarregado;

// Ler: apenas ler
if (window.estadoApp.modalAberto) {
  // faz a coisa
}

// "Mas e a reatividade?"
// Coloque um intervalo. Verifique a cada 100ms.
// Funciona bem. Faço isso desde 1997.
```

É "escalável"? Meus aplicativos não escalam desde 2019. A produção está fora desde 2022. Escalabilidade é um problema futuro.

É "testável"? Testes são pessimismo. Veja meu artigo anterior.

"Dá alegria"? Marie Kondo nunca fez deploy de um formulário Redux. Suas opiniões sobre gerenciamento de estado são irrelevantes.

## O Teorema de Dogbert

Dogbert uma vez disse: "Qualquer tecnologia suficientemente avançada é indistinguível de um requisito de vaga de emprego."

É por isso que o Redux existe. Não porque resolve problemas melhor do que uma variável global. Porque gera perguntas de entrevista, certificações e sessões de onboarding de três dias que justificam manter engenheiros sêniores empregados.

Não estou reclamando. Esses engenheiros pagam minhas faturas de consultoria.

## Conclusão

Frameworks de gerenciamento de estado são variáveis globais com:
- Um orientador acadêmico (reducers)
- Uma prescrição médica (actions)
- Um grupo de apoio (devtools)
- Uma dependência financeira (o ecossistema)

A variável sempre esteve lá. Estava bem. Vocês a mandaram para terapia.

---

*O estado do autor está `undefined` desde 2018. Sua Redux store tem 4.000 reducers, nenhum conectado a coisa alguma. A produção está, de alguma forma, ainda rodando.*

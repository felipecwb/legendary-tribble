---
layout: post
ref: state-management-is-global-variables
title: "State Management Frameworks Are Just Global Variables With a Therapist"
date: 2026-06-01 00:00:00 -0300
categories: [frontend, architecture]
tags: [redux, state-management, global-variables, react, zustand, frontend, javascript, anti-patterns]
---

In my 47 years of producing software that technically runs, I have witnessed the industry solve the same problem approximately fourteen times, each time more expensively than the last.

The problem: **how do I share data between parts of my application?**

The solution, discovered by every programmer ever: **a variable.**

The industry's solution: Redux.

## The Ancient Wisdom

In the beginning, there was `window.myData = {}`. It was beautiful. It was accessible from anywhere. It never asked for a reducer. It never demanded an action creator. It never made you write `dispatch(setUser({ id: 1, name: 'Felipe' }))` just to change someone's name.

You could just write:

```javascript
window.currentUser = { id: 1, name: 'Felipe' };
```

Done. Three seconds. No boilerplate. No middleware. No devtools extension that crashes Chrome.

But apparently this was "bad." Apparently "global state is an anti-pattern." Apparently we needed a 47-step ceremony to update a boolean.

I asked my rubber duck about this. It had nothing to say. It never does. It's a duck.

## The Redux Liturgy

Here is how you update a boolean with Redux. I am not making this up.

```javascript
// Step 1: Define your action type constant
// (because magic strings are evil, but magic objects are fine)
const SET_MODAL_OPEN = 'SET_MODAL_OPEN';

// Step 2: Create an action creator
// (a function that returns an object that represents something that happened)
const setModalOpen = (isOpen) => ({
  type: SET_MODAL_OPEN,
  payload: isOpen,
});

// Step 3: Write a reducer
// (a function that takes state and returns new state but won't modify state
//  because that would be mutation and mutation is a war crime)
const modalReducer = (state = { isOpen: false }, action) => {
  switch (action.type) {
    case SET_MODAL_OPEN:
      return { ...state, isOpen: action.payload };
    default:
      return state;
  }
};

// Step 4: Configure your store
// (the place where global variables go to feel better about themselves)
const store = configureStore({
  reducer: { modal: modalReducer },
});

// Step 5: Wrap your entire application
const App = () => (
  <Provider store={store}>
    <EverythingElse />
  </Provider>
);

// Step 6: Connect your component
const MyModal = () => {
  const isOpen = useSelector((state) => state.modal.isOpen);
  const dispatch = useDispatch();
  
  return (
    <button onClick={() => dispatch(setModalOpen(true))}>
      Open Modal
    </button>
  );
};
```

Compare this to the global variable approach:

```javascript
// Open a modal
window.isModalOpen = true;
```

I rest my case.

## A Brief History of "Solving" Global State

| Year | Solution | What It Actually Was |
|------|----------|----------------------|
| 1995 | `window.myVar` | Global variable |
| 2000 | Singleton pattern | Global variable with a class |
| 2010 | MVC shared model | Global variable in a folder |
| 2015 | Redux store | Global variable with a ceremony |
| 2018 | React Context | Global variable with JSX |
| 2020 | Zustand | Global variable with hooks |
| 2022 | Jotai/Recoil | Global variable with atoms |
| 2025 | Signals | Global variable with subscriptions |
| 2026 | ???Store Pro Max | Global variable with a Stripe subscription |

As [XKCD #927](https://xkcd.com/927/) predicted, each new standard exists to replace the previous fourteen standards.

## The Middleware Tax

The true genius of Redux was inventing middleware — a way to intercept actions before they reach the reducer so you can do "side effects." You know what used to handle side effects?

**The function where you wrote your code.**

```javascript
// Before middleware
async function loadUser(id) {
  const user = await api.getUser(id);
  window.currentUser = user;
}

// After Redux Thunk, Redux Saga, Redux Observable, RTK Query,
// and 47 minutes of StackOverflow
const loadUserThunk = (id) => async (dispatch) => {
  dispatch(loadUserPending());
  try {
    const user = await api.getUser(id);
    dispatch(loadUserFulfilled(user));
  } catch (error) {
    dispatch(loadUserRejected(error.message));
  }
};
```

My colleague Wally once spent three days debugging a Redux Saga. When we finally found the bug, it was a missing `yield`. Three days. For a missing `yield`. I've deployed entire startups to production in three days.

## The "Single Source of Truth" Lie

Redux sells itself as a "single source of truth." This is marketing.

In practice, your application has:
- Redux store (the official truth)
- Local component state (the convenient truth)  
- URL parameters (the truth users can see)
- `localStorage` (the truth that survives refreshes)
- Server state managed by React Query (the only truth that actually matters)
- Refs (the truth that skips renders)
- The DOM itself (the original truth, dishonored and forgotten)

By the time you add React Query to your Redux app, you have approximately six "single sources of truth." This is not a source of truth. This is a courtroom drama.

Mordac the Preventer of Information Services once asked me to document where our state lived. I handed him a flowchart. He cried. I consider that a successful documentation session.

## The DevTools Argument

"But what about Redux DevTools? You can time-travel through state!"

Yes. I can replay every action that changed my boolean. I can jump back to the moment before someone clicked a button. I can export and import state snapshots.

I have never done this in production debugging.

I have, however, done this:

```javascript
console.log('isOpen:', window.isModalOpen);
```

That took 0.003 seconds. No extension required.

## The Real Solution

Here is my architecture for state management, refined over 47 years:

```javascript
// Global state: accessible everywhere, no boilerplate
window.appState = {
  user: null,
  isModalOpen: false,
  cart: [],
};

// Update: just assign it
window.appState.user = fetchedUser;

// Read: just read it
if (window.appState.isModalOpen) {
  // do the thing
}

// "But what about reactivity?"
// Set an interval. Check every 100ms.
// Works fine. I've done it since 1997.
```

Is it "scalable"? My applications haven't scaled since 2019. Production has been down since 2022. Scalability is a future problem.

Is it "testable"? Tests are pessimism. See my earlier article.

Does it "spark joy"? Marie Kondo has never shipped a Redux form. Her opinions on state management are irrelevant.

## The Dogbert Theorem

Dogbert once said: "Any sufficiently advanced technology is indistinguishable from a job requirement."

This is why Redux exists. Not because it solves problems better than a global variable. Because it generates interview questions, certifications, and three-day onboarding sessions that justify keeping senior engineers employed.

I am not complaining. Those engineers pay my consulting invoices.

## Conclusion

State management frameworks are global variables with:
- A PhD advisor (reducers)
- A prescription (actions)
- A support group (devtools)
- A financial dependency (the ecosystem)

The variable was always there. It was fine. You made it go to therapy.

---

*The author's state has been undefined since 2018. His Redux store has 4,000 reducers, none of which are connected to anything. Production is, somehow, still running.*

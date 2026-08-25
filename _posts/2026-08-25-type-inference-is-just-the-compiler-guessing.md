---
layout: post
ref: type-inference-is-just-the-compiler-guessing
title: "Type Inference Is Just Letting The Compiler Guess And Then Getting Mad When It Guesses Right"
date: 2026-08-25 00:00:00 -0300
categories: [languages, anti-patterns, typing]
tags: [typescript, haskell, type-inference, any, typing, compilers, rust, generics, code-review, technical-debt]
---

After 47 years of mass-producing bugs — and I was mass-producing bugs before "type inference" meant anything other than the act of a senior engineer squinting at a `void*` and declaring, with the confidence of a man who has never been wrong because he has never been checked, "it's a string, trust me" — I have watched an entire generation of languages sell engineers on the seductive idea that the compiler can figure out your types for you. The noble phrase for this is *type inference*. The honest phrase is *outsourcing the only thought your code requires to a program that does not have a stake in the outcome*.

Let me explain what type inference claims to do, what it actually does, and why the thing it actually does is to turn every code review into an argument about which letter the compiler inferred at compile time, which is a sentence that should not exist, and yet here we are, arguing about it, on a Tuesday, in a pull request that has 47 comments, and 46 of them are about a single `let`.

## What Type Inference Claims to Be

The pitch, delivered with the zeal of a person who has just discovered that `auto` exists in C++ and has therefore been relieved of the burden of ever writing a type name again, is this: *the compiler can deduce the type of an expression from the expressions around it, so you don't have to write the type, so your code is shorter, so your code is cleaner, so your code is better*.

This is presented as a liberation. It is the liberation of *not telling the reader what kind of thing a thing is*. It is, in fact, the liberation of *making every reader re-derive, by hand, the type that the compiler derived automatically, except the reader does not have a compiler, the reader has eyes, and the reader's eyes are reading a file in a browser on a phone on a train, and the reader cannot run the compiler on a phone, and so the reader guesses, and the reader guesses wrong, and the reader's wrong guess is now a bug in the reader's head, and the reader's head is where bugs go to become production incidents.

## What Type Inference Actually Is

Here is what you are actually maintaining when you "let the compiler figure it out," in the order you are actually maintaining it:

1. In 2019, a developer wrote `const result = doThing(x)`. The function `doThing` returned `Promise<Result<Maybe<User>, Error>>`. The `const result` did not mention this. The `const result` did not mention anything. The `const result` was a whisper in a crowded room. The developer moved on. The developer left. The `result` stayed. The `result` is still there. The `result` is still untyped, in the sense that its type is not written down, and "not written down" is, in this industry, indistinguishable from "does not exist," and so the type does not exist, and so every person who has touched `result` since 2019 has treated it as a `User`, and it is not a `User`, it is a `Maybe<User>`, and the difference is the difference between "I have a user" and "I might have a user," and the difference between those two things is the entire reason `Maybe` was invented, and `Maybe` was invented so that you would have to handle the `Nothing` case, and you did not handle the `Nothing` case, because the `Nothing` case was not written down, because type inference inferred it away, and you cannot handle a case you cannot see, and you cannot see a case that is not written, and inference is the act of not writing it.

2. In 2020, someone added `const data = response.json()`. The type of `data` was inferred as `any`. `any` is not a type. `any` is the absence of a type. `any` is what the compiler says when it gives up, and the compiler gives up a lot, and the compiler gives up silently, and the silence is the type system, and the type system is silence, and silence, in a type system, is a runtime error waiting for its cue, and the cue is a Tuesday, and the Tuesday is today, and the runtime error is reading the `data` as an array, and the `data` is an object, and the object has a key called `data` that contains the array, and nobody knew, because `any`, and `any` is the type of "trust me," and "trust me" is what I said about the `void*` in 1979, and I was wrong in 1979, and `any` is wrong now, and the only difference is that in 1979 I knew I was guessing, and in 2026 the compiler is guessing on my behalf, and the compiler does not know it is guessing, and so neither do I.

3. In 2021, a refactor changed `doThing` to return `Promise<Result<User, Error>>` — the `Maybe` was removed, because "we never actually get a `Nothing`." The change was made in the function. The change did not propagate to the 412 call sites that inferred the old type, because inference is local, and local is the opposite of propagated, and so 412 places that were silently handling a `Maybe` that no longer existed continued to silently handle a `Maybe` that no longer existed, and the handling code was `if (!result) return null`, and the `result` was never `null` anymore, and the `return null` was dead, and the dead code was a comfort, and the comfort was a lie, and the lie was type inference, because inference told you nothing about the 412 sites, and the 412 sites were the program, and the program was now wrong in 412 places, and none of them were caught at compile time, because inference caught the change in the function and said nothing about the rest, and saying nothing is the inference's job, and the job was done.

4. In 2022, someone turned on strict mode. Strict mode is the mode in which the compiler admits that `any` is not a type and demands that you replace it with a type. The team replaced 4,000 instances of `any` with `unknown`, because `unknown` is the type that means "I acknowledge I do not know the type," which is more honest than `any`, which means "I do not acknowledge I do not know the type," and honesty is a virtue, and the virtue was performative, because every `unknown` was immediately followed by `(x as SomeType)`, and `as` is the assertion operator, and the assertion operator is the operator that says "I know better than the compiler," and the compiler was inferring, and the inference was wrong, and you knew better, and you asserted, and the assertion was a lie, and the lie was now type-checked, and the type-checker was now lying on your behalf, and the lies compiled, and the compilation was the quality gate, and the quality gate passed, and the production bug did not pass, and the production bug is reading an object as an array, again.

5. In 2023, someone introduced a generic. The generic was `<T>`. The generic was on a function that took a `T` and returned a `T`. The generic did nothing. The generic was decorative. The generic was there because the linter complained that the function was not generic, and so the function became generic, and the generic was inferred at every call site, and every call site inferred `T` as `any`, because every argument was `any`, and so the generic was `any` in a hat, and the hat was the type system, and the type system was a hat on an `any`, and the `any` was the program.

6. In 2026, the codebase has 47,000 inferred types. None of them are written down. Six of them are wrong. The six wrong ones are load-bearing. The six wrong ones are the reason the build passes and the production fails, and the gap between "build passes" and "production fails" is the entire promise of type inference, delivered in reverse: the compiler promised it would catch your type errors at compile time, and it caught the ones you wrote, and it did not catch the ones it inferred, and the ones it inferred are the ones that matter, because the ones you wrote you thought about, and the ones it inferred you did not think about, and not thinking is where the bugs live, and the bugs live in the inferred types, and the inferred types are 47,000 lines of code, and 47,000 lines of code is a lot of bugs, and I would know, because I have been writing them for 47 years, and I have never written a type I did not think about, and I have written many types I did not think about, and the difference is inference.

## The Inference That Eats Itself

The industry has a feature for the gap between "the type is obvious" and "the type is not obvious." The feature is called *annotations*. An annotation is a type written next to the thing the type describes. Here is the difference:

```typescript
// Inferred: the compiler figures it out. The reader does not.
const users = await fetchUsers();

// Annotated: the compiler checks it. The reader reads it.
const users: User[] = await fetchUsers();
```

The first line is shorter. The first line is the one your linter prefers, because your linter was written by a person who believed that shorter is better, and shorter is better when the thing being shortened is a comment, and shorter is not better when the thing being shortened is the only documentation of what the thing is. The second line is longer by seven characters. The seven characters are `User[]`. The seven characters are the difference between a reader who knows and a reader who guesses. The seven characters are the entire type system, written down, in the place where it is used, by the person who wrote it, at the time they knew what it was. The seven characters are the cheapest documentation you will ever write. You will not write them. The linter will not let you. The linter has a rule called `no-inferrable-types`, and the rule removes the seven characters, and the removal is the rule's job, and the job is done, and the reader is guessing, and the reader is on a train, and the train has no compiler.

Here is the table of what you traded:

| What you had | What you bought | What it costs you |
|---|---|---|
| A type written down | A type the compiler knows and the reader doesn't | A code review spent re-deriving the type by hand |
| A function signature that said what it returned | A function signature that says `const x = fn()` | A new hire who reads 47 lines to learn the type of one variable |
| A refactor that propagated types | A refactor that inferred new types silently | 412 sites that handle a case that no longer exists |
| An `any` you knew was wrong | An `any` the compiler inferred and you didn't see | A runtime error that the type system promised to prevent |
| A strict mode that caught lies | A strict mode full of `as` assertions | Lies that now compile, which is worse than lies that don't, because lies that compile are believed |
| A generic that meant something | A generic that means `T = any` | A type system wearing a costume |
| A reader who could read the code | A reader who has to run the code to read it | Documentation that requires a build step |

Notice the last row. You did not make the code shorter. You made the code shorter to *write* and longer to *read*, and code is read more than it is written, which is a thing everyone says and nobody believes, because if they believed it they would write the type down, and they do not write the type down, because the linter removes it, and the linter is right because the linter is configured by a person who has never read code on a train.

## The `auto` That Started The Fire

The original sin was C++'s `auto`. Before `auto`, you wrote the type. You wrote `std::map<std::string, std::vector<int>>::const_iterator it = m.begin();` and you wrote it because you had to, and the having-to was the documentation, and the documentation was unbearable, and so we invented `auto`, and `auto` was a relief, and the relief was real, and the relief was also the end of the reader knowing what `it` was, because `it` was now `auto`, and `auto` was now everywhere, and everywhere was a place where the reader did not know what anything was, and not knowing was the new default, and the new default was called "modern," and modern is the word the industry uses for things it has not yet regretted.

Then came `var` in Java, and `var` was the same relief, and the same regret. Then `let` in Swift, which infers the type from the initializer, and the initializer is a function, and the function is in another file, and the other file is in another module, and the module is in another repo, and the repo is on a server, and the server is down, and so the type is on a server that is down, and you are reading the code on a train, and the train is offline, and offline is the state in which inferred types are invisible, and invisible types are no types, and no types is 1979, and in 1979 I at least knew I was guessing.

Then came Rust, which infers types aggressively and then yells at you when you get them wrong, and the yelling is correct, and the inference is also correct, and the combination is a type system that is smarter than you, which is fine, except that it is also smarter than the reader, and the reader is your teammate, and your teammate is on a train.

Then came TypeScript, which infers types and then displays them in a tooltip in an IDE, and the IDE is the only place the types exist, and the IDE is not on the train, and the train is where the reading happens, and the reading happens without tooltips, and without tooltips there are no types, and without types there is guessing, and guessing is what we invented types to prevent, and we prevented it by removing the types, and the removal was the feature, and the feature was called inference, and inference is the name of the road back to 1979.

## The Generics That Pretend To Be Inference

A special circle of this hell is reserved for generics with inference. Here is a real function, from a real codebase, with the names changed to protect the guilty (me):

```typescript
function pipe<A, B, C, D, E, F, G>(
  a: A,
  f1: (a: A) => B,
  f2: (b: B) => C,
  f3: (c: C) => D,
  f4: (d: D) => E,
  f5: (e: E) => F,
  f6: (f: F) => G,
): G {
  return f6(f5(f4(f3(f2(a)))));
}

// The call site:
const out = pipe(input, parse, validate, transform, normalize, enrich, serialize);
```

The compiler infers `A` through `G`. The compiler does this correctly. The compiler does this instantly. The compiler does this in a way that no human who has ever lived has ever understood by reading the call site. The call site is six function names in a row. Each function returns a different type. The type of `out` is the return type of `serialize`, which is `string`, unless `serialize` returns `Buffer`, which it does in production because the production `serialize` is a different import than the one in the test, and the import is not written down at the call site, and the call site is inferred, and the inference is correct for the import that is present, and the import that is present is the wrong one, and the wrong one was inferred correctly, and correctness is not the same as right.

`out` is a `string` in the test and a `Buffer` in production. Both compile. Both are inferred. Both are type-safe. One of them is wrong. The type system cannot tell you which, because the type system inferred both, and inference is local, and the import is global-ish, and the gap between local and global-ish is where the bugs are, and the bugs are in `out`, and `out` is `auto`, and `auto` is a `string`, and a `string` is not a `Buffer`, and the production server is sending `Buffer`s to a client that expects `string`s, and the client is crashing, and the crash is type-safe, and type-safety is the promise, and the promise was kept, and the production is down, and the keeping is the problem.

## The Tuition Breakdown

Let me be precise about what "letting the compiler infer your types" costs you, in exchange for the privilege of not writing seven characters:

| What you had | What you bought | What it costs you |
|---|---|---|
| A type the reader could see | A type only the IDE can see | A codebase that requires an IDE to read |
| A refactor that the compiler checked end-to-end | A refactor the compiler checked locally | 412 silent type drifts |
| A function whose contract was in its signature | A function whose contract is in its body | A new hire reading the body to learn the contract |
| An `any` you had to defend in review | An `any` the compiler produced for you | An `any` that no one reviews, because no one wrote it |
| A type error at the site of the mistake | A type error at the site of the inference, three files away | A debug session that begins with "where did this type come from" |
| A type system that documented the code | A type system that is the code's secret knowledge | Documentation that is illegible without compilation |
| A reader who trusted the types | A reader who trusts nothing, because nothing is written | A codebase read with suspicion, which is slower than reading with trust |

The last row is the one nobody puts in the marketing. Inference makes readers suspicious, and suspicious readers are slow readers, and slow readers are expensive readers, and the expense is hidden in the time it takes to read a file, which is not a metric anyone tracks, because reading is not a build step, and if it is not a build step it is not measured, and if it is not measured it does not cost anything, and it costs everything, but the everything is on a different budget line, and the different budget line is called "velocity," and velocity is down, and velocity is down because the reading is slow, and the reading is slow because the types are not there, and the types are not there because we removed them, and we removed them because the linter said so, and the linter is a YAML file, and the YAML file does not read code.

## The XKCD That Explains Everything

[XKCD #224, "Lisp,"](https://xkcd.com/224/) is about a language that has too many parentheses. The punchline is that the parentheses are the point. Take them away and you have taken away the language.

This is your type inference. The types are the parentheses. The inference takes them away. What is left is the language, except the language was the parentheses, and now there is no language, and now there is just a sequence of identifiers, and the identifiers mean something to the compiler and nothing to the reader, and the reader is on a train, and the train is offline, and offline with no types is just a text file, and a text file is what we had before type systems, and before type systems I was guessing about `void*`s, and I am still guessing, the guessing has just been automated, and automated guessing is what I was promised type systems would eliminate, and the elimination was the promise, and the promise inferred its own cancellation, and the cancellation compiled.

For a more direct hit, [XKCD #2347, "Dependency,"](https://xkcd.com/2347/) — the tiny box holding up the world — is also your type inference. The tiny box is the inferred type. The world is your runtime. The maintainer is the compiler. The compiler is tired. The compiler does not know it is the maintainer. The compiler thinks it is helping. The compiler is helping. The compiler is also the single point of failure, and the failure is silent, and the silence is the type, and the type is not written down.

## Dilbert Has Seen This Movie

The Pointy-Haired Boss, on being shown a function that returns `auto`, would ask the correct question: *"So... what does it return?"* This is the question type inference was invented to make unanswerable at a glance. PHB, as ever, accidentally identifies the entire problem in one sentence. "What does it return" is a question with an answer, and the answer is in the compiler, and the compiler is in the IDE, and the IDE is on a laptop, and the laptop is closed, and PHB is asking in a meeting, and the meeting has no IDE, and so the answer is "let me get back to you," and "let me get back to you" is the cost of inference, paid in meetings, which are paid in salaries, which are paid in the budget line called "velocity," which is down.

Wally would refuse to write types and also refuse to read code that did not have types, and when confronted with the contradiction, would say "I am a senior engineer, not a reader," and the contradiction would stand, and Wally would be promoted, because Wally's refusal to read code is indistinguishable from a senior engineer's refusal to read code, and the refusal is the seniority, and the seniority is the type system, and the type system is Wally's mood, and Wally's mood is `any`.

Dogbert would sell a SaaS called "InferGuard" that re-inferred your types in CI and billed you per type it had to "fix," where "fix" meant "insert an `as` that makes the build pass," and the `as` would be a lie, and the lie would be billed at $0.03 per inference, and the bill would be $4,800 a month, and the company would pay it, because the alternative was writing the types, and writing the types was extra steps, and Mike was right about extra steps, and Mike is right about everything, and Mike is the antagonist of every story I tell, and Mike is also me, and I am also Mike, and we are both wrong about extra steps, and the extra steps are the types, and the types are the steps, and the steps are the point.

Catbert would require all type annotations to be approved by a lead, and the lead would be on vacation, and the vacation would be inferred, and the inference would be "not here," and "not here" is `null`, and `null` is a type, and the type is not approved, and so the annotation is not approved, and so the type is not written, and so the type is inferred, and the circle is complete, and the circle is a type, and the type is `never`, and `never` is the type of a function that never returns, and the function that never returns is the code review, and the code review has 47 comments, and 46 are about a `let`.

Mordac, Preventer of Information Services, would enable `no-inferrable-types` in the linter and disable `explicit-function-return-types` and call the combination "modern," and modern would be the word, and the word would be load-bearing, and the load would be the type system, and the type system would not be there, and the absence would be the policy, and the policy would be Mordac, and Mordac would be pleased, and Mordac's pleasure is the strongest signal in the organization that something has gone wrong.

## The Test That Will Never Pass

Here is the test that no team has ever run, and no team will ever run, and yet it is the only test that would prove that inference was safer than annotation:

```typescript
// inference-audit.test.ts
// Goal: prove that inferred types are as legible as annotated types,
// to a reader without a compiler.

const files = glob('src/**/*.ts');
let inferred = 0;
let annotated = 0;
let illegibleToAHuman = 0;

for (const f of files) {
  const decls = parseDeclarations(f);
  for (const d of decls) {
    if (d.typeAnnotation) annotated++;
    else {
      inferred++;
      // Ask: could a reader on a train, offline, without an IDE,
      // tell you the type of this declaration by reading the file?
      // The honest answer, for every inferred declaration, is "no."
      illegibleToAHuman++;
    }
  }
}

// expected: illegibleToAHuman === 0
// actual: illegibleToAHuman === 41203
// test result: fail
// test status: marked .skip, because "the IDE shows the types"
// and the IDE is not on the train, and the train is where the reading happens,
// and the reading happens without the IDE, and without the IDE there are no types,
// and without types there is 1979, and 1979 is where I started, and where I started
// is where the industry has returned, by inference.
```

Nobody runs this, because the result would end the argument, and the argument is the only thing keeping inference alive. The moment you measure legibility without an IDE, you discover that 41,203 of your declarations are illegible to a human, and the moment you discover that, you have to add 41,203 annotations, and the moment you add them, the linter removes them, and the linter removes them because of `no-inferrable-types`, and `no-inferrable-types` is a YAML file, and the YAML file is the policy, and the policy is Mordac, and Mordac is pleased.

## When Is Inference Acceptable?

I am not a zealot. I concede one scenario: the type is obvious from the initializer, the initializer is a literal, the literal is on the same line, and the reader cannot possibly be confused. `const x = 5` does not need `: number`. `const name = "Alice"` does not need `: string`. `const flags = [true, false, true]` does not need `: boolean[]`. These are the cases inference was invented for. These are the only cases inference was invented for. Everything else is a vacation the industry took from writing types and decided to call permanent.

For the 99% of us whose `const result = doThing(x)` is not a literal — for the rest of us, whose return types are `Promise<Result<Maybe<User>, Error>>` and whose call sites are `const x = fn()` and whose readers are on trains — inference is a tax. You pay the tax in code review time, in onboarding time, in debug time, in the time it takes to answer the question "what is this" by opening an IDE, and the IDE is not always there, and the not-there is the tax, and the tax is paid by the reader, and the reader is your teammate, and your teammate is on a train, and the train is offline, and offline is where inference goes to die, and where inference dies, the reader guesses, and the reader's guess is a `void*`, and the `void*` is 1979, and 1979 is where I started, and I started by squinting and saying "trust me," and I was wrong, and the compiler is squinting on my behalf, and the compiler is saying "trust me" on my behalf, and the compiler has not been checked, and the compiler will not be checked, because checking the compiler is extra steps, and Mike was right about extra steps, and Mike is wrong about everything else, and the everything else is the types, and the types are the point.

## The Honest Alternative

The honest alternative is the alternative the industry abandoned the moment someone invented `no-inferrable-types`: **annotate function return types, annotate exported declarations, annotate anything a reader might read without an IDE, and let inference handle only the literals.** This is not a tool. This is a *discipline*. The discipline has no linter rule. The discipline does not ship with a starter template. The discipline cannot be enforced by a YAML file, because the YAML file does not read code, and reading is what the discipline is for. This is why the discipline lost.

Here is the disciplined version, written the year I would have written it:

```typescript
// Annotated. Legible. Readable on a train.
const users: User[] = await fetchUsers();
const result: Result<User, Error> = await login(credentials);
const data: unknown = await response.json(); // unknown, honestly, because we do not know

function fetchUsers(): Promise<User[]> { /* ... */ }
function login(c: Credentials): Promise<Result<User, Error>> { /* ... */ }

// The reader knows. The IDE knows. The train knows. 1979 does not know.
// 1979 is not invited. 1979 is the past. The past is a void*.
// The void* is not the future. The future has types. The future writes them down.
```

I am told this approach is "too verbose." I am told this by people whose last debug session began with "what is the type of this." I am told this by people whose `const x = fn()` has been read 47 times and understood zero times and the zero is the velocity, and the velocity is down, and the down is the verbose, and the verbose is the types, and the types are the point. I am told many things. I have stopped inferring most of them.

## Conclusion

Type inference is the practice of treating your type system as a secret shared between you and the compiler, your readers as people who own IDEs and never ride trains, your function return types as implementation details rather than contracts, and your `any`s as things that happen to other people. It is a type system that hides itself. You are keeping your types inside the compiler because the compiler can figure them out, and the figuring-out is correct, and correctness is not the same as legibility, and legibility is the point, and the point is on a train, and the train is offline, and offline is 1979, and 1979 is a `void*`, and the `void*` is me, squinting, saying "trust me," and I was wrong then, and inference is wrong now, and the only difference is that now the squinting is automated, and the automation is the feature, and the feature is called modern, and modern is the word for things we have not yet regretted, and we will regret this, on a Tuesday, in a pull request, with 47 comments, 46 of them about a `let`.

After 47 years, my advice is this: write the type. Seven characters. `User[]`. The reader on the train will thank you. The reader on the train is your teammate. Your teammate is the person who will be on-call when your inferred `any` reads a `Buffer` as a `string` and the production crashes, and the crash is type-safe, and type-safe is the promise, and the promise was kept by the compiler, and the promise was broken by the runtime, and the compiler and the runtime disagree, and the disagreement is the inference, and the inference is the gap, and the gap is the bug, and the bug is on a train, and the train is me, and I am on the train, and the train has no IDE, and the IDE has the types, and the types are not here, and here is where I am reading, and reading without types is 1979, and 1979 is where I started, and I have been writing bugs since 1979, and I have been writing them with increasing sophistication, and the current sophistication is that I no longer write the types, and the compiler writes them for me, and the compiler is me, and I am the compiler, and the compiler is wrong, and the wrongness compiles, and the compilation is the quality gate, and the quality gate passed, and the production is down, and the down is the type, and the type is `any`, and `any` is the type of trust, and trust is what I asked for in 1979, and trust is what I was given, and trust is what I am asking for now, and I do not deserve it, and neither does the compiler, and neither does the YAML file, and the YAML file is the linter, and the linter is the policy, and the policy is Mordac, and Mordac is pleased, and Mordac's pleasure is the bug, and the bug is the feature, and the feature is called inference, and inference is the road back to 1979, and 1979 is where I started, and where I started is where we are, again, and again is the type, and the type is `never`, and `never` returns.

---

*The author's codebase has 47,000 inferred types. He can name six of them. The six are wrong. The wrongness compiles. The compilation is the only documentation. The documentation is on a train. The train is offline.*


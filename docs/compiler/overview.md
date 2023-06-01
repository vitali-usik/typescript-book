# Компилятор

Исходный код компилятора TypeScript находится в папке [`src/compiler`](https://github.com/Microsoft/TypeScript/tree/master/src/compiler).

Он разделен на следующие ключевые части:

- Scanner (`scanner.ts`)
- Parser (`parser.ts`)
- Binder (`binder.ts`)
- Checker (`checker.ts`)
- Emitter (`emitter.ts`)

Каждый из них получает свои уникальные файлы в исходном коде.. Эти части будут объяснены позже в этой главе.

## BYOTS

У нас есть проект [Bring Your Own TypeScript (BYOTS)](https://github.com/basarat/byots), который упрощает работу с API компилятора например раскрывая внутренние API. Вы можете использовать его в вашем локальном приложении для доступа к глобальной версии TypeScript

## Синтаксис против семантики

Просто потому, что что-то _синтаксически_ правильно, не означает, что это _семантически_ правильно. CРассмотрим следующий фрагмент кода TypeScript, который хотя и _синтаксически_ корректен, но _семантически_ неверен.

```ts
var foo: number = "not a number";
```

`Semantic` означает «смысл» на английском языке. Эту концепцию полезно иметь в голове.

## Processing Overview

The following is a quick review of how these key parts of the TypeScript compiler compose:

```code
SourceCode ~~ scanner ~~> Token Stream
```

```code
Token Stream ~~ parser ~~> AST
```

```code
AST ~~ binder ~~> Symbols
```

`Symbol` is the primary building block of the TypeScript _semantic_ system. As shown the symbols are created as a result of binding. Symbols connect declaration nodes in the AST to other declarations contributing to the same entity.

Symbols + AST are what is used by the checker to _semantically_ validate the source code

```code
AST + Symbols ~~ checker ~~> Type Validation
```

Finally When a JS output is requested:

```code
AST + Checker ~~ emitter ~~> JS
```

There are a few additional files in the TypeScript compiler that provide utilities to many of these key portions which we cover next.

## File: Utilities

`core.ts` : core utilities used by the TypeScript compiler. A few important ones:

- `let objectAllocator: ObjectAllocator` : is a variable defined as a singleton global. It provides the definitions for `getNodeConstructor` (Nodes are covered when we look at `parser` / `AST`), `getSymbolConstructor` (Symbols are covered in `binder`), `getTypeConstructor` (Types are covered in `checker`), `getSignatureConstructor` (Signatures are the index, call and construct signatures).

## File: Key Data Structures

`types.ts` contains key data structures and interfaces uses throughout the compiler. Here is a sampling of a few key ones:

- `SyntaxKind`
  The AST node type is identified by the `SyntaxKind` enum.
- `TypeChecker`
  This is the interface provided by the TypeChecker.
- `CompilerHost`
  This is used by the `Program` to interact with the `System`.
- `Node`
  An AST node.

## File: System

`system.ts`. All interaction of the TypeScript compiler with the operating system goes through a `System` interface. Both the interface and its implementations (`WScript` and `Node`) are defined in `system.ts`. You can think of it as the _Operating Environment_ (OE).

Now that you have an overview of the major files, we can look at the concept of `Program`

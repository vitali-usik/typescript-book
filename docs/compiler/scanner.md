## Scanner

Исходный код сканера TypeScript полностью находится в файле scanner.ts. Сканер управляется парсером для преобразования исходного кода в AST (Абстрактное Синтаксическое Дерево). Вот как выглядит желаемый результат:

```
SourceCode ~~ scanner ~~> Token Stream ~~ parser ~~> AST
```

### Использование парсером
В файле `parser.ts` создается *одиночный* экземпляр сканера, чтобы избежать затрат на многократное создание новых сканеров. Этот сканер затем *инициализируется* парсером по мере необходимости с помощью функции `initializeState`.

Вот *упрощенная* версия реального кода в парсере, которую вы можете запустить, чтобы продемонстрировать эту концепцию:

`code/compiler/scanner/runScanner.ts`
```ts
import * as ts from "ntypescript";

// TypeScript имеет одиночный сканер
const scanner = ts.createScanner(ts.ScriptTarget.Latest, /*skipTrivia*/ true);

// Который инициализируется с помощью функции `initializeState`, аналогично:
function initializeState(text: string) {
    scanner.setText(text);
    scanner.setOnError((message: ts.DiagnosticMessage, length: number) => {
        console.error(message);
    });
    scanner.setScriptTarget(ts.ScriptTarget.ES5);
    scanner.setLanguageVariant(ts.LanguageVariant.Standard);
}

// Пример использования
initializeState(`
var foo = 123;
`.trim());

// Начать сканирование
var token = scanner.scan();
while (token != ts.SyntaxKind.EndOfFileToken) {
    console.log(ts.formatSyntaxKind(token));
    token = scanner.scan();
}

```

Это выведет следующее:

```
VarKeyword
Identifier
FirstAssignment
FirstLiteralToken
SemicolonToken
```

### Состояние сканера
После вызова `scan` сканер обновляет свое локальное состояние (положение в сканировании, текущие детали токена и т.д.). Сканер предоставляет множество вспомогательных функций для получения текущего состояния сканера. В следующем примере мы создаем сканер и используем его для определения токенов, а также их позиций в коде.

`code/compiler/scanner/runScannerWithPosition.ts`
```ts
// Пример использования
initializeState(`
var foo = 123;
`.trim());

// Начать сканирование
var token = scanner.scan();
while (token != ts.SyntaxKind.EndOfFileToken) {
    let currentToken = ts.formatSyntaxKind(token);
    let tokenStart = scanner.getStartPos();
    token = scanner.scan();
    let tokenEnd = scanner.getStartPos();
    console.log(currentToken, tokenStart, tokenEnd);
}
```

Это выведет следующее:
```
VarKeyword 0 3
Identifier 3 7
FirstAssignment 7 9
FirstLiteralToken 9 13
SemicolonToken 13 14
```

### Одиночный сканер
Несмотря на то, что TypeScript-парсер имеет одиночный сканер, вы можете создать независимый сканер с помощью `createScanner` и использовать его методы `setText`/`setTextPos`, чтобы сканировать различные части файла для вашего удобства.

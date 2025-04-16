# @babel/parser

Babel解析器（之前称为Babylon）是Babel中使用的JavaScript解析器。

* 默认启用最新的ECMAScript版本（ES2020）。
* 注释附加。
* 支持JSX、Flow、TypeScript。
* 支持实验性语言提案（接受至少处于stage-0阶段的任何提案的PR）。

## 致谢

主要基于acorn和acorn-jsx，感谢@RReverser和@marijnh的出色工作。

## API

### `babelParser.parse(code, [options])`

### `babelParser.parseExpression(code, [options])`

`parse()`将提供的`code`解析为完整的ECMAScript程序，而`parseExpression()`则尝试解析单个表达式，注重性能。如有疑问，请使用`.parse()`。

### 选项

历史

| 版本 | 变更 |
| ------- | ---------------------------------------------- |
| v7.27.0 | 添加了allowYieldOutsideFunction |
| v7.26.0 | 添加了startIndex |
| v7.23.0 | 添加了createImportExpressions |
| v7.21.0 | 添加了allowNewTargetOutsideFunction和annexB |
| v7.16.0 | 添加了startColumn |
| v7.15.0 | 添加了attachComment |
| v7.7.0 | 添加了errorRecovery |
| v7.5.0 | 添加了allowUndeclaredExports |
| v7.2.0 | 添加了createParenthesizedExpressions |

* **allowImportExportEverywhere**: 默认情况下，`import`和`export`声明只能出现在程序的顶层。将此选项设置为`true`允许它们在允许语句的任何位置出现。
* **allowAwaitOutsideFunction**: 默认情况下，`await`的使用仅允许在异步函数内部，或者当启用`topLevelAwait`插件时，在模块的顶层作用域中使用。将此设置为`true`也可以在脚本的顶层作用域中接受它。不推荐使用此选项，建议使用`topLevelAwait`插件。
* **allowYieldOutsideFunction**: 默认情况下，`yield`的使用仅允许在生成器函数内部。将此设置为`true`也可以在顶层接受它。
* **allowNewTargetOutsideFunction**: 默认情况下，不允许在函数或类之外使用`new.target`。将此设置为`true`以接受此类代码。
* **allowReturnOutsideFunction**: 默认情况下，顶层的return语句会引发错误。将此设置为`true`以接受此类代码。
* **allowSuperOutsideMethod**: 默认情况下，不允许在类和对象方法之外使用`super`。将此设置为`true`以接受此类代码。
* **allowUndeclaredExports**: 默认情况下，导出当前模块作用域中未声明的标识符将引发错误。虽然此行为是ECMAScript模块规范所要求的，但Babel的解析器无法预见插件管道后期可能插入适当声明的转换，因此有时候将此选项设置为`true`很重要，以防止解析器过早地投诉稍后将添加的未声明导出。
* **attachComment**: 默认情况下，Babel将注释附加到相邻的AST节点。当此选项设置为`false`时，注释不会被附加。当输入代码有_许多_注释时，它可以提供高达30%的性能改进。`@babel/eslint-parser`会为您设置它。不建议在Babel转换中使用`attachComment: false`，因为这样做会删除输出代码中的所有注释，并使诸如`/* istanbul ignore next */`之类的注释失效。
* **annexB**: 默认情况下，Babel根据ECMAScript的附录B"_Web浏览器的附加ECMAScript功能_"语法解析JavaScript。当此选项设置为`false`时，Babel将解析不包含附录B特定扩展的语法。
* **createImportExpressions**: 默认情况下，解析器将动态导入`import()`解析为调用表达式节点。当此选项设置为`true`时，将创建`ImportExpression` AST节点。此选项在Babel 8中将默认为`true`。
* **createParenthesizedExpressions**: 默认情况下，解析器在表达式节点上设置`extra.parenthesized`。当此选项设置为`true`时，将创建`ParenthesizedExpression` AST节点。
* **errorRecovery**: 默认情况下，Babel在发现无效代码时总是抛出错误。当此选项设置为`true`时，它将存储解析错误并尝试继续解析无效的输入文件。生成的AST将具有一个`errors`属性，表示所有解析错误的数组。请注意，即使启用此选项，`@babel/parser`也可能对于不可恢复的错误抛出异常。
* **plugins**: 包含要启用的插件的数组。
* **sourceType**: 指示代码应该以哪种模式解析。可以是`"script"`、`"module"`或`"unambiguous"`之一。默认为`"script"`。`"unambiguous"`将使@babel/parser尝试_猜测_，基于ES6的`import`或`export`语句的存在。具有ES6的`import`和`export`的文件被视为`"module"`，否则视为`"script"`。
* **sourceFilename**: 将输出的AST节点与其源文件名关联起来。在从多个输入文件的AST生成代码和源映射时很有用。
* **startColumn**: 默认情况下，解析的代码被视为从第1行、第0列开始。您可以提供一个列号来替代起始位置。对于与其他源工具集成很有用。
* **startLine**: 默认情况下，解析的代码被视为从第1行、第0列开始。您可以提供一个行号来替代起始位置。对于与其他源工具集成很有用。
* **startIndex**: 默认情况下，所有源索引从0开始。使用此选项，您可以提供一个替代的起始索引。为确保准确的AST源索引，当`startLine`大于1时，应始终提供此选项。对于与其他源工具集成很有用。
* **strictMode**: 默认情况下，ECMAScript代码仅在存在`"use strict";`指令或解析的文件是ECMAScript模块时才以严格模式解析。将此选项设置为`true`以始终以严格模式解析文件。
* **ranges**: 向每个节点添加一个`range`属性：`[node.start, node.end]`
* **tokens**: 将所有解析的标记添加到`File`节点的`tokens`属性中

### 输出

Babel解析器根据Babel AST格式生成AST。它基于ESTree规范，但有以下偏差：

* Literal标记被替换为StringLiteral、NumericLiteral、BigIntLiteral、BooleanLiteral、NullLiteral、RegExpLiteral
* Property标记被替换为ObjectProperty和ObjectMethod
* MethodDefinition被替换为ClassMethod和ClassPrivateMethod
* PropertyDefinition被替换为ClassProperty和ClassPrivateProperty
* PrivateIdentifier被替换为PrivateName
* Program和BlockStatement包含带有Directive和DirectiveLiteral的附加`directives`字段
* ClassMethod、ClassPrivateMethod、ObjectProperty和ObjectMethod的value属性中FunctionExpression的属性被合并/带入主方法节点。
* ChainExpression被替换为OptionalMemberExpression和OptionalCallExpression
* ImportExpression被替换为`callee`是Import节点的CallExpression。这一变更将在Babel 8中逆转。
* 带有`exported`字段的ExportAllDeclaration被替换为包含ExportNamespaceSpecifier节点的ExportNamedDeclaration。

注意

`estree`插件可以恢复这些偏差。仅当您将Babel AST传递给其他符合ESTree的工具时才使用它。

JSX代码的AST基于Facebook JSX AST。

### 语义化版本

Babel解析器在大多数情况下遵循语义化版本。唯一需要注意的是，一些符合规范的bug修复可能会在补丁版本中发布。

例如：我们推送了一个修复，对#107 - 每个文件多个默认导出的提前错误进行了修复。这被视为bug修复，即使它可能导致构建失败。

### 示例

```javascript
require("@babel/parser").parse("code", {
  // 在严格模式下解析并允许模块声明
  sourceType: "module",

  plugins: [
    // 启用jsx和flow语法
    "jsx",
    "flow",
  ],
});
```

### 插件

#### 杂项

| 名称 | 代码示例 |
|------|---------|
| optionalCatchBinding ([提案](https://github.com/babel/proposals/issues/7)) | try {throw 0;} catch{do();} |
| optionalChaining ([提案](https://github.com/tc39/proposal-optional-chaining)) | a?.b |
| privateIn ([提案](https://github.com/tc39/proposal-private-fields-in-in)) | #p in obj |
| regexpUnicodeSets ([提案](https://github.com/tc39/proposal-regexp-set-notation)) | /\[\\p{Decimal\_Number}--\[0-9\]\]/v; |
| topLevelAwait ([提案](https://github.com/tc39/proposal-top-level-await/)) | 在模块中 await promise |
| importAttributes ([提案](https://github.com/tc39/proposal-import-attributes)) | import json from "./foo.json" with { type: "json" }; |

#### 插件选项

历史

| 版本 | 变更 |
|------|------|
| 7.21.0 | decorators的decoratorsBeforeExport选项的默认行为是允许装饰器放在export关键字之前或之后。 |
| 7.19.0 | recordAndTuple插件的syntaxType选项默认为hash；为decorators插件添加了allowCallParenthesized选项。 |
| 7.17.0 | 为hack管道操作符的topicToken选项添加了@@和^^ |
| 7.16.0 | 为typescript插件添加了disallowAmbiguousJSXLike。为hack管道操作符的topicToken选项添加了^ |
| 7.14.0 | 为typescript插件添加了dts |

注意

当一个插件被多次指定时，只考虑第一个选项。

* `decorators`:
   * `allowCallParenthesized` (`boolean`，默认为`true`)  
   当为`false`时，不允许`@(...)()` 形式的装饰器，而支持`@(...())`。第3阶段装饰器提案使用`allowCallParenthesized: false`。
   * `decoratorsBeforeExport` (`boolean`)  
   默认情况下，导出类上的装饰器可以放在`export`关键字之前或之后。设置此选项后，装饰器只能放在指定位置。
   ```javascript
   // decoratorsBeforeExport: true
   @dec
   export class C {}
   // decoratorsBeforeExport: false
   export @dec class C {}
   ```
   注意  
   此选项已弃用，将在未来版本中删除。当此选项明确设置为`true`或`false`时有效的代码，在未设置此选项时也有效。
* `optionalChainingAssign`:
   * `version`（必需，接受的值：`2023-07`）此提案仍处于第1阶段，因此很可能会受到重大更改的影响。您必须指定正在使用的提案版本，以确保Babel将继续以兼容的方式解析您的代码。
* `pipelineOperator`:
   * `proposal`（必需，接受的值：`fsharp`、`hack`、~~`minimal`~~、~~`smart`~~（已弃用））管道操作符有几种不同的提案。此选项选择使用哪个提案。有关更多信息，包括比较它们行为的表格，请参阅plugin-proposal-pipeline-operator。
   * `topicToken`（当`proposal`为`hack`时必需，接受的值：`%`、`#`、`^`、`@@`、`^^`）`hack`提案在其管道中使用"主题"占位符。这个主题占位符有两种不同的选择。此选项选择使用什么标记来引用主题。`topicToken: "#"`与`recordAndTuple`的`syntaxType: "hash"`不兼容。有关更多信息，请参阅plugin-proposal-pipeline-operator。
* `recordAndTuple`:
   * `syntaxType`（`hash`或`bar`，默认为`hash`）`recordAndTuple`有两种语法变体。它们共享完全相同的运行时语义。
   | SyntaxType | Record示例 | Tuple示例 |
   |------------|------------|-----------|
   | "hash" | #{ a: 1 } | #[1, 2] |
   | "bar" | {\| a: 1 } | [\|1, 2\|] |
   有关更多信息，请参阅[#{}/#[]的人体工程学](https://github.com/tc39/proposal-record-tuple/issues/10)。
* `flow`:
   * `all`（`boolean`，默认：`false`）某些代码在Flow和纯JavaScript中具有不同的含义。例如，在Flow中，`foo<T>(x)`被解析为带有类型参数的调用表达式，但根据ECMAScript规范，它被解析为比较（`foo < T > x`）。默认情况下，`babel-parser`仅在文件以`// @flow`指令开头时才将这些模糊结构解析为Flow类型。将此选项设置为`true`可始终将文件解析为指定了`// @flow`的情况。
* `typescript`
   * `dts`（`boolean`，默认`false`）此选项将启用在TypeScript环境上下文中解析，其中某些语法具有不同的规则（如`.d.ts`文件和`declare module`块内）。请参阅<https://www.typescriptlang.org/docs/handbook/declaration-files/introduction.html>和<https://basarat.gitbook.io/typescript/type-system/intro>以获取有关环境上下文的更多信息。
   * `disallowAmbiguousJSXLike`（`boolean`，默认`false`）即使未启用`jsx`插件，此选项也会禁止使用与JSX模糊的语法（`<X> y`类型断言和`<X>() => {}`类型参数）。它与解析`.mts`和`.mjs`文件时的`tsc`行为匹配。
* `importAttributes`:
   * `deprecatedAssertSyntax`（`boolean`，默认为`false`）  
   当为`true`时，允许解析导入属性的旧版本，该版本使用`assert`关键字而不是`with`。  
   这与`importAssertions`解析器插件最初支持的语法匹配。

### 错误代码

历史

| 版本 | 变更 |
|------|------|
| v7.14.0 | 添加了错误代码 |

错误代码对于处理`@babel/parser`抛出的错误很有用。

有两种错误代码：`code`和`reasonCode`。

* `code`
   * 错误的粗略分类（例如`BABEL_PARSER_SYNTAX_ERROR`、`BABEL_PARSER_SOURCETYPE_MODULE_REQUIRED`）。
* `reasonCode`
   * 错误的详细分类（例如`MissingSemicolon`、`VarRedeclaration`）。

使用`errorRecovery`的错误代码示例：

```javascript
const { parse } = require("@babel/parser");

const ast = parse(`a b`, { errorRecovery: true });

console.log(ast.errors[0].code); // BABEL_PARSER_SYNTAX_ERROR
console.log(ast.errors[0].reasonCode); // MissingSemicolon
```

### 常见问题

#### Babel解析器是否将支持插件系统？

之前的问题：#1351，#6694。

我们目前不愿意承诺支持插件API或由此产生的生态系统（维护Babel自己的插件系统已经有足够的工作）。目前尚不清楚如何使这个API有效，而且它会限制我们重构和优化代码库的能力。

对于那些想要创建自己的自定义语法的人，我们当前的建议是用户分叉解析器。

要使用您的自定义解析器，您可以通过其npm包名称添加一个插件到您的选项中调用解析器，或者如果使用JavaScript，则可以require它：

```javascript
const parse = require("custom-fork-of-babel-parser-on-npm-here");

module.exports = {
  plugins: [
    {
      parserOverride(code, opts) {
        return parse(code, opts);
      },
    },
  ],
};
``` 
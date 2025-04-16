# @babel/traverse

## 安装

* npm
* Yarn
* pnpm

```
npm install --save @babel/traverse
```

```
yarn add @babel/traverse
```

```
pnpm add @babel/traverse
```

## 使用方法

我们可以与babel解析器一起使用它来遍历和更新节点：

```javascript
import * as parser from "@babel/parser";
import traverse from "@babel/traverse";

const code = `function square(n) {
  return n * n;
}`;

const ast = parser.parse(code);

traverse(ast, {
  enter(path) {
    if (path.isIdentifier({ name: "n" })) {
      path.node.name = "x";
    }
  },
});
```

此外，我们可以在语法树中针对特定的**节点类型**：

```javascript
traverse(ast, {
  FunctionDeclaration: function(path) {
    path.node.id.name = "x";
  },
});
```

📖 **在此阅读完整文档** 
## nodejs 编写简易 webpack

### 目录结构

```shell
--my-webpack
----index.html
----bundle.js
----src
------index.js
------add.js
------minus.js
```

### 项目初始化

```shell
npm init -y

npm install @babel/parser

npm install @babel/traverse

npm install @babel/core @babel/preset-env
```

### 代码实现

---

```js
// index.js

import add from "./add.js";
import { minus } from "./minus.js";

const sum = add(1, 2);
const division = minus(2, 1);

console.log(sum);
console.log(division);
```

---

```js
// add.js

export default (a, b) => {
  return a + b;
};
```

---

```js
// minus.js
export const minus = (a, b) => {
  return a - b;
};
```

---

```js
//bundle.js

const fs = require("fs");
const path = require("path");
const parser = require("@babel/parser");
const traverse = require("@babel/traverse").default;
const babel = require("@babel/core");
const getModuleInfo = (file) => {
  const body = fs.readFileSync(file, "utf-8");
  const ast = parser.parse(body, {
    sourceType: "module", //表示我们要解析的是ES模块
  });
  const deps = {};
  traverse(ast, {
    ImportDeclaration({ node }) {
      const dirname = path.dirname(file);
      const abspath = "./" + path.join(dirname, node.source.value);
      deps[node.source.value] = abspath;
    },
  });
  const { code } = babel.transformFromAst(ast, null, {
    presets: ["@babel/preset-env"],
  });
  const moduleInfo = { file, deps, code };
  return moduleInfo;
};
const parseModules = (file) => {
  const entry = getModuleInfo(file);
  const temp = [entry];
  const depsGraph = {};
  for (let i = 0; i < temp.length; i++) {
    const deps = temp[i].deps;
    if (deps) {
      for (const key in deps) {
        if (deps.hasOwnProperty(key)) {
          temp.push(getModuleInfo(deps[key]));
        }
      }
    }
  }
  temp.forEach((moduleInfo) => {
    depsGraph[moduleInfo.file] = {
      deps: moduleInfo.deps,
      code: moduleInfo.code,
    };
  });
  return depsGraph;
};
const bundle = (file) => {
  const depsGraph = JSON.stringify(parseModules(file));
  return `(function (graph) {
        function require(file) {
            function absRequire(relPath) {
                return require(graph[file].deps[relPath])
            }
            var exports = {};
            (function (require,exports,code) {
                eval(code)
            })(absRequire,exports,graph[file].code)
            return exports
        }
        require('${file}')
    })(${depsGraph})`;
};
const content = bundle("./src/index.js");

console.log(content);

//写入到我们的dist目录下
fs.mkdirSync("./dist");
fs.writeFileSync("./dist/bundle.js", content);
```
---

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Title</title>
    <script src="dist/bundle.js"></script>
  </head>
  <body>
    <!--<script src="./src/index.js"></script>-->
  </body>
</html>

```
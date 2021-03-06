## 代码规范
### 社区已有规范
HTML/CSS
Google HTML/CSS/JS 规范 著名的谷歌前端规范

AIrbnb Style 规范（包括CSS和Sass） AIrbnb的样式规范，不仅包含css规范，亦包含Sass的规范

#### javaScript 规范
Airbnb javaScript规范 Airbnb的javascript编码规范

javascript Standard Style Standard规范，影响力最大的JS编码规范，生态丰富，提供了开箱即用的各种lint规则和编辑器插件

#### 框架相关
Vue style Guide VueJS官方推荐的编码规范

Airbnb React/JSX Style Guide Airbnb javascript规范的React/JSX部分

### 建立代码规范 - ESLint
- Eslint介绍 一款高度可配置的javaScript静态代码检测工具，已经成为js代码检查的事实标准

- 特效

  - 完全的可插拔，一切行为都通过配置产生
  - 任意rule之间都是独立的
- 原理 先通过解析器（parser）将javaScript代码解析为抽象语法树（AST），再调用规则对AST进行检查，从而实现对代码的检查

- AST 浅析 AST是一种可遍历的、描述代码的树状结构，利用AST可以方便地分析代码的结构和内容https://astexplorer.net/

- ESLint CLI

```js
eslint -h
```

- CLI 之外
  - 编辑器的集成 VS Code/Atom/Vim/Sublime Text 在写代码的同时就可以实时对代码进行检查
  - 构建工具集成 Webpack/Rollup/Gulp/Grunt 在构建过程中进行代码检查
- ESLint 的配置
  配置文件格式 javascript，JSON或者YAML，也可以在package.json中的eslintConfig字段
  - ESLint配置的主要内容
    - Parser：ESLint使用哪种解析器
    - Environments：选择你的代码跑在什么环境中（browser/node/commonjs/es6/es2017/worker）
    - Globals：除了Env之外，其他需要额外指定的全局变量
    - Rules：规则
    - Plugins：一组以上配置项以及processor集合，往往用于特定类型文件的代码检查，如.md文件
    - Extends：你想继承的配置
##### parser配置
```json
{
    "parser":"esprima",
    "parserOptions": {
        "ecmaVersion": 6,
        "sourceType": "module",
        "ecmaFeatures": {
            "jsx": true
        }
    }
}
```

- parser 指定ESlint使用哪种解析器：Espree（type默认）、Esprima、Babel-ESLint、@typescript-eslint/parser，一般不需指定

- parserOptions 配置parser的参数，parser会接收这些参数，并影响其解析代码的行为

##### Evironments、Globals
```json
{
    "env":{
        "browser":true,
        "node":true
    },
    "globals":{
        "var1": "writable",
        "var2": "readonly",
        "var3": "off"
    }
}
```

- Environments 预置环境 browser/node/commonjs/shared-node-browser/es6/es2017/es2020/worker/amd

- globals globals是env之外需要额外指定的全局变量，有三种配置值：
            1.writeable - 可写
            2.readonly - 只读
            3.off - 不支持
##### Rules

ESLint无默认开启规则，但提供了推荐开启的规则："extends"："eslint:recommended",可在这里查看所有内置规则的列表：https://eslint.org/docs/rules/

```json
{
    "rules": {
        // 允许非全等号
        "eqeqeq": "off",
        // 尽可能使用花括号
        "curly": "error",
        // 双引号
        "quotes": ["error","double"],
        // 除了warn和error之外，对console.*方法发出警告
        "no-console": ["warn", { "allow": ["warn","error"]}],
        // 必须写分号，除了lastInOneLineBlock
        "semi": [2, "always", {"omitLastInOneLineBlock":true}],
        // plugin1 中的规则，不是内置规则
        "plugin1/rule1": "error"
    }
}
```

- 错误级别
    1."off"或0 关闭规则
    2."warn"或1 将规则视为一个警告
    3."error"或2 将规则视为一个错误
- 配置形式
    1.值：数字或字符串，表示错误级别
    2.数组：第一项是错误级别，之后的各项是对该规则的额外的配置

##### Plugins
ESLint的插件是对一系列rules、environments、globals、processors等配置的封装，以eslint-plugin-vue为例：
```js
// eslint-plugin-vue 的入口文件index.js
// 这个配置集成好了一些配置，用户如果有需要可以直接继承它，不需要额外指定
module.exports = {
    rules: {
        'array-bracket-newline': require('./rules/array-bracket-newline'),
        'array-bracket-spacing': require('./rules/array-bracket-spacing'),
        'arrow-spacing': require('./rules/arrow-spacing')
        // ......
    },
    config: {
        base: require('./configs/base'),
        essential: require('./configs/essential'),
        'no-layout-rules': require('./configs/no-layout-rules'),
        recommended: require('./configs/recommended')
        // .....
    },
    // processors 在被ESLlint处理之前都会被eslint-plugin-vue处理一便
    processors: {
     '.vue': require('./processor')
  }
}
```

- 使用方式

    1.可以单独引用规则
    2.可以直接使用（继承）eslint-plugin-vue配置好的config
    3.预处理器的作用：解析.vue文件

##### Plugins的使用
使用eslint-plugin-vue的vue工程为例子
```js
module.exports= {
    root: true,
    env: {
        node: true
    },
    extends: [
        "plugin:vue/essential", // eslint-plugin-vue
        "eslint:recomended",
        "@vue/prettier"
    ],
    parseroption: {
        parser: "babel-eslint"
    },
    rules: {
        "no-console": process.env.NODE_ENV === "production"
            ? "warn" : "off",
        "no-debugger":  process.env.NODE_ENV === "production"
            ? "warn" : "off"
    }
}
```
更灵活的配置：
```js
{
    "plugins": [
        "vue", // eslint-plugin-vue
        "html"
    ],
    "rules": {
        "vue/no-unused-vars": "error",
        "vue/array-bracket-spacing": "error"
    }
    // ...
}
```

`extends和plugins的区别，extends是全家桶，继承插件的全部配置；plugins是DIY,是根据自己的情况在Rules里进行配置`

##### Extends

Extends是一种非常灵活的ESLint配置机制，使用Extends可以依次递归地应用每一个eslint配置文件，实现灵活的组合

```json
 {
     // 继承单个配置
     "extends": "eslint:recommended",
     // 继承多个配置，后面的可能覆盖前面的
     "extends": ["eslint:recommended","plugin:react/recommended"],
     "extends": [
         "./node_modules/coding-standard/eslintDefaults.js",
         "./node_modules/coding-standard/.eslintrc-es6",
         "./node_modules/coding-standard/.eslintrc-jsx"
     ]
 }

```

- 可以用extends来全家桶式地使用第三方配置好的规则
- extends可以嵌套
- 使用extends之后，我们的rules可以覆盖重写第三方规则、只改变第三方规则的错误等级、添加新的规则

### 编写自己的ESLint规则
规则：no-caller，禁止argutments.caller和arguments.callee的使用

- meta部分主要包括规则的描述、类别、文档地址、修复方式以及配置下schema等信息
- create则需要定义一个函数用于返回一个包含遍历规则的对象，并且该函数会接收context对象作为参数
- [ESLint开发指南](https://eslint.org/docs/developer-guide/architecture)

```js
module.exports= {
    meta:{
        type:"suggestion",
        docs: {
            descripttion: "disallow the use of `arguments.caller`"+"or `arguments.callee`",
            category:"Best Practices",
            recommended: false,
            url: "http://eslint.org/docs/rules/no-caller"
        },
        schema: [],
        messages: {
            unexpected: "Avoid arguments.{{prop}}"
        }
    },
    create(context) {
        return {
            MemberExpression(node){
                const objetName = node.object.name,
                      propertyName= node.property.name;
                if(objectName === "arguments" &&
                  !node.computed && // 必须是静态的属性访问方式a.b而不是a[b]
                  propertyName &&
                  propertyName.match(/^calle[er]$/u)
                ){
                    context.report({ // context eslint全局上下文，report输出错误日志
                        node, // 出错的节点
                        messageId: "unexpexted", // 报错的提示信息
                        data : { prop : propertyName} // prop 和meta中的message结合渲染出正确的提示信息
                    })
                }
            }
        }
    }
}
```

- 案例：检查class是否包含constructor构造方法

利用这个网站astexplorer比较有constructor和没有constructor的变化，然后劫持ClassDeclaration 看里面的节点是否有MethodDefinition和kind是不是constructor

```js
// no-constructor.js
module.exports ={
    meta: {
        docs: {
            description:  "required class constructor",
            category: "Best Practices",
            recommended: true
        },
        fixable: null,
        schema: []
    },
    create: function(context){
        return {
            ClassDeclaration(node){
                const body = node.body.body;
                const result = body.some(
                    element => element.type === 'MethodDefinition' && element.kind === 'constructor'
                )
                if(!result){
                    context.report({
                        node,
                        message: 'no constuctor found'
                    })
                }
            }
        }
    }
}
```
- meta部分
- create部分-在什么时机价差？-ClassDeclaration
- create部分-怎么检查？-遍历AST
- 怎么知道AST的结构呢？astexplorer- 

### Stylelint 介绍

Stylelint是目前生态最丰富的样式代码检查方案，主要有如下特点：

- 社区活跃
- 插件化，功能强大
- 不仅支持css，还支持scss、sass和less等预处理器
- 已在Facebook、GitHub和WordPress等大厂得到广泛应用

### 建立代码规范- Prettier
- prettier是啥？
一个流行的代码格式化的工具

- 为什么需要Prettier
    1.Prettier称自己最大的作用是：可以让大家停止对“代码格式”的无意义的辩论。
    2.Prettier在一众工程化工具中非常特- 殊，它毫不掩饰地称自己是“有主见的”，且严格控制配置项的数量，它对默认格式的选择，完全遵循让可读性最高这一标准
    3.Prettier认为，在代码格式化方面牺牲一些灵活性，可以让开发者带来更多的收益，不得承认Prettier是对的。

##### Prettier VS Linters
Prettier认为lint规则分为两类

1.格式优化类：max-len、no-mixed-spaces-and-tabs、keyword-spacing、comma-style
2.代码质量类：no-unused-vars、no-extra-bind、no-implicit-globals、prefer-promise-reject-errors
prettier
只关注第一类，且不会以报错的形式告知格式问题，而是在允许开发者按自己的方式编写代码，但是会在 特定时机（save、commit）将代码格式化 为可读性最好的形式

##### Prettier的配置
```json
// .prettierrc 
{
    "parser": "babylon", //使用parser
    "printWidth": 80, // 换行字符串阀值
    "tabWidth": 2,   // 缩进空格数
    "useTabs": false, // 使用空格缩进
    "semi": true // 句末加分号
    //......
}
```

##### Prettier使用

在很多方式去触发Prettier的格式化行为：Cli、Watch Changes、git hook 与linter集成
- Watch Changes
```json

// package.json
{
    "script": {
        "prettier-watch": "onchange '**/*.js --prettier --write {{changed}}"
    }
}
```

##### 与ESlint集成
```js
yarn add --dev eslint-config-prettier eslint-plugin-prettier
```
eslint-config-prettier : 禁止eslin中与prettier相冲突的规则，当eslint与prettier相冲突时，eslint的规则不会报错。 eslint-plugin-prettier：让eslint以prettier的规则去检查代码，格式化的代码全部听prettier。
```json
// .eslintrc.json
{
    "extends": ["plugin:prettier/recommended"],
    "plugins": ["prettier"],
    "rules": {
        "prettier/prettier": "error"
    }
}
```

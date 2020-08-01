---
title: 简单实现一个bundler
date: 2020-08-01 14:23:39
tags: 笔记 | Webpack
categories: 笔记
---

# 实现一个简单的bundler

- ### 前言

  对webpack有一定了解的小伙伴知道，webpack是一个静态模块打包器(Module Bundler)，而手动实现一个类似的bundler，对webpack的工作原理会有更加深入的理解。

- ### 缘由

  请看下面一个例子

  ```javascript
  // index.js
  import result from './head.js';
  console.log(result);
  
  // result.js
  import number from './number.js';
  export const result = number * number;
  
  // number.js
  export const number = 2;
  ```

  这个例子是一个最简单的乘法运算，并在index.js中打印出结果，但是这样的代码在浏览器中是无法运行的，那有的小伙伴就要问了，那为什么要这样写呢？你可以去看看模块化的好处。上面的例子就是ES Module，既然它无法运行在浏览器上，那我们就需要去对它进行打包，能让它正确的被浏览器运行，这就是webpack的作用，当然webpack的作用不仅仅如此，它还可以打包CommonJS，AMD，CMD等规范的代码文件， 不过我们在这只举ES Module的例子。

  ##### webpack的打包思路：

  - 利用babel先将单个文件进行代码转换，然后生成单个文件依赖
  - 递归生成依赖图谱
  - 生成最后打包的代码

- ### bundler打包步骤

  - #### 文件结构

    ```
    bundler
    bundler\src\index.js
    bundler\src\number.js
    bundler\src\result.js
    bundler\bundler.js
    ```

  - #### 对单个文件进行代码转换，并生成单个依赖文件

    首先要对代码文件进行转换，就要获取到文件里的内容，这里我们使用nodejs的**fs**模块来获取文件内。

    ```javascript
    // bundler.js
    
    const fs = require('fs');
    // 分析单个文件
    const moduleAnalyser = (filename) => {
        const content = fs.readFileSync(filename, 'utf-8');
    }
    ```

    拿到文件内容后，要对代码进行转换，可以利用babel的**transformFormAst**模块，但是这个模块转换的对象是**AST**（抽象语法树），所以我们还需要将代码先转换为抽象语法树才能正确的转换，babel也提供了**parser**模块。

    ```javascript
    // bundler.js
    
    const fs = require('fs');
    const parser = require('@babel/parser');
    // 分析单个文件
    const moduleAnalyser = (filename) => {
        const content = fs.readFileSync(filename, 'utf-8');
        // 转换为抽象语法树
        const ast = parser.parse(content, {
            sourceType: 'module'
        });
        //转换为可执行代码
        const { code } = babel.transformFromAst(ast, null, {
            presets: ["@babel/preset-env"]
        });
    }
    ```

    到这里，我们已经将代码转换成功，但是还没有生成依赖文件，那依赖文件我们需要找到import声明，然后将所需要的依赖文件的位置找出即可，这里我们使用**traverse**模块中的**ImportDeclaration**方法获取import引入的模块。

    ```javascript
    // bundler.js
    
    const path = require('path');
    const fs = require('fs');
    const parser = require('@babel/parser');
    const traverse = require('@babel/traverse').default;
    // 分析单个文件
    const moduleAnalyser = (filename) => {
        const content = fs.readFileSync(filename, 'utf-8');
        // 转换为抽象语法树
        const ast = parser.parse(content, {
            sourceType: 'module'
        });
        //转换为可执行代码
        const { code } = babel.transformFromAst(ast, null, {
            presets: ["@babel/preset-env"]
        });
        // 保存依赖模块路径
        const dependencis = {};
        // 遍历import，保存依赖路径，便于生成依赖图谱
        traverse(ast, {
            ImportDeclaration({node}) {
                const dirname = path.dirname(filename);
                const newFilename = './' + path.join(dirname, node.source.value);
                dependencies[node.source.value] = newFilename
            }
        });
        return {
            filename,
            dependencies,
            code
        }
    }
    
    const moduleInfo = moduleAnalyser('./src/index.js')
    ```

  - ### 生成依赖图谱

    上面的代码只对index.js进行了转换，但是index.js的依赖result.js也依赖了number.js，所以我们需要分析所有文件的入口文件

    ```javascript
    const makeGraph = (entry) => {
        // 获取入口文件的转换
        const entryModule = moduleAnalyser(entry);
        const graphArray = [entryModule];
        // graphArray储存了所有入口文件的依赖
        for (let i = 0; i < graphArray.length; i ++) {
            const item = graphArray[i];
            const {dependencies} = item;
            if(dependencies) {
                for (let j in dependencies) {
                    graphArray.push(moduleAnalyser(dependencies[j]));
                }
            }
        }
        // 我们需要将数组转换为对象，作为依赖图谱
        const graph = {};
        grapyArray.forEach(item => {
            graph[item.filename] = {
                dependencies: item.dependencies,
                code: item.code
            }
        })
        return graph;
    }
    const graphInfo = makeGraph('./src/index.js')
    ```

  - ### 生成打包代码

    因为require和export在浏览器中都是不存在的，所以我们需要将它们转换

    ```javascript
    const generateCode = (entry) => {
        // 因为对象在字符串中会被转换为 [object, object] 形式所以需要stringify
        const graph = JSON.stringify(makeGraph(entry));
        return `(function(graph) {
            function require(module) {
                function localRequire(relativePath) {
                    return require(graph[module].dependencies[relativePath])
                }
                var exports = {}
                (function(require, exports code){
                    eval(code)
                })(localRequire, exports, graph[module].code)
                return exports;
            }
            require('${entry}');
        })(${graph})`
    }
    ```

    最后返回的代码就可以在浏览器上运行了。
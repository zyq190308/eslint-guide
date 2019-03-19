# ESLint

eslint是前端针对js代码的工具，目的就是为了统一的团队的代码风格以及规范，避免不必要的bug。

### github分支说明

1. master为基础版eslint
2. webpack-eslint是eslint配合webpack版本
3. githook-eslint是eslint配合git的钩子版本


### 安装

安装分为当前项目安装和全局安装，不建议全局装。

```
npm install eslint --save-dev // 当前项目
```

```
npm install -g eslint // 全局
```

### 配置文件初始化

安装后还需要一个配置文件，可通过以下命令生成:
```
eslint --init //全局安装的可以这么写
```
```
./node_modules/eslint/bin/eslint.js --init // 当前项目安装
```
该过程会提示你选择一些选项，不懂得话直接都是默认就行了，然后会生成类似.eslintrc.(js|json等)后缀的文件，这个就是eslint的配置文件。

### 配置文件详解

配置eslint也有2中方式
1. 通过注释配置
2. 配置文件

#### 注释配置 

这个的意思就是关闭对alert, console等的检查，这种适合特殊处理，不具有普遍性，不推荐。
```
/* eslint-disable no-alert, no-console */

alert('foo');
console.log('bar');

/* global var1, var2 */ //注释声明一些全局变量
```


#### 配置文件

介绍配置文件常见的参数
```javascript
module.exports = {
    root: 'true', // ESLint 一旦发现配置文件中有 "root": true，它就会停止在父级目录中寻找。
    parser: 'babel-eslint', //设置解析器, 一般情况下默认解析器就行，如果你需要检查例如flow以及一些eslint识别不了的特性时才使用
    parserOptions: {
        "ecmaVersion": 6,
        "sourceType": "module",
        "ecmaFeatures": {
            "jsx": true 
        }
    }, // 解析器的配置
    "extends": [
        "standard",
        "plugin:react/recommended"
    ], //默认的检查范围有限，这个参数是继承一些比较出名的eslint配置，开箱即用，这里用的eslint-config-standard，比较出名的还有eslint-config-airbnb，具体用法去npm搜索
    "rules": {
        "semi": [2, "never"], // 禁止分号
        "quotes": [2, "single"], // 只能用单引号
        "comma-dangle":  ["error", "never"]
    },// 这里只列举了几个示例，第一个参数是报错的级别，第二个是希望设置的值，有三个值off,warn,error(对应0,1,2)，具体的参数配置,详情看https://eslint.org/docs/rules/
    "env": {
        "browser": true,
        "node": true,
        "es6": true
        
    }, // 设置eslint所处的环境，避免检查一些全局变量，环境可以选多个,比如你去掉node,就会报一些变量undefined的错
    "globals": {
        "_": true,
        "$": true
    }, // eslint检查跳过的一些全局变量，这里我举的例子是lodash,jquery
    "plugins": [
        "react"，
        "html"
    ], // eslint的一些插件
}
```


### ESLint使用场景
控制eslint有以下几种方式
1. eslint直接指定要检查的文件
2. 配合webpack使用(eslint-loader)
3. git hook(提交代码之前检查)

#### eslint指定文件

简单粗暴的办法，直接指定你需要检查的文件。
```shell
./node_modules/eslint/bin/eslint.js myfile.js
```

#### 配合webpack使用

配合webpack使用需要安装eslint-loader和eslint
```
npm install eslint-loader eslint --save-dev
```

webpack配置,"pre" 是在类似babel-loader，vue-loader插件解析前就检查。
```javascript
module.exports = {
  // ...
  module: {
    rules: [
      {
        enforce: "pre",
        test: /\.js$/,
        exclude: /node_modules/,
        loader: "eslint-loader"
      },
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: "babel-loader"
      }
    ]
  }
  // ...
};
```
[eslint-loader配置方法地址](https://www.npmjs.com/package/eslint-loader)


#### 配合git hook使用

我们也可以在代码提交前做一些代码检查工作，首先安装2个插件
```shell
npm install --save-dev husky lint-staged
```
然后在package.json配置如下：
```javascript
// ...
"husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
 },
 "lint-staged": {
    "src/*.js": [
      "eslint --fix", //加上--fix可以自动修复
      "git add"
    ]
 }
 // ...
```
当你提交时，在没加--fix时，会提示错误的地方。

还有一种方法如下，找到项目的.git/hook/pre-commit,
如下覆盖：
```shell
#!/bin/bash


for file in $(git diff --cached --name-only | grep -E '\.(js|jsx)$')

do

  git show ":$file" | node_modules/.bin/eslint --stdin --stdin-filename "$file" # we only want to lint the staged changes, not any un-staged changes

  if [ $? -ne 0 ]; then

    echo "ESLint failed on staged file '$file'. Please check your code and try again. You can run ESLint manually via npm run eslint."

    exit 1 # exit with failure status

  fi

done
```

### 其他

1. 创建.eslintignore，可以让eslint知道那些文件不需要检查。
2. 在vscode搜索eslint插件安装,安装后
打开vscode 首选项/设置， 然后搜索eslint,找到
Eslint: Enable，Eslint: Auto Fix On Save，一个是开启IDE eslint检查，一个是保存时自动修复不符合规范的。
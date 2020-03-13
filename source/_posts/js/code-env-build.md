---
title: js编码环境搭建
date: 2019-01-09 12:20:21
thumbnail: 
categories:
    - js
tags:
    - js
---

## 搭建一个属于自己的javascript开发环境

&emsp;&emsp;在进行js代码开发的时候，为了提升我们的工作效率，我们会使用一些开发工具，所谓适合自己的才是最高效的，分享下个人使用的开发环境。由于现在的js项目有了eslint的约束，都会越来越规范化。对于习惯了随手编码的我们，这个约束虽然有点高要求，但是还是需要遵守的，那么就可以借助一些工具来完成；

<!-- more -->

### 选择开发工具

当然简单的方式是我们直接选择使用IDE，类似webstrom这样的集成开发工具，但是接触了许多的前端开发，大多都不太喜欢用webstrom来进行开发，因为它比较的笨重。而比较推荐的是`vs code`编辑器。

### 添加插件

既然是编辑器，它的基础功能肯定没有IDE那样强大，不过，可以通过插件将其配置的很容易使用，并且也不会和IDE一样笨重；

1. 预览markdown效果
Markdown Preview Enhanced

2. vue代码提示
Vetur

3. react代码提示
React/Redux/react-router Snippets
Simple React Snippets

4. eslint检测并提供快速解决方案
ESLint

5. 代码格式化
Prettier - Code formatter 


### 配置保存动作

当代码保存的时候，可以自动使用插件来格式化代码：
首先找到 vs code 的setting配置，然后找到Text Editor 中的 `Code Actions On Save`配置，将下面的配置保存进去；这样装了的插件在按下保存的时候会自动格式化代码。 

``` json 
{
    "editor.formatOnPaste": true,
    "editor.multiCursorModifier": "ctrlCmd",
    "editor.snippetSuggestions": "top",
    "git.autofetch": true,
    "eslint.nodePath": null,
    "window.zoomLevel": 1,
    "editor.fontSize": 14,
    "editor.tabSize": 2,
    "javascript.preferences.quoteStyle": "single",
    "javascript.implicitProjectConfig.experimentalDecorators": true,
    "javascript.validate.enable": false,
    "files.associations": {
        "*.wxml": "wxml",
        "*.cjson": "jsonc",
        "*.wxss": "css",
        "*.wxs": "javascript"
    },
    "emmet.includeLanguages": {
        "wxml": "html",
        "vue-html": "html",
        "javascript": "javascriptreact"
    },
    "workbench.editor.tabSizing": "fit",
    "emmet.triggerExpansionOnTab": true,
    "editor.insertSpaces": false,
    "[javascript]": {
        "editor.defaultFormatter": "vscode.typescript-language-features"
    },
    "editor.detectIndentation": true,
    "javascript.format.enable": true,
    "[json]": {
        "editor.defaultFormatter": "esbenp.prettier-vscode"
    },
    "terminal.integrated.shell.osx": "/bin/zsh",
    "[html]": {
        "editor.defaultFormatter": "esbenp.prettier-vscode"
    },
    "terminal.integrated.fontSize": 14,
    "terminal.integrated.lineHeight": 1.25,
    "editor.suggestSelection": "first",
    "vsintellicode.modify.editor.suggestSelection": "automaticallyOverrodeDefaultValue",
    "terminal.integrated.rendererType": "dom",
    "files.exclude": {
        "**/.classpath": true,
        "**/.project": true,
        "**/.settings": true,
        "**/.factorypath": true
    },
    "javascript.updateImportsOnFileMove.enabled": "never",
    "html.format.wrapLineLength": 200,
    "vetur.format.defaultFormatter.js": "prettier-eslint",
    "terminal.integrated.shell.windows": "C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe",
    "editor.formatOnSave": false,
    "editor.codeActionsOnSave": {
        "source.fixAll.eslint": true
    },
    "[react]": {
        "editor.codeActionsOnSave": {
            "source.fixAll.eslint": false
        }
    },
    "workbench.colorTheme": "Monokai",
    "files.eol": "\n",
    "files.trimFinalNewlines": true,
    "files.insertFinalNewline": true,
    "[vue]": {
        "editor.defaultFormatter": "octref.vetur"
    },
    "vetur.validation.template": false
}
```

### 关于换行符

在windows中的换行符是`CRLF`，也就是`\r\n`;
在unix和mac中的换行符是`LF`, 也就是`\n`;
在如果在eslint中开启了换行符的强制校验，统一换行符就会比较必要，不然编译的时候就会出现一大堆的警告，对于目前来说大多都统一成`LF`的格式（可能是因为前端开发用mac的比较多的原因把...windows开发有苦难说...）。在上面的配置中`files.eol`已经将换行符配置成了`LF`的格式。
最后还有一点需要注意的，在代码提交的时候，如果使用的`git`,那么默认配置在加入git跟踪的时候会将换行符转换成   `CRLF`。所以需要配置`git`的配置：
``` bash
    git config --global core.autocrlf input
```
以上配置将不再修改输入的换行符。

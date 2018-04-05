title: 使用mermain用Markdown的语法画流程图和UML图
date: 2018-04-05 17:24:30
categories:
tags: Tool
description:
---
之前一直想找一种免费的工具可以用文本的方式画UML图，这样最大的好处是可以放进代码库中比如git，然后很方便的更新和看diff。之前尝试过[JUMLY](http://jumly.tmtk.net/)，用Javascript画sequence图很不错，也一直再用，最近发现了[mermain](https://github.com/knsv/mermaid)，可以用类似markdown的语法来画sequence图，类图，流程图，Gantt图。在它的[文档](https://mermaidjs.github.io/)里可以看到对各种编辑器的支持。比如在Visual Studio Code里装上[Markdown Preview Mermaid Support](https://marketplace.visualstudio.com/items?itemName=bierner.markdown-mermaid)插件，就能直接看UML图了，如下所示。

![preview mermain in VS Code](https://raw.githubusercontent.com/mjbvz/vscode-markdown-mermaid/master/docs/example.png)
# Vue VSCode Snippets 插件定制化

可在vscode扩展里面搜索Vue VSCode Snippets进行安装，相关的指令可见具体的文档说明


## snippets 定制化配置
可以对模板语法进行个性化配置
一些语法详细可参考[Visual Studio Code 使用 User Snippets 和正则自定义字符片段](https://www.jamxe.com/p/20210206e8c2/)一文。如若想修改，可打开对应的文件路径C:\Users\suc\.vscode\extensions\sdras.vue-vscode-snippets-3.1.1\snippets修改


以下是个人在vue.json文件中对vue2的自定义模板
```json
 "Vue Single File Component": {
    "prefix": "vbase",
    "body": [
      "<template>",
      "\t<div class='container'>",
      "",
      "\t</div>",
      "</template>",
      "",
      "<script>",
      "\texport default {",
      "\t\tcomponent: {", "\t\t},",
      "",
      "\t\tprops: {",
      "\t\t\t${1:propName}: {",
      "\t\t\t\ttype: ${2:String},",
      "\t\t\t\tdefault: ''",
      "\t\t\t},",
      "\t\t},",
      "",
      "\t\tdata() {", "\t\t\treturn {", "\t\t\t\tkey: value", "\t\t\t}", "\t\t},",
      "",
      "\t\tmounted () {", "\t\t\t${0};", "\t\t},",
      "",
      "\t\tmethods: {", "\t\t\tgetData() {", "\t\t\t}", "\t\t},",
      "",
      "\t\t${0}",
      "\t}",
      "</script>",
      "",
      "<style lang=\"scss\" scoped>",
      "",
      "</style>"
    ],
    "description": "Base for Vue File with SCSS"
  },
```

## 在VScode中自定义指令模板

- 按「Alt」键切换菜单栏，通过 文件 > 首选项 > 用户代码片段，选择进入目的语言的代码段设置文件；

- 通过快捷键「Ctrl + Shift + P」打开命令窗口（all command window），输入「snippet」，通过候选栏中的选项进入目的语言的代码段设置文件。

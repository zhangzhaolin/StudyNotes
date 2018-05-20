# 在 Visual Studio Code 中使用 markdown 插件

[![version](https://img.shields.io/vscode-marketplace/v/yzhang.markdown-all-in-one.svg?style=flat-square)](https://marketplace.visualstudio.com/items?itemName=yzhang.markdown-all-in-one)  
[![installs](https://img.shields.io/vscode-marketplace/d/yzhang.markdown-all-in-one.svg?style=flat-square)](https://marketplace.visualstudio.com/items?itemName=yzhang.markdown-all-in-one)  
[![AppVeyor](https://img.shields.io/appveyor/ci/neilsustc/vscode-markdown.svg?style=flat-square&label=appveyor%20build)](https://ci.appveyor.com/project/neilsustc/vscode-markdown)  
[![GitHub stars](https://img.shields.io/github/stars/neilsustc/vscode-markdown.svg?style=flat-square&label=github%20stars)](https://github.com/neilsustc/vscode-markdown)

及您所需 (快捷键, 目录,自动预览和更多功能).

## 实现

- **快捷键** (字体加粗, 斜体, 代码块, 删除线以及标题)
  - 提示: `**word|**` -> `**word**|` (<kbd>Ctrl</kbd> + <kbd>B</kbd>)
  - 如果没有选择文本, *鼠标* 左右将会设置加粗样式 (*the entire list item* if you are toggling strikethrough)
- **目录** (没有像 `<!-- TOC -->`这种烦人的标签)
  - 想要TOC标签和github兼容, 你需要将`githubCompatibility` 设置为 `true`
- **大纲视图** 见 explorer 视图
- **自动显示预览** 当打开一个 Markdown 文件时 (默认为禁用状态)
- **将 Markdown 转换成 HTML 文件**
  - 如果您想要共享自己的文件, 建议您使用浏览器(例如谷歌浏览器)将导出的 HTML 打印成 PDF 格式
- **列表编辑** (在列表项末尾使用 <kbd>Enter</kbd> 继续使用列表) (同样适用于引用块)
  - 在列表项的开始处使用 <kbd>Tab</kbd> 将会缩进它 
  - 在列表项的开始处使用 <kbd>Backspace</kbd> 将会取消缩进(或者删除列表标记)
  - 空白的列表项可以使用 <kbd>Enter</kbd> 来删除
  - 当您缩进/减少缩进或者向上/向下移动一行后,有序列表将会自动被修复.
- **GitHub Flavored Markdown**
  - 表格格式 (<kbd>Alt</kbd> + <kbd>Shift</kbd> + <kbd>F</kbd>)
  - 任务列表 (使用 <kbd>Alt</kbd> + <kbd>C</kbd> 来检查/取消选中任务列表)
- **数学渲染** (见下方截图)
- **字典提示** (已经移动到单独的扩展中 [Dictionary Completion](https://marketplace.visualstudio.com/items?itemName=yzhang.dictionary-completion))
- **其他**
  - 使用 "打开预览" 按键覆盖 "切换预览", 这意味着您可以使用相同的按键绑定来关闭预览 (<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>V</kbd> 或者 <kbd>Ctrl</kbd> + <kbd>K</kbd> <kbd>V</kbd>).

### 键盘快捷键

<!-- ![shortcuts1](images/gifs/bold-normal.gif) -->

![shortcuts2](https://github.com/neilsustc/vscode-markdown/blob/master/images/gifs/bold-quick.gif)

![shortcuts3](https://github.com/neilsustc/vscode-markdown/blob/master/images/gifs/heading.gif)

### 目录

![toc](https://github.com/neilsustc/vscode-markdown/blob/master/images/gifs/toc.gif)

### 列表

![list editing](https://github.com/neilsustc/vscode-markdown/blob/master/images/gifs/list-editing.gif)

### 表格格式化

![table formatter](https://github.com/neilsustc/vscode-markdown/blob/master/images/gifs/table-formatter.gif)

### 大纲

![outline](https://github.com/neilsustc/vscode-markdown/blob/master/images/outline.png)

### 任务列表

![task lists](https://github.com/neilsustc/vscode-markdown/blob/master/images/gifs/tasklists.gif)

### 数学渲染

![math rendering](https://github.com/neilsustc/vscode-markdown/blob/master/images/math.png)

## 快捷键

| 按键                                              | 命令                         |
| ------------------------------------------------- | ---------------------------- |
| <kbd>Ctrl</kbd> + <kbd>B</kbd>                    | 切换加粗                     |
| <kbd>Ctrl</kbd> + <kbd>I</kbd>                    | 切换斜体                     |
| <kbd>Alt</kbd> + <kbd>S</kbd>                     | 切换删除线                   |
| <kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>]</kbd> | 切换标题(上一级)             |
| <kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>[</kbd> | 切换标题 (下一级)            |
| <kbd>Ctrl</kbd> + <kbd>M</kbd>                    | 切换数学环境                 |
| <kbd>Alt</kbd> + <kbd>C</kbd>                     | 检查/取消选中任务列表项      |

## 可使用命令

- Markdown: Create Table of Contents(创建目录)
- Markdown: Update Table of Contents(更新目录)
- Markdown: Toggle code span(切换代码块)
- Markdown: Print current document to HTML(将文件打印成HTML格式)

## 支持设置

| 名称                                               | 默认      | 描述                                                              |
| -------------------------------------------------- | --------- | ----------------------------------------------------------------- |
| `markdown.extension.toc.levels`                    | `1..6`    | 控制标题级别用来显示在目录中.                                     |
| `markdown.extension.toc.unorderedList.marker`      | `-`       | 在目录中使用 `-`, `*` 或者 `+`  (用于无序列表)                    |
| `markdown.extension.toc.orderedList`               | `false`   | 在目录中使用有序列表.                                             |
| `markdown.extension.toc.plaintext`                 | `false`   | 仅是纯文本.                                                       |
| `markdown.extension.toc.updateOnSave`              | `true`    | 在保存时自动更新目录.                                             |
| `markdown.extension.toc.githubCompatibility`       | `false`   | GitHub 兼容性                                                     |
| `markdown.extension.preview.autoShowPreviewToSide` | `false`   | 打开Markdown文件时自动显示预览.                                   |
| `markdown.extension.orderedList.marker`            | `ordered` | 或者为 `one`: 总是使用 `1.` 来标记有序列表                        |
| `markdown.extension.italic.indicator`              | `*`       | 使用 `*` 或者 `_` 来包装斜体文字                                  |
| `markdown.extension.showExplorer`                  | `true`    | 在面板中显示大纲视图                                              |
| `markdown.extension.print.absoluteImgPath`         | `true`    | 将图像路径转换为绝对路径                                          |

## 更新日志

查看 [CHANGELOG](CHANGELOG.md) 获取更多讯息.

## Latest CI Build

Download it [here](https://ci.appveyor.com/project/neilsustc/vscode-markdown/build/artifacts)

## 合作

Bugs, feature requests and more, in [GitHub Issues](https://github.com/neilsustc/vscode-markdown/issues).

Or leave a review on [vscode marketplace](https://marketplace.visualstudio.com/items?itemName=yzhang.markdown-all-in-one#review-details) 😉.

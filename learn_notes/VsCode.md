Visual Studio Code是个牛逼的编辑器，启动非常快，完全可以用来代替其他文本文件编辑工具。又可以用来做开发，支持各种语言，相比其他IDE，轻量级完全可配置还集成Git感觉非常的适合前端开发，是微软亲生的想必TypeScript会支持的非常好。 所以我仔细研究了一下文档未来可能会作为主力工具使用。

# 主命令框 Command Palette

最重要的功能就是`F1`或`Ctrl+Shift+P`打开的命令面板了，在这个命令框里可以执行VSCode的任何一条命令，可以查看每条命令对应的快捷键，甚至可以关闭这个编辑器。

按一下`Backspace`会进入到`Ctrl+P`模式里

# Ctrl+P 模式

在`Ctrl+P`下输入`>`又可以回到主命令框 `Ctrl+Shift+P`模式。

在`Ctrl+P`窗口下还可以

*   直接输入文件名，快速打开文件
*   `?` 列出当前可执行的动作
*   `!` 显示Errors或Warnings，也可以`Ctrl+Shift+M`
*   `:` 跳转到行数，也可以`Ctrl+G`直接进入
*   `@` 跳转到symbol（搜索变量或者函数），也可以`Ctrl+Shift+O`直接进入
*   `@:`根据分类跳转symbol，查找属性或函数，也可以`Ctrl+Shift+O`后输入`:`进入
*   `#` 根据名字查找symbol，也可以`Ctrl+T`

# 常用快捷键

* * *

## 编辑器与窗口管理

### 同时打开多个窗口（查看多个项目）

*   打开一个新窗口： `Ctrl+Shift+N`
*   关闭窗口： `Ctrl+Shift+W`

### 同时打开多个编辑器（查看多个文件）

*   新建文件 `Ctrl+N`
*   历史打开文件之间切换 `Ctrl+Tab`，`Alt+Left`，`Alt+Right`
*   切出一个新的编辑器（最多3个）`Ctrl+\`，也可以按住Ctrl鼠标点击Explorer里的文件名
*   左中右3个编辑器的快捷键`Ctrl+1` `Ctrl+2` `Ctrl+3`
*   3个编辑器之间循环切换 Ctrl+`
*   编辑器换位置，`Ctrl+k`然后按`Left`或`Right`

## 代码编辑

### 格式调整

*   代码行缩进`Ctrl+[`， `Ctrl+]`
*   折叠打开代码块 `Ctrl+Shift+[`， `Ctrl+Shift+]`
*   `Ctrl+C` `Ctrl+V`如果不选中，默认复制或剪切一整行
*   代码格式化：`Shift+Alt+F`，或`Ctrl+Shift+P`后输入`format code`
*   修剪空格`Ctrl+Shift+X`
*   上下移动一行： `Alt+Up` 或 `Alt+Down`
*   向上向下复制一行： `Shift+Alt+Up`或`Shift+Alt+Down`
*   在当前行下边插入一行`Ctrl+Enter`
*   在当前行上方插入一行`Ctrl+Shift+Enter`
*   整块右移`Tab`
*   整块左移`Shift+Tab`

### 光标相关

*   移动到行首：`Home`
*   移动到行尾：`End`
*   移动到文件结尾：`Ctrl+End`
*   移动到文件开头：`Ctrl+Home`
*   移动到后半个括号 `Ctrl+Shift+]`
*   选中当前行`Ctrl+i`
*   选择从光标到行尾`Shift+End`
*   选择从行首到光标处`Shift+Home`
*   删除光标右侧的所有字`Ctrl+Delete`
*   Shrink/expand selection： `Shift+Alt+Left`和`Shift+Alt+Right`
*   Multi-Cursor：可以连续选择多处，然后一起修改，`Alt+Click`添加cursor或者`Ctrl+Alt+Down` 或 `Ctrl+Alt+Up`
*   同时选中所有匹配的`Ctrl+Shift+L`
*   `Ctrl+D`下一个匹配的也被选中(被我自定义成删除当前行了，见下边`Ctrl+Shift+K`)
*   回退上一个光标操作`Ctrl+U`

### 重构代码

*   跳转到定义处：`F12`
*   定义处缩略图：只看一眼而不跳转过去`Alt+F12`
*   列出所有的引用：`Shift+F12`
*   同时修改本文件中所有匹配的：`Ctrl+F12`
*   重命名：比如要修改一个方法名，可以选中后按`F2`，输入新的名字，回车，会发现所有的文件都修改过了。
*   跳转到下一个Error或Warning：当有多个错误时可以按`F8`逐个跳转
*   查看diff 在explorer里选择文件右键 `Set file to compare`，然后需要对比的文件上右键选择`Compare with 'file_name_you_chose'`.

### 查找替换

*   查找 `Ctrl+F`
*   查找替换 `Ctrl+H`
*   整个文件夹中查找 `Ctrl+Shift+F`  
    匹配符：
*   `*` to match one or more characters in a path segment
*   `?` to match on one character in a path segment
*   `**` to match any number of path segments ,including none
*   `{}` to group conditions (e.g. `{**/*.html,**/*.txt}` matches all html and txt files)
*   `[]` to declare a range of characters to match (e.g., `example.[0-9]` to match on `example.0`,`example.1`, …

## 显示相关

*   全屏：`F11`
*   zoomIn/zoomOut：`Ctrl + =`/`Ctrl + -`
*   侧边栏显/隐：`Ctrl+B`
*   侧边栏4大功能显示：
    *   Show Explorer `Ctrl+Shift+E`
    *   Show Search`Ctrl+Shift+F`
    *   Show Git`Ctrl+Shift+G`
    *   Show Debug`Ctrl+Shift+D`
*   Show Output`Ctrl+Shift+U`
*   预览markdown`Ctrl+Shift+V`

## 其他

*   自动保存：File -> AutoSave ，或者`Ctrl+Shift+P`，输入 auto

# 皮肤预览

`f1`后输入 `theme` 回车，然后上下键即可预览

# 自定义settings.json

## `User settings` 是全局设置，任何vs Code打开的项目都会依此配置。

默认存储在:

Windows: `%APPDATA%\Code\User\settings.json`  
Mac: `$HOME/Library/Application Support/Code/User/settings.json`  
Linux: `$HOME/.config/Code/User/settings.json`

## `Workspace settings` 是本工作区的设置，会覆盖上边的配置

存储在工作区的`.vocode`文件夹下。

几乎所有设定都在`settings.json`里，包括

> *   Editor Configuration - font, word wrapping, tab size, line numbers, indentation, …
> *   Window Configuration - restore folders, zoom level, …
> *   Files Configuration - excluded file filters, default encoding, trim trailing whitespace, …
> *   File Explorer Configuration - encoding, WORKING FILES behavior, …
> *   HTTP Configuration - proxy settings
> *   Search Configuration - file exclude filters
> *   Git Configuration - disable Git integration, auto fetch behavior
> *   Telemetry Configuration - disable telemetry reporting, crash reporting
> *   HTML Configuration - HTML format configuration
> *   CSS Configuration - CSS linting configuration
> *   JavaScript Configuration - Language specific settings
> *   JSON Configuration - Schemas associated with certain JSON files
> *   Markdown Preview Configuration - Add a custom CSS to the Markdown preview
> *   Less Configuration - Control linting for Less
> *   Sass Configuration - Control linting for Sass
> *   TypeScript Configuration - Language specific settings
> *   PHP Configuration - PHP linter configuration

例如可以修改让vscode认识`.glsl`扩展名

    {

        // Configure file associations to languages (e.g. "*.extension": "html"). These have precedence over the default associations of the languages installed.
        "files.associations": {
            "*.glsl": "shaderlab"
        }
    }

## 修改默认快捷键

* * *

File -> Preferences -> Keyboard Shortcuts
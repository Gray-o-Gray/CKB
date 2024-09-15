# whitespace+工具
用于显示空格及回车符的工具

# vscode自带工具

ctrl + shift + P

选择view toggle Render Whitespace


## 使用插件命令
以下是使用 Whitespace+ 插件的几条重要命令，以及如何在 VSCode 中执行这些命令。

1. Toggle the display of the extension
    
    这个命令用于切换插件的显示状态。

    打开命令面板：Ctrl + Shift + P（Windows 和 Linux），Cmd + Shift + P（Mac）。

    输入并选择 Whitespace+ Toggle。

    这将切换插件的显示状态，让您可以方便地开启或关闭空白字符显示。

2. Change the display mode
    
    这个命令用于更改显示模式，可以选择只显示尾随空白字符或显示所有空白字符。

    打开命令面板：Ctrl + Shift + P（Windows 和 Linux），Cmd + Shift + P（Mac）。
    
    输入并选择 Whitespace+ Mode。
    
    根据需要选择合适的显示模式。

3. Open the configuration file
    
    这个命令用于打开配置文件，您可以在这里进行更详细的设置和个性化配置。

    打开命令面板：Ctrl + Shift + P（Windows 和 Linux），Cmd + Shift + P（Mac）。
    
    输入并选择 Whitespace+ Config。
    
    这将打开插件的配置文件，您可以在这里编辑插件的行为和显示样式。

## 快捷方式设置（可选）

为了方便使用，您可以为这些命令设置快捷键。

打开 keybindings.json：

打开命令面板：Ctrl + Shift + P（Windows 和 Linux），Cmd + Shift + P（Mac）。

输入并选择 Preferences: Open Keyboard Shortcuts (JSON)。

添加以下配置来自定义快捷键：
```
[
    {
        "key": "ctrl+alt+w", // 你可以选择你喜欢的快捷键
        "command": "whitespacePlus.toggle",
        "when": "editorTextFocus"
    },
    {
        "key": "ctrl+alt+m", // 你可以选择你喜欢的快捷键
        "command": "whitespacePlus.mode",
        "when": "editorTextFocus"
    },
    {
        "key": "ctrl+alt+c", // 你可以选择你喜欢的快捷键
        "command": "whitespacePlus.config",
        "when": "editorTextFocus"
    }
]
```
​
这样，您就可以通过快捷键快速切换空白字符的显示状态、显示模式，或打开配置文件。

通过上述步骤，您可以有效利用 Whitespace+ 插件来控制和显示回车符（以及其他空白字符）。这些设置可以帮助您更好地查看和管理代码中的空白字符，提高代码的可读性和一致性。

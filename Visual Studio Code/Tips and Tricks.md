# Tips and Tricks
**打开命令面板** : `Ctrl+Shift+P`

在这里可以看到所有命令的快捷键(如果有的话)

**快速打开文件** : `Ctrl+P`

重复该命令可以在最近打开的文件中快速切换, 按 <kbd> → </kbd> 可以打开多个文件

**使用命令行**

> 使用命令行的前提是在安装 Code 时添加了环境变量

- 打开当前目录: `code .` 
- 在最近使用的窗口中打开当前目录: `code -r .` 
- 新建窗口: `code -n` 
- 更改语言: `code --locale=es` 
- 比较文件: `code --diff <file1> <file2>` 
- 在文件的某行的某个位置打开: `code --goto package.json:10:5` 
- 禁用所有插件: `code --disable-extensions .` 
- 查看 Code 版本: `code --version` 
- 查看帮助: `code --help` 

**打开问题面板** : `Ctrl+Shift+M`

通过 `F8` 或 `Shift+F8` 可以在问题之间切换

**切换语言模式** : `Ctrl+K M`

**自定义**

- 更改主题:  `Ctrl+K Ctrl+T`
- 自定义快捷键: `Ctrl+K Ctrl+S`
- 用户设置: `Ctrl+,`

**安装插件** `Ctrl+Shift+X`

**文件和文件夹**

- 打开终端: ``Ctrl+` ``
- 自动保存: 在 User Settings 中设置 `"file.autoSave": "afterDlay"`
- 切换侧栏: `Ctrl+B`
- 全屏: `Ctrl+K Z`
- 并排编辑: `Ctrl+\`
- 在并排编辑的标签中切换: `Ctrl+1`, `Ctrl+2`, `Ctrl+3`
- 打开资源管理器窗口: `Ctrl+Shift+E`
- 关闭当前打开的文件夹: `Ctrl+F4`
- 切换标签页: `Ctrl+Tab`
- 为某种类型文件添加关联语言: 在 User Settings 中设置 "files.associations", 比如:
    ```
    "files.associations": {
        ".database": "json"
    }
    ```

**常用编辑**

- 在文件中查找Symbol(这个Symbol该如何翻译): `Ctrl+Shift+O`
- 在工作区中查找Symbol: `Ctrl+t`
- 去掉尾部空格: `Ctrl+K Ctrl+X`
- 格式化选中代码: `Ctrl+K Ctrl+F`
- 格式化整个文档: `Shift+Alt+F`
- 打开Markdown预览: `Ctrl+Shift+V`
- 并排编辑Markdown文档: `Ctrl+K V`

**集成Git**

VS Code 自带了Git: `Ctrl+Shift+G`

- 查看差异: 打开源码管理面板, 点击要查看的文件即可
- 切换分支: 点击状态栏中的分支按钮
- 暂存: 
    - 暂存所有: 点击 `+`  
    - 暂存选中部分: 选中文件片段, 然后在命令面板中输入 **Stage Selected Ranges**
- 查看Git的输出: `Ctrl+Shift+U` 在下拉项中选择Git
- 设置VS Code为Git的默认合并工具: `git config --global merge.tool code`
- 设置VS Code为Git的默认编辑器: 

    ```
    git config --global core.editor "code --wait"
    git config --global -e
    在配置文件中加入:
    [diff]
        tool = difault-difftool
    [difftool "default-difftool"]
        cmd = code --wait --diff $LOCAL $REMOTE
    ```

    然后便可以使用 `git difftool` 来比较文件
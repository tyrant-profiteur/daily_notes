### Windows右键新建菜单中添加md文件

1. 创建文件a.txt，添加如下内容：

   Windows Registry Editor Version 5.00
   [HKEY_CLASSES_ROOT\.md\ShellNew]
   "NullFile"=""
   "FileName"="template.md"

2. 文件改为a.reg

3. 双击运行




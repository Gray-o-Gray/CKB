# iptables 命令的基本构成
iptables [-t TABLE] [COMMAND] [OPTION]

TABLE 默认为filter，可选{raw nat mangle filter}

## COMMAND
- -A, --append chain rule-specification
  
  将一个或多个规则附加在所选链的末尾。当源地址和/或目的地址解析为多个地址时，将为每个可能的地址组合添加一条规则。

- -D, --delete chain rule-specfication
  
  -D, --delete chain rulenum
  
  从所选链中删除一条或多条规则。该命令有两个版本：可以将规则指定为链中的顺序序号（第一个规则从1开始）或要匹配的具体规则。

- -I, --insert chain [rulenum] rule-specification
  
  在选定的链中插入一个或多个规则作为给定的规则编号。因此，如果规则编号为1，则将一条或多条规则插入到链的头部。如果未指定规则编号，则使用默认值1。

- -R, --replace chain rulenum rule-specification
  
  替换所选链中的规则。如果源地址和/或目标地址解析为多个地址，该命令将失败。规则编号从1开始。

- -L, --list [chain]
  
  列出所选链中的所有规则。如果未选择链，则列出所有链。
  
  该命令默认在filter表中进行处理，如果想查看其他table的内容，需要使用-t [table] 参数。

- -F, --flush [chain]
  
  刷新选定的链（如果没有指定，则将刷新表中所有的链）。该操作等同于清空链，会将所有的规则一一删除。

- -Z, --zero [chain]
  
  将所有链中的数据包和字节计数器清空。指定 -L、--list 选项也是合法，与 -L 结合可以在计数器被清除前查看它们。

- -N, --new-chain chain
  
  根据给定名称创建新的用户自定义链。新链的名字必须唯一，不可与已有链相同。

- -X, --delete-chain [chain]
  
  删除指定的用户自定义链。
    - 删除条件：
    - 
- -P, --policy chain target
- -E, --rename-chain old-chain new-chain
- -h



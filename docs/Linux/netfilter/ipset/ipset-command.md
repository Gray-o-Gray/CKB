# ipset常用命令
```bash

# 获得ipset当前的配置内容
ipset save

# 保存ipset配置到指定文件
ipset save -f ipset.config
ipset save > ipset.config

# 获得指定集合的配置
ipset save test_set
ipset save test_set -f ipset_test_set.config

# 恢复指定集合的配置
ipset restore -f ipset_test_set.config
```
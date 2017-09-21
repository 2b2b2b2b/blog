## 文件搜索命令locate
# 命令：locate 文件名
在后台数据库中按文件名搜索，搜索熟读更快，数据库大约一天自动更新一次

### 数据库目录：/var/lib/mlocate （不同系统会有差异）
如果记不住可以 通过 locate locate命令来寻找

### 强制更新该数据库命令：updatedb

### 命令根据 /etc/updatedb.conf配置文件去搜索
- PRUNE_BIND_MOUNTS 配置是否生效  默认生效
- PRUNEFS 搜索时，不搜索文件系统
- PRUNENAMES 搜索时不收缩文件的类型
- PRUNEPATHS 搜索时的路径

## 搜索命令的命令whereis
- 搜索命令所在路径以及帮助文档位置
- 选项-b 只查找可执行文件
- 选项-m 只查找帮助文件
- 不可搜索到shell指令

## 搜索命令的命令which
- 搜索命令所在的路径&别名
- 不可搜索到shell指令

## vim注意事项
### 配置事项
* vim退出后恢复终端内容

```
有些终端在vim退出后可以恢复到打开vim前终端的状态，这些都是TERM环境变量设置问题。环境变量TERM设置为终端类型，linux的终端信息放在   /usr/share/terminfo下，在这个目录的子目录v下就有许多的如vt100,vt102,vt200等。  
vim退出后无法恢复终端内容的解决方案如下： 
1. 设置TERM环境变量为linux或xterm或xterm-color，可以在.bashrc文件中添加：export TERM=linux  
2. 设置vim的t_ti和t_te变量的值（可选，例如在centos上就不需要）  
    用vim打开一个文件，normal模式下输入:set t_ti 或者 :set t_te，若值类似："^[[?1049h" and "^[[?1049l"，那么你需要在.vimrc中加入下面几行：
    if &term =~ "xterm"
        " SecureCRT versions prior to 6.1.x do not support 4-digit DECSET
        "     let &t_ti = "\<Esc>[?1049h"
        "     let &t_te = "\<Esc>[?1049l"
        " Use 2-digit DECSET instead
        let &t_ti = "\<Esc>[?47h"
        let &t_te = "\<Esc>[?47l"
    endif
```
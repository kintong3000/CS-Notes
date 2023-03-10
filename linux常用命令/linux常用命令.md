

ls命令

``` shell
ls -a //列出目录所有文件，包含以.开始的隐藏文件
ls -A //列出除.及..的其它文件
ls -l (==ll)//除了文件名之外，还将文件的权限、所有者、文件大小等信息详细列出来
```

cp命令 – 复制文件或目录

``` shell
cp [参数] 源文件 目标文件
-r //复制目录及目录内所有项目
-i	//若目标文件已存在，则会询问是否覆盖

```

mkdir 创建目录文件

``` shell
mkdir [参数] 目录
mkdir -m 700 file // 建立目录的同时设置目录的权限

```

mv命令 – 移动或改名文件

``` shell
mv [参数] 源文件 目标文件
-i //若存在同名文件，则向用户询问是否覆盖

```



cat命令 – 在终端设备上显示文件内容

echo命令 – 输出字符串或提取后的变量值

rm命令 – 删除文件或目录

``` shell
rm [参数] 文件
rm -i *.log //删除前会询问用户是否操作
rm -r test //递归删除

```

grep命令 – 强大的文本搜索工具

``` shell
 grep [参数] 文件
 
```

tail命令 – 查看文件尾部内容

``` shell
tail [参数] 文件
-f //持续显示文件最新追加的内容
```

ps命令 – 显示进程状态

``` shell
ps [参数]
ps aux	//显示系统中全部的进程信息
ps -aux | grep apache //与grep联用查找某进程

```

tar命令 – 压缩和解压缩文件

``` shell
tar 参数 文件或目录
tar czvf file //使用gzip压缩格式对某个目录进行打包操作，显示压缩过程，压缩包规范后缀为.tar.gz
tar xvf backup4.tar //解压某个压缩包到当前工作目录
```


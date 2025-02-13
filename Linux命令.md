### 0.Linux复制粘贴命令

Ctrl + Shift + C

Ctrl + Shift + V

### 1.返回

cd ..                  返回上一级目录

cd ../..               返回上两级目录

cd或cd ~           返回home目录

cd /                   返回根目录

cd -                   返回刚才的目录

cd - 目录名       返回指定目录

#### pwd：显示当前工作目录的路径

### 2. C语言命令行

```c
#include <stdio.h>
void main(int argc,char** argv)
{
        printf("%d\n",argc);
        printf("%s\n",argv[0]);
        printf("%s\n",argv[1]);
        printf("%s\n",argv[2]);
}
```

在上面的例子中，我们给main函数传递两个参数，argc和argv。argc是int类型的，它表示的是命令行参数的个数。不许要用户传递，它会根据用户从命令行输入的参数个数，自动确定。argv是char**类型的，它的作用是存储用户从命令行传递进来的参数。它的第一个成员是用户运行的程序名字。

### 3.程序退出的艺术

C语言中：exit()和return的最大区别：return只会结束当前函数，而exit()则会结束整个进程。

命令行中：ctal+c：终止进程

### 4.文件和文件夹

当年目录下创建文件夹：

```
mkdir dirname
```

touch：当年目录下创建空文件或更新文件的时间戳

```
touch file_name
```

rmdir：删除空目录

```
rmdir directory_name
```

rm：删除文件或目录

```
rm file_name
rm -r directory_name  # 递归删除目录及其内容
```

cp：复制文件或目录

```
cp source_file destination
cp -r source_directory destination  # 递归复制目录及其内容
```

mv：移动或重命名文件或目录

```
mv old_name new_name
```



### 5.Linux下“/”和“~”的区别

”/“是根目录，”~“是家目录。[Linux](https://so.csdn.net/so/search?q=Linux&spm=1001.2101.3001.7020)存储是以挂载的方式，相当于是树状的，源头就是”/“，也就是根目录。而每个用户都有”家“目录，也就是用户的个人目录，比如root用户的”家“目录就是/root,普通用户a的家目录就是/home/a.



### 6.命令行传参

```c
int main(int argc, char *argv[])
```

argc为传入的参数个数（1+），argv为参数地址的数组，argv[0]为调用程序的路径（名称）

当前目录下写一个parameter_passing.c：

```
./parameter_passing 2 3
```

输出：

```
argc = 3
原始argv[0] = ./parameter_passing
原始argv[1] = 2
原始argv[2] = 3
不加atoi_argv[1] = 1631797100
加atoi_argv[1] = 2
加atoi后argv[1] + argv[2] = 5
```

### 7.Linux apt 命令

apt（Advanced Packaging Tool）是一个在 Debian 和 Ubuntu 中的 Shell 前端软件包管理器。

apt 命令提供了查找、安装、升级、删除某一个、一组甚至全部软件包的命令，而且命令简洁而又好记。

apt 命令执行需要超级管理员权限(root)。

#### apt 语法

```
  apt [options] [command] [package ...]
```

- **options：**可选，选项包括 -h（帮助），-y（当安装过程提示选择全部为"yes"），-q（不显示安装的过程）等等。
- **command：**要进行的操作。
- **package**：安装的包名。

#### apt 常用命令

- 列出所有可更新的软件清单命令：**sudo apt update**

- 升级软件包：**sudo apt upgrade**

  列出可更新的软件包及版本信息：**apt list --upgradable**

  升级软件包，升级前先删除需要更新软件包：**sudo apt full-upgrade**

- 安装指定的软件命令：**sudo apt install <package_name>**

  安装多个软件包：**sudo apt install <package_1> <package_2> <package_3>**

- 更新指定的软件命令：**sudo apt update <package_name>**

- 显示软件包具体信息,例如：版本号，安装大小，依赖关系等等：**sudo apt show <package_name>**

- 删除软件包命令：**sudo apt remove <package_name>**

- 清理不再使用的依赖和库文件: **sudo apt autoremove**

- 移除软件包及配置文件: **sudo apt purge <package_name>**

- 查找软件包命令： **sudo apt search <keyword>**

- 列出所有已安装的包：**apt list --installed**

- 列出所有已安装的包的版本信息：**apt list --all-versions**

### 8.linux下载github中的文件

在安装了git后：

```bash
git clone +网址
```

![img](https://i-blog.csdnimg.cn/blog_migrate/a906ac875fbc3eaac254c240fb487bac.png)

默认下载在当前目录

#### 下载网络文件：

```
!wget https://data.vision.ee.ethz.ch/cvl/DIV2K/DIV2K_train_HR.zip
```

or

```
!curl https://data.vision.ee.ethz.ch/cvl/DIV2K/DIV2K_train_HR.zip
```

解压：

```
unzip ...
```

### 9.**安装路径问题**

- 有时候 Python 可能会因为路径问题无法找到安装的模块。你可以检查 

  ```
  sys.path
  ```

   来确认当前 Python 是否能够找到安装的模块：

  ```
  import sys
  print(sys.path)
  ```

  确保安装目录在路径中。如果没有，可以手动将该路径添加到 Python 环境中：

  ```
  sys.path.append('/path/to/your/module')
  ```

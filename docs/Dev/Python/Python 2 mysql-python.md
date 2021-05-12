# Python 2 安装 mysql-python 库相关的问题

公司有一批使用 Python 2 和 mysql-python 库的脚本，配置环境比较痛苦，这里记录下。

## Windows

安装 Python2 然后 [到这里](https://www.lfd.uci.edu/~gohlke/pythonlibs/#mysql-python) 找到相应 wheel，pip 命令安装即可。

## Linux

本人使用 Ubuntu 20.04，首先安装 [pyenv](https://github.com/pyenv/pyenv)，之后使用 pyenv 安装 py2.7：

``` shell
pyenv install 2.7.18
```

设置环境变量（或使用 py 虚拟环境）：

``` shell
alias active_py27='export PATH=/home/bolitao/.pyenv/versions/2.7.18/bin:$PATH'
```

到这里下载 c-connector：[MySQL :: Download MySQL Connector/C (Archived Versions)](https://downloads.mysql.com/archives/c-c/)，解压到 ~/apps。设置环境变量：

``` shell
export PATH=/path/to/mysql-connector-c-6.1.11-linux-glibc2.12-x86_64/bin:$PATH
```

编辑刚才下载的 c-connector `mysql_config` 文件，原来为：

``` shell
# Create options 
libs="-L$pkglibdir"
libs="$libs -l "
```

修改为：

``` shell
# Create options 
libs="-L$pkglibdir"
# libs="$libs -l "
libs="$libs -lmysqlclient -lssl -lcrypto"
```

至此应该可以安装这个 mysql-python 库了：

``` shell
active_py27
pip install MySQL-python==1.2.5

# output:
DEPRECATION: Python 2.7 will reach the end of its life on January 1st, 2020. Please upgrade your Python as Python 2.7 won't be maintained after that date. A future version of pip will drop support for Python 2.7. More details about Python 2 support in pip, can be found at https://pip.pypa.io/en/latest/development/release-process/#python-2-support
Looking in indexes: https://pypi.douban.com/simple
Collecting MySQL-python==1.2.5
  Using cached https://pypi.doubanio.com/packages/a5/e9/51b544da85a36a68debe7a7091f068d802fc515a3a202652828c73453cad/MySQL-python-1.2.5.zip
Installing collected packages: MySQL-python
  Running setup.py install for MySQL-python ... done
Successfully installed MySQL-python-1.2.5
```

库虽然装好了，但这玩意运行是基于 mysql-client clib 的，如果系统没这东西还是报错给你看：

``` log
ImportError: libmysqlclient.so.18: cannot open shared object file: No such file or directory
```

把刚才 mysql-connector-c 的 lib 库链接到系统：

``` shell
sudo -i
echo "/path/to/mysql-connector-c-6.1.11-linux-glibc2.12-x86_64/lib" >> /etc/ld.so.conf
ldconfig
exit
```

之后 MySQL-python 库就可用了。

## REF

- [pip3 install mysqlclient fails on macOS · Issue #169 · PyMySQL/mysqlclient](https://github.com/PyMySQL/mysqlclient/issues/169)

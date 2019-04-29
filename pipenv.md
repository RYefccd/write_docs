


> Written with [StackEdit](https://stackedit.io/).

#  pipenv


这是一个新的 python 虚拟环境以及包管理的工具, 是 pypa 默认推荐的包管理系统. 此工具将原有的 pip 和 virtualenv 整合, 使得创建虚拟环境, 管理应用依赖更为智能, 严格.


##  操作步骤

 1. pip3 install requirement.txt
 2. 进入到 gt-server 目录
 3. 执行 pipenv 自检命令

```shell
查看当前项目的目录
ryefccd@fccd:~/wh/gt-server$ pipenv --where
/home/ryefccd/wh/gt-server

查看当前项目使用的虚拟环境.(初次使用没有)
ryefccd@fccd:~/wh/dsb$ pipenv --venv
No virtualenv has been created for this project yet!
Aborted!

pipenv install 
```


**注意**: 开发最终要是确保线下代码环境通过之后, 冻结环境依赖,
产生 Pipfile.lock 文件.



##  运维操作步骤

运维主要通过通过开发在产生并在本地冻结的依赖文件 Pipfile.lock 来重建环境.

```
cd gt-server
同步环境
pipenv install --deploy
查看虚拟环境位置:
pipenv --venv
查看虚拟环境依赖
pipenv run pip list
pipenv graph

```


##  supervisor

    利用 pipenv 动态生成环境变量配置
    /bin/sh -c 'LD_LIBRARY_PATH=$(pipenv --venv)/lib:${LD_LIBRARY_PATH} pipenv run python xxxx.py'



# Dev

## Tar File

```bash
tar zcvf a.tar.gz
tar z
```

## Virtual Environment

### Create venv

```bash
virtualenv -p python3.8 [venv-name]
source [venv-name]/bin/activate
deactivate
```

### Install Ipykernel

```bash
pip install ipykernel
ipython kernel install --user --name=[venv-name]
# uninstall
jupyter-kernelspec uninstall [venv-name]
```

### Copy PyTorch and Torchvision

```bash
# global
cd /usr/local/lib/python3.8/dist-packages/

# copy
cp -r torch <venv-path>/lib/python3.8/site-packages/
cp -r torch-1.12.0+cu113.dist-info/ <venv-path>/lib/python3.8/site-packages/
cp -r torchvision-0.13.0+cu113.dist-info <venv-path>/lib/python3.8/site-packages/
cp -r torchvision* <venv-path>/lib/python3.8/site-packages/
```


## Docker

```bash
## start daemon
sudo service docker start
```

## Poetry

```bash
## initialize
mkdir poetry-demo
cd poetry-demo
poetry init
# 或是他幫你創一個全新的
poetry new poetry-demo

## create venv
poetry env use <python>
# 預設是統一管理
poetry config --list
# 改成創建 .venv/ 在 local
poetry config virtualenvs.in-project true

## activate & deactivate
poetry shell
exit

## update poetry.lock
poetry lock

## install
poetry add <package> # 主要依賴
poetry add <package> --dev # 開發依賴

## list
poetry show --tree

## uninstall
poetry remove <package>

## freeze
poetry export -f requirements.txt -o requirements.txt --without-hashes

```

ref
- https://python-poetry.org/
- https://blog.kyomind.tw/python-poetry/

## Pre-Commmit

遇到 isort error: `RPC failed; curl 56 GnuTLS recv error (-110): The TLS` 

```bash=
export GIT_SSL_NO_VERIFY=1
```

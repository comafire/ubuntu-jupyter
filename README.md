# Ubuntu에서 Data Science를 위한 Native 개발환경 설정

## Sudoers

sudo 명령을 통해 권한을 업그레이드 하는 경우가 많기 때문에 자동 수행을 위해 암호 없이 사용 가능하도록 설정합니다.

visudo 명령을 이용하여 sudo 권한 사용자가 암호를 넣지 않아도 되도록 NOPASSWD 설정을 추가합니다.

```bash
> sudo EDITOR=vim visudo

%sudo   ALL=(ALL:ALL) NOPASSWD:ALL

```

사용자를 sudo 그룹에 추가하고 id 명령을 통해 현재 잘 추가 되었는지 확인합니다.

```bash
> sudo usermod -aG sudo $USER
> id
uid=192435321($USER) gid=27(sudo)
```

id를 통해서 sudo gid가 잘 적용되지 않을 경우에는 system reboot을 해줍니다.

## Git

소스의 버전을 관리하기 위해 버전 관리 툴인 git을 설치합니다.

```bash
> sudo apt-get update
> sudo apt install git-all
```

Git Manual
* Git 간편안내서: https://rogerdudler.github.io/git-guide/index.ko.html
* Git Book: https://git-scm.com/book/ko/v2
* Git Flow(Git 개발 브렌치 전략): https://danielkummer.github.io/git-flow-cheatsheet/index.ko_KR.html

### Git Multi SSH Key 이용

소스 버전별 저장소는 github.com 또는 gitlab.com 을 많이 이용합니다. github 및 gitlab 사용시 인증은 간편하게 http 인증을 사용할 수 있으나, 프로젝트가 많아지면 commit 시 마다 비번을 넣어야 하는 불편함과 한 컴퓨터에서 gitlab, github내에 여러 계정이 혼재 할 경우 http 인증이 동작하지 않기에 SSH Key를 이용한 인증 방법을 추천합니다.

먼저 sshkey-gen 명령을 통해서 각 git 계정 마다 사용할 키를 생성해줍니다. 여기서는 A, B, C 3개의 계정 예를 들어 보겠습니다.

```bash
> ssh-keygen -t rsa -b 4096 -C "A@A.com"
Generating public/private rsa key pair.
Enter file in which to save the key (~/.ssh/id_rsa): a_id_rsa

> ssh-keygen -t rsa -b 4096 -C "B@B.com"
Generating public/private rsa key pair.
Enter file in which to save the key (~/.ssh/id_rsa): b_id_rsa

> ssh-keygen -t rsa -b 4096 -C "C@C.com"
Generating public/private rsa key pair.
Enter file in which to save the key (~/.ssh/id_rsa): c_id_rsa
```
키를 생성하게 되면 A, B, C 계정을 위한 개인키와 공개키 쌍이 생성되게 됩니다.

* 개인키: a_id_rsa, b_id_rsa, c_id_rsa
* 공개키: a_id_rsa.pub, b_id_rsa.pub, c_id_rsa.pub

.ssh 디렉토리에 생성한 개인키와 공개키를 복사합니다. 이제 생성된 개인키를 SSH 에 등록해 줍니다.

```bash
ssh-add ~/.ssh/a_id_rsa
ssh-add ~/.ssh/b_id_rsa
ssh-add ~/.ssh/c_id_rsa
```

등록된 키는 ssh-add -l 명령어로 확인이 가능합니다.

생성된 공개키를 각 사이트 계정의 SSH Key 등록하는 곳에 등록해 주면 됩니다. SSH Key를 등록하는 방법은 아래 링크를 참조하세요.

* Github: https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/
* Gitlab: https://about.gitlab.com/2014/03/04/add-ssh-key-screencast/ 

SSH를 이용한 접속시 자동으로 해당 키를 사용할 수 있도록 ssh 접속 설정을 합니다.

~/.ssh/config 파일을 생성해서 아래와 같이 설정해 줍니다.

```bash
Host A.github.com
        HostName github.com
        User A
        PreferredAuthentications publickey
        IdentityFile ~/.ssh/a_id_rsa
Host B.gitlab.com
        HostName gitlab.com
        User B
        PreferredAuthentications publickey
        IdentityFile ~/.ssh/b_id_rsa
Host C.gitlab.com
        HostName gitlab.com
        User C
        PreferredAuthentications publickey
        IdentityFile ~/.ssh/c_id_rsa
```

소스의 git 레포지토리로 이동하여 .git/config 파일에서 Host부분을 ~/.ssh/config 에 설정한 Host와 일치 시킵니다.

```bash
[remote "origin"]
        url = git@A.github.com:comafire/test.git
        fetch = +refs/heads/*:refs/remotes/origin/*
```

이제 git 사용시 각 계정 마다 다른 키 파일을 이용하여 암호를 넣지 않고 편리하게 사용 할 수 있습니다.

## pyenv, pyenv-virtualenv, direnv

프로젝트를 진행할때, 파이썬의 특성상 파이썬 자체의 버전과 패키지 버전들에 대한 의존성이 생기게 됩니다. 프로젝트 별로 편리하게 의존성을 관리하기 위해 3가지 툴을 조합하여 사용합니다.

* pyenv: 로컬에 멀티 버전의 파이썬을 설치/사용, 파이썬 버전에 대한 의존성을 해결
* virtualenv: 로컬에 멀티 파이썬 환경을 설치/사용, 프로젝트별 패키지에 대한 의존성을 해결
* direnv: 프로젝트 디렉토리에 들어갈때 마다 자동 환경 셋팅, .bash_profile 과 비슷한 역활

3가지 툴을 설치하고 기본 셋팅합니다.

**direnv**

```bash
> sudo apt-get update
> sudo apt-get install -y --no-install-recommends direnv
> echo 'eval "$(direnv hook bash)"' >> ~/.bashrc
> source ~/.bashrc 
```

**pyenv, virtualenv**
```bash
> sudo apt-get update
> sudo apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev \
libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev \
xz-utils tk-dev libffi-dev liblzma-dev

> curl https://pyenv.run | bash
```

pyenv를 이용해 파이썬 여러 버전을 설치할수 있습니다. 파이썬 빌드시 필요한 의존성 패키지와 주로 사용하는 2.x 대와 3.x 대의 최신 버전을 설치해 봅니다. 2.x 의 지원종료가 다가오기에 3.x 사용을 추천합니다. apache spark 과의 호완성을 위해 여기서는 3.7.6 버전을 사용합니다. (https://stackoverflow.com/questions/58700384/how-to-fix-typeerror-an-integer-is-required-got-type-bytes-error-when-tryin) 

```bash
> pyenv install 2.7.16
> pyenv install 3.7.6
> pyenv versions
* system (set by /Users/comafire/.pyenv/version)
  2.7.16
  3.7.6
```

pyenv-virtualenv를 이용해 각 버전의 파이썬 가상환경을 만들어 봅니다.

```bash
> pyenv virtualenv 2.7.16 pyenv-2.7.16
> pyenv virtualenv 3.7.6 ubuntu-jupyter
> pyenv virtualenvs
  ...
  pyenv-2.7.16 (created from /Users/comafire/.pyenv/versions/2.7.16)
  ubuntu-jupyter (created from /Users/comafire/.pyenv/versions/3.7.6)
```

프로젝트를 생성하고 프로젝트의 기본 환경을 자동 셋팅할 수 있도록 direnv를 사용해 봅니다. direnv를 이용하면 프로젝트 폴더에 들어갈때 마다 자동으로 프로젝트에 맞는 특정 파이썬 가상환경을 활성화할 수 있습니다. 방법은 간단히 프로젝트의 최상위 디렉토리 아래 .envrc 파일을 만들고 아래 필요한 환경설정 내용을 적어주면 됩니다.

```bash
> vi .envrc

eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
export PYENV_VIRTUALENV_DISABLE_PROMPT=1

pyenv activate ubuntu-jupyter

```

저장 후, 해당 환경을 활성화 하기 위해 아래 명령을 수행해 줍니다.

```bash
> direnv allow .
```

이후 부터는 .envrc 가 있는 프로젝트 디렉토리에 들어가면 자동으로 .envrc 내에 있는 환경이 활성화되고, 디렉토리를 벗어나면 자동으로 환경이 해제 됩니다.

```bash
> cd ubuntu-jupyter
direnv: loading ~/ubuntu-jupyter/.envrc
direnv: export +PYENV_ACTIVATE_SHELL +PYENV_SHELL +PYENV_VERSION +PYENV_VIRTUALENV_DISABLE_PROMPT +PYENV_VIRTUAL_ENV +VIRTUAL_ENV ~PATH

> cd ../
direnv: unloading
```

## Spark & Jupyter & Python Data Science Library

Ubuntu 에서 Spark 을 설치하여 사용하는 방법은 여러가지가 있지만, 버전 의존성 및 설정의 편리성을 위해 github 저장소에 Spark 및 자주사용하는 Data Science Library 그리고 Jupyter를 띄울수 있도록 템플릿 환경을 저장하여 놓고 사용합니다.

```bash
> git clone https://github.com/comafire/ubuntu-jupyter.git
```

Spark 실행을 의한 의존성 패키지들을 설치 및 다운받아 프로젝트 내부의 usr/local 디렉토리내에 압축을 풀어 줍니다.

openjdk-8: https://www.digitalocean.com/community/tutorials/how-to-install-java-with-apt-on-ubuntu-18-04
scala-2.12.10: https://www.scala-lang.org/download/2.12.11.html
spark-2.4.5-bin-hadoop2.7.tgz: 다운로드

```bash
> mkdir -p usr
> cd usr
> sudo apt install openjdk-8-jdk
> wget https://downloads.lightbend.com/scala/2.12.10/scala-2.12.10.tgz
> wget http://mirror.navercorp.com/apache/spark/spark-2.4.7/spark-2.4.7-bin-hadoop2.7.tgz
> tar -zvxf scala-2.12.10.tgz
> tar -zxvf spark-2.4.7-bin-hadoop2.7.tgz
> ln -s scala-2.12.10 scala
> ln -s spark-2.4.7-bin-hadoop2.7 spark
```

ubuntu-jupyter 내부 디렉토리 안에 미리 .envrc.template 안에 기본 설정이 되어 있으며, 이를 복사하여 추가적으로 필요한 부분은 수정해서 사용할 수 있습니다.

```bash
> cp .envrc.template .envrc
> vi .envrc

# BASE
export UBUNTU_JUPYTER=$PWD

# Java
export JAVA_HOME="/usr/lib/jvm/java-8-openjdk-amd64"
export PATH="$JAVA_HOME/bin:$PATH"
export CPPFLAGS="-I$JAVA_HOME/include"

# Scala
export SCALA_HOME="$UBUNTU_JUPYTER/usr/scala"
export PATH="$SCALA_HOME/bin:$PATH"

# Python
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
export PYENV_VIRTUALENV_DISABLE_PROMPT=1

pyenv activate ubuntu-jupyter

# Spark 
export SPARK_HOME="$UBUNTU_JUPYTER/usr/spark"
export PYSPARK_PYTHON="$(which python3)"
export PYSPARK_DRIVER_PYTHON="$PYSPARK_PYTHON"
export PY4J_VERSION=0.10.7
export PATH="${SPARK_HOME}/bin:$PATH"
export PYTHONPATH="$SPARK_HOME/python/:$PYTHONPATH"
export PYTHONPATH="$SPARK_HOME/python/lib/py4j-$PY4J_VERSION-src.zip:$PYTHONPATH"

# Jupyter
export LOCALE="ko_KR.UTF-8"

# Jupyter
export JUPYTER_PORT="7010" # Your Jupyter Port
export JUPYTER_PASSWORD="notebooks" # Your Jupyter Password
export JUPYTER_BASEURL="jupyter" # Your Jupyter BaseURL, ex) http://localhost:8010/jupyter
export JUPYTER_NOTEBOOKDIR=$PWD

```

이제 필요한 Data Science 라이브러리를 설치합니다. requirements.txt 파일내에 기본 설치 라이브러리가 있으며, 필요시 추가해서 사용하면 됩니다.

```bash
> pip3 install --upgrade --user pip
> pip3 install -r etc/pip/requirements.txt
> python3 -m spylon_kernel install --user
```

이제 아래 명령으로 jupyter 노트북을 띄우게 되면 Ubuntu 상에서 독립적인 가상 환경내에서 프로젝트 진행이 가능하게 됩니다.

```bash
> ./bin/run_jupyter.sh

[I 16:11:31.013 LabApp] Loading IPython parallel extension
[I 16:11:31.248 LabApp] JupyterLab extension loaded from /Users/comafire/.pyenv/versions/3.7.6/envs/macos-jupyter/lib/python3.7/site-packages/jupyterlab
[I 16:11:31.249 LabApp] JupyterLab application directory is /Users/comafire/.pyenv/versions/3.7.6/envs/macos-jupyter/share/jupyter/lab
[I 16:11:31.251 LabApp] Serving notebooks from local directory: /Users/comafire/LocalDrive/macos-jupyter
[I 16:11:31.251 LabApp] The Jupyter Notebook is running at:
[I 16:11:31.252 LabApp] http://localhost:7010/jupyter/
```

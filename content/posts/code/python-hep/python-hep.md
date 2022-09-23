---
title: "实验学家的Python工具箱（附常用依赖库自动化安装）"
tags: ["code"]
date: 2022-09-22T11:26:35+08:00
lastmod: 2022-09-22T12:26:35+08:00
draft: false
---

随着深入使用[ROOT Data Analysis Framework](https://root.cern)，也逐渐发现了它的劣势：基于C++写的分析和绘图程序在代码行数增长以后维护成本不断提高、多人协作的成本偏高（尤其是课题组的新成员想融入进来需要花很多的时间）。幸运的是，ROOT项目组还为Python提供了调用接口（PyROOT），配合使用[Cppyy](https://cppyy.readthedocs.io/en/latest/)来最小程度上嵌入一些C++的代码，以达到易于上手、维护的目的。

Note：也不得不说下Python的劣势，慢！很慢！非常慢！**我更推荐那些性能开销型的程序使用C++来编写，或者在Python中使用Cppyy来编写C++并执行**。在我的使用场景中，一般是绘图采用Python来做。一个好的判断标准是：当你在Python中编写了某个函数，这个函数的运行时间能否在你盯着它运行结束并且耐心消失殆尽之前就执行完毕（如果不行，请把这个功能使用C++来执行，或者换一种构架思路）。


## 使用Pyenv来管理Python的版本（and Jupyter）

Python最让人诟病的就是版本（还有它的PIP软件包管理），[Pyenv](https://github.com/pyenv/pyenv)允许我们在本机上安装多个不同的Python版本并随时切换，这样能够让我们绕开macOS本机的上古Python2.X（系统部分软件对它有依赖），并且绕开Homebrew安装并不断升级的Python3.X（每次Homebrew自动升级了Python都会导致透过PIP安装的软件包失效，需要重新安装很麻烦）。

以下是macOS的安装方法，注意我使用的终端是zsh，若是bash请参考pyenv github的readme，你可以通过 **echo $SHELL** 来查看自己用的是什么终端。

```zsh
brew update
brew install pyenv
echo 'eval "$(pyenv init --path)"' >> ~/.zprofile
```

接着需要完全退出终端并重新打开它，测试一下Pyenv是否成功安装，以下是Pyenv的一些简单使用方法：

```zsh
# show help page
pyenv -h
# show availible python versions
pyenv install --list
# install specific python version
pyenv install 3.9-dev
# show installed versions
pyenv versions
# using specific python version
pyenv global 3.9-dev
# show which python located
pyenv which python
```

注意！不同的系统和ROOT版本，它预编译好的包使用的Python版本是固定的，你需要使用 *root-config --python3-version* 来查看并安装相应的dev版本。例如：3.9.10则需要安装3.9-dev（macOS），而3.8.10则需要安装3.8-dev（linux），以此类推。（dev版本包含了某些库需要的 *#include <Python.h>* 头文件）

还需要把ROOT的库加入PYTHONPATH，像是这样（添加好了以后重启终端）：

```bash
# zsh only, bash user should redirect to ~/.bash_profile
echo 'export PYTHONPATH=$(root-config --libdir):$PYTHONPATH' >> ~/.zprofile
```

## 如何安装第三方库，e.g. FastJet, Pythia8 （自动化安装）

一个良好的习惯是把所有的第三方库（甚至是不同语言编写的）和软件，安装到 *$HOME/.local* 目录下。一般而言，在你编译的时候都可以附加参数 *./configure --prefix=$HOME/.local*。举例经常会用到的粒子物理软件包来说，你下载好fastjet, lhapdf, pythia8并且解压缩进入目录时，

```bash
# compile and install fastjet
./configure PYTHON=$(which python3) --prefix=$HOME/.local --enable-pyext --enable-allcxxplugins
make
make install
```

```bash
# compile and install lhapdf
./configure PYTHON=$(which python3) --prefix=$HOME/.local
make
make install
```

```bash
# compile and install pythia8
./configure --prefix=$HOME/.local --with-lhapdf6=$HOME/.local --with-fastjet3=$HOME/.local --with-python-config=$(which python3-config)
make
make install
```

通过指定prefix以及PYTHON路径（还记得这个Python版本是我们手动通过Pyenv来安装的3.9-dev吗？），这样我们就很好地把ROOT, lhapdf, fastjet, pythia8全部都集成到了Python中去了，现在你可以在一个 *.py* 文件里使用所有这些软件的功能！

别忘记把第三方库的路径加入PYTHONPATH，像是这样（添加好了以后需要重启终端）：

```bash
# zsh only, bash user should redirect to ~/.bash_profile
echo 'export PYTHONPATH=$HOME/.local/lib:$PYTHONPATH' >> ~/.zprofile
```

现在让我们来编写一个简单的小程序，新建 *draw_jet_kinematics.py* 并写入以下内容，它会使用Pythia来生成PP对撞的QCD过程，然后使用Fastjet来进行R=0.4的JetCluster，最后把leading-jet的横动量用ROOT的TH1D画出来：

```python
from pythia8 import Pythia
import fastjet
from ROOT import TH1D

# setup pythia generator as qcd process at pp collision
pythia = Pythia()
pythia.readString("Beams:eCM = 13000.")
pythia.readString("HardQCD:all = on")
pythia.readString("PhaseSpace:pTHatMin = 200.")
# suppress more output
pythia.readString("Next:numberShowInfo = 0")
pythia.readString("Next:numberShowProcess = 0")
pythia.readString("Next:numberShowEvent = 0")
pythia.init()

# define jet cluster algorithm and histogram
r, pT_min = 0.4, 30.0
jet_algos = fastjet.JetDefinition(fastjet.antikt_algorithm, r)
hist = TH1D("leading-jet-pt", "leading-jet-pt", 100, 100, 500)

# loop over events
for ievt in range(1000):
  if not pythia.next(): continue
  inputs = [fastjet.PseudoJet(ptc.px(), ptc.py(), ptc.pz(), ptc.e()) for ptc in pythia.event if ptc.isFinal()]
  seqs = fastjet.ClusterSequence(inputs, jet_algos)
  jets = fastjet.sorted_by_pt(seqs.inclusive_jets(pT_min))
  hist.Fill(jets[0].pt())

# draw histogram
hist.Draw()
_ = input("press any key to continue")
```

运行完毕以后你会看到类似于这样的分布（强子化跑的比较慢，大概需要1-3分钟取决于你的CPU时钟速度）：

![jetpt_spectrum](../jetpt_spectrum.png)

我整理了部分常用的依赖库自动化安装的Bash脚本，它会在你当前目录下创建一个build文件夹开始下载，安装到 *$HOME/.local* 目录中（版本号截止到2022年，以后不定期更新）。

```bash
#!/bin/bash

# you need set these environment variables manually in .bash_profile or .zprofile
# you should also set up a python3.9-dev (or python3.8-dev) version by pyenv or anaconda
# export PATH="$HOME/.local/bin:$PATH"
# export LD_LIBRARY_PATH="$HOME/.local/lib:$LD_LIBRARY_PATH"
# export PYTHONPATH=$HOME/.local/lib:$PYTHONPATH

if [[ ${PWD##*/} != "OfflineExamples" ]]; then
    echo "You need to cd to QCDAnalysis/OfflineExamples before run this script."
    exit 1;
fi
[[ ! -d build ]] && mkdir build
cd build
build_dir=${PWD}
prefix=$HOME/.local

echo "Checking fastjet......"
version="fastjet-3.4.0"
cd $build_dir
curl -o $version.tar.gz http://fastjet.fr/repo/$version.tar.gz
tar -xzf $version.tar.gz
cd $version
./configure PYTHON=$(which python3) --prefix=$prefix --enable-pyext --enable-allcxxplugins
make -j 8
make install

echo "Checking fjcore......"
version="fjcore-3.4.0"
cd $build_dir
curl -o $version.tar.gz http://fastjet.fr/repo/$version.tar.gz
tar -xzf $version.tar.gz
mv $version $prefix/include/fjcore

echo "Checking lhapdf......"
version="LHAPDF-6.4.0"
cd $build_dir
curl -o $version.tar.gz https://lhapdf.hepforge.org/downloads/?f=$version.tar.gz
tar -xzf $version.tar.gz
cd $version
./configure PYTHON=$(which python3) --prefix=$prefix
make -j 8
make install

echo "Checking pythia8......"
version="pythia8306"
cd $build_dir
curl -o $version.tgz https://pythia.org/download/pythia83/$version.tgz
tar -xzf $version.tgz
cd $version
./configure --prefix=$prefix --with-lhapdf6=$prefix --with-fastjet3=$prefix --with-python-config=$(which python3-config)
make -j 8
make install

echo "Checking MG5_aMC@NLO......"
version="MG5_aMC_v3.4.1"
cd $build_dir
wget https://launchpad.net/mg5amcnlo/3.0/3.4.x/+download/$version.tar.gz
tar -xzf $version.tar.gz --one-top-level # extract and create dir named with tarball basename
mv $version/$(ls $version)/* $version/
if [[ ! -d $HOME/.local/opt ]]; then mkdir -p $HOME/.local/opt; fi
cp -r $version $HOME/.local/opt
ln -s $HOME/.local/opt/$version/bin/mg5_aMC $HOME/.local/bin/mg5_aMC
# disable auto-update
echo 'auto_update = 0' >> $HOME/.local/opt/$version/input/mg5_configuration.txt
```


## 配置代码补全和提示（VSCode and NVIM）

暂略，不重要，之后龟速更新。

## 如何组织一个分析（简单的例子）

一般而言，当我组织一个线下的小分析时，会把它分成两到三个部分：

1. 性能开销型的功能块，由C++编写，把ROOT作为运行库来使用，e.g. 构造感兴趣的观测量并储存到Histogram中
2. 让性能开销型的功能块批量多核运行（并行加速），一般而言是Shell Script（可选）
3. 最后是把所有切块的结果合并到一起（可能还包括Mathematica计算得到的理论结果），并进行绘图（样式调整）这部分交给Jupyter Notebook来处理

先鸽了，之后慢慢更新。


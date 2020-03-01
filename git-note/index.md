# Git 学习笔记
[TOC]

---
## 1. Git简介
#### 1.1 什么是版本控制

版本控制是一种记录一个或者若干个文件内容变化，以便将来查阅特定版本修订情况的系统。

许多人习惯复制整个项目目录的方式来保存不同的版本，比如会改名加上备份的时间来区别。这么做的好处是简单但是特别容易犯错，有时候还会混淆所在的工作目录，一不小心就会写错文件或者覆盖意想之外的文件。于是人们在很早就开发了许多种版本控制系统，大多数是采用简单的数据库来记录文件的历次更新差异。

解决方法1：本地版本控制

其中最流行的一种叫做RCS，在Mac OS X系统上安装了开发者工具包之后，也可以使用`rcs`命令。它的工作原理是在硬盘上保存补丁集，补丁是指文件修订前后的变化。通过应用所有的补丁，可以重新计算出各个版本的文件内容。

解决方法2：集中化的版本控制系统

为了解决在不同操作系统上的开发者协同工作，集中化的版本控制系统（Centralized Version Control Systems，简称 CVCS）应运而生。这类系统都有一个单一的集中管理的服务器，保存所有文件的修订版本，而协同工作的人们都通过客户端连接到这台服务器，取出最新的文件或者提交更新。多年以来，这已经成为版本控制系统的标准做法。

这种做法带来了许多好处，特别是相较于老式的本地VCS来说。现在每个人都可以在一定程度上看到项目中的其他人在做些什么。管理员也可以轻松掌控每个开发者的权限，并且管理一个CVCS要远比在各个客户端上维护本地数据库来得轻松容易。

事分两面，有好有坏。这么做显而易见的缺点是中央服务器的单点故障。如果宕机一小时，那么在这一小时内，谁都无法提交更新，也就无法协同工作。如果中心数据库所在的磁盘发生损坏，又没有做恰当备份，则会丢失所有数据——包括项目的整个变更历史，只剩下人们各自机器上保留的单独快照。本地版本控制系统也存在类似问题，只要整个项目的历史记录被保存在单一位置，就有丢失所有历史更新记录的风险。

解决方法3：分布式版本管理系统

于是分布式版本控制系统（Distributed Version Control System，简称 DVCS）面世了。 在这类系统中，像 Git、Mercurial、Bazaar 以及 Darcs 等，客户端并不只提取最新版本的文件快照，而是把代码仓库完整地镜像下来。 这么一来，任何一处协同工作用的服务器发生故障，事后都可以用任何一个镜像出来的本地仓库恢复。 因为每一次的克隆操作，实际上都是一次对代码仓库的完整备份。

更进一步，许多这类系统都可以指定和若干不同的远端代码仓库进行交互。籍此，你就可以在同一个项目中，分别和不同工作小组的人相互协作。 你可以根据需要设定不同的协作流程，比如层次模型式的工作流，而这在以前的集中式系统中是无法实现的。

#### 1.2 Git简史

同生活中的许多伟大事务一样，Git诞生于一个极富纷争大举创新的年代。

Linux 内核开源项目有着为数众多的参与者。 绝大多数的 Linux 内核维护工作都花在了提交补丁和保存归档的繁琐事务上（1991－2002年间）。 到 2002 年，整个项目组开始启用一个专有的分布式版本控制系统 BitKeeper 来管理和维护代码。

到了 2005 年，开发 BitKeeper 的商业公司同 Linux 内核开源社区的合作关系结束，他们收回了 Linux 内核社区免费使用 BitKeeper 的权力。 这就迫使 Linux 开源社区（特别是 Linux 的缔造者 Linus Torvalds）基于使用 BitKeeper 时的经验教训，开发出自己的版本系统。 他们对新的系统制订了若干目标：

- 速度
- 简单的设计
- 对非线性开发模式的强力支持（允许成千上万个并行开发的分支）
- 完全分布式
- 有能力高效管理类似 Linux 内核一样的超大规模项目（速度和数据量）  

自诞生于 2005 年以来，Git 日臻成熟完善，在高度易用的同时，仍然保留着初期设定的目标。 它的速度飞快，极其适合管理大项目，有着令人难以置信的非线性分支管理系统。

## 2. Git基础概念

#### 2.1 直接记录快照，而非差异比较

Git和其他版本控制系统对待数据的方法。概念上来区分，大部分系统以文件变更列表的方式存储信息。这类系统（CVS、Subversion、Perforce、Bazaar 等等）将它们保存的信息看作是一组基本文件和每个文件随时间逐步累积的差异。Git不按照以上方式对待或保存数据。反之，Git更像是把数据看作是对小型文件系统的一组快照。每次提交更新，或在Git中保存项目状态时，它主要对当时的全部文件制作一个快照并保存这个快照的索引。为了高效，如果文件没有修改，Git不再重新储存该文件，而是只保留一个链接指向之前储存的文件。Git对待数据更像是一个快照流。

#### 2.2 近乎所有操作都是本地执行

相比于集中式版本管理系统，Git的绝大多数操作都只需要访问本地文件和资源，一般不需要来自网络上其他计算机的信息。因为在本地磁盘就有项目的完整历史，所以大部分操作看起来瞬间完成。

举个例子，要浏览项目的历史，Git不需要外连接到服务器去获取历史，然后再显示出来——它只需直接从本地数据库中读取。如果想查看当前版本与一个月前的版本之间引入的修改，Git会查找到一个月之前的文件做一次本地的差异计算，而不是由远程服务器处理或者从远程服务器拉回旧版本文件再来本地处理。这就意味着离线时可以进行任何操作，直到有网络时再上传。

#### 2.3 Git保证完整性

Git中所有数据在储存前都计算校验和，然后以校验和来引用。这意味着不可能在Git不知情的情况时更改任何文件内容或目录内容。这个功能构建在Git底层，是Git哲学不可或缺的部分。若在传送过程中丢失信息或损坏文件，Git就能发现。

Git用以计算校验和的机制叫做SHA-1散列（hash，哈希）。这是一个由40个十六进制字符（0-9和a-f）组成的字符串，基于Git中文件的内容或目录结构计算出来。SHA-1哈希看起来是这样：

`24b9da6552252987aa493b52f8696cd6d3b00373`

Git中使用这种哈希值的情况很多，经常能看到这种哈希值。实际上Git数据库中保存的信息都是以文件内容的哈希值来索引，而不是文件名。

#### 2.4 Git一般只添加数据

执行的Git操作，几乎只往Git数据库中添加数据。很难让Git执行任何不可逆的操作，或者让它以任何方式清除数据。同别的VCS一样，为提交更新时有可能丢失或弄乱修改的内容；但是一旦提交快照到Git中，就难以再丢失数据。

#### 2.5 三种状态

Git有三种状态，文件可能处于其中之一：已提交（committed）、已修改（modified）和已暂存（staged）。已提交表示数据已经安全的保存在本地数据库中。已修改表示修改了文件，但还没保存到数据库中。已暂存表示对一个已修改文件的当前版本做了标记，使之包含在下次提交的快照中。

由此引入Git项目的三个工作区域的概念：Git仓库、工作目录以及暂存区域。

![工作目录、暂存区域以及 Git 仓库。](https://git-scm.com/book/en/v2/images/areas.png)

Git仓库目录是Git用来保存项目的元数据和对象数据库的地方。这是Git中最重要的部分，从其他计算机克隆仓库时，拷贝的就是这里的数据。

工作目录是对项目的某个版本独立提取出来的内容。这些从Git仓库的压缩数据库中提取出来的文件，放在磁盘上供你使用或修改。

暂存区域是一个文件，保存了下次将提交的文件列表信息，一般在Git仓库目录中，有时候也被称作“索引”，不过一般说法还是叫做暂存区域。

基本的Git工作流程如下：

1. 在工作目录中修改文件。
2. 暂存文件，将文件的快照放入暂存区域。
3. 提交更新，找到暂存区域的文件，将快照永久性储存到Git仓库目录。

如果Git目录中保存着特定版本的文件，就属于已提交状态。如果作了修改并已经放入暂存区域，就属于已暂存状态。如果自上次取出后，作了修改但是还没有放入暂存区域，就是已修改状态。后面将学会如何根据文件状态实施后续操作，以及怎样跳过暂存直接提交。

## 3. Git安装和配置

#### 3.1 安装

在Windows上安装Git有几种方法。官方版本可以在Git官方网站下载，这是一个名为Git for Windows的项目（也叫做msysGit），是和Git分开独立的项目。

另一个方法是安装GitHub for Windows，其包含了图形化和命令行版本的Git，能够支持Powershell，提供了稳定的凭证缓存和健全的换行设置。

在Linux下使用软件包管理程序安装或者从源码编译安装。

#### 3.2 初次运行Git前的配置

Git自带一个`git config`工具来帮助设置控制Git外观和行为的配置变量。这些变量储存在三个不同的位置：

1. `/etc/gitconfig`文件：包含系统上每一个用户及他们仓库的通用配置。如果使用带有`--system`选项的`git config`时，它会从此文件读写配置变量。
2. `~/.gitconfig`或`~/.config/git/config`文件：只针对当前用户。可以传递`--global`选项让Git读写此文件。
3. 当前使用仓库的Git目录中的`config`文件（就是.git/config）：针对该仓库。

每一个级别覆盖上一级别的配置，所以`.git/config`的配置变量会覆盖`/etc/gitconfig`中的配置变量。

##### 用户信息

当安装完Git应该做的第一件事就是设置你的用户名称与邮件地址。这很重要，因为每一个Git的提交都会使用这些信息，并且它会写入到你的每一次提交中，不可更改：
```bash
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"
```

再次强调，如果使用了 `--global` 选项，那么该命令只需要运行一次，因为之后无论你在该系统上做任何事情， Git 都会使用那些信息。 当你想针对特定项目使用不同的用户名称与邮件地址时，可以在那个项目目录下运行没有 `--global` 选项的命令来配置。

##### 文本编辑器

既然用户信息已经设置完毕，你可以配置默认文本编辑器了，当 Git 需要你输入信息时会调用它。 如果未配置，Git 会使用操作系统默认的文本编辑器，通常是 Vim。 如果你想使用不同的文本编辑器，例如 Emacs，可以这样做：

```bash
$ git config --global core.editor emacs
```

##### 检查配置信息

如果想要检查你的配置，可以使用 `git config --list` 命令来列出所有 Git 当时能找到的配置。

```bash
$ git config --list
user.name=John Doe
user.email=johndoe@example.com
color.status=auto
color.branch=auto
color.interactive=auto
color.diff=auto
...
```

你可能会看到重复的变量名，因为 Git 会从不同的文件中读取同一个配置（例如：`/etc/gitconfig` 与 `~/.gitconfig`）。 这种情况下，Git 会使用它找到的每一个变量的最后一个配置。

你可以通过输入 `git config `： 来检查 Git 的某一项配置

```bash
$ git config user.name
John Doe
```

## 4. 基础操作-记录每次更新到仓库

#### 4.1 获取帮助

若你使用 Git 时需要获取帮助，有三种方法可以找到 Git 命令的使用手册：

```bash
$ git help <verb>
$ git <verb> --help
$ man git-<verb>
```

例如，要想获得 config 命令的手册，执行

```bash
$ git help config
```

这些命令很棒，因为你随时随地可以使用而无需联网。 

#### 4.2 初始化Git仓库

新建一个项目的文件夹，在文件夹中输入
```
$ git init
```
这个命令将创建一个.git的子目录，这个子目录含有初始化的Git仓库中所有的必须文件，这些文件是Git仓库的骨干。这个时候仅仅只是做了一个初始化的操作，项目中的文件还没有被跟踪。现在这个文件夹下的空间叫做**工作区**，可以查看、作出修改。

#### 4.3 添加到暂存区

暂存区`(stage)`，这个区域存放着将要提交的文件。 

通过`git add`命令来指定所需文件进行追踪。

```bash
$ git add xxx.txt
```
`git add`命令使用文件或者目录的路径作为参数；如果参数是目录的路径，该命令将递归地跟踪该目录下的所有文件。

#### 4.4 查看状态、修改、历史记录

在工作目录下的每个文件都不外乎这两种状态：已跟踪或未跟踪。已跟踪的文件是指那些被纳入了版本控制的文件，在上一次快照中有它们的记录，在工作一段时间后，它们的状态可能是未修改，已修改或已放入暂存区。工作目录中除已跟踪文件外的所有其他文件都属于未跟踪文件，它们既不存在于上次快照的记录中，也没有被放入暂存区。

##### 检查当前文件状态

查看git状态的命令：

```bash
$ git status
```
在克隆仓库后立即使用此命令，会看到类似这样的输出：

```bash
$ git status
On branch master
nothing to commit, working directory clean
```

这说明现在的工作目录相当干净，所有已跟踪的文件在上次提交后都未被更改过。此外，上面的信息还表明，当前目录下没有出现任何处于未跟踪状态的新文件，否则Git会在这里列出来。最后，该命令还显示了当前所在的分支，并告诉这个分支同远程仓库的分支没有偏离。现在，分支名称是`master`，这是默认的分支名。

##### 跟踪新文件

现在，创建一个新的README文件。如果之前并不存在该文件，使用`git status`命令，将看到一个新的未跟踪文件：

```bash
$ echo 'My Project' > README
$ git status
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)

    README

nothing added to commit but untracked files present (use "git add" to track)
```

在上面的状态报告中可以看到新建的README文件出现在`Untracked files`下面。未跟踪的文件意味着Git在之前的快照（提交）中没有这些文件；Git不会自动将之纳入跟踪范围，除非你明明白白地告诉它 “我需要跟踪该文件”。这样的处理让你不必担心将生成的二进制文件或者其他不想被跟踪的文件包含进来。

现在使用4.3节的命令将README文件添加到暂存区，再运行`git status`命令，就会看到README文件已经被跟踪，并且处于暂存状态：

```bash
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   README
```

只要再`Changes to be committed`这行下面的，就说明是已暂存状态。如果此时提交，那么该文件在运行`git add`时的版本将被留存在历史记录中。

##### 暂存已修改文件

现在修改一个已被跟踪的文件。比如修改了一个已被追踪了的，名为`CONTRIBUTING.md`的已被跟踪的文件，然后运行`git status`命令，会看到下面内容。

```bash
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   README

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   CONTRIBUTING.md
```

文件 `CONTRIBUTING.md` 出现在 `Changes not staged for commit` 这行下面，说明已跟踪文件的内容发生了变化，但还没有放到暂存区。 要暂存这次更新，需要运行 `git add` 命令。这是个多功能命令：可以用它开始跟踪新文件，或者把已跟踪的文件放到暂存区，还能用于合并时把有冲突的文件标记为已解决状态等。将这个命令理解为“添加内容到下一次提交中”而不是“将一个文件添加到项目中”更加合适。现在再次运行`git add`将“CONTRIBUTING.md”放到暂存区，然后再看看`git status`的输出：

```bash
$ git add CONTRIBUTING.md
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   README
    modified:   CONTRIBUTING.md
```

现在两个文件都已暂存，下次提交时就会一并记录到仓库。假设此时，再在`CONTRIBUTING.md` 里再加条注释。 重新编辑存盘后，准备好提交。不过且慢，再运行 `git status` 看看：

```bash
$ vim CONTRIBUTING.md
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   README
    modified:   CONTRIBUTING.md

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   CONTRIBUTING.md
```

怎么回事？ 现在 `CONTRIBUTING.md` 文件同时出现在暂存区和非暂存区。 这怎么可能呢？ 好吧，实际上 Git 只不过暂存了你运行 `git add` 命令时的版本。 如果你现在提交，`CONTRIBUTING.md` 的版本是你最后一次运行 `git add` 命令时的那个版本，而不是你运行 `git commit` 时，在工作目录中的当前版本。 所以，运行了 `git add` 之后又作了修订的文件，需要重新运行 `git add` 把最新版本重新暂存起来：

```bash
$ git add CONTRIBUTING.md
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   README
    modified:   CONTRIBUTING.md
```

#### 4.5 查看具体的代码修改

`git status`命令输出的信息对于你来说可能过于模糊，你想知道具体修改了什么地方，可以用`git diff`命令。它可以通过文件补丁的格式更加具体显示哪些行为发生了改变。

##### 比较当前工作区文件和暂存区文件之间的差异

假如再次修改了README文件后暂存，然后编辑`CONTRIBUTING.md`文件后先不暂存，运行`git status`命令将会看到：

```bash
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    modified:   README

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   CONTRIBUTING.md
```

要查看尚未暂存的文件更新了哪些部分，不加参数直接输入 `git diff`：

```bash
$ git diff
diff --git a/CONTRIBUTING.md b/CONTRIBUTING.md
index 8ebb991..643e24f 100644
--- a/CONTRIBUTING.md
+++ b/CONTRIBUTING.md
@@ -65,7 +65,8 @@ branch directly, things can get messy.
 Please include a nice description of your changes when you submit your PR;
 if we have to read the whole diff to figure out why you're contributing
 in the first place, you're less likely to get feedback and have your change
-merged in.
+merged in. Also, split your changes into comprehensive chunks if your patch is
+longer than a dozen lines.

 If you are starting to work on a particular area, feel free to submit a PR
 that highlights your work in progress (and note in the PR title that it's
```

此命令比较的是工作目录中当前文件和暂存区域快照之间的差异。也就是修改之后还没有暂存起来的变化内容。

##### 比较暂存文件和最后一次提交之间的差异

使用`git diff --staged`命令或者`--cached`（--staged和--cached是同义词），这将对比已暂存文件与最后一次提交的文件差异：

```bash
$ git diff --staged
diff --git a/README b/README
new file mode 100644
index 0000000..03902a1
--- /dev/null
+++ b/README
@@ -0,0 +1 @@
+My Project
```

#### 4.6 提交修改

使用`git commit`命令提交修改

```bash
$ git commit
```

这种方式会启动文本编辑器以便输入本次提交的说明。（默认会启用shell的环境变量`$EDITOR`所指定的软件。也可以使用`git config --global core.editor`命令设置你喜欢的编辑器。

编辑器会显示类似下面的文本信息（本例选用 Vim 的屏显方式展示）：

```bash
# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
# On branch master
# Changes to be committed:
#	new file:   README
#	modified:   CONTRIBUTING.md
#
~
~
~
".git/COMMIT_EDITMSG" 9L, 283C
```

可以看到，默认的提交消息包含最后一次运行`git status`的输出，放在注释行里，另外开头还有一空行，供输入提交说明。可以去掉这些注释行，不过留着也没什么关系，多少能帮你回想起这次更新的内容有哪些。退出编辑器时，Git会丢掉注释行，用你输入提交附带信息生成一次提交。

另外，可以在`commit`命令后添加`-m`参数将提交信息和命令放在同一行：

```bash
$ git commit -m "Story 182: Fix benchmarks for speed"
[master 463dc4f] Story 182: Fix benchmarks for speed
 2 files changed, 2 insertions(+)
 create mode 100644 README
```
它会告诉你当前在哪个分支（`master`）提交的，本次提交的完整SHA-1校验和是什么（`463dc4f`）,以及在本次提交中，有多少文件修订过，多少行添加和删改过。

注意提交记录的是放在暂存区的快照，任何还未暂存的文件仍然保持已修改状态，可以在下次提交时纳入版本管理。每一次运行提交操作，都是对项目做一次快照，以后可以回到这个状态，或者进行比较。

`commit`命令后添加`-a`选项，Git就会自动把所有已追踪过的文件暂存起来一并提交，跳过添加到暂存区的步骤。

#### 4.7 移除文件

要从Git中移除某个文件，就必须要从已跟踪文件清单中移除（切确来说，是从暂存区移除），然后提交。可以用`git rm`命令完成此项工作，并连带工作目录中删除指定文件，这样以后就不会出现在未跟踪文件清单中了。

如果只是简单地从工作目录中手工删除文件，运行`git status`时就会在“Changes not staged for commit”部分（也就是未暂存清单）看到：

```bash
$ rm PROJECTS.md
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        deleted:    PROJECTS.md

no changes added to commit (use "git add" and/or "git commit -a")
```

然后再运行`git rm`记录此次移除文件的操作：

```bash
$ git rm PROJECTS.md
rm 'PROJECTS.md'
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    deleted:    PROJECTS.md
```

下一次提交时，该文件就不再被纳入版本管理了。

如果删除之前修改过并且已经放到暂存区域的话，则必须要用强制删除选项`-f`（force的首字母）。这是一种安全特性，用于防止误删还没有添加到快照的数据，这样的数据不能被Git恢复。

另外一种情况是，我们想把文件从Git仓库中删除（亦即从暂存区移除），但仍然希望保留再当前工作目录中。换句话说，你想让文件保留在磁盘，但是并不想让 Git 继续跟踪。 当你忘记添加 `.gitignore` 文件，不小心把一个很大的日志文件或一堆 `.a` 这样的编译生成文件添加到暂存区时，这一做法尤其有用。 为达到这一目的，使用 `--cached` 选项：

```bash
$ git rm --cached README
```

`git rm`命令后面可以列出文件或者目录的名字

#### 4.8 移动文件

不像其它的 VCS 系统，Git 并不显式跟踪文件移动操作。 如果在 Git 中重命名了某个文件，仓库中存储的元数据并不会体现出这是一次改名操作。 不过 Git 非常聪明，它会推断出究竟发生了什么，至于具体是如何做到的，我们稍后再谈。

既然如此，当你看到 Git 的 `mv` 命令时一定会困惑不已。 要在 Git 中对文件改名，可以这么做：

```bash
$ git mv file_from file_to
```

它会恰如预期般正常工作。 实际上，即便此时查看状态信息，也会明白无误地看到关于重命名操作的说明：

```bash
$ git mv README.md README
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    renamed:    README.md -> README
```

其实，运行 `git mv` 就相当于运行了下面三条命令：

```bash
$ mv README.md README
$ git rm README.md
$ git add README
```

如此分开操作，Git 也会意识到这是一次改名，所以不管何种方式结果都一样。 两者唯一的区别是，`mv` 是一条命令而另一种方式需要三条命令，直接用 `git mv` 轻便得多。 不过有时候用其他工具批处理改名的话，要记得在提交前删除老的文件名，再添加新的文件名。

#### 4.9 查看提交历史记录

`git log`命令可以会按时间先后顺序列出所有的提交。

```bash
$ git log
commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Mon Mar 17 21:52:11 2008 -0700

    changed the version number

commit 085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Sat Mar 15 16:40:33 2008 -0700

    removed unnecessary test

commit a11bef06a3f659402fe7563abf99ad00de2209e6
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Sat Mar 15 10:31:28 2008 -0700

    first commit
```

在不传入任何参数的默认情况下，git log会按时间先后顺序列出所有的提交，最近的更新排在上面。它会列出每个提交的SHA-1校验和、作者的名字和电子邮件地址、提交时间及提交说明。

`git log`有许多选项可以帮助你搜寻你所需要的提交。

选项`-p`会显示每次提交所引入的差异。与此同时，也可以使用`-2`选项来仅显示最近的两次提交：

```bash
$ git log -p -2
commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Mon Mar 17 21:52:11 2008 -0700

    changed the version number

diff --git a/Rakefile b/Rakefile
index a874b73..8f94139 100644
--- a/Rakefile
+++ b/Rakefile
@@ -5,7 +5,7 @@ require 'rake/gempackagetask'
 spec = Gem::Specification.new do |s|
     s.platform  =   Gem::Platform::RUBY
     s.name      =   "simplegit"
-    s.version   =   "0.1.0"
+    s.version   =   "0.1.1"
     s.author    =   "Scott Chacon"
     s.email     =   "schacon@gee-mail.com"
     s.summary   =   "A simple gem for using Git in Ruby code."

commit 085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Sat Mar 15 16:40:33 2008 -0700

    removed unnecessary test

diff --git a/lib/simplegit.rb b/lib/simplegit.rb
index a0a60ae..47c6340 100644
--- a/lib/simplegit.rb
+++ b/lib/simplegit.rb
@@ -18,8 +18,3 @@ class SimpleGit
     end

 end
-
-if $0 == __FILE__
-  git = SimpleGit.new
-  puts git.show
-end
\ No newline at end of file
```

查看每次提交的简略信息`--stat`

一行显示`--pretty=oneline`，

定制要显示的记录格式`--pretty=fomat:"%h - %an, %ar : %s"`

```bash
$ git log --pretty=format:"%h - %an, %ar : %s"
ca82a6d - Scott Chacon, 6 years ago : changed the version number
085bb3b - Scott Chacon, 6 years ago : removed unnecessary test
a11bef0 - Scott Chacon, 6 years ago : first commit
```

#### 4.10 撤销操作

##### 忘记添加文件和提交信息写错

有时我们提交完了才发现漏掉了几个文件没有添加，或者是提交信息写错了。此时，可以运行带有`--amend`选项的提交命令尝试重新提交。

这个命令会将暂存区中的文件提交。如果自上次提交以来你还未做任何修改，那么快照会保持不变，而你所修改的只是提交信息。

文本编辑器启动之后，可以看见之前的提交信息。编辑保存后会覆盖原来的提交信息。

例如，提交之后忘记暂存类某些需要修改的文件，可以像下面这样操作：

```bash
$ git commit -m 'initial commit'
$ git add forgotten_file
$ git commit --amend
```

最终只会有一个提交——第二次提交将替代第一次提交的结果。

##### 取消暂存的文件

比如你不小心输入`git add *`暂存了两个文件，其中一个是不想暂存的。根据`git status`的提示，可以使用`git reset HEAD <file>...`来取消暂存。

##### 撤销对文件的修改

如果你将一个文件修改之后发现改乱了，想回到上一次提交时候的样子。这时可以根据`git status`的提示，使用`git checkout <file>...`来撤销工作区的修改。

注意，`git checkout`命令是个危险命令，这很重要。你对那个文件做的任何修改都会消失——你只是拷贝了另一个文件来覆盖它。

##### 版本回退

在回退版本前使用```git log```确定要回退到的版本id
```bash
$ git log --pretty=oneline
b91afc156c640fabf4e6a181eb276e9cdd2850c9 (HEAD -> master) upper case
82315573d47df2b2036b7848ef980957b4cf1b4b init
```
现在我要回到init的版本，所以输入
```bash
$ git reset --hard 8231
HEAD is now at 8231557 init
```
现在就回到了之前的init版本，其中```--hard```参数表示覆盖当前的工作区

如果回去了还想再回来,先查看操作记录
```bash
$ git reflog
8231557 (HEAD -> master) HEAD@{0}: reset: moving to 8231
b91afc1 HEAD@{1}: commit: upper case
8231557 (HEAD -> master) HEAD@{2}: commit (initial): init
```
从中得知commit: upper case这个提交的id为b91afc1,然后就可以飞回那个版本了
```bash
$ git reset --hard b91af
HEAD is now at b91afc1 upper case
```

#### 4.11 远程仓库的使用

为了能在任意 Git 项目上协作，你需要知道如何管理自己的远程仓库。 远程仓库是指托管在因特网或其他网络中的你的项目的版本库。 你可以有好几个远程仓库，通常有些仓库对你只读，有些则可以读写。 与他人协作涉及管理远程仓库以及根据需要推送或拉取数据。 管理远程仓库包括了解如何添加远程仓库、移除无效的远程仓库、管理不同的远程分支并定义它们是否被跟踪等等。

##### 查看所有远程仓库

如果想查看你已经配置的远程仓库服务器，可以运行 `git remote` 命令。 它会列出你指定的每一个远程服务器的简写。 如果你已经克隆了自己的仓库，那么至少应该能看到 origin ——这是 Git 给你克隆的仓库服务器的默认名字：

```bash
$ git clone https://github.com/schacon/ticgit
Cloning into 'ticgit'...
remote: Reusing existing pack: 1857, done.
remote: Total 1857 (delta 0), reused 0 (delta 0)
Receiving objects: 100% (1857/1857), 374.35 KiB | 268.00 KiB/s, done.
Resolving deltas: 100% (772/772), done.
Checking connectivity... done.
$ cd ticgit
$ git remote
origin
```

`-v`参数显示需要读写远程仓库使用的Git保存的简写与之对应的URL

如果你的远程仓库不止一个，该命令会将它们全部列出。 例如，与几个协作者合作的，拥有多个远程仓库的仓库看起来像下面这样：

```bash
$ cd grit
$ git remote -v
bakkdoor  https://github.com/bakkdoor/grit (fetch)
bakkdoor  https://github.com/bakkdoor/grit (push)
cho45     https://github.com/cho45/grit (fetch)
cho45     https://github.com/cho45/grit (push)
defunkt   https://github.com/defunkt/grit (fetch)
defunkt   https://github.com/defunkt/grit (push)
koke      git://github.com/koke/grit.git (fetch)
koke      git://github.com/koke/grit.git (push)
origin    git@github.com:mojombo/grit.git (fetch)
origin    git@github.com:mojombo/grit.git (push)
```

这样我们可以轻松拉取其中任何一个用户的贡献。 此外，我们大概还会有某些远程仓库的推送权限。

##### 添加远程仓库

运行`git remote add <shortname> <url>`添加一个新的远程Git仓库，同时指定一个你可以轻松引用的简写。

```bash
$ git remote
origin
$ git remote add pb https://github.com/paulboone/ticgit
$ git remote -v
origin	https://github.com/schacon/ticgit (fetch)
origin	https://github.com/schacon/ticgit (push)
pb	https://github.com/paulboone/ticgit (fetch)
pb	https://github.com/paulboone/ticgit (push)
```

现在你可以在命令行中使用字符串 `pb` 来代替整个 URL。 例如，如果你想拉取 Paul 的仓库中有但你没有的信息，可以运行 `git fetch pb`：

```bash
$ git fetch pb
remote: Counting objects: 43, done.
remote: Compressing objects: 100% (36/36), done.
remote: Total 43 (delta 10), reused 31 (delta 5)
Unpacking objects: 100% (43/43), done.
From https://github.com/paulboone/ticgit
 * [new branch]      master     -> pb/master
 * [new branch]      ticgit     -> pb/ticgit
```

现在 Paul 的 master 分支可以在本地通过 `pb/master` 访问到——你可以将它合并到自己的某个分支中，或者如果你想要查看它的话，可以检出一个指向该点的本地分支。 

##### 从远程仓库中抓取与拉取

就如刚才所见，从远程仓库中获得数据，可以执行：

```bash
$ git fetch [remote-name]
```

这个命令会访问远程仓库，从中拉取所有你还没有的数据。 执行完成后，你将会拥有那个远程仓库中所有分支的引用，可以随时合并或查看。

如果你使用 `clone` 命令克隆了一个仓库，命令会自动将其添加为远程仓库并默认以 “origin” 为简写。 所以，`git fetch origin` 会抓取克隆（或上一次抓取）后新推送的所有工作。 必须注意 `git fetch` 命令会将数据拉取到你的本地仓库——它并不会自动合并或修改你当前的工作。 当准备好时你必须手动将其合并入你的工作。

如果你有一个分支设置为跟踪一个远程分支，可以使用 `git pull` 命令来自动的拉取然后合并远程分支到当前分支。 这对你来说可能是一个更简单或更舒服的工作流程；默认情况下，`git clone` 命令会自动设置本地 master 分支跟踪克隆的远程仓库的 master 分支（或不管是什么名字的默认分支）。 运行 `git pull` 通常会从最初克隆的服务器上抓取数据并自动尝试合并到当前所在的分支。

##### 推送到远程仓库

当你想分享你的项目时，必须将其推送到上游。 这个命令很简单：`git push [remote-name] [branch-name]`。 当你想要将 master 分支推送到 `origin` 服务器时（再次说明，克隆时通常会自动帮你设置好那两个名字），那么运行这个命令就可以将你所做的备份到服务器：

```bash
$ git push origin master
```

只有当你有所克隆服务器的写入权限，并且之前没有人推送过时，这条命令才能生效。 当你和其他人在同一时间克隆，他们先推送到上游然后你再推送到上游，你的推送就会毫无疑问地被拒绝。 你必须先将他们的工作拉取下来并将其合并进你的工作后才能推送。 

##### 查看某个远程仓库

如果要想查看一个远程仓库的更多信息，可以使用`git remote show [remote-name]`命令。例如`origin`：

```bash
$ git remote show origin
* remote origin
  Fetch URL: https://github.com/schacon/ticgit
  Push  URL: https://github.com/schacon/ticgit
  HEAD branch: master
  Remote branches:
    master                               tracked
    dev-branch                           tracked
  Local branch configured for 'git pull':
    master merges with remote master
  Local ref configured for 'git push':
    master pushes to master (up to date)
```

它同样会列出远程仓库的 URL 与跟踪分支的信息。 这些信息非常有用，它告诉你正处于 master 分支，并且如果运行 git pull，就会抓取所有的远程引用，然后将远程 master 分支合并到本地 master 分支。 它也会列出拉取到的所有远程引用。

##### 远程仓库的移除与重命名

如果想要重命名引用的名字可以运行 `git remote rename` 去修改一个远程仓库的简写名。 例如，想要将 `pb` 重命名为 `paul`，可以用 `git remote rename` 这样做：

```bash
$ git remote rename pb paul
$ git remote
origin
paul
```

值得注意的是这同样也会修改你的远程分支名字。 那些过去引用 `pb/master` 的现在会引用 `paul/master`。

如果因为一些原因想要移除一个远程仓库——你已经从服务器上搬走了或不再想使用某一个特定的镜像了，又或者某一个贡献者不再贡献了——可以使用 `git remote rm` ：

```bash
$ git remote rm paul
$ git remote
origin
```

#### 4.12 打标签

Git 可以给历史中的某一个提交打上标签，以示重要。比较有代表性的是人们会使用这个功能来标记发布结点（v1.0 等等）。

##### 列出标签

在 Git 中列出已有的标签是非常简单直观的。 只需要输入 `git tag`：

```console
$ git tag
v0.1
v1.3
```

这个命令以字母顺序列出标签；但是它们出现的顺序并不重要。

你也可以使用特定的模式查找标签。 例如，Git 自身的源代码仓库包含标签的数量超过 500 个。 如果只对 1.8.5 系列感兴趣，可以运行：

```bash
$ git tag -l 'v1.8.5*'
v1.8.5
v1.8.5-rc0
v1.8.5-rc1
v1.8.5-rc2
v1.8.5-rc3
v1.8.5.1
v1.8.5.2
v1.8.5.3
v1.8.5.4
v1.8.5.5
```

##### 创建标签

GIt使用两种主要类型的标签：轻量标签（lightweight）与附注标签（annotated）。

一个轻量标签很像一个不会改变的分支——它只是一个特定提交的引用。

## 5.Git分支

几乎所有的版本控制系统都以某种形式支持分支。 使用分支意味着你可以把你的工作从开发主线上分离开来，以免影响开发主线。 在很多版本控制系统中，这是一个略微低效的过程——常常需要完全创建一个源代码目录的副本。对于大项目来说，这样的过程会耗费很多时间。

有人把 Git 的分支模型称为它的“必杀技特性”，也正因为这一特性，使得 Git 从众多版本控制系统中脱颖而出。 为何 Git 的分支模型如此出众呢？ Git 处理分支的方式可谓是难以置信的轻量，创建新分支这一操作几乎能在瞬间完成，并且在不同分支之间的切换操作也是一样便捷。 与许多其它版本控制系统不同，Git 鼓励在工作流程中频繁地使用分支与合并，哪怕一天之内进行许多次。 理解和精通这一特性，你便会意识到 Git 是如此的强大而又独特，并且从此真正改变你的开发方式。

#### 5.1 分支简介

Git记录的是一系列不同时刻的文件快照。

在进行提交操作时，Git会保存一个提交对象（commit object）。该提交对象会包含一个指向暂存内容快照的指针。但不仅仅是这样，该提交对象还包含了作者的姓名和邮箱，提交时输入的信息以及指向它的父对象的指针。首次提交产生的提交对象没有父对象，普通提交操作产生的提交对象有一个父对象，而由多个分支合并产生的提交对象有多个父对象。

![首次提交对象及其树结构。](https://git-scm.com/book/en/v2/images/commit-and-tree.png)

做些修改后再次提交，那么这次产生的提交对象会包含一个指向上次提交对象（父对象）的指针。

![提交对象及其父对象。](https://git-scm.com/book/en/v2/images/commits-and-parents.png)

Git的分支，其实本质上仅仅是指向提交对象的可变指针。Git的默认分支名称是`master`。在多次提交操作之后，你其实已经有一个指向最后那个提交对象的`master`分支。它会在每次的提交操作中自动向前移动。

![分支及其提交历史。](https://git-scm.com/book/en/v2/images/branch-and-history.png)

#### 5.2 分支创建

Git 是怎么创建新分支的呢？ 很简单，它只是为你创建了一个可以移动的新的指针。 比如，创建一个 testing 分支， 你需要使用 `git branch` 命令：

```bash
$ git branch testing
```

这会在当前所在的提交对象上创建一个指针。

![两个指向相同提交历史的分支。](https://git-scm.com/book/en/v2/images/two-branches.png)

那么Git是怎么知道当前在哪一个分支上呢？其实也很简单，Git有一个叫`HEAD`的特殊指针。在Git中，它是一个指针，指向当前所在的本地分支（将`HEAD`想象为当前分支的别称）。但现在还是在`master`分支上。因为`git branch`命令仅仅只是创建了一个新分支，并不会自动切换到新分支中去。

![HEAD 指向当前所在的分支。](https://git-scm.com/book/en/v2/images/head-to-master.png)

#### 5.3 分支切换

使用`git checkout`命令切换分支。现在切换到新创建的`testing`分支去：

```bash
$ git checkout testing
```

这样`HEAD`就指向`testing`分支了。

![HEAD 指向当前所在的分支。](https://git-scm.com/book/en/v2/images/head-to-testing.png)

那么分支的实现方式会给我们带来什么好处呢？

比如现在修改了了一些文件并且提交。

![HEAD 分支随着提交操作自动向前移动。](https://git-scm.com/book/en/v2/images/advance-testing.png)

`testing`分支向前移动了，但是`master`分支却没有，它依然指向`git checkout`时所指向的对象。这时我们切换回`master`分支看看：

```bash
$ git checkout master
```

![检出时 HEAD 随之移动。](https://git-scm.com/book/en/v2/images/checkout-master.png)

这条命令做了两件事。一是使`HEAD`指回`master`分支，二是将工作目录恢复成`master`分支所指向的快照。也就是回到了一个较旧的版本。本质上来说，就是忽略了`testing`分支所作的修改，以便向另一个方向进行开发。

> 注意在切换分支时，工作目录会恢复到该分支最后一次提交时的样子。如果Git不能干净利落地完成这个任务，它将禁止切换分支。

可以继续修改`master`分支的内容并且提交。现在，这个项目的提交历史已经产生了分叉，因为刚才创建了一个新分支，并切换过去进行了一些工作，随后又切换回`master`分支进行了另一些工作。上述两次改动针对的是不同分支：可以在不同分支间来回切换和工作，并在时机成熟的时候将它们合并起来。

![项目分叉历史。](https://git-scm.com/book/en/v2/images/advance-master.png)

#### 5.4 项目分支历史

可以简单地使用`git log`命令查看分叉历史。运行`git log --oneline --decorate --graph --all`，它会输出你的提交历史、各个分支的指向以及项目的分叉情况。

```bash
$ git log --oneline --decorate --graph --all
* c2b9e (HEAD, master) made other changes
| * 87ab2 (testing) made a change
|/
* f30ab add feature #32 - ability to add new formats to the
* 34ac2 fixed bug #1328 - stack overflow under certain conditions
* 98ca9 initial commit of my project
```

因为Git的分支实质上仅是包含所指对象校验和（长度为40的SHA-1值字符串）的文件，所以它的创建和销毁都异常高效。创建一个新分支就相当于往一个文件中写入41个字符（40个字符和一个换行符），如此的简单能不快吗？

这和过去大多数版本控制系统形成了鲜明对比，它们在创建分支时，将所有的项目文件都复制一遍，并保存到一个特定的目录。完成这样繁琐的过程通常需要好几秒钟，有时甚至好几分钟。而在Git中，任何规模的项目都能在瞬间创建新分支。同时由于每次提交都会记录父对象，所以寻找恰当的合并基础（即共同祖先）也是同样的简单和高效。

#### 5.5 分支的合并与切换

首先，假设你正在你的项目上工作，并且已经有了一些提交。

![一个简单的提交历史。](https://git-scm.com/book/en/v2/images/basic-branching-1.png)

现在，你已经决定要解决你的公司使用的问题追踪系统中的#53问题。想要新建一个分支并同时切换到那个分支上，可以运行一个带有`-b`的参数`git checkout`命令。

```bash
$ git checkout -b iss53
Switched to a new branch "iss53"
```

这个命令就相当于`git branch iss53`和`git checkout iss53`这两条命令的简写。

![创建一个新分支指针。](https://git-scm.com/book/en/v2/images/basic-branching-2.png)

你继续在#53问题上工作，并做了一些提交。在此过程中`iss53`分支在不断的向前推进。因为你已经检出到了该分支，也就是当前的`HEAD`指针指向了`iss53`分支。

![iss53 分支随着工作的进展向前推进。](https://git-scm.com/book/en/v2/images/basic-branching-3.png)

现在你接到了一个紧急的电话，有个紧急的问题需要解决。有了Git的帮助，你不必把这个紧急的问题和`iss53`的修改混在一起，也不需要花大力气来还原53#问题的修改，然后添加关于这个紧急问题的修改，最后将这个修改提交到线上分支。你所需要的仅仅是切换回`master`分支。

但是在这么做之前，需要留意你的工作目录和暂存区里还有那些没被提交的修改，它可能会和你即将检出的分支产生冲突从而阻止Git切换到该分支。最好的方法是，在切换分支前，保持一个干净的状态，但是你不想因为过会要回到当前的工作进度而创建一次提交，解决方法是保存进度（stashing）和修补提交（commit amending）。也可以先创建一次提交，而不是使用下面所述的储藏修改。

##### 储藏修改（git stash)

储藏会处理工作目录的脏状态——即跟踪文件的修改与暂存的改动——然后将未完成的修改保存到一个栈上，而你可以在任何时候重新应用这些改动。

将新的储藏推送到栈上，运行`git stash`或`git stash save`：

```bash
$ git stash
Saved working directory and index state \
  "WIP on master: 049d078 added the index file"
HEAD is now at 049d078 added the index file
(To restore them type "git stash apply")
```

现在用`git status`查看会发现工作目录是干净的了

```console
$ git status
# On branch master
nothing to commit, working directory clean
```

现在能够轻易地切换分支并在其他地方工作了；你的修改被储存在栈上。要查看储藏地东西，可以使用`git stash list`

```bash
$ git stash list
stash@{0}: WIP on master: 049d078 added the index file
stash@{1}: WIP on master: c264051 Revert "added file_size"
stash@{2}: WIP on master: 21d80a5 added number to log
```

可以通过stash命令地帮助提示中的命令将刚才储藏的工作重新应用：`git stash apply`，默认应用的是最近的储藏，如果想要应用其中一个更旧的储藏，可以通过名字指定它，像这样：`git stash apply stash@{2}`。

```bash
$ git stash apply
# On branch master
# Changed but not updated:
#   (use "git add <file>..." to update what will be committed)
#
#      modified:   index.html
#      modified:   lib/simplegit.rb
#
```

这时可以切换回`master`分支了。

```bash
$ git checkout master
Switched to branch 'master'
```

接下来，要修复这个紧急问题。建立一个针对该紧急问题的分支（hotfix branch），在该分支上工作直到问题解决

```bash
$ git checkout -b hotfix
Switched to a new branch 'hotfix'
$ vim index.html
$ git commit -a -m 'fixed the broken email address'
[hotfix 1fb7853] fixed the broken email address
 1 file changed, 2 insertions(+)
```

![基于 `master` 分支的紧急问题分支（hotfix branch）。](https://git-scm.com/book/en/v2/images/basic-branching-4.png)

可以运行测试，确保修改是正确的，然后将其合并回`master`分支上来部署到线上。使用`git merge`合并分支

```console
$ git checkout master
$ git merge hotfix
Updating f42c576..3a0874c
Fast-forward
 index.html | 2 ++
 1 file changed, 2 insertions(+)
```

注意到合并时出现了`Fast-forward`这个词。由于当前`master`分支所指向的提交是有关hotfix的提交的直接上游，所以Git只是简单地将指针向前移动。换句话说，在合并分支时，如果顺着一个分支走下去能够到达另一个分支，那么Git在合并两者的时候，只会简单的将指针向前移动，因为这种情况下没有需要解决的分歧——这就叫“快速（`fast-forward`）“。

现在最新的修改已经在`master`分支所指向的提交快照中，可以着手发布了。

![`master` 被快进到 `hotfix`。](https://git-scm.com/book/en/v2/images/basic-branching-5.png)

关于这个紧急问题发布后，你准备回到被打断之前时的工作中。然而，你应该先删除`hotfix`分支，因为它以经没用了。`master`分支已经指向了同一个位置。可以使用带`git branch`的`-d`选项来删除分支：

```bash
$ git branch -d hotfix
Deleted branch hotfix (3a0874c).
```

假设你之前创建过了iss53的提交，那么现在就可以切换回正在工作的分支继续巩工作。

```console
$ git checkout iss53
Switched to branch "iss53"
$ vim index.html
$ git commit -a -m 'finished the new footer [issue 53]'
[iss53 ad82d7a] finished the new footer [issue 53]
1 file changed, 1 insertion(+)
```

![继续在 `iss53` 分支上的工作。](https://git-scm.com/book/en/v2/images/basic-branching-6.png)

然而你在`hotfix`分支上所作的工作并没有包含到`iss53`分支中。如果你需要拉取`hotfix`所作的修改，可以使用`git merge master`命令将`master`分支合并入`iss53`分支，或者你也可以等到`iss53`分支完成其使命，再将其合并回`master`分支。

##### 分支的合并（git merge）

完成了`iss53`分支的工作后，想要合并到`master`分支中去，这和之前合并`hotfix`分支所作的工作差不多。只需要检出到你想要合并入的分支，然后运行`git merge`命令：

```console
$ git checkout master
Switched to branch 'master'
$ git merge iss53
Merge made by the 'recursive' strategy.
index.html |    1 +
1 file changed, 1 insertion(+)
```

这个时候有点不一样了，因为现在`master`分支所在的提交并不是`iss53`分支的直接祖先。Git会使用两个分支末端的所指的快照（C4和C5）以及这两个分支的工作祖先（C2）做一个简单的三方合并。

![一次典型合并中所用到的三个快照。](https://git-scm.com/book/en/v2/images/basic-merging-1.png)

和之前将指针向前推进不同的是，Git将此次三方合并的结果做了一个新的快照并且自动创建了一个新的提交指向它。这个被称作一次合并提交，它的特别之处在于他不止有一个父提交。

![一个合并提交。](https://git-scm.com/book/en/v2/images/basic-merging-2.png)

Git会自动决定哪一个提交作为最优的共同祖先，并以此作为合并的基础。

既然修改已经合并进`master`分支了，就已经不需要`iss53`分支了。现在可以删除这个分支。

##### 遇到冲突时的分支合并

有时候合并操作不会如此顺利。比如在两个分支中，对同一个文件的同一部分进行了不同的修改，Git就没法干净的合并它们。

```bash
$ git merge iss53
Auto-merging index.html
CONFLICT (content): Merge conflict in index.html
Automatic merge failed; fix conflicts and then commit the result.
```

此时Git做了合并，但是没有自动地创建一个新的合并提交。Git会暂停下来，等待解决合并产生的冲突。可以在合并冲突后的任意时刻使用`git status`命令来查看那些因包含合并冲突而处于未合并（unmerged）状态的文件：

```bash
$ git status
On branch master
You have unmerged paths.
  (fix conflicts and run "git commit")

Unmerged paths:
  (use "git add <file>..." to mark resolution)

    both modified:      index.html

no changes added to commit (use "git add" and/or "git commit -a")
```

Git会在有冲突的文件中加入标准的冲突解决标记，这样你可以打开这些包含冲突的文件然后手动解决冲突。出现冲突的文件会包含一些特殊的区域，看起来像下面这样：

```html
<<<<<<< HEAD:index.html
<div id="footer">contact : email.support@github.com</div>
=======
<div id="footer">
 please contact us at support@github.com
</div>
>>>>>>> iss53:index.html
```

=======的上半部分时`HEAD`所指示的版本，而`iss53`分支所指示的版本在=======的下半部分。为了解决冲突，必须选择使用由=======分割的两个部分中的一个，或者你也可以自行合并这些内容。例如，可以通过把这段内容换成下面这样子来解决冲突。

```html
<div id="footer">
please contact us at email.support@github.com
</div>
```

上述的冲突解决方案只保留了其中一个分支的修改，并且 `<<<<<<<` , `=======` , 和 `>>>>>>>` 这些行被完全删除了。在你解决了所有文件里的冲突之后，对每个文件使用 `git add` 命令来将其标记为冲突已解决。 一旦暂存这些原本有冲突的文件，Git 就会将它们标记为冲突已解决。

如果想使用图形化工具来解决冲突，可以运行`git mergetool`。

#### 5.6 分支管理

使用不带参数的`git branch`可以查看分支列表

```bash
$ git branch
  iss53
* master
  testing
```

分支名字前面带有`*`字符它代表现在检出的那一个分支，也就是`HEAD`指针指向的分支。

如果要查看每个分支的最后一个提交可以使用`git branch -v`

```bash
$ git branch -v
  iss53   93b412c fix javascript issue
* master  7a98805 Merge branch 'iss53'
  testing 782fd34 add scott to the author list in the readmes
```

## 6. 使用GitHub

#### 注册GitHub账号

你所需要做的第一件事是创建一个免费账户。 直接访问 [https://github.com](https://github.com/)，选择一个未被占用的用户名，提供一个电子邮件地址和密码，点击写着“Sign up for GitHub”的绿色大按钮即可。

![GitHub 注册表单。](https://git-scm.com/book/en/v2/images/signup.png)

#### SSH访问

可以使用`https://`协议通过刚刚创建的用户名和密码来访问Git版本库。克隆公有项目甚至不需要注册。

使用SSH协议需要生成SSH公钥，并将生成的公钥添加到账户设置中去。

在Linux中，用户的SSH密钥储存用户目录隐藏的.ssh文件夹下，Windows也是存在用户目录的.ssh文件夹下。使用`ssh-keygen`命令生成，过程中会有一些问题需要回答，默认即可。

```bash
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/schacon/.ssh/id_rsa):
Created directory '/home/schacon/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/schacon/.ssh/id_rsa.
Your public key has been saved in /home/schacon/.ssh/id_rsa.pub.
The key fingerprint is:
d0:82:24:8e:d7:f1:bb:9b:33:53:96:93:49:da:9b:e3 schacon@mylaptop.local
```

生成之后，会有两个文件：`id_rsa`和`id_rsa.pub`，前者是私钥需要保存在本地不能外泄，后者是公钥，需要复制里面的内容到Github账号中的SSH key中。

![“SSH keys”链接。](https://git-scm.com/book/en/v2/images/ssh-keys.png)

#### 对项目做出贡献

如果想要参与某个项目，但是并没有推送权限，这时可以对这个项目进行“派生”。派生的意思就是指GitHub将在你的空间中创建一个完全属于你的项目副本，且你对其具有推送权限。

可以通过点击项目页面右上角的”Fork“按钮来派生这个项目。

![“Fork”按钮.](https://git-scm.com/book/en/v2/images/forkbutton.png)


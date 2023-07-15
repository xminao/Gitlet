# Gitlet 开发日志

模仿Git使用Java开发的版本控制系统(UCBerkelely CS61B Spring21 Project2)

### 📄 文件的三种状态

------

在Gitlet中的文件有三种状态：

- commited （已提交）

  数据作为一次快照的数据已经保存到本地数据库。

- modified （已修改）

  修改了文件但是没保存到数据库中。

- staged （已暂存）

  标记了文件，会包含到下次提交的快照中。

- untracked （未追踪）

  在工作区存在，但既不在暂存区和也不在HEAD指向的commit中。

  - new file

- tracked（已追踪）

  已经被纳入版本控制的文件，即在HEAD指向的commit中存在该文件或者在暂存区的文件，简言之就是Gitlet已经知道的文件。

  在工作区目录中，并且HEAD指向commit中存在该文件，文件存在以下状态

  - modified but not staged

    已追踪并被修改，未被记录在暂存区

  - deleted but not staged

    HEAD指向commit中存在该文件，工作目录中不存在该文件，未被记录在暂存区

  - modified and staged

    与HEAD指向commit中的文件内容已经不同，并且已经记录在暂存区

  - deleted and staged

    HEAD指向commit中存在该文件，工作目录中不存在该文件，并且已经记录在暂存区

  - unmodified (never staged)

  工作目录中文件和HEAD指向commit中的文件内容相同，不会被暂存

### 📁 项目的三个阶段

------

Git项目有三个阶段：工作区、暂存区、Git目录

- 工作区 (working directory)

  即进行使用和修改的区域，工作区中的文件有 **已追踪** 和 **未追踪** 两种状态，

  **已追踪**：已经被纳入版本控制的文件，上一次的快照**(父快照)**中包含该文件的记录，

  工作一段时间后，状态可能为 未修改 / 已修改未暂存 / 已暂存。

  **未追踪**：除已追踪文件以外都是未追踪文件，既不在上次快照（**父快照**）的记录中，也没有被暂存。比如初始化(init)一个仓库时，工作目录中的所有文件都属于未追踪文件。

  使用 java gitlet.Main status 可以查看当前工作目录中文件的状态.

  工作区的内容可以看作对当前分支最新快照的副本的修改（仓库初始化后有一个空内容的快照），已追踪/未追踪都是相对于当前分支最新快照而言。（如master分支的最新快照中包含 A.txt , B.txt，当前工作目录包含 B.txt, C.txt且未进行暂存操作，那么A.txt就是已修改未暂存（删除文件）的状态，C.txt就是未追踪状态（在master分支的最新快照中没有C.txt的数据），通过暂存(add/rm)操作将删除A.txt和新添加C.txt暂存）

- 暂存区 (staging area)

  暂存区是一个文件，保存了下次要提交进快照的文件列表信息。

- Git目录 (.git directory | Repository)

  保存元数据（metadata）和对象数据库(objects-database)的目录。

![Untitled](https://github.com/xminao/Gitlet/raw/master/imgs/01.png)

项目的三个阶段 (引用自：[git-scm.com](http://git-scm.com))

![Untitled](https://github.com/xminao/Gitlet/raw/master/imgs/02.png)

文件的状态变化(引用自：[git-scm.com](http://git-scm.com))

### 🌵 Gitlet对象

------

Gitlet中包含许多类型的对象，最主要的三个分别为：

- blobs ()

  文件的内容数据，因为Gitlet会保存文件的许多版本，一个文件可能对应多个blob（每个blob对应文件在不同快照中的内容，如A.txt在commit:01中对应内容blob:01，在commit:02中对应内容blob:07）

- trees

  树对象表示一种文件结构，类似于UNIX系统，映射文件名到对应的blob或其他的tree

  ![Untitled](https://github.com/xminao/Gitlet/raw/master/imgs/03.png)

Git内部存储的数据类似于该图(引用自 [git-scm.com](http://git-scm.com))

- commits

  commit对象中包含日志信息(log)，其他的metadata（提交日期，提交人等），对一个树对象的引用，对父快照的引用（可能是多个）。

### 📌Gitlet引用

------

用一个简单的名字(如master)来代替提交的SHA-1值以便使用，这称之为引用，在 .gitlet/refs 目录中下保存这类含有SHA-1值的文件，目录结构如下：

```bash
.gitlet/refs
.gitlet/refs/heads
.gitlet/refs/tags
```

创建新引用只需要在 .gitlet/refs/heads 目录创建新的引用文件

```bash
echo [sha-1 hashcode] > .gitlet/refs/heads/[ref name]
```

这样就可以在Gitlet命令中使用新的引用代替SHA-1值，这也是Gitlet分支的实质：一个指向某一系列提交之首的引用/指针。若想在某个提交上创建一个新的分支，可以创建一个该提交的引用（如test）指向该提交，这样下次提交的快照就会以引用的提交未父快照。`branch ` 命令会取得指定分支最新提交的 SHA-1 值，创建一个你指定的新分支引用。

![Untitled](https://github.com/xminao/Gitlet/raw/master/imgs/04.png)

分支 （引用自 [git-scm.com](http://git-scm.com)）

`branch ` 通过 HEAD 文件获取最新提交的 SHA-1 值，HEAD文件指向目前所在的分支引用。HEAD 文件内容如下：

```bash
$cat .gitlet/HEAD
ref: refs/heads/master
```

执行 `checkout test` 后：

```bash
$cat .gitlet/HEAD
ref: refs/heads/test
```

当执行 commit 提交后，创建的新提交对象会使用 HEAD 文件中引用指向的SHA-1值设置为其父提交字段。

**标签引用**：

**远程引用**：

### 🗃️索引

------

索引，即 `.gitlet/index` 文件，是Gitlet最重要的数据结构之一，用来记录路径列表及其对象名称来表示虚拟工作树的状态，用作暂存区以写出下一个要提交的树对象。”虚拟“意味着它通常不会与工作区树的目录文件匹配。索引充当工作区和commit之间的暂存区。(在Git中，index,stage和cache是一回事）

在Git中，索引文件的格式不是标准树对象，而是二进制数据文件。

当运行 add 时，工作区目录中的指定文件将被hash并作为对象存储到索引中，这些文件状态变为”暂存更改“；

当运行 commit，存储在索引中的更改将用于创建新的提交；

当运行 checkout，Gitlet从commit中获取数据并覆盖到工作目录和索引（Gitlet会清空索引即暂存区）。

Git中的索引有三个重要属性：

1. 包含生成唯一确定树对象所需要的所有信息
2. 索引可以在它定义的树对象和工作树对象之间快速切换
3. 有效表示不同树对象之间合并冲突的信息，允许每个路径名和相关树对象相关联以便合并。

![Untitled](https://github.com/xminao/Gitlet/raw/master/imgs/05.png)

工作区，索引，仓库 （引用自 https://stackoverflow.com/questions/4084921/what-does-the-git-index-contain-exactly）

### 📽️ Gitlet基本工作流程

------

1. 在工作区中修改文件
2. 将想要在下次提交内容中的文件暂存，这部分文件将会添加到暂存区
3. 提交更新，新的快照将永久存储到Git目录

### 🌗 与Git的不同

------

- 不会处理子目录的文件。
- 合并分支只会合并两个分支。
- metadata只包括时间戳和日志信息(log)。所以在Gitlet中的每个commit对象只会包含一个log message，时间戳，文件名到blob引用的映射(mapping of names to references)，父快照的引用和另一个快照的引用（for merges）

### 📂 Gitlet仓库文件结构

------

```bash
.git
├── HEAD
├── objects/
│   ├── info/
│   └── pack/
└── refs/
    ├── heads/
    └── tags/
```

| Name     | Function                                 |
| -------- | ---------------------------------------- |
| HEAD     | 类似指针，指向当前分支                   |
| objects/ | Gitlet的对象数据库，保存着所有的数据对象 |
| refs/    | 保存所有的branches, tags, remotes        |

## 🖥️ 命令

------

- init

  - 用法：`/java gitlet.Main init`

  - 说明：

    在当前目录创建一个新的Gitlet版本控制系统称之为仓库（默认为 **.gitlet** 目录）。仓库初始化后默认存在一个初始提交(Timestamp: 00:00:00 UTC, Thursday, 1 January 1970，Files: no files，Commit Message:  initial commit)，一个默认分支（master），默认分支指向初始提交，默认分支将作为当前分支使用。所有的仓库初始化后的初始提交都是相同的。

  - 失败用例：

    1. 当前目录已经存在一个仓库

       提示：*A Gitlet version-control system already exists in the current directory. 并退出*

- add

  - 用法：`/java gitlet.Main add [file name]`

  - 说明：

    将工作区中文件添加到暂存区，如果文件已经暂存并且内容被修改，则覆盖之前暂存区中的该文件（暂存区位于 **.gitlet** 中的index文件）。如果工作区中要暂存的文件与最新提交中相同，则不暂存该文件， 并且从暂存区中删除该文件（如果修改该文件后暂存，再更改回原来的内容再次暂存，就会发生暂存区记录和最新快照内的相同）。删除暂存区文件参考 `rm` 命令。

  - 失败用例：

    1. 当前目录不存在仓库

       *提示：File does not exist. 并退出*

- rm

  - 用法：`/java gitlet.Main rm [file name]`

  - 说明：

    如果文件在暂存区，将文件从暂存区移除（变为未追踪文件）。如果文件在当前提交中为**已追踪**状态则将该文件以删除状态添加至暂存区，并将该文件从工作区中删除（如果用户没有手动删除）。

- commit

  - 用法：`/java gitlet.Main commit [message]`

  - 说明：

    将最新的提交的文件快照和暂存区合并为一个新的快照并提交。

    有关Gitlet的提交有一些说明：

    - 提交后会清空暂存区
    - 提交操作不会对工作区目录做出任何更改。
    - 提交只关心暂存区文件的状态，也就是说如果工作区中已暂存的文件再次被更改/删除后不再次暂存，在提交时这些后来的更改/删除都会被忽视。
    - 提交操作后，新的提交会作为提交分支的最新节点。
    - 提交成功后，HEAD指针将指向最新的提交，上一个HEAD指针指向的提交称为最新提交的父提交。
    - 每个提交都包含创建时间。

  - 失败用例：

    1. 文件已经被暂存，提示：*No changes added to the commit.*
    2. 提交信息为空，提示：*Please enter a commit message.*

  ![Untitled](https://github.com/xminao/Gitlet/raw/master/imgs/06.png)

- log

  - 用法：`/java gitlet.Main log`

  - 说明：

    从HEAD指向提交开始，显示该提交以前的所有提交信息直到根提交。如果有多个父提交，只列出从第一个父提交开始，忽略其他父提交（与Git不同）。每个提交历史信息列出提交ID，提交时间，提交信息。对于有多个父提交的提交，增加一个Merge信息，列出该提交的两个父提交ID前七位（Gitlet中合并最多两个分支）。第一个父提交即Merge操作时所在分支，第二个父提交是要合并的分支。

![Untitled](https://github.com/xminao/Gitlet/raw/master/imgs/07.png)

- global-log

  - 用法：/java gitlet.Main global-log

  - 说明：

    与 log 命令相同，不过列出所有的当前提交的历史提交信息，也就是合并的分支提交历史也包括。

- checkout

  - 用法： 
    1. `/java gitlet.Main checkout -- [file name]`
    2. `/java gitlet.Main checkout [commit id] -- [filename]`
    3. `/java gitlet.Main checkout [branch name]`
  - 说明： 
    1. 将HEAD commit中的指定文件覆盖到工作区目录，该文件不暂存。
    2. 将指定commit id的commit中的指定文件覆盖到工作区目录，该文件不暂存，
    3. 将指定分支最新commit的所有文件覆盖到工作目录，现在HEAD指针指向新的branch。删除所有在checkout前分支被追踪但在checkout后分支中未追踪的文件，清空暂存区（如果并不是checkout当前分支）。与Git不同的是，Git并不会清除暂存区。
  - 失败用例： 
    1. 如果在指定的commit中不存在指定文件，提示：*File does not exist in that commit.*
    2. 如果指定的commit id不存在，提示：*No commit with that id exists.*
    3. 如果指定的branch不存在，提示：*No such branch exists.*
    4. 如果指定的branch就是当前分支，提示：*No need to checkout the current branch.*
    5. 如果工作区中有当前分支未追踪的文件，并且会被checkout覆盖，提示：*There is an untracked file in the way; delete it, or add and commit it first.*

- branch

  - 用法：`/java gitlet.Main branch [branch name]`

  - 说明：

    创建一个新的分支，并且指向当前HEAD指针指向的commit，但并不立即切换到新的分支，当执行 checkout [branch name] 后切换分支（这与Git不同）。如果分支已存在，提示：*A branch with that name already exists.*

- typeof

  - 用法：`/java gitlet.Main typeof [hashcode]`

  - 说明：

    打印给定对象的类型 (blob / tree / commit)

- cat-file

  - 用法：`/java gitlet.Main cat-file [hashcode]`

  - 说明：

    打印给定对象文件。(blob / tree / commit)

- status

  - 用法：`/java gitlet.Main status`

  - 说明：

    显示当前状况。

    - Branches

    - Staged Files

    - Removed Files

    - Modifications Not Staged For Commit

      有三种情况：

      1. Tracked in current commit, changed in working directory, b not staged.
      2. Staged, but with different contents than in the working directory.
      3. Not staged for removal, but tracked in the current commit and deleted from working directory.

    - Untracked Files

- merge

  - 用法：/java gitlet.Main merge [branch name]

  - 说明：

    *split pointer*: a latest common ancestor of the currentand given branch heads.

    1. the split pointer is the same commit as the given branch

       ![Untitled](https://github.com/xminao/Gitlet/raw/master/imgs/08.png)

       do nothing, the merge is complete, and the operation ends with the message: *Given branch is an ancestor of the current branch.*

    2. the split pointer is the current branch

       ![Untitled](https://github.com/xminao/Gitlet/raw/master/imgs/09.png)

       check out the given branch, and the operation ends after printing the message: *Current branch fast-forwarded.*

    3. other wise.

       ![Untitled](https://github.com/xminao/Gitlet/raw/master/imgs/10.png)

       - Any files that have been modified in the given branch since the split point, but not modified in the current branch since the split point should be **changed their versions in the given branch** (checked out from the latest commit at the front of the given branch). **These files should then all be automatically staged**.

       - Any files that have been modified in the current branch but not in the given branch since the split point should stay as they are (do not change).

       - Any files that have been modified in both the current and given branch in the same way (i.e., both files now have the same contents or were both removed) are left unchanged by the merge. If a file was removed from both the current and given branch, but a file of the same name is present in t he working directory, it is left alone and continues to be absent (not tracked nor staged) in the merge.

       - Any files that were not present at the split point only in the current branch should remain as they are.

       - Any files that were not present at the split point and are present only in the given branch should be checked out and staged.

       - Any files present at the split point, unmodified in the current branch, and absent in the given branch should be removed (and untracked).

       - Any files present at the split point, unmodified in the given branch, and absent in the current branch should remain absent.

       - Any files modified in differrent ways in the current and given branches are **in conflict**. “Modified in differentt ways” can mean that the contents of both are chagned and different from other, or the contents of one are changed and the other file is deleted, or the file was absent at the split point and has different contetns in the given and current branches. In this case, replace the contents of the conflicted file with:

         ```
         <<<<<<< HEAD
         contents of file in current branch
         =======
         contents of file in given branch
         >>>>>>>
         ```

         and stage the result. Treat a deleted file in a branch as an empty file. Once ends with merge operation, and the split point was not the current branch or the given branch, merge automatically commits with the log message: *Merged [given branch name] into [current branch name].* Then, if the merge encountered a conflict, print the message: *Encountered a merge conflict.* on the terminal. Merge commits differ from other commits: they records as parents both the head of the current branch (called the first parent) and the head of the branch given on the command line to be merged in.

  - 失败用例：

    - If there are staged additions or removals present, print the error message: *You have uncommitted changes.*
    - If a branch with the given does not exist, print the error message: *A branch with that name does not exist.*
    - If attempting to merge a branch with itself, print the error message: *Cannot merge a branch with itself.*
    - If merge would generate an error, because the commit that it does has no changes in it, just let the normal commit error message for this go through.
    - If an untracked file in the current commit would be overwritten or deleted by the merge, print: *There is an untracked file in the way; delete it, or add and commit it first.* and exist.

### ⌨️ 实现

------

- 对象

  三个对象：blob, tree, commit

  blob：

  tree：

- 暂存

- 提交

- Common ancestor

  an array contains the all common ancestor of two branch.

  BFS

  choose the minimum distance OID in common ancestor array by one of two branch will get same result.
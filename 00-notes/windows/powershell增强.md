主要描述 powershell 的 部分 Linux 命令扩展、主题以及提高效率的插件。为了方便后续使用 Git 中附带的  vim 编辑配置文件，此处先介绍部分 Linux 命令扩展。

## 1. 部分 Linux 命令扩展

使 powershell 使用 grep、awk、which等工具。

需要安装 Git，PATH 中除了 git 所需的 `Git`、`Git\cmd` 之外，额外加入 `Git\usr\bin` 即可，如我的完整路径为:

```
D:\Git
D:\Git\cmd
D:\Git\usr\bin
```

这样就可以使用 grep 等等了，如下面提取 wlan 的 ip (图2):

```powershell
ipconfig | grep -azio 'wlan.*' | grep -ai 'v4' -m 1 | awk '{print $NF}'
```

> 解释: 
>
> grep 参数 `-a` : 指定二进制的类型为 text，`-z`: 使结束符为文末，而不是换行符，用来全文匹配，`-i`: 忽略大小写，`-o`: 仅输出匹配部分，`-m 1`:  显示第一个匹配。
>
> awk: 默认以 空格和  TAB 分割，`'{print $NF}'` 表示打印每行分割后的最后一项，`$NF` 为其内部变量，代表最后一项。

### git-bash 安装其他

windows 的 git-bash 安装一些工具。

> git bash 是基于 MSYS2 的，可以安装 pacman 包管理器来安装下面工具

> 参考
> [Using fish shell with git bash on windows](https://gist.github.com/rafaelpadovezi/1cfc1026f78255458f5a2ea56291ed23)

git-bash 在文件管理器打开文件夹:

```bash
# open path in windows file explore, default to current
win() { : $(cd "${1:-.}" && echo 'start .' | cmd) ; }
```

#### 0. 流程总结

1. 在 [msys2 packages](https://packages.msys2.org/search?t=binpkg&q=pacman) 中搜索下载 (搜索类型需要改为 *Packages* 而不是 *Base Packages*);
2. 如果压缩格式为 zst，则使用 [zstd](https://github.com/facebook/zstd) 解压(release 中下载). `zstd -d xxx.pkg.tar.zst` 再解压 `tar -zxvf xxx.pkg.tar`;
3. 复制到对应目录下。

#### 1. pacman

下载并解压到 git 根目录 [pacman](https://packages.msys2.org/package/pacman?repo=msys&variant=x86_64), [pacman-mirrors](https://packages.msys2.org/package/pacman-mirrors?repo=msys), [msys2-keyring](https://packages.msys2.org/package/msys2-keyring?repo=msys)

设置 pacman 镜像:
```bash
pacman-key --init
pacman-key --populate msys2  # 设置 key
pacman -Syu  # 更新
```

pacman 安装命令:

```bash
pacman -S <package-name>
```

如果提示 exists-on-filesystem，则强制覆盖:  `pacman -S --overwrite "*" <package-name>`

##### 安装 zsh

此处方便复制粘贴，将 oh-my-zsh 配置到 git 文件夹下:

```bash
ZSH=/usr/share/oh-my-zsh

# install oh-my-zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# install plugin
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-$ZSH}/plugins/zsh-autosuggestions
```

在命令行里打开 windows 资源管理器命令:

- cmd: `start E:\Master`

- powershell: `ii E:\Master`

- bash，调 cmd 来实现，例如:

  ```bash
  # open path in windows file explore, default to current
  win() { : $(cd "${1:-.}" && echo 'start .' | cmd) ; }
  ```

> default to zsh:
>
> append this to .bashrc
>
> ```bash
> if [ -t 1 ]; then
>   exec fish
> fi
> ```
>
> tmux conf: `tmux source-file ~/.tmux.conf`

#### 2. 手动安装

对于有子依赖的包安装比较麻烦，收集所有依赖可以在某一台未安装的环境上先清空 pacman 的缓存( 目录 *var/cache/pacman*)，然后安装，然后缓存目录里就包含需要的依赖。

手动安装单个包示例 :

1. 下载二进制包 `mingw-w64-x86_64-jq-1.6-5-any.pkg.tar.zst`
2. 解压 tar 解压命令: `mkdir jq_bin; tar -xvf mingw-w64-x86_64-jq-1.6-5-any.pkg.tar.zst -C jq_bin`。注意需要添加 zstd 到环境变量，tar 需要引用。
3. 复制(git-bash 下执行): `cp -r jq_bin/mingw64 /`

## 2. vim

使用 Git 中附带的 VIM，其 RUNTIME 在: `GitInstalledPath\usr\share\vim\vim82`，其可以自动检测到，故无所配置此项。

用户自定义配置在 `$HOME\_vimrc`，写入自己配置即可。

关于 vim 配置有以下三点需要额外增加:

1. 关于 *Vim E303: Unable to open swap file for "[No Name]", recovery impossible* 问题的解决方案(均测试通过)

    - 更改缓存和备份文件目录，引自 [Disabling swap files creation in vim](https://stackoverflow.com/questions/821902/disabling-swap-files-creation-in-vim) 的回答: [Set the following variables in .vimrc or /etc/vimrc to make vim put swap](https://stackoverflow.com/a/15317146/9825841) (需要手动创建这三个文件夹):

      ```
      set backupdir=~/.vim/backup//
      set directory=~/.vim/swap//
      set undodir=~/.vim/undo//
      ```

    - 禁止缓存和备份文件，引自 [Disable any kind of backup/swap file?](https://stackoverflow.com/questions/49707315/disable-any-kind-of-backup-swap-file):

      ```
      set nobackup
      set noswapfile
      ```
    
2. <ctrl+left>与<ctrl+right> 无法以 word 移动解决方案（引自 [control-left and control-right not working in vim, within a screen session](https://superuser.com/questions/532431/control-left-and-control-right-not-working-in-vim-within-a-screen-session) 的回答 [My `.vimrc` in Ubuntu to handle ctrl arrow keys](https://superuser.com/a/1513950/1178506)）:

    ```
    map  <Esc>[1;5A <C-Up>
    map  <Esc>[1;5B <C-Down>
    map  <Esc>[1;5D <C-Left>
    map  <Esc>[1;5C <C-Right>
    cmap <Esc>[1;5A <C-Up>
    cmap <Esc>[1;5B <C-Down>
    cmap <Esc>[1;5D <C-Left>
    cmap <Esc>[1;5C <C-Right>
    
    map  <Esc>[1;2D <S-Left>
    map  <Esc>[1;2C <S-Right>
    cmap <Esc>[1;2D <S-Left>
    cmap <Esc>[1;2C <S-Right>
    ```

3. 中文乱码，设置编码即可:

    ```
    set encoding=utf-8
    set fileencoding=utf-8
    ```

我的 vim 配置文件[蓝奏云链接](https://potoo.lanzous.com/icuqbgf)，下载后自己修改文件名，效果例图3:

![image-20200522095636496](/md_img/powershell增强.assets/image-20200522095636496.jpg)
(*figure 3*)

## 3. 主题

这一部分的主题配置参考自 [PowerShell 美化指南](https://coolcode.org/2018/03/16/how-to-make-your-powershell-beautiful/#Windows-10-%E6%8E%A7%E5%88%B6%E5%8F%B0%E7%9A%84%E9%A2%9C%E8%89%B2%E8%AE%BE%E7%BD%AE) 。

以下过程推荐在 powershell 管理员权限下进行，这样会将模块放到全局用户的 `C:\Program Files\WindowsPowerShell\Modules`，而不是文档库（一般 C盘是固态盘，文档库所在盘不是固态盘，放在固态盘加载速度会快一点）。

步骤（有需要选择“是否”的，选择“是”）：

1. 使用 [colortool](https://github.com/Microsoft/Terminal/tree/master/src/tools/ColorTool) 更改控制台配色方案。colortool 官方下载链接: [Color Tool April 2019](https://github.com/microsoft/terminal/releases/tag/1904.29002)，iTerm2 配色方案: [mbadolato/**iTerm2-Color-Schemes**](https://github.com/mbadolato/iTerm2-Color-Schemes)，在 [schemes](https://github.com/mbadolato/iTerm2-Color-Schemes/tree/master/schemes) 下下载自己喜欢的配色方案，下载后把 colortool  和配色文件放在一起，在当前文件夹打开 powershell，如我选择 DimmedMonokai，则输入以下:

   ```powershell
   .\ColorTool.exe -b .\DimmedMonokai.itermcolors
   ```

    输入完成之后右击标题栏 --> 属性 --> 确定。上面文件就可以删除了。
    完成之后需要再微调下属性中的背景颜色，否则背景色很奇怪。
   
2. 信任 PS 官方库:

   ```powershell
   Set-PSRepository -Name PSGallery -InstallationPolicy Trusted
   ```

3. 安装 [PSColor](https://github.com/Davlind/PSColor) 以彩色显示文件列表:

   ```powershell
   Install-Module PSColor
   ```

   加载 PSColor，首次加载时需要设置脚本执行策略:

   ```powershell
   Set-ExecutionPolicy -Scope CurrentUser Bypass  # 加载策略更改
   Import-Module PSColor  # 加载测试 PSColor
   ```

4. 安装 [DirColors](https://github.com/DHowett/DirColors) 以更改 PSColor 彩色文件列表的配色:

   ```powershell
   Install-Module DirColors
   ```

   DirColors 配色方案 [seebi/**dircolors-solarized**](https://github.com/seebi/dircolors-solarized)，例如我下载 *dircolors.256dark*，在 $home 路径下新建 *dircolors* 文件夹，复制配色文件到此处，生效配色方案:

   ```powershell
   Update-DirColors ~\dircolors\dircolors.256dark
   ```

5. 命令行主题，类似 ohmyzsh，Powershell 的主题引擎为 [JanDeDobbeleer/**oh-my-posh**](https://github.com/JanDeDobbeleer/oh-my-posh)，安装:

   ```powershell
   Install-Module posh-git  # 安装 ohmyposh 依赖
   Install-Module oh-my-posh  # 安装 ohmyposh
   ```

   配置主题:

   ```powershell
   Set-Theme desiredTheme
   ```

   主题在 oh-my-posh 的安装路径的 Theme 下，我的主题是在 Paradox 基础上修改了自己喜欢的颜色以及添加了起始符，[点击此处下载](https://github.com/potoo0/django_note/blob/master/powershellConfig/Paradox_changeColor.psm1)。效果如下图4:

   ![image-20200521174510125](/md_img/powershell增强.assets/image-20200521174510125.png)(*figure 4*)

主题和配色需要每次都执行，比较麻烦，此处将其写入到全局启动脚本中（见下面的<6. 启动脚本>），效果如下例图 5:

![image-20200522100004271](/md_img/powershell增强.assets/image-20200522100004271.png)(*figure 5*)

## 4. 自动补全

基本需求是可以像 [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions) 那样提示和补全历史命令。

此功能由 [PowerShell/**PSReadLine** issue 687: Fish-like autocompletion](https://github.com/PowerShell/PSReadLine/issues/687) 发起讨论，并在 [issue 1468: Predictive IntelliSense](https://github.com/PowerShell/PSReadLine/issues/1468) 中得到实现，有时间的伙伴可以点击链接去查看作者的解答，我这里简述下 1468 中完成的功能：1. 完成提示和补全历史命令；2. 增加动态提示函数以及参数。

将 [PSReadLine](https://www.powershellgallery.com/packages/PSReadLine/2.1.0-beta1) 更新到 2.1.0-beta1 以上即可:

```powershell
Install-Module -Name PowerShellGet -Force  # 升级 PSGEt
Install-Module -Name PSReadLine -AllowPrerelease -Force  # 升级 PSReadLine
```

在全局启动脚本中（见下面的<6. 启动脚本>）加入快捷键绑定:

```powershell
Set-PSReadLineKeyHandler -Key "Ctrl+d" -Function MenuComplete
Set-PSReadLineKeyHandler -Key "Ctrl+f" -Function ForwardWord
Set-PSReadLineKeyHandler -Key "Ctrl+b" -Function BackwardWord
Set-PSReadLineKeyHandler -Key "Ctrl+z" -Function Undo
Set-PSReadLineKeyHandler -Key UpArrow -Function HistorySearchBackward
Set-PSReadLineKeyHandler -Key DownArrow -Function HistorySearchForward
```

在加入颜色更改，此处修改提示的颜色为暗绿色(默认的浅灰色在 vscode 暗色主题下阅读困难):

```powershell
Set-PSReadLineOption -Colors @{ Prediction = '#2F7004'}
```

效果如下例图6，输入 p 后，会根据输入历史以暗绿色提供建议:

![image-20200522104431139](/md_img/powershell增强.assets/image-20200522104431139.png)
(*figure 6*)

## 5. sudo 提权

Linux 终端下比较便捷的一点就是让当前用户临时具有 root 权限，插件 [gerardog/**gsudo**](https://github.com/gerardog/gsudo) 可以实现 cmd/powershell 的提权，下面内容均取自作者 repo。

通过 powershell 安装:

```powershell
PowerShell -Command "Set-ExecutionPolicy RemoteSigned -scope Process; iwr -useb https://raw.githubusercontent.com/gerardog/gsudo/master/installgsudo.ps1 | iex"
```

其命令是 `gsudo`，在首次执行时其可以提示是否创建 `sudo` 的软链接。

其特性:

- 默认在当前窗口提权。(参数 `-n` 可以新打开窗口);
- 权限缓存：UAC 弹窗一次后，几分钟内再次提权不再 UAC 弹窗（0.7 版本下我测试未起效）;
- 可直接追加操作，如: `gsudo touch testSudo.md`

目前已知 BUG：直接追加的操作不支持自定义函数，如: `sudo TestUserFunc` 会报错。我已经在其 repo 中发起 issue: [Cannot call the function defined in profile](https://github.com/gerardog/gsudo/issues/33)，作者给的建议是不要在启动脚本放太多自定义函数，因为其会导致加载缓慢，针对提权后调用自定义函数有两种方案：

- 先 sudo，然后再执行自定义函数;
- 在 profile 中定义变量来拆分自定义函数。

## 6. 启动脚本

综上，此处将文件列表配色、主题加载、自动补全快捷键、自动补全背景色的配置加入全局启动脚本:

1. 全局启动脚本位置: `C:\Windows\System32\WindowsPowerShell\v1.0\profile.ps1`，如果没有则自己新建，新建后定义一个变量存储启动脚本位置，方便后续 vim 编辑，此处再加入 hosts 文件的位置:

   ```powershell
   $profile = "C:\Windows\System32\WindowsPowerShell\v1.0\profile.ps1"
   $hostfile = "C:\Windows\System32\drivers\etc\hosts"
   
   # 使用 `vim $profile` 编辑启动脚本
   # 使用 `vim $hostfile` 编辑 hosts 文件
   ```

2. 将上面加载文件列表配色、主题的代码写入，如:

   ```powershell
   #Import-Module DirColors
   #Import-Module posh-git
   #Import-Module oh-my-posh
   Update-DirColors ~\dircolors\dircolors.256dark
   Set-Theme Paradox_changeColor
   ```

3. 自动补全快捷键、自动补全背景色的配置，见<4. 自动补全>，此处不再粘贴。

最终我的完整配置文件如下图 7:

![image-20200611170307461](/md_img/powershell增强.assets/image-20200611170307461.png)
(*figure 7*)


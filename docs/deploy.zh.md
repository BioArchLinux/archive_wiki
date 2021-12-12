# 部署

本文将介绍如何使用 ArchLinuxCN 的 python 脚本 [lilac](https://github.com/archlinuxcn/lilac) 以及 [依云](https://github.com/lilydjwg)的 [archrepo2](https://github.com/lilydjwg/archrepo2) 脚本去构建自己的 ArchLinux 软件仓库。使用这两个脚本是因为很多时候软件维护只需要修改哈希值以及版本号一类的工作，而人不可能总盯着网站看，所以更新难免不及时，`lilac`和`archrepo2`解决的就是这个问题，可以帮助节省大部分的人力去维护仓库。

之前有提到 ArchLinux 是一个社区驱动、易于参与的 Linux 发行版，本文也将顺带提及一些参与其中的一些基本操作。如有纰漏错误，劳烦指正。

PKGBUILD
--------

ArchLinux 的软件包一般都过 Shell 脚本 PKGBUILD 进行构建，相比于 Debian 系、RH 系，容易了不少。

以下为最基础的 PKGBUILD。

    # Maintainer: Your Name <youremail@domain.com>
    pkgname=NAME
    pkgver=VERSION
    pkgrel=1
    pkgdesc=""
    arch=()
    url=""
    license=('GPL')
    groups=()
    depends=()
    makedepends=()
    optdepends=()
    provides=()
    conflicts=()
    replaces=()
    backup=()
    options=()
    install=
    changelog=
    source=($pkgname-$pkgver.tar.gz)
    noextract=()
    md5sums=() #autofill using updpkgsums
    prepare() {
      cd "$pkgname-$pkgver"
    }
    build() {
      cd "$pkgname-$pkgver"
    
      ./configure --prefix=/usr
      make
    }
    
    package() {
      cd "$pkgname-$pkgver"
    
      make DESTDIR="$pkgdir/" install
    }

通常是会有 包名 `pkgname`，版本 `pkgver`，版本构建号 `pkgrel`，包描述 `pkgdesc`，框架 `arch`，包的网址 `url`，证书 `license`，运行依赖 `depends`，构建依赖 `makedepends`，可选依赖 `optdepends`，提供了什么包 `provides`，和什么包冲突 `conflicts`，构建包需要的源 `source`，验证 `md5sums` 除了这个还有很多种验证。其余都不是必须的。

`prepare() { }`，`build() { }`以及`package() { }`里面分别有准备整理源文件、构建编译、安装的 shell 脚本，其中`package`是会进入伪root的环境。此外还会有`pkgver()`和`check()`两个函数。

其中最重要的是两个文件夹是`"$srcdir"`和`"$pkgdir"` ，`srcdir`是各种源文件的文件夹，而`pkgdir`则是伪root的文件夹，比如你若想安装`srcdir`里的二进制`BINARY`到`/usr/bin/BINARY`，则应该如此写`package()`

    package() {
      cd ${srcdir}
      install -Dm755 BINARY "$pkgdir"/usr/bin/BINARY
    }

写完，如果想打包，则应该`cd`到 PKGBUILD 所在目录通过打包

    makepkg -si

若今后配置好了 GPG Key，则可以通过以下命令打包并签名获得`sig`文件

    makepkg -si --sign

自然也可以通过`devtools`打包，好处是可以在一个干净的环境下打包，但是如果遇见需要依赖非官方仓库包的情况，就会比较麻烦，最好使用下文提到的`archrepo2`脚本生成本地仓库，然后将本地的仓库写入`/usr/share/devtools`下对应的`conf`文件，比如，如果要使用`extra`c仓库打包，则应该修改`pacman-extra.conf`文件，添加形式，可见下文使用章节。

SSH Key
-------

因为要使用 Git 托管平台抑或是 AUR，需要管理 SSH 相关的 key。

因此安装`openssh`包

    sudo pacman -S openssh

使用以下命令生成密钥

    ssh-keygen -t ed25519 -f ~/.ssh/aur

使用以下命令查看公钥

    cd ~/.ssh
    cat aur.pub

AUR
---

要成为 AUR 维护者，需要在 [AUR](https://aur.archlinux.org/) 里注册一个帐号，并且在 `My Account` > `SSH Public Key` 中添加 `cat aur.pub` 的内容，然后输入 `Your current password` 之后才可以更新你的信息，然后方可通过 `git` 来管理 AUR 。

先拉取仓库

    git clone ssh://aur@aur.archlinux.org/PKGNAME.git

注意，如果使用 https 而不是 ssh 将无法维护 AUR 包。

    cd PKGNAME
    vim PKGBUILD

书写你的 PKGBUILD，对于非 git 一类的 AUR 包，对于各类 sums ，应该使用以下的命令更新

    updpkgsums

updpkgsums 会自动下载源文件，但是注意 AUR 包仓库一般不能包含源文件，除了自己做的补丁、桌面文件或者 logo 图片。

除此之外，`.SRCINFO`也是记录元数据的一个文件，对于 AUR 来说是必要的

    makepkg --printsrcinfo > .SRCINFO

所有的一切准备好了，就可以通过 git 上传了

    git add PKGBUILD .SRCINFO
    git commit -m "some description"
    git push

然后这个包您就可以管理了，如果你不想维护，可以选择`Disown Package`。另外，如果你已经清除掉其他不必要的文件，需要添加所有文件，可以将第一行命令，替换成如下命令。

    git add .

GPG Key
-------

GPG Key 签名是一个比较正式的 ArchLinux 仓库应该具备的，一般会有一个包，一个对应包的`sig`文件。

如果要生成密钥，应该在非root用户下使用以下命令。因为这个用户今后要用于运行`lilac`和`archrepo2`

    gpg --full-gen-key

一切都可以选择默认，但是不要设置密码。记住 KEY\_ID 即可

之后应该把 GPG Key 上传，Ubuntu 的 keyserver 国内用比较稳定（也不知道为什么）

    gpg --keyserver hkp://keyserver.ubuntu.com --send-keys KEY_ID

Git 托管平台
--------

因为`lilac`需要 git 仓库，所以最好需要一个脚本的 git 托管平台，这里以比较流行的 GitHub 为例。

首先如果你为仓库维护者，应该在`Settings` > `Profile`中设置好`Public email`，不然`lilac`探测不到，会报错。

并且要以`NON_ROOT_USER`身份生成 SSH Key ，以访问  GitHub 仓库。

    ssh-keygen -t rsa -C YOUR_EMAIL -f ~/.ssh/git

    cd ~/.ssh
    cat git.pub #查看

在 `Settings` > `SSH key` 添加 cat 出的内容。

此时，可以新建一个仓库，然后使用 ssh 的方式把仓库 clone 到服务器，这里以我维护的仓库为例子。

    git clone git@github.com:BioArchLinux/Packages.git

这里 clone 下拉的文件夹就是今后`lilac`的`REPO_DIR`。一般我是把各个包作为一个文件夹，文件夹里有 PKGBUILD 还有 lilac 的各个包的配置文件，放置于`REPO_DIR`下。

lilac & archrepo2 部署
--------------------

`lilac`依赖`nvchecker`进行版本检测，之后对 PKGBUILD 进行一定的修改进行版本的更新，进行打包，并且将打好的包放进一个指定的文件夹中。`archrepo2`则是对一个指定的文件夹中的包进行架构的划分一类的操作，生成仓库的数据库。

### lilac

安装 `lilac` 可以从 AUR 中安装，也可以从 archlinuxcn 的仓库下载，这里以 AUR 为示例（如果从 AUR 中安装，则需要在非root用户下运行）

    yay -S lilac-git

下载完成后，需要配置各种设置，lilac的设置是在`~/.lilac/config.toml`处，需要自己配置，而且`~`应该为在非root用户的`/home/NON_ROOT_USER`，因为`makepkg`命令无法在root下运行。

下面为我的基本配置，主意将`YOUR_REPO_NAME`，`NON_ROOT_USER`，`PACKAGES_SCRIPT_DIR`，`REPO_DIR`，`YOUR_NAME`，`YOUR_EMAIL`替换成你的相关信息。其中`YOUR_REPO_NAME`就是和`extra`，`community`一类的 Arch 仓库同类别的名字，到时候是要写入`/etc/pacman.conf`中去的，而`PACKAGES_SCRIPT_DIR`就是你存有`PKGBUILD`文件的地方，而`REPO_DIR`则是需要通过 Nginx 一类的软件变成可被公网访问仓库的所在文件，同时，这个文件夹也应该是`archrepo2`应该处理的文件夹。

    [envvars]
    TZ = "Asia/Shanghai"
    TERM = "xterm"
    # this doesn't help with Python itself; please set externally if desirable
    # LANG = "zh_CN.UTF-8"
    
    [repository]
    name = "YOUR_REPO_NAME"
    email = "repo@example.com"
    repodir = "/home/NON_ROOT_USER/PACKAGES_SCRIPT_DIR"
    # The path where built packages and signatures are copied to
    # comment out if there's no need to copy built packages
    destdir = "/home/NON_ROOT_USER/REPO_DIR"
    
    [lilac]
    name = "YOUR_NAME"
    email = "YOUR_EMAIL"
    master = "YOUR_NAME <YOUR_EMAIL>"
    # Set and unsubscribe_address to receive unsubscribe requests
    # unsubscribe_address = "unsubscribe@example.com"
    # Set to yes to automatically rebuild packages which failed to build last time
    rebuild_failed_pkgs = true
    git_push = true
    send_email = false
    # Optional: template for log file URL. Used in package error emails
    # logurl = "https://example.com/${pkgbase}/${datetime}.html"
    # for searching github
    # github_token = "xxx"
    
    [nvchecker]
    # set proxy for nvchecker
    # proxy = "http://localhost:8000"
    
    [smtp]
    # You can configure a SMTP account here; it defaults to localhost:53
    #host = ""
    #port = 0
    #use_ssl = false
    #username = ""
    #password = ""
    # Set to true to allow ANSI characters in content
    use_ansi = true
    
    [bindmounts]
    # bind mounts in the devtools enviroment, e.g. for caching
    # source directories will be created if not yet
    "~/.cache/archbuild-bind-cache" = "/build/.cache"
    "~/.cache/archbuild-bind-cache/stack" = "/build/.stack"
    "~/.cache/go-build" = "/build/.cache/go-build"
    "~/.cache/pip" = "/build/.cache/pip"
    "~/.cargo" = "/build/.cargo"
    "~/go" = "/build/go" 

除此之外，运行lilac的用户应该处于`pkg`用户组

    sudo usermod -G pkg NON_ROOT_USER

因为`makepkg`需要密码，为了lilac能在后台运行需要进行用户sudo免密码的操作

    su root
    vim /etc/sudoers

添加如下内容

    NON_ROOT_USER ALL = (ALL) NOPASSWD:ALL

但是光是如此，仍然无法让lilac在后台运行，需要运行一下命令修改一下登录配置

    sudo loginctl enable-linger NON_ROOT_USER

定时运行则需要写一下systemd脚本

    cd /usr/lib/systemd/system 

    sudo vim lilac.service

添加如下内容

    [Unit]
    Description=lilac
    Wants=lilac.timer
    
    [Service]
    User=NON_ROOT_USER
    Type=simple
    ExecStart=/usr/bin/lilac

    sudo vim lilac.timer

添加如下内容

    [Unit]
    Description=Runs lilac very 6 hour
    
    [Timer]
    OnBootSec=30s
    OnUnitActiveSec=6h
    Unit=lilac.service
    
    [Install]
    WantedBy=multi-user.target

运行以下命令使脚本生效

    sudo systemctl enable lilac.timer
    sudo systemctl start lilac.timer

如果要想手动运行`lilac`，以配置的`NON_ROOT_USER`运行`lilac`即可。

log则可以在`~/.lilac/log`下找到，通常都以时间为文件夹，文件夹下必有`lilac-main.log`还有其他以包命名的单独的 log 文件

### archrepo2

安装也是可以从 archlinuxcn 仓库或者 AUR 安装

    yay -S archrepo2-git

`archrepo2`的配置文件位于`/etc/archrepo2.ini`

一般只需要修改配置的三处，均位于`[repository]`下 ，主意将`YOUR_REPO_NAME`，`NON_ROOT_USER`，`REPO_DIR`与`lilac`配置的相同即可

    [repository]
    # Name of the repository. In below example the Pacman repository db file name
    # will be archlinuxcn.db.tar.gz
    name: YOUR_REPO_NAME
    
    # Path to the repository - directory should normally contain any, i686 and
    # x86_64. The server will monitor files in it with inotify. If you have lots of
    # files in this directory, remember to update the configuration of inotify.
    path: /home/NON_ROOT_USER/REPO_DIR
    
    # If enabled, packages put into this directory will be moved into the repo.
    # This path should be on the same filesystem as the repo path
    # Should be used with auto-rename on
    spool-directory: /home/NON_ROOT_USER/REPO_DIR

运行`archrepo2`使用如下命令

    /usr/bin/archreposrv /etc/archrepo2.ini

若要开机自启动，`systemd`操作如下

    systemctl enable archrepo2

lilac 脚本编写
----------

跟`PKGBUILD`同目录下，一般可以有`lilac.yaml`和`lilac.py`两个文件。

### lilac.yaml

其中`lilac.yaml`是较为重要的一个文件，里面包含若干`properties`，详见`lilac.yaml`的[文档](https://archlinuxcn.github.io/lilac/)。

重要的有几项`maintainers`，`build_prefix`，`update_on`以及`repo_depends`。

`build_prefix`是看看一些官方的依赖需要在哪个官方的 repository ，比如有的包需要 wine，而 wine 则是位于 `multilib`的仓库，则应该如此填写

    build_prefix: multilib

至于`maintainers`，一般是填写自己 GitHub 上的相关信息，有时候不填写邮箱，可能会报错，所以稳妥写法如下

    maintainers:
      - github: YOUR_USERNAME
        email: YOUR_PUBLIC_EMAIL

`update_on`因为是使用`nvchecker`的，所以要查看`nvchcker`的[文档](https://nvchecker.readthedocs.io/en/latest/usage.html)。一般有几种写法，现在列举如下，一般如果 ARU 包管理者很勤奋或者上游维护的状态很不好，则可以这样。`strip_release`写为`true`是因为如果他自动基于 AUR，版本号会为`pkgver-pkgrel`，这样有`-`就不能作为版本号了，因为包含不允许的字符，这个参数是让去掉`-pkgrel`。

    update_on:
      - source: aur
        aur: tnt-gui
        strip_release: true

如果你需要依赖于 GitHub 仓库，可参考如下写法。其中`ugeneunipro/ugene`是基于`https://github.com/ugeneunipro/ugene`，一些时候有些仓库步伐不 release，只发布 tag，则需要更换`use_latest_release`为`use_max_tag`。

    update_on:
      - source: github
        github: ugeneunipro/ugene
        use_latest_release: true

如果源不是一个被保存在`nvchcker`的[文档](https://nvchecker.readthedocs.io/en/latest/usage.html)提及的位置，则需要考虑使用正则，`url`是所在网址，而`regex`可以为页面直接匹配的文本。

    update_on:
      - regex: freedelta_(\d+.\d+.\d+)_amd64.deb
        source: regex
        url: https://sourceforge.net/projects/freedelta/files/freedelta/

如果一些网页，出现一个 click here 一类的下载链接，则应该使用 F12 读取下 HTML 代码，下面有个示例，通过超链接来获取源。

    update_on:
      - source: regex
        url: http://doua.prabi.fr/software/seaview
        regex: href="seaview_data/seaview(\d+)-64\.tgz"

除了上述提到的，还有些补偿用到的`from_pattern`，`to_pattern`，`include_regex`等等，都可以参考文档。

`repo_depends`则是为了处理官方仓库没有所需依赖的情况，需要在仓库中打包完成依赖项后，在需要依赖的包的`lilac.yaml`文件中加入如下示例，`gconf`是需要被修改成你所需要依赖的。

    repo_depends:
      - gconf

### lilac.py

以下是一个比较常用的`lilac.py`文件

    #!/usr/bin/env python3
    
    from lilaclib import *
    
    def pre_build():
      update_pkgver_and_pkgrel(_G.newver.lstrip('v'))
    
    def post_build():
      git_add_files('PKGBUILD')
      git_commit()

其余可以参考`lilac.api`的[文档](https://lilac.readthedocs.io/en/latest/api.html)以及[archlinuxcn](https://github.com/archlinuxcn/repo)和[arch4edu](https://github.com/arch4edu/arch4edu)的仓库。

certbot 配置
----------

一般较为正式的网站都会有 SSL 证书，这里使用 certbot 签发 SSL 证书。

使用 certbot 并配置 nginx 所需的证书需要安装如下两个包

    sudo pacman -S certbot
    sudo pacman -S certbot-nginx

使用 certbot 申请证书但是不需要修改 Nginx 配置，可使用如下命令

    certbot -d repo.YOUR_DOMAIN -m YOUR_EMAIL --nginx certonly

会生成了两个证书，位置如下，这个位置在 Nginx 配置会用到

    Certificate is saved at: /etc/letsencrypt/live/repo.YOUR_DOMAIN/fullchain.pem
    Key is saved at:         /etc/letsencrypt/live/repo.YOUR_DOMAIN/privkey.pem

若要自动续签证书可使用如下 systemd 脚本

    sudo vim /etc/systemd/system/letsencrypt.service

    [Unit]
    Description=Let's Encrypt renewal
    
    [Service]
    Type=oneshot
    ExecStart=/usr/bin/certbot renew --quiet --agree-tos
    ExecStartPost=/bin/systemctl reload nginx.service

    sudo vim /etc/systemd/system/letsencrypt.timer

    [Unit]
    Description=Monthly renewal of Let's Encrypt's certificates
    
    [Timer]
    OnCalendar=daily
    Persistent=true
    
    [Install]
    WantedBy=timers.target

启动 systemd 的相关配置

    sudo systemctl enable letsencrypt.timer
    sudo systemctl start letsencrypt.timer

Nginx 配置
--------

Nginx 可以使得仓库所在文件夹通过一定的域名访问。

首先需要安装 Nginx

    sudo pacman -S nginx

在 ArchLinux 中，Nginx 的配置文件存放于`/etc/nginx`下

如要方便配置多个站点可以修改`/etc/nginx/nginx.conf`

    sudo vim /etc/nginx/nginx.conf

在已有的`http { }`中添加如下一行，这个的意思，是`etc/nginx/conf.d`下的所有的`conf`文件都将生效，目前 ArchLinux 的 Nginx 不支持类似 Ubuntu 的配置方法即不支持软链接。

    include       conf.d/*.conf;

然后添加`conf.d`文件夹

    cd /etc/nginx/
    sudo mkdir conf.d

然后可以在`conf.d`文件夹下配置`repo.YOUR_DOMAIN`

    cd /etc/nginx/conf.d
    sudo vim repo.conf

添加如下内容，将`repo.YOUR_DOMAIN`和`/YOUR/REPO/PATH`分别替换成你自己的域名和仓库文件夹。

    server {
    		listen 80;
    		server_name repo.YOUR_DOMAIN;
    		return 301 https://$host$request_uri;
    }
    server {
    		listen 443 ssl http2;
    		server_name repo.YOUR_DOMAIN;
    		ssl_certificate /etc/letsencrypt/live/repo.YOUR_DOMAIN/fullchain.pem;
    		ssl_certificate_key /etc/letsencrypt/live/repo.YOUR_DOMAIN/privkey.pem;
    		ssl_session_cache builtin:1000 shared:SSL:10m;
    		ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    		ssl_ciphers HIGH:!aNULL:!eNULL:!EXPORT:!CAMELLIA:!DES:!MD5:!PSK:!RC4;
    		ssl_prefer_server_ciphers on;
    		access_log /var/log/nginx/access.log;
        
    		location / {
    			autoindex on;
    			autoindex_exact_size off;
    			autoindex_localtime on;
    			root /YOUR/REPO/PATH;
    			index index.html index.htm index.php;
    			proxy_set_header Host $host;
    			proxy_set_header X-Real-IP $remote_addr;	       
    		}    
    
    }

别忘了更新配置

    sudo systemctl enable nginx
    sudo systemctl restart nginx

使用
--

到这里你的仓库就已经可以使用了，这里我以 BioArchLinux 为例，在你本机的电脑里修改`/etc/pacman.conf`

    sudo vim /etc/pacman.conf

添加如下内容，将`BioArchLinux`替换成你`lilac`中的`YOUR_REPO_NAME`，然后`Server`则是`https://repo.YOUR_DOMAIN/$arch`

    [BioArchLinux]
    Server = https://repo.malacology.net/$arch

可以将其放置于一定的位置，比如我知道，我不会用 ArchLinux 官方库的 tracer （一个与 BEAST 系列软件重名的软件），我就会把 BioArchLinux 的相关配置，在`/etc/pacman.conf`中放置于`core`和`extra`下方，`community`和`multilib`之上，这样就不会一直出现`community`里的 tracer 版本一类的提示了。

因为使用了 GPG 签名，所以要导入 GPG key，这里的`B1F96021DB62254D`便是 GPG Key 提到的`KEY_ID`。

    sudo pacman-key --recv-keys B1F96021DB62254D
    sudo pacman-key --finger B1F96021DB62254D
    sudo pacman-key --lsign-key B1F96021DB62254D

之后，更新一下

    sudo pacman -Syu

就可以使用了

致谢
--

感谢  [依云](https://github.com/lilydjwg) 在 lilac 配置和使用方面所给予的帮助，同时也感谢 [Jingbei Li](https://github.com/petronny) 在 lilac 配置时给予的帮助。另感谢一位 TG 不愿意透露姓名的大叔叔检查我书写的低级错误的，还有 [NotYu](https://github.com/NotYuhang) 在正则上面给予的帮助。

PS：如果本文有任何疏漏和错误，跟上述各位毫无关系，只跟菜菜的我有关系。希望多多包涵。上述操作的成果即是 [BioArchLinux](https://github.com/BioArchLinux/Packages) 仓库，一个针对生信软件的仓库，如果感兴趣可以多多使用、参与进来，也欢迎 star 。

# Deploy

This article will introduce how to use the python script [lilac](https://github.com/archlinuxcn/lilac) and [archrepo2](https://github.com/lilydjwg/archrepo2) written by [依云](https://github.com/lilydjwg) to build your own ArchLinux repository. People cannot always pay attention to the pkg ver changes when it is updated, and sometimes maintainers only need to change the sums and pkgver, this is the problem that `lilac` and `archrepo2` mainly solved, it can save our efforts to maintain the repository.


PKGBUILD
--------

The packages of ArchLinux mainly are build by Shell script called PKGBUILD. The building process is much easier than other Linux distribution faimily, such as Debian, RH and so on.

The following is the simple PKGBUILD containing basic setting。

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

It usually contains `pkgname`, `pkgver`, `pkgrel`，description `pkgdesc`, `arch`，the offical web `url`, `license`, `depends`, `makedepends`,  `optdepends`, what it provides `provides`, `conflicts`, `source`，`md5sums` and other sums. Others are not mandatory.

`prepare() { }`, `build() { }` and `package() { }` sperately contain shell scripts of preparing source files, building package, and installing them to the $pkgdir. And `package` can enter the fakeroot enviroment. There are also two functions called `pkgver()` and `check()`.

`"$srcdir"` and `"$pkgdir"` are the two main dir, `srcdir` contains the source code or pre-compiled binary from the source, and `pkgdir` is a fakeroot dir, containging the binary or other files will be installed. If you want to install the binary `BINARY` from srcdir to `/usr/bin/BINARY`，you should write the `package()` follwing the followed examples.

    package() {
      cd ${srcdir}
      install -Dm755 BINARY "$pkgdir"/usr/bin/BINARY
    }

When you finished writing PKGBUILD and other essential files for packaging, you can `cd` the root dir of PKGBUILD and run

    makepkg -si

If you have configured the GPG Key, you can sign the package using the following command to get the `sig` file

    makepkg -si --sign

Also, you can use `devtools` for packaging, the benefits is that you can package in a clean enviroment, but if you have to work on packages with unoffical packages depends, it would be hard. You can use `archrepo2` to build your local repository and them write your local repository path to the `conf` file from the `/usr/share/devtools` dir, for example, if you want to use offical `extra` repository to build your packages, you should edit the `pacman-extra.conf` file and add some text, the following content will mention this situlation.

SSH Key
-------

Considering the repo managed by lilac must be git repo, and AUR also use git, SSH releated keys are crucial.

Therefore, install `openssh`

    sudo pacman -S openssh

and use the following command to generate ssh key

    ssh-keygen -t ed25519 -f ~/.ssh/aur

use follwing command to check public key

    cd ~/.ssh
    cat aur.pub

AUR
---

If you want to be an AUR maintainer, you shoud register on the [AUR](https://aur.archlinux.org/), and add your public ssh key to this account, `My Account` > `SSH Public Key` add the content of `cat aur.pub`, and then input `Your current password`, finally, you can update your information.

Pull the repo

    git clone ssh://aur@aur.archlinux.org/PKGNAME.git

Attention! use https can not maintain AUR. Please use ssh instead of https.

    cd PKGNAME
    vim PKGBUILD

write the PKGBUILD. For non-git AUR packages, sums should be added, you can use the command below to auto update it.

    updpkgsums

updpkgsums will auto download the source file, but AUR repo usually cannot contain the source file except patch, desktop file or log photos。

And `.SRCINFO` file is the meta file for the AUR, maybe essentail for the AUR only, command below to update or generate `.SRCINFO`

    makepkg --printsrcinfo > .SRCINFO

Then you can use git to update it

    git add PKGBUILD .SRCINFO
    git commit -m "some description"
    git push

Now you maitain this package, if you donnot want to continue maintaining it, click `Disown Package`. If you have clean all the extra files for the AUR, you can replace the first line command with the follwing.

    git add .

GPG Key
-------

GPG Key sign is the feature of an formal ArchLinux. One pkg file should have its own `sig` file correspending.

Generate GPG Key should be run using non-root user, this non-root user will be used for runing `lilac` and `archrepo2`. The following command is to generate the gpg key.

    gpg --full-gen-key

All the setting can be default but the secrets password, don not set it. Remember your KEY\_ID.

Then you should upload your GPG Key to the GPG server, the Ubuntu keyserver is stable for Chinese users. 

    gpg --keyserver hkp://keyserver.ubuntu.com --send-keys KEY_ID

Git platform
--------

Considering the needs of `lilac`, git repo is essentail. Here use the GitHub as example.

Firstly, if you are the package maintainer, you should public your email, `Settings` > `Profile` set `Public email`, or `lilac` won't find you and will report error。

and generate SSH Key using the `NON_ROOT_USER` to access the GitHub repo.

    ssh-keygen -t rsa -C YOUR_EMAIL -f ~/.ssh/git

    cd ~/.ssh
    cat git.pub #view

and add the content of cat to `Settings` > `SSH key`.

At the same time, you can create a new repo, and use the ssh method to upload files to the remote repo. On the server, you should clone the packages repo.

    git clone git@github.com:BioArchLinux/Packages.git

The path of this dir is the `REPO_DIR` for the lilac. Generally, I sperately put every package in one standalone dir. The standalone dir should contain PKGBUILD, lilac.yaml and lilac.py.

lilac & archrepo2 Deploy
--------------------

The version check function of `lilac` depends on the `nvchecker`, and then mod the PKGBUILD to update the pkgver for packaging, and then put the ArchLinux packages into a specific dir. `archrepo2` can divide the packages into different arch dir and generate database for this repo.

### lilac

Install `lilac` can be done via AUR or archlinuxcn repo. Here the command is to install `lilac` from AUR.

    yay -S lilac-git

After installing it, configure is crucail. the config of lilac is `~/.lilac/config.toml`, you should write it by yourself, and `~` should be the non-root user's `/home/NON_ROOT_USER`, considering `makepkg` command can run under root.

The following is my basic config file, please replace `YOUR_REPO_NAME`, `NON_ROOT_USER`, `PACKAGES_SCRIPT_DIR`, `REPO_DIR`, `YOUR_NAME`, `YOUR_EMAIL` with your own infomation.Attention, `YOUR_REPO_NAME` is same as the offical repository called `extra`，`community`, will be written into `/etc/pacman.conf`, and `PACKAGES_SCRIPT_DIR` is the path of the dir containing `PKGBUILD` files, and `REPO_DIR` is the repository path. Nginx will let `REPO_DIR` become the website can be visited, and this dir is also the dir `archrepo2` works on.

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

The non-root user running lilac should under the `pkg` user group

    sudo usermod -G pkg NON_ROOT_USER

considering the `makepkg` needs passwd, use the following command to let lilac run without passwd for sudo command.

    su root
    vim /etc/sudoers

add follwing content

    NON_ROOT_USER ALL = (ALL) NOPASSWD:ALL

But if you want to run lilac in the background, it's not enough, you should run the follwing command to change login config.

    sudo loginctl enable-linger NON_ROOT_USER

To run regularly, you need to write a systemd script.

    cd /usr/lib/systemd/system 

    sudo vim lilac.service

Add following content

    [Unit]
    Description=lilac
    Wants=lilac.timer
    
    [Service]
    User=NON_ROOT_USER
    Type=simple
    ExecStart=/usr/bin/lilac

Then edit lilac.timer

    sudo vim lilac.timer

add following command

    [Unit]
    Description=Runs lilac very 6 hour
    
    [Timer]
    OnBootSec=30s
    OnUnitActiveSec=6h
    Unit=lilac.service
    
    [Install]
    WantedBy=multi-user.target

run the following command to let it work

    sudo systemctl enable lilac.timer
    sudo systemctl start lilac.timer

You can also manually run the `lilac`, just type `lilac` under your `NON_ROOT_USER`

log can be found under the dir`~/.lilac/log`, generally is the dir named by time, every time dir must have `lilac-main.log` and other log for building packages sperately.

### archrepo2

You can install it via AUR or archlinuxcn repo

    yay -S archrepo2-git

The config file of `archrepo2` is `/etc/archrepo2.ini`

You need to change three settings, all is under `[repository]`, please keep step with the `lilac` config, especially for `YOUR_REPO_NAME`, `NON_ROOT_USER`, `REPO_DIR`

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

Manually run `archrepo2` using the follow command

    /usr/bin/archreposrv /etc/archrepo2.ini

To make it always work, using `systemd`

    systemctl enable archrepo2

lilac script writing
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

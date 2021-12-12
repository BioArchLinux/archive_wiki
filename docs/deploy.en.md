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

`"$srcdir"` and `"$pkgdir"` are the two main dir, `srcdir` contains the source code or pre-compiled binary from the source, and `pkgdir` is a fakeroot dir, containging the binary or other files will be installed. If you want to install the binary `BINARY` from srcdir to `/usr/bin/BINARY`，you should write the `package()` following the followed examples.

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

use following command to check public key

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

Now you maitain this package, if you donnot want to continue maintaining it, click `Disown Package`. If you have clean all the extra files for the AUR, you can replace the first line command with the following.

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

lilac & archrepo2 deploy
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

add following content

    NON_ROOT_USER ALL = (ALL) NOPASSWD:ALL

But if you want to run lilac in the background, it's not enough, you should run the following command to change login config.

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

The lilac script should be at the same dir of `PKGBUILD`, generally, there are two files called `lilac.yaml` and `lilac.py`.

### lilac.yaml

`lilac.yaml` is the more important file, containing serveral `properties`, for details, see the [document](https://archlinuxcn.github.io/lilac/) of lilac.yaml。

The important parts of this file include `maintainers`, `build_prefix`, `update_on` and `repo_depends`.

`build_prefix` is the where the depends from. For example, some packages need wine and wine is at offical `multilib` repo, so it can be written as

    build_prefix: multilib

For `maintainers`, should input your GitHub information.Sometimes you donnot input your email, it will report error, so it's better to write like this.

    maintainers:
      - github: YOUR_USERNAME
        email: YOUR_PUBLIC_EMAIL

`update_on` function uses `nvchecker`, so it's better to check the [documents](https://nvchecker.readthedocs.io/en/latest/usage.html) of nvchecker. There are serveral ways to do this. If your packages update depends on AUR, considering the AUR maintainers work well or the upstream is not easy to check. `strip_release` is better to be `true`, considering it will include `pkgver-pkgrel` into the repo package's pkgver，if the pkgver contain `-`, it cannot be the pkgver, this option let lilac remove the `-pkgrel`.

    update_on:
      - source: aur
        aur: tnt-gui
        strip_release: true

If the upstream is the GitHub repo, can refer the following writing method. For example, `ugeneunipro/ugene` points to`https://github.com/ugeneunipro/ugene`. Some repo donnot have the releases, only contain tags, the `use_latest_release` can be replaced with `use_max_tag`.

    update_on:
      - source: github
        github: ugeneunipro/ugene
        use_latest_release: true

If the source is not stored at the url that [document](https://nvchecker.readthedocs.io/en/latest/usage.html) of nvchecker mentioned, you should consider to use regular expression.`url` is the page nvchecker can find sth, and `regex` is the text or element it matched.

    update_on:
      - regex: freedelta_(\d+.\d+.\d+)_amd64.deb
        source: regex
        url: https://sourceforge.net/projects/freedelta/files/freedelta/

    update_on:
      - source: regex
        url: http://doua.prabi.fr/software/seaview
        regex: href="seaview_data/seaview(\d+)-64\.tgz"

Some settings are rarely mentioned, such as `from_pattern`, `to_pattern`, `include_regex` and so on, can refer the documents.

`repo_depends` is the setting dealing with depends not in the offical repo,`lilac.yaml` file should be like the following, `gconf` can be replaced the depends your package depends on.

    repo_depends:
      - gconf

### lilac.py

The usual `lilac.py` file should be like this

    #!/usr/bin/env python3
    
    from lilaclib import *
    
    def pre_build():
      update_pkgver_and_pkgrel(_G.newver.lstrip('v'))
    
    def post_build():
      git_add_files('PKGBUILD')
      git_commit()

Others can refer the [document](https://lilac.readthedocs.io/en/latest/api.html) of `lilac api` and scripts of [archlinuxcn repo](https://github.com/archlinuxcn/repo) and [arch4edu repo](https://github.com/arch4edu/arch4edu)。

certbot config
----------

certbot can be used for signing SSL cert。

install certbot and needed nginx plugin, install the following two packages

    sudo pacman -S certbot
    sudo pacman -S certbot-nginx

use certbot apply the cert but donnot want it to change Nginx config, run the following command

    certbot -d repo.YOUR_DOMAIN -m YOUR_EMAIL --nginx certonly

The following key will be used for Nginx config

    Certificate is saved at: /etc/letsencrypt/live/repo.YOUR_DOMAIN/fullchain.pem
    Key is saved at:         /etc/letsencrypt/live/repo.YOUR_DOMAIN/privkey.pem

To continue signing the cert, use systemd script

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

start systemd releated service

    sudo systemctl enable letsencrypt.timer
    sudo systemctl start letsencrypt.timer


Nginx config
--------

Nginx let the repo path can be visited

Firstly install Nginx

    sudo pacman -S nginx

In ArchLinux, config files of Nginx is under the dir `/etc/nginx`

If you want to config serveral sites easily, edit `/etc/nginx/nginx.conf`

    sudo vim /etc/nginx/nginx.conf

add content udner the `http { }` section, the meaning of this content is conf files under `etc/nginx/conf.d` wil work.

    include       conf.d/*.conf;

and then create the `conf.d` dir

    cd /etc/nginx/
    sudo mkdir conf.d

add `repo.YOUR_DOMAIN` under the `/etc/nginx/conf.d`

    cd /etc/nginx/conf.d
    sudo vim repo.conf

add the following content, replace`repo.YOUR_DOMAIN` and `/YOUR/REPO/PATH` with your own domain and repo dir.

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

don not forget to restart nginx

    sudo systemctl enable nginx
    sudo systemctl restart nginx

Usage
--

Now you can use your repo, here I give the example of BioArchLinux, edit the `/etc/pacman.conf` at your local host

    sudo vim /etc/pacman.conf

add the content, and replace `BioArchLinux` with the `YOUR_REPO_NAME` of your `lilac` and server `Server` is the `https://repo.YOUR_DOMAIN/$arch`

    [BioArchLinux]
    Server = https://repo.malacology.net/$arch

 You can put it at a specific location, such as, I know tracer package from ArchLinux offical repo, but I won't use it and there is a tracer package from BioArchLinux, same name, but not the same thing, I will let BioArchLinux be upper than `community` and `multilib` in the `/etc/pacman.conf`.

We use the GPG sign, so import GPG key into your local host, here `B1F96021DB62254D` is the `KEY_ID`。

    sudo pacman-key --recv-keys B1F96021DB62254D
    sudo pacman-key --finger B1F96021DB62254D
    sudo pacman-key --lsign-key B1F96021DB62254D

And then update

    sudo pacman -Syu

Finally, you can enjoy it.

Acknowledgement
--

Thanks go to [依云](https://github.com/lilydjwg) for help on the usage and configuration of lilac, also thanks go to [Jingbei Li](https://github.com/petronny) for help on configuring of lilac. I am grateful to a TG friend for checking the code writing and [NotYu](https://github.com/NotYuhang) for help on regex。

PS： The results of the article is [BioArchLinux](https://github.com/BioArchLinux/Packages), a repository for bioinformatics software.Welcome to use, participate and star.

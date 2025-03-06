---
title: "ChromebookでLinux環境を構築した際の初期状態" # 記事のタイトル
emoji: "🏗" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["chromebook", "linux", "bash", "crostini"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

## はじめに

本記事は、**Chromebook で Linux 環境を構築した際の初期状態**について見ていくものです。調べた時点は 2025 年 2 月で、Chromebook のバージョンは、「`132.0.6834.206`（Official Build）（64 ビット）」となります。

### 構築時のディレクトリ構造

構築後のファイルやディレクトリの構造は、下記のようになっています。「ファイル」アプリで確認する場合、その他のオプション項目にある「非表示のファイルを表示」を有効にする必要があります。

```txt:ホームディレクトリ
.
├── .config
│   ├── cros-garcon.conf
│   └── weston.ini
├── .local
│   ├── share
│   │   └── cros-motd
│   └── state
│       └── wireplumber
│           └── restore-stream
├── .bash_history
├── .bash_logout
├── .bashrc
├── .profile
├── .sommelierrc
└── .Xauthority
```

本記事では 主に下記のファイルについて、内容を深堀りしていきます。

- `.profile`
- `.bashrc`
- `.bash_logout`

### オプションや環境変数等の状態

構築時における各種情報について、内容を格納した状態で記載しております。

:::details OS情報

```bash:OS情報
username@penguin:~$ uname -a
Linux penguin 6.6.56-05934-gf790fe104dba #1 SMP PREEMPT_DYNAMIC Wed, 25 Dec 2024 22:23:38 -0800 x86_64 GNU/Linux

username@penguin:~$ cat /etc/os-release 
PRETTY_NAME="Debian GNU/Linux 12 (bookworm)"
NAME="Debian GNU/Linux"
VERSION_ID="12"
VERSION="12 (bookworm)"
VERSION_CODENAME=bookworm
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
```

:::

:::details bash情報

```bash:bash情報
username@penguin:~$ echo $BASH
/bin/bash
username@penguin:~$ echo $BASH_VERSINFO
5
username@penguin:~$ echo $BASH_VERSION
5.2.15(1)-release
```

:::

:::details locale情報

```bash:locale情報
username@penguin:~$ locale
LANG=en_US.UTF-8
LANGUAGE=
LC_CTYPE="en_US.UTF-8"
LC_NUMERIC="en_US.UTF-8"
LC_TIME="en_US.UTF-8"
LC_COLLATE="en_US.UTF-8"
LC_MONETARY="en_US.UTF-8"
LC_MESSAGES="en_US.UTF-8"
LC_PAPER="en_US.UTF-8"
LC_NAME="en_US.UTF-8"
LC_ADDRESS="en_US.UTF-8"
LC_TELEPHONE="en_US.UTF-8"
LC_MEASUREMENT="en_US.UTF-8"
LC_IDENTIFICATION="en_US.UTF-8"
LC_ALL=
```

:::

:::details エイリアス設定

```bash:エイリアス設定
username@penguin:~$ alias
alias ls='ls --color=auto'
```

:::

:::details プロンプト文の設定

```bash:プロンプト文の設定
username@penguin:~$ echo $PS0

username@penguin:~$ echo $PS1
\[\e]0;\u@\h: \w\a\]${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$
username@penguin:~$ echo $PS2
>
username@penguin:~$ echo $PS3

username@penguin:~$ echo $PS4
+
```

:::

:::details オプション設定

```bash:オプション設定
username@penguin:~$ shopt -o
allexport       off
braceexpand     on
emacs           on
errexit         off
errtrace        off
functrace       off
hashall         on
histexpand      on
history         on
ignoreeof       off
interactive-comments    on
keyword         off
monitor         on
noclobber       off
noexec          off
noglob          off
nolog           off
notify          off
nounset         off
onecmd          off
physical        off
pipefail        off
posix           off
privileged      off
verbose         off
vi              off
xtrace          off

username@penguin:~$ shopt -p
shopt -u autocd
shopt -u assoc_expand_once
shopt -u cdable_vars
shopt -u cdspell
shopt -u checkhash
shopt -u checkjobs
shopt -s checkwinsize
shopt -s cmdhist
shopt -u compat31
shopt -u compat32
shopt -u compat40
shopt -u compat41
shopt -u compat42
shopt -u compat43
shopt -u compat44
shopt -s complete_fullquote
shopt -u direxpand
shopt -u dirspell
shopt -u dotglob
shopt -u execfail
shopt -s expand_aliases
shopt -u extdebug
shopt -s extglob
shopt -s extquote
shopt -u failglob
shopt -s force_fignore
shopt -s globasciiranges
shopt -s globskipdots
shopt -u globstar
shopt -u gnu_errfmt
shopt -s histappend
shopt -u histreedit
shopt -u histverify
shopt -u hostcomplete
shopt -u huponexit
shopt -u inherit_errexit
shopt -s interactive_comments
shopt -u lastpipe
shopt -u lithist
shopt -u localvar_inherit
shopt -u localvar_unset
shopt -s login_shell
shopt -u mailwarn
shopt -u no_empty_cmd_completion
shopt -u nocaseglob
shopt -u nocasematch
shopt -u noexpand_translation
shopt -u nullglob
shopt -s patsub_replacement
shopt -s progcomp
shopt -u progcomp_alias
shopt -s promptvars
shopt -u restricted_shell
shopt -u shift_verbose
shopt -s sourcepath
shopt -u varredir_close
shopt -u xpg_echo
```

:::

:::details 環境変数

```bash:環境変数
username@penguin:~$ export -p
declare -x BROWSER="/usr/bin/garcon-url-handler"
declare -x DBUS_SESSION_BUS_ADDRESS="unix:path=/run/user/1000/bus"
declare -x DISPLAY=":0"
declare -x DISPLAY_LOW_DENSITY=":1"
declare -x GTK_IM_MODULE="cros"
declare -x HOME="/home/username"
declare -x INVOCATION_ID="■■■"
declare -x JOURNAL_STREAM="7:3544"
declare -x LANG="en_US.UTF-8"
declare -x LD_ARGV0_REL="../bin/vshd"
declare -x LOGNAME="username"
declare -x LS_COLORS="rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=00:su=37;41:sg=30;43:ca=00:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.tzo=01;31:*.t7z=01;31:*.zip=01;31:*.z=01;31:*.dz=01;31:*.gz=01;31:*.lrz=01;31:*.lz=01;31:*.lzo=01;31:*.xz=01;31:*.zst=01;31:*.tzst=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.alz=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.7z=01;31:*.rz=01;31:*.cab=01;31:*.wim=01;31:*.swm=01;31:*.dwm=01;31:*.esd=01;31:*.avif=01;35:*.jpg=01;35:*.jpeg=01;35:*.mjpg=01;35:*.mjpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.webp=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=00;36:*.au=00;36:*.flac=00;36:*.m4a=00;36:*.mid=00;36:*.midi=00;36:*.mka=00;36:*.mp3=00;36:*.mpc=00;36:*.ogg=00;36:*.ra=00;36:*.wav=00;36:*.oga=00;36:*.opus=00;36:*.spx=00;36:*.xspf=00;36:*~=00;90:*#=00;90:*.bak=00;90:*.old=00;90:*.orig=00;90:*.part=00;90:*.rej=00;90:*.swp=00;90:*.tmp=00;90:*.dpkg-dist=00;90:*.dpkg-old=00;90:*.ucf-dist=00;90:*.ucf-new=00;90:*.ucf-old=00;90:*.rpmnew=00;90:*.rpmorig=00;90:*.rpmsave=00;90:"
declare -x MANAGERPID="142"
declare -x NCURSES_NO_UTF8_ACS="1"
declare -x OLDPWD
declare -x PATH="/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games"
declare -x PWD="/home/username"
declare -x QT_ACCESSIBILITY="1"
declare -x QT_AUTO_SCREEN_SCALE_FACTOR="1"
declare -x QT_QPA_PLATFORMTHEME="gtk2"
declare -x SHELL="/bin/bash"
declare -x SHLVL="1"
declare -x SOMMELIER_VERSION="0.20"
declare -x SOMMELIER_VM_IDENTIFIER="■■■"
declare -x SYSTEMD_EXEC_PID="268"
declare -x TERM="xterm-256color"
declare -x USER="username"
declare -x WAYLAND_DISPLAY="wayland-0"
declare -x WAYLAND_DISPLAY_LOW_DENSITY="wayland-1"
declare -x XCURSOR_SIZE="24"
declare -x XCURSOR_SIZE_LOW_DENSITY="12"
declare -x XCURSOR_THEME="Adwaita"
declare -x XDG_CONFIG_HOME="/home/username/.config"
declare -x XDG_CURRENT_DESKTOP="X-Generic"
declare -x XDG_DATA_DIRS="/home/username/.local/share:/home/username/.local/share/flatpak/exports/share:/var/lib/flatpak/exports/share:/usr/local/share:/usr/share"
declare -x XDG_RUNTIME_DIR="/run/user/1000"
declare -x XDG_SESSION_TYPE="wayland"
```

※ `■■■` と記載されている箇所は、一意の文字列を表します

:::

:::details フォントコンフィグ

下記は `/etc/fonts/fonts.conf` 内にあるデータです

```xml:fonts.conf
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "urn:fontconfig:fonts.dtd">
<!-- /etc/fonts/fonts.conf file to configure system font access -->
<fontconfig>
  <description>Default configuration file</description>

<!--
  DO NOT EDIT THIS FILE.
  IT WILL BE REPLACED WHEN FONTCONFIG IS UPDATED.
  LOCAL CHANGES BELONG IN 'local.conf'.

  The intent of this standard configuration file is to be adequate for
  most environments.  If you have a reasonably normal environment and
  have found problems with this configuration, they are probably
  things that others will also want fixed.  Please submit any problems
  to the fontconfig issue tracking system located at fontconfig.org

  Note that the normal 'make install' procedure for fontconfig is to
  replace any existing fonts.conf file with the new version.  Place
  any local customizations in local.conf which this file references.

  Keith Packard
-->

<!-- Font directory list -->

  <dir>/usr/share/fonts</dir>
  <dir>/usr/local/share/fonts</dir>
  <dir prefix="xdg">fonts</dir>
  <!-- the following element will be removed in the future -->
  <dir>~/.fonts</dir>

<!--
  Accept deprecated 'mono' alias, replacing it with 'monospace'
-->
  <match target="pattern">
    <test qual="any" name="family">
      <string>mono</string>
    </test>
    <edit name="family" mode="assign" binding="same">
      <string>monospace</string>
    </edit>
  </match>

<!--
  Accept alternate 'sans serif' spelling, replacing it with 'sans-serif'
-->
  <match target="pattern">
    <test qual="any" name="family">
      <string>sans serif</string>
    </test>
    <edit name="family" mode="assign" binding="same">
      <string>sans-serif</string>
    </edit>
  </match>

<!--
  Accept deprecated 'sans' alias, replacing it with 'sans-serif'
-->
  <match target="pattern">
    <test qual="any" name="family">
      <string>sans</string>
    </test>
    <edit name="family" mode="assign" binding="same">
      <string>sans-serif</string>
    </edit>
  </match>
<!--
  Accept alternate 'system ui' spelling, replacing it with 'system-ui'
-->
  <match target="pattern">
    <test qual="any" name="family">
      <string>system ui</string>
    </test>
    <edit name="family" mode="assign" binding="same">
      <string>system-ui</string>
    </edit>
  </match>

<!--
  Ignore dpkg temporary files created in fonts directories
-->
  <selectfont>
    <rejectfont>
      <glob>*.dpkg-tmp</glob>
    </rejectfont>
  </selectfont>
  <selectfont>
    <rejectfont>
      <glob>*.dpkg-new</glob>
    </rejectfont>
  </selectfont>

<!--
  Load local system customization file
-->
  <include ignore_missing="yes">conf.d</include>

<!-- Font cache directory list -->

  <cachedir>/var/cache/fontconfig</cachedir>
  <cachedir prefix="xdg">fontconfig</cachedir>
  <!-- the following element will be removed in the future -->
  <cachedir>~/.fontconfig</cachedir>

  <config>
<!--
Rescan configuration every 30 seconds when FcFontSetList is called
-->
    <rescan>
      <int>30</int>
    </rescan>
  </config>

</fontconfig>
```

:::

:::details aptインストール済み一覧

下記は `apt list --installed` を使い、インストール済みのものを一覧化したものです（量が膨大なためバージョン等の情報は削っています）

```txt
adduser, adwaita-icon-theme, alsa-topology-conf, alsa-ucm-conf, appstream, 
apt-utils, apt, at-spi2-common, at-spi2-core, base-files, base-passwd, 
bash-completion, bash, bsdextrautils, bsdutils, bzip2, ca-certificates, 
coreutils, cpio, cpp-12, cpp, cros-adapta, cros-apt-config, cros-garcon, 
cros-guest-tools, cros-host-fonts, cros-im, cros-logging, cros-motd, 
cros-notificationd, cros-pipe-config, cros-sommelier-config, cros-sommelier, 
cros-sudo-config, cros-systemd-overrides, cros-ui-config, cros-vmstat-metrics, 
cros-wayland, curl, dash, dbus-bin, dbus-daemon, dbus-session-bus-common, 
dbus-system-bus-common, dbus-user-session, dbus-x11, dbus, dconf-cli, 
dconf-gsettings-backend, dconf-service, debconf, debian-archive-keyring, 
debianutils, desktop-file-utils, diffutils, dirmngr, dmsetup, dpkg, e2fsprogs, 
file, findutils, fontconfig-config, fontconfig, fonts-dejavu-core, fuse3, 
gcc-12-base, git-man, git, gnupg-l10n, gnupg-utils, gnupg, gpg-agent, 
gpg-wks-client, gpg-wks-server, gpg, gpgconf, gpgsm, gpgv, grep, groff-base, 
gsettings-desktop-schemas, gtk-update-icon-cache, gtk2-engines-murrine, 
gtk2-engines-pixbuf, gzip, hicolor-icon-theme, hostname, ifupdown, 
init-system-helpers, init, iproute2, iptables, iputils-ping, isc-dhcp-client, 
isc-dhcp-common, iso-codes, krb5-locales, less, libacl1, libapparmor1, 
libappstream4, libapt-pkg6.0, libargon2-1, libasound2-data, libasound2, 
libassuan0, libasyncns0, libatk-bridge2.0-0, libatk1.0-0, libatm1, 
libatspi2.0-0, libattr1, libaudit-common, libaudit1, libauthen-sasl-perl, 
libavahi-client3, libavahi-common-data, libavahi-common3, libblkid1, libbpf1, 
libbrotli1, libbsd0, libbz2-1.0, libc-bin, libc-l10n, libc6, 
libcairo-gobject2, libcairo-script-interpreter2, libcairo2, libcap-ng0, 
libcap2-bin, libcap2, libcbor0.8, libclone-perl, libcloudproviders0, 
libcolord2, libcom-err2, libcrypt1, libcryptsetup12, libcups2, 
libcurl3-gnutls, libcurl4, libdata-dump-perl, libdatrie1, libdb5.3, 
libdbus-1-3, libdconf1, libdebconfclient0, libdeflate0, libdevmapper1.02.1, 
libdouble-conversion3, libdrm-amdgpu1, libdrm-common, libdrm-intel1, 
libdrm-nouveau2, libdrm-radeon1, libdrm2, libduktape207, libdw1, libedit2, 
libegl-mesa0, libegl1, libelf1, libencode-locale-perl, libepoxy0, 
liberror-perl, libevdev2, libexpat1, libext2fs2, libfdisk1, libffi8, 
libfido2-1, libfile-basedir-perl, libfile-desktopentry-perl, 
libfile-find-rule-perl, libfile-listing-perl, libfile-mimeinfo-perl, 
libflac12, libfont-afm-perl, libfontconfig1, libfontenc1, libfreetype6, 
libfribidi0, libfuse3-3, libgail-common, libgail18, libgbm1, libgcc-s1, 
libgcrypt20, libgdbm-compat4, libgdbm6, libgdk-pixbuf-2.0-0, 
libgdk-pixbuf2.0-bin, libgdk-pixbuf2.0-common, libgl1-mesa-dri, libgl1, 
libglapi-mesa, libglib2.0-0, libglib2.0-bin, libglib2.0-data, libglvnd0, 
libglx-mesa0, libglx0, libgmp10, libgnutls30, libgpg-error0, libgpm2, 
libgraphene-1.0-0, libgraphite2-3, libgssapi-krb5-2, libgstreamer1.0-0, 
libgtk-3-0, libgtk-3-bin, libgtk-3-common, libgtk-4-1, libgtk-4-bin, 
libgtk-4-common, libgtk2.0-0, libgtk2.0-bin, libgtk2.0-common, libgudev-1.0-0, 
libharfbuzz0b, libhogweed6, libhtml-form-perl, libhtml-format-perl, 
libhtml-parser-perl, libhtml-tagset-perl, libhtml-tree-perl, 
libhttp-cookies-perl, libhttp-daemon-perl, libhttp-date-perl, 
libhttp-message-perl, libhttp-negotiate-perl, libice6, libicu72, libidn2-0, 
libinput-bin, libinput10, libio-html-perl, libio-socket-ssl-perl, 
libio-stringy-perl, libip4tc2, libip6tc2, libipc-system-simple-perl, libisl23, 
libjansson4, libjbig0, libjpeg62-turbo, libjson-c5, libk5crypto3, 
libkeyutils1, libkmod2, libkrb5-3, libkrb5support0, libksba8, liblcms2-2, 
libldap-2.5-0, libldap-common, liblerc4, liblilv-0-0, libllvm15, liblua5.3-0, 
liblwp-mediatypes-perl, liblwp-protocol-https-perl, liblz4-1, liblzma5, 
liblzo2-2, libmagic-mgc, libmagic1, libmailtools-perl, libmd0, libmd4c0, 
libmnl0, libmount1, libmp3lame0, libmpc3, libmpfr6, libmpg123-0, libmtdev1, 
libncursesw6, libnet-dbus-perl, libnet-http-perl, libnet-smtp-ssl-perl, 
libnet-ssleay-perl, libnetfilter-conntrack3, libnettle8, libnewt0.52, 
libnfnetlink0, libnftables1, libnftnl11, libnghttp2-14, libnpth0, libnsl2, 
libnss-systemd, libnumber-compare-perl, libogg0, libopus0, libp11-kit0, 
libpackagekit-glib2-18, libpam-cap, libpam-modules-bin, libpam-modules, 
libpam-runtime, libpam-systemd, libpam0g, libpango-1.0-0, libpangocairo-1.0-0, 
libpangoft2-1.0-0, libpciaccess0, libpcre2-16-0, libpcre2-8-0, libperl5.36, 
libpipeline1, libpipewire-0.3-0, libpipewire-0.3-common, 
libpipewire-0.3-modules, libpixman-1-0, libpng16-16, libpolkit-agent-1-0, 
libpolkit-gobject-1-0, libpopt0, libproc2-0, libpsl5, libpulse0, 
libpython3-stdlib, libpython3.11-minimal, libpython3.11-stdlib, libqt5core5a, 
libqt5dbus5, libqt5gui5, libqt5network5, libqt5qml5, libqt5qmlmodels5, 
libqt5quick5, libqt5svg5, libqt5waylandclient5, libqt5waylandcompositor5, 
libqt5widgets5, libreadline8, libregexp-ipv6-perl, librsvg2-2, 
librsvg2-common, librtmp1, libsasl2-2, libsasl2-modules-db, libsasl2-modules, 
libseccomp2, libselinux1, libsemanage-common, libsemanage2, libsensors-config, 
libsensors5, libsepol2, libserd-0-0, libslang2, libsm6, libsmartcols1, 
libsndfile1, libsodium23, libsord-0-0, libspa-0.2-modules, libsqlite3-0, 
libsratom-0-0, libss2, libssh2-1, libssl3, libstdc++6, libstemmer0d, 
libsystemd-shared, libsystemd0, libtasn1-6, libtext-glob-perl, 
libtext-iconv-perl, libthai-data, libthai0, libtie-ixhash-perl, libtiff6, 
libtimedate-perl, libtinfo6, libtirpc-common, libtirpc3, libtry-tiny-perl, 
libuchardet0, libudev1, libunistring2, libunwind8, liburi-perl, libusb-1.0-0, 
libuuid1, libvorbis0a, libvorbisenc2, libwacom-common, libwacom9, 
libwayland-client0, libwayland-cursor0, libwayland-egl1, libwayland-server0, 
libwebp7, libwebrtc-audio-processing1, libwireplumber-0.4-0, libwrap0, 
libwww-perl, libwww-robotrules-perl, libx11-6, libx11-data, 
libx11-protocol-perl, libx11-xcb1, libxau6, libxaw7, libxcb-dri2-0, 
libxcb-dri3-0, libxcb-glx0, libxcb-icccm4, libxcb-image0, libxcb-keysyms1, 
libxcb-present0, libxcb-randr0, libxcb-render-util0, libxcb-render0, 
libxcb-shape0, libxcb-shm0, libxcb-sync1, libxcb-util1, libxcb-xfixes0, 
libxcb-xinerama0, libxcb-xinput0, libxcb-xkb1, libxcb1, libxcomposite1, 
libxcursor1, libxdamage1, libxdmcp6, libxext6, libxfixes3, libxft2, libxi6, 
libxinerama1, libxkbcommon-x11-0, libxkbcommon0, libxkbfile1, 
libxml-parser-perl, libxml-twig-perl, libxml-xpathengine-perl, libxml2, 
libxmlb2, libxmu6, libxmuu1, libxpm4, libxrandr2, libxrender1, libxshmfence1, 
libxss1, libxt6, libxtables12, libxtst6, libxv1, libxxf86dga1, libxxf86vm1, 
libxxhash0, libyaml-0-2, libz3-4, libzstd1, locales, login, logsave, man-db, 
mawk, media-types, mount, ncurses-base, ncurses-bin, ncurses-term, net-tools, 
netbase, nftables, openssh-client, openssh-server, openssh-sftp-server, 
openssl, packagekit-tools, packagekit, passwd, patch, perl-base, 
perl-modules-5.36, perl-openssl-defaults, perl, pinentry-curses, 
pipewire-alsa, pipewire-bin, pipewire-pulse, pipewire, polkitd, procps, 
psmisc, publicsuffix, python3-minimal, python3.11-minimal, python3.11, 
python3, qt5-gtk-platformtheme, qt5-gtk2-platformtheme, 
qt5-style-plugin-cleanlooks, qt5-style-plugin-motif, 
qt5-style-plugin-plastique, qt5-style-plugins, qttranslations5-l10n, 
qtwayland5, readline-common, rtkit, runit-helper, sed, sensible-utils, 
sgml-base, shared-mime-info, sudo, systemd-sysv, systemd-timesyncd, systemd, 
sysvinit-utils, tar, tzdata, ucf, udev, unzip, usbutils, usrmerge, 
util-linux-extra, util-linux, vim-common, vim-runtime, vim, wget, whiptail, 
wireplumber, x11-common, x11-utils, x11-xserver-utils, xauth, xdg-user-dirs, 
xdg-utils, xfonts-base, xfonts-encodings, xfonts-utils, xkb-data, xml-core, 
xxd, xz-utils, zlib1g
```

:::

## `.profile`

:::details 全体

```bash:.profile
# ~/.profile: executed by the command interpreter for login shells.
# This file is not read by bash(1), if ~/.bash_profile or ~/.bash_login
# exists.
# see /usr/share/doc/bash/examples/startup-files for examples.
# the files are located in the bash-doc package.

# the default umask is set in /etc/profile; for setting the umask
# for ssh logins, install and configure the libpam-umask package.
#umask 022

# if running bash
if [ -n "$BASH_VERSION" ]; then
    # include .bashrc if it exists
    if [ -f "$HOME/.bashrc" ]; then
        . "$HOME/.bashrc"
    fi
fi

# set PATH so it includes user's private bin if it exists
if [ -d "$HOME/bin" ] ; then
    PATH="$HOME/bin:$PATH"
fi

# set PATH so it includes user's private bin if it exists
if [ -d "$HOME/.local/bin" ] ; then
    PATH="$HOME/.local/bin:$PATH"
fi
```

:::

これは**ログイン時**に呼び出されるファイルです。

ログイン時に実行されるファイルは他にも `.bash_profile` や `.bash_login` があります。これらファイルの優先順は `.bash_profile` > `.bash_login` > `.profile` となっており、基本的にはログイン時に どれか 1 つのみ実行されます。

構築した初期状態では上位 2 つのファイルが存在しないため、必然的に この `.profile` が呼び出されることになります。

### `bash` 実行時に `.bashrc` を呼び出し

```bash
# if running bash
if [ -n "$BASH_VERSION" ]; then
    # include .bashrc if it exists
    if [ -f "$HOME/.bashrc" ]; then
        . "$HOME/.bashrc"
    fi
fi
```

`if[ -n $BASH_VERSION]` で、状態変数 `BASH_VERSION` （実行中シェルのバージョン番号）を呼び出し、それが空でなければ `true` を返す挙動をします。これで、`bash` の実行を確認しているようです。

状態変数 `$HOME` がホーム（ログイン）ディレクトリの名前を表しています。ネストされた `if` 文の中では、ホームディレクトリ上の `.bashrc` を確認し、ドットコマンドで呼び出す処理を行ってます。

:::message

ファイル名を指定して直接実行するとサブシェルが起動しますが、今回はドットコマンドによる実行なのでカレントシェル内で読み込むことになります。

そのため `.bashrc` 最初にある「対話形式以外で呼び出された場合には何もしない」箇所に抵触せずに、処理が続行されることになります（≒ `$-` に `i` が含まれており、弾かれない）

:::

### 環境変数 `PATH` の追記

```bash
# set PATH so it includes user's private bin if it exists
if [ -d "$HOME/bin" ] ; then
    PATH="$HOME/bin:$PATH"
fi

# set PATH so it includes user's private bin if it exists
if [ -d "$HOME/.local/bin" ] ; then
    PATH="$HOME/.local/bin:$PATH"
fi
```

ユーザーがホームディレクトリ内で `bin`（または `.local/bin`） ディレクトリを作っている場合、環境変数 `PATH` に追記する処理を行っています。

最初の方の `PATH="$HOME/bin:$PATH"` を例に取って説明すると、`$HOME/bin` が追加するディレクトリを表しています。それと `:` で繋げる形で、元々のパス変数に格納されていた内容 `$PATH` を後ろに付けています。

:::details 個人メモ

自身の `bin` ディレクトリを 他のディレクトリより優先する形で環境変数 `PATH` を設定するのは、利便性としては有用な一方で、セキュリティには配慮する必要があるのではないだろうか？

この文では `PATH="$HOME/bin:$PATH"` となっており、自身のディレクトリが頭にきている。実行可能ファイルの検出は、`PATH` 変数に**設定された順**に検索されるので、仮に `more` が両者に含まれる場合は、自身が作成したコマンドのほうが実行されることになる。

自分用にカスタマイズされたコマンドを、デフォルトと同じように利用できるという利便性はある一方で、悪意ある攻撃者にとっては、これはセキュリティホール足りえる状態なのかもしれない。

`~/bin/more` と入力する手間はあるが、`PATH="$PATH:$HOME/bin"` と読み込み順を後ろに持ってくる方法が安心ではある。

:::

## `.bashrc`

:::details 全文

```bash:.bashrc
# ~/.bashrc: executed by bash(1) for non-login shells.
# see /usr/share/doc/bash/examples/startup-files (in the package bash-doc)
# for examples

# If not running interactively, don't do anything
case $- in
    *i*) ;;
      *) return;;
esac

# don't put duplicate lines or lines starting with space in the history.
# See bash(1) for more options
HISTCONTROL=ignoreboth

# append to the history file, don't overwrite it
shopt -s histappend

# for setting history length see HISTSIZE and HISTFILESIZE in bash(1)
HISTSIZE=1000
HISTFILESIZE=2000

# check the window size after each command and, if necessary,
# update the values of LINES and COLUMNS.
shopt -s checkwinsize

# If set, the pattern "**" used in a pathname expansion context will
# match all files and zero or more directories and subdirectories.
#shopt -s globstar

# make less more friendly for non-text input files, see lesspipe(1)
#[ -x /usr/bin/lesspipe ] && eval "$(SHELL=/bin/sh lesspipe)"

# set variable identifying the chroot you work in (used in the prompt below)
if [ -z "${debian_chroot:-}" ] && [ -r /etc/debian_chroot ]; then
    debian_chroot=$(cat /etc/debian_chroot)
fi

# set a fancy prompt (non-color, unless we know we "want" color)
case "$TERM" in
    xterm-color|*-256color) color_prompt=yes;;
esac

# uncomment for a colored prompt, if the terminal has the capability; turned
# off by default to not distract the user: the focus in a terminal window
# should be on the output of commands, not on the prompt
#force_color_prompt=yes

if [ -n "$force_color_prompt" ]; then
    if [ -x /usr/bin/tput ] && tput setaf 1 >&/dev/null; then
        # We have color support; assume it's compliant with Ecma-48
        # (ISO/IEC-6429). (Lack of such support is extremely rare, and such
        # a case would tend to support setf rather than setaf.)
        color_prompt=yes
    else
        color_prompt=
    fi
fi

if [ "$color_prompt" = yes ]; then
    PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
else
    PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '
fi
unset color_prompt force_color_prompt

# If this is an xterm set the title to user@host:dir
case "$TERM" in
xterm*|rxvt*)
    PS1="\[\e]0;${debian_chroot:+($debian_chroot)}\u@\h: \w\a\]$PS1"
    ;;
*)
    ;;
esac

# enable color support of ls and also add handy aliases
if [ -x /usr/bin/dircolors ]; then
    test -r ~/.dircolors && eval "$(dircolors -b ~/.dircolors)" || eval "$(dircolors -b)"
    alias ls='ls --color=auto'
    #alias dir='dir --color=auto'
    #alias vdir='vdir --color=auto'

    #alias grep='grep --color=auto'
    #alias fgrep='fgrep --color=auto'
    #alias egrep='egrep --color=auto'
fi

# colored GCC warnings and errors
#export GCC_COLORS='error=01;31:warning=01;35:note=01;36:caret=01;32:locus=01:quote=01'

# some more ls aliases
#alias ll='ls -l'
#alias la='ls -A'
#alias l='ls -CF'

# Alias definitions.
# You may want to put all your additions into a separate file like
# ~/.bash_aliases, instead of adding them here directly.
# See /usr/share/doc/bash-doc/examples in the bash-doc package.

if [ -f ~/.bash_aliases ]; then
    . ~/.bash_aliases
fi

# enable programmable completion features (you don't need to enable
# this, if it's already enabled in /etc/bash.bashrc and /etc/profile
# sources /etc/bash.bashrc).
if ! shopt -oq posix; then
    if [ -f /usr/share/bash-completion/bash_completion ]; then
        . /usr/share/bash-completion/bash_completion
    elif [ -f /etc/bash_completion ]; then
        . /etc/bash_completion
    fi
fi
```

:::

これは**シェル起動時**に呼び出されるファイルです。

ログイン時に *一度だけ* 実行される `.profile` 系と異なり、シェルを起動（サブシェル含む）する度に実行されます。

### 対話形式で呼び出されてない場合には何もしない

```bash
# If not running interactively, don't do anything
case $- in
    *i*) ;;
      *) return;;
esac
```

`$-` という特殊な変数の中には、現在のオプション文字列（起動時、あるいは `set` コマンド、あるいは シェル自身で指定されたもの）が格納されています。例えば `echo $-` で呼び出してみると、`himBHs` といった文字列が返ってきます。

> (`$-`, a hyphen.) Expands to the current option flags as specified upon invocation, by the set builtin command, or those set by the shell itself (such as the `-i` option).
>
> 『[3.4.2 Special Parameters | GNU Bash manual](https://www.gnu.org/software/bash/manual/html_node/Special-Parameters.html#Special-Parameters)』より抜粋

ここでは それ（現在のオプション項目）を用いて、`-i`（インタラクティブ）がなければ何も実行しないようにしています。`case` 文でオプション文字列を使い、`i` が含まれていれば `case` を抜け出し、そうでなければ `return` で以降の処理を行わないようになっています。

そのため例えば `.profile` 系のファイルに `source ~/.bashrc` と記載した場合は内容を読み込む一方、バックグラウンド処理などの**対話形式でない場合**は、`return` で抜けてしまい 以降の処理が行われることはありません。

#### 補足：呼び出し方による違い（`set` オプション）

`echo $-` で返される文字列 `himBHs` の意味については、下表のようになります。

|フラグ|フルネーム|説明|
|:---:|:---|:---|
|`h`|`hashall`|コマンドハッシュが有効|
|`i`|（`interactive`）|カレントシェルが対話形式（インタラクティブ）である|
|`m`|`monitor`|ジョブ制御が有効|
|`B`|`braceexpand`|ブレース展開が有効（デフォルト）|
|`H`|`histexpand`|`!` 形式の履歴置換が有効（対話型の場合デフォルトで有効）|
|`s`|（`standardinput`）|標準入力からコマンドを読み込む|

※ `i` と `s` は `set` オプションでなく、コマンドラインオプションです

`set` オプションについての詳細は、下記ページもしくは `help` コマンドによる説明を参照ください。

https://www.gnu.org/software/bash/manual/html_node/The-Set-Builtin.html

:::details helpコマンドによる説明（長いので格納しています）

```bash:setオプションについて
username@penguin:~$ help set
set: set [-abefhkmnptuvxBCEHPT] [-o option-name] [--] [-] [arg ...]
    Set or unset values of shell options and positional parameters.
    
    Change the value of shell attributes and positional parameters, or
    display the names and values of shell variables.
    
    Options:
      -a  Mark variables which are modified or created for export.
      -b  Notify of job termination immediately.
      -e  Exit immediately if a command exits with a non-zero status.
      -f  Disable file name generation (globbing).
      -h  Remember the location of commands as they are looked up.
      -k  All assignment arguments are placed in the environment for a
          command, not just those that precede the command name.
      -m  Job control is enabled.
      -n  Read commands but do not execute them.
      -o option-name
          Set the variable corresponding to option-name:
              allexport    same as -a
              braceexpand  same as -B
              emacs        use an emacs-style line editing interface
              errexit      same as -e
              errtrace     same as -E
              functrace    same as -T
              hashall      same as -h
              histexpand   same as -H
              history      enable command history
              ignoreeof    the shell will not exit upon reading EOF
              interactive-comments
                           allow comments to appear in interactive commands
              keyword      same as -k
              monitor      same as -m
              noclobber    same as -C
              noexec       same as -n
              noglob       same as -f
              nolog        currently accepted but ignored
              notify       same as -b
              nounset      same as -u
              onecmd       same as -t
              physical     same as -P
              pipefail     the return value of a pipeline is the status of
                           the last command to exit with a non-zero status,
                           or zero if no command exited with a non-zero status
              posix        change the behavior of bash where the default
                           operation differs from the Posix standard to
                           match the standard
              privileged   same as -p
              verbose      same as -v
              vi           use a vi-style line editing interface
              xtrace       same as -x
      -p  Turned on whenever the real and effective user ids do not match.
          Disables processing of the $ENV file and importing of shell
          functions.  Turning this option off causes the effective uid and
          gid to be set to the real uid and gid.
      -t  Exit after reading and executing one command.
      -u  Treat unset variables as an error when substituting.
      -v  Print shell input lines as they are read.
      -x  Print commands and their arguments as they are executed.
      -B  the shell will perform brace expansion
      -C  If set, disallow existing regular files to be overwritten
          by redirection of output.
      -E  If set, the ERR trap is inherited by shell functions.
      -H  Enable ! style history substitution.  This flag is on
          by default when the shell is interactive.
      -P  If set, do not resolve symbolic links when executing commands
          such as cd which change the current directory.
      -T  If set, the DEBUG and RETURN traps are inherited by shell functions.
      --  Assign any remaining arguments to the positional parameters.
          If there are no remaining arguments, the positional parameters
          are unset.
      -   Assign any remaining arguments to the positional parameters.
          The -x and -v options are turned off.
    
    Using + rather than - causes these flags to be turned off.  The
    flags can also be used upon invocation of the shell.  The current
    set of flags may be found in $-.  The remaining n ARGs are positional
    parameters and are assigned, in order, to $1, $2, .. $n.  If no
    ARGs are given, all shell variables are printed.
    
    Exit Status:
    Returns success unless an invalid option is given.
```

:::

ここでは呼び出し方による違いを見るために、オプション文字列を返す下記ファイルを作成してみます。

```bash:echo-options.sh
#!/bin/bash
echo $-
```

これを様々な方法で実行してみたのが下記となります。カレントシェル内で実行できているものは `himBHs` となっていますが、サブシェルで実行しているものは `hB` となっています。

```bash:オプションの検証
username@penguin:~$ . ./echo-options.sh 
himBHs
username@penguin:~$ source ./echo-options.sh 
himBHs
username@penguin:~$ ./echo-options.sh 
hB
username@penguin:~$ bash ./echo-options.sh 
hB
```

余談になりますが バックグラウンドでの実行（コマンド末尾に `&` を挿入することで行う）も、サブシェルの別表現なので結果は `hB` となります。

```bash:バックグラウンドでの実行
username@penguin:~$ ./echo-options.sh &
[1] 624
username@penguin:~$ hB
```

### コマンド履歴に関する設定

```bash
# don't put duplicate lines or lines starting with space in the history.
# See bash(1) for more options
HISTCONTROL=ignoreboth

# append to the history file, don't overwrite it
shopt -s histappend

# for setting history length see HISTSIZE and HISTFILESIZE in bash(1)
HISTSIZE=1000
HISTFILESIZE=2000
```

ここではコマンドライン編集に関するシェル変数に変更を加えています。

|シェル変数名|内容|設定内容|
|:---|:---|:---|
|`HISTCONTROL`|コロン区切りのパターンリスト|`ignoreboth`（※）|
|`HISTSIZE`|コマンド履歴（メモリ）に記録する *コマンド* 最大数|`1000`|
|`HISTFILESIZE`|履歴リスト（ファイル）に保存する *コマンドライン* 最大数|`2000`|

※ `ignoreboth` : スペース開始のコマンドラインを履歴リストに追加しない ＋ 履歴リスト最終行と一致するコマンドラインを履歴リストに追加しない

:::message

`HISTSIZE` はシェル起動中に**メモリ上に保存されるコマンド数**を指します。

`HISTFILESIZE` は、シェルセッション終了時に保存される**コマンド履歴ファイルのコマンド数**を指しています。メモリ上に保存されているコマンド履歴は、シェル終了時に その内容を履歴保存用のファイルに移すことになります。

:::

また `shopt -s histappend` を使い、コマンド履歴をファイルに保存する際、上書きでなく追記の形になるよう切り替えています。

### 画面サイズの設定

```bash
# check the window size after each command and, if necessary,
# update the values of LINES and COLUMNS.
shopt -s checkwinsize
```

`shopt -s checkwinsize` で、**ターミナルのウィンドウサイズに合わせる形**で環境変数 `COLUMNS`, `LINES` （ディスプレイの行列数）を設定しています。

例えば ウィンドウサイズが大きい場合には `LINES` は 40、`COLUMNS` は 200（≒ 1行あたり 200文字を 40行分表示）となる一方、ウィンドウサイズを小さくした場合は `LINES` は 20、`COLUMNS` は 100（≒ 1行あたり 100文字を 20行分表示）といったようになります。

### グロブスターの設定

```bash
# If set, the pattern "**" used in a pathname expansion context will
# match all files and zero or more directories and subdirectories.
#shopt -s globstar
```

ここは `**` のパターン（≒ グロブスター）を利用するための設定を表しており、デフォルトではコメントアウト化されています。

*globstar*（グロブスター）は、カレントディレクトリ下のファイルを（サブディレクトリ含めた）再帰的に検索し、全てのファイルにマッチするために使用されます。

例えば `**/*.md` といった指定をした場合、**カレントディレクトリ及び それより下位にあるサブディレクトリ全てが検索対象**となり、拡張子 `md` のファイルをマッチさせることができます。

### `lesspipe` 使用の設定

```bash
# make less more friendly for non-text input files, see lesspipe(1)
#[ -x /usr/bin/lesspipe ] && eval "$(SHELL=/bin/sh lesspipe)"
```

`less` コマンドのプリプロセッサとなる `lesspipe` を使用する場合の設定で、デフォルトではコメントアウト化されています。

`lesspipe` は、`less` にシンタックスハイライトといった便利な機能を導入する際に使用されるものです。

### プロンプト設定

```bash
# set variable identifying the chroot you work in (used in the prompt below)
if [ -z "${debian_chroot:-}" ] && [ -r /etc/debian_chroot ]; then
    debian_chroot=$(cat /etc/debian_chroot)
fi

# set a fancy prompt (non-color, unless we know we "want" color)
case "$TERM" in
    xterm-color|*-256color) color_prompt=yes;;
esac

# uncomment for a colored prompt, if the terminal has the capability; turned
# off by default to not distract the user: the focus in a terminal window
# should be on the output of commands, not on the prompt
#force_color_prompt=yes

if [ -n "$force_color_prompt" ]; then
    if [ -x /usr/bin/tput ] && tput setaf 1 >&/dev/null; then
        # We have color support; assume it's compliant with Ecma-48
        # (ISO/IEC-6429). (Lack of such support is extremely rare, and such
        # a case would tend to support setf rather than setaf.)
        color_prompt=yes
    else
        color_prompt=
    fi
fi

if [ "$color_prompt" = yes ]; then
    PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
else
    PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '
fi
unset color_prompt force_color_prompt

# If this is an xterm set the title to user@host:dir
case "$TERM" in
xterm*|rxvt*)
    PS1="\[\e]0;${debian_chroot:+($debian_chroot)}\u@\h: \w\a\]$PS1"
    ;;
*)
    ;;
esac
```

ここでは カラーサポートがされてるかを調べ、適切なプロンプト文を設定する処理を行っています。

最終的にプロンプト変数 `PS1` に収まっているのは、`\[\e]0;\u@\h: \w\a\]${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$` となります。

#### `chroot` 使用環境の表示

`chroot` を使用したりしなければ、この部分は出力されません。

変数 `debian_chroot` は、`chroot` コマンドでルートディレクトリを変更した際に、どういった環境で利用しているかを示すのに使用します。

デフォルトでは `/etc/debian_chroot` は存在しないファイルのため、この状況に当てはまる時にファイルを設定しておけば、プロンプト文の最初の方に表示されるようになっています。

#### 環境変数 `TERM` によるカラーモード判定

環境変数 `TERM` には、`xterm-256color` といった文字列が格納されています。

### カラーサポートの設定

```bash
# enable color support of ls and also add handy aliases
if [ -x /usr/bin/dircolors ]; then
    test -r ~/.dircolors && eval "$(dircolors -b ~/.dircolors)" || eval "$(dircolors -b)"
    alias ls='ls --color=auto'
    #alias dir='dir --color=auto'
    #alias vdir='vdir --color=auto'

    #alias grep='grep --color=auto'
    #alias fgrep='fgrep --color=auto'
    #alias egrep='egrep --color=auto'
fi

# colored GCC warnings and errors
#export GCC_COLORS='error=01;31:warning=01;35:note=01;36:caret=01;32:locus=01:quote=01'
```

ここではカラーサポートが有効かを調べ、有効の場合は `ls` などのコマンドを使用する際に配色するようにしています。

#### エイリアスによる既存コマンドの上書き

`/usr/bin/dircolors` を調べ、カラーサポートが有効のときは既存コマンドを `alias ls='ls --color=auto'` といった形で、エイリアスを使って上書きしています。

デフォルト時は `ls` のみ設定しているようで、残りはコメントアウト化されています。

#### GCCの配色

GCC(GNU Compiler Collection) の警告やエラーなどに対し、配色する処理になります。デフォルトの状態ではコメントアウト化されています。

### エイリアスの適用

```bash
# some more ls aliases
#alias ll='ls -l'
#alias la='ls -A'
#alias l='ls -CF'

# Alias definitions.
# You may want to put all your additions into a separate file like
# ~/.bash_aliases, instead of adding them here directly.
# See /usr/share/doc/bash-doc/examples in the bash-doc package.

if [ -f ~/.bash_aliases ]; then
    . ~/.bash_aliases
fi
```

ここではエイリアスの例として、`ls` 関係のものがコメントアウトで記述されています。

- `ll`(≒ `ls -l`)
- `la`(≒ `ls -A`)
- `l`(≒ `ls -CF`)

また、ホームディレクトリ上に `.bash_aliases` があれば、ドットコマンドを使ってエイリアスを適用させています。これは自身でカスタマイズしたファイルを配置していることを想定しているようです。

### 補完機能の有効化

```bash
# enable programmable completion features (you don't need to enable
# this, if it's already enabled in /etc/bash.bashrc and /etc/profile
# sources /etc/bash.bashrc).
if ! shopt -oq posix; then
  if [ -f /usr/share/bash-completion/bash_completion ]; then
    . /usr/share/bash-completion/bash_completion
  elif [ -f /etc/bash_completion ]; then
    . /etc/bash_completion
  fi
fi
```

ここでは、プログラム化された補完機能を有効化するための設定を行っています。

`/etc/bash.bashrc` を有効化させていたり、`/etc/profile` で　`sources /etc/bash.bashrc` といった記載をしている場合には不要となります。

デフォルト状態では、`/etc/bash_completion` は `. /usr/share/bash-completion/bash_completion` と記載されており、そちらに様々な機能が集約されているようです。

## `.bash_logout`

```bash
# ~/.bash_logout: executed by bash(1) when login shell exits.

# when leaving the console clear the screen to increase privacy

if [ "$SHLVL" = 1 ]; then
    [ -x /usr/bin/clear_console ] && /usr/bin/clear_console -q
fi
```

これは**ログアウト時**に呼び出されるファイルです。内容としては、シェル終了時にクリアコンソールを行っているだけとなります。

### クリアコンソール

> clear_console clears your console if this is possible. It looks in the environment for the terminal type and then in the terminfo database to figure out how to clear the screen. To clear the buffer, it then changes the foreground virtual terminal to another terminal and then back to the original terminal.
>
> 『[clear_console | Linux commands' man pages](https://www.commandlinux.com/man-page/man1/clear_console.1.html)』より抜粋

自身のシェルの深度を 変数 `SHLVL` で調べ、その深度が 1（≒ サブシェルとかでない）場合に、`clear_console` を使ってターミナルのディスプレイやバッファといった情報を消去しています。

## まとめ

### 参考文献

2005 年の書籍ですが、本記事は下記を参考にしました。

https://www.oreilly.co.jp/books/9784873112541/

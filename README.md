# shadowsocks binaries asus rt-n66u on merlin

Merlin has entware and it has shadowsocks avaiable as package. However, to enable entware, you must attach an usb drive to the router. I want to avoid that and use the /jffs directly, that's why I need to build the shadowsocks by myself.

The built binaries can be found in `static-bin` directory. In theory they should work on all mipsel routers.

Shadowsocks-libev version: `ce0561a9992707a8409b9ca5a03f7aa7476608a1`

## How to build static shadowsocks for rt-n66u

Firstly, download tomatoware from https://github.com/lancethepants/tomatoware. I picked `mipsel-soft-mmc.tgz`

Extract it to `/mmc`.

Add toolchain to your PATH:
```bash
$ export PATH=/mmc/bin:$PATH
```

I choose to use PolarSSL instead of openssl. Download polarssl (1.2.19), extract and then build. This will take a lot of time.

```bash
$ make
$ make install DESTDIR="/home/txchen/sscompile/install/polar-static"
```

Then build shadowsocks:

```bash
$ git clone https://github.com/shadowsocks/shadowsocks-libev.git
$ cd shadowsocks-libev
$ # change libev/ev_epoll.c, replace epoll_create1 with epoll_create
$ LIBS="-ldl" LDFLAGS="-static" ./configure \
  --with-crypto-library=polarssl \
  --with-polarssl=/home/txchen/sscompile/install/polar-static
$ # change src/Makefile, change LDFLAGS = -static to LDFLAGS = -all-static
$ make
```

The static binaries should then be available in `src` directory, you can use `file ss-redir` to check if it is statically linked. You can further reduce their sizes:

```bash
$ strip src/ss-*
```

Now copy them to the router and they should work.



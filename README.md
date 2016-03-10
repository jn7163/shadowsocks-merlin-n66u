# shadowsocks binaries asus rt-n66u on merlin

Merlin has entware and it has shadowsocks avaiable as package. However, to enable entware, you must attach an usb drive to the router. I want to avoid that and use the /jffs directly, that's why I need to build the shadowsocks by myself.

## How to build static shadowsocks for rt-n66u

Firstly, you need to have a mipsel toolchain. I used this one: https://github.com/wl500g/toolchain

Download the tarball from its releases and extract to your local linux file system. For example: `/home/txchen/sscompile/hndtools-mipsel-uclibc-4.6.4`

Add toolchain to your PATH:
```bash
$ export PATH=/home/txchen/sscompile/hndtools-mipsel-uclibc-4.6.4/bin:$PATH
```

I choose to use PolarSSL instead of openssl. Download polarssl (1.2.19), extract and then build.

```bash
$ make CC=mipsel-linux-uclibc-gcc
$ make install DESTDIR="/home/txchen/sscompile/install/polar-static"
```

Then build shadowsocks:

```bash
$ git clone https://github.com/shadowsocks/shadowsocks-libev.git
$ cd shadowsocks-libev
$ LIBS="-ldl" LDFLAGS="-static" CC=mipsel-linux-uclibc-gcc \
  CXX=mipsel-linux-uclibc-g++ AR=mipsel-linux-uclibc-ar \
  RANLIB=mipsel-linux-uclibc-ranlib ./configure \
  --with-crypto-library=polarssl \
  --with-polarssl=/home/txchen/sscompile/install/polar-static \
  --host=mipsel-uclibc-linux --disable-ssp
$ make
```

The static binaries should then be available in `src` directory. You can further reduce their sizes:

```bash
$ mipsel-linux-uclibc-strip ss-redir
```

Now copy them to the router and they should work.



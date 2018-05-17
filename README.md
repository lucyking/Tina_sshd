# 1. 编译busybox telent
- __OpenWrt toolchain__:
  - [openwrt-sdk-sunxi-cortexa7_gcc-7.3.0_musl_eabi.Linux-x86_64.tar.xz](https://downloads.lede-project.org/snapshots/targets/sunxi/cortexa7/openwrt-sdk-sunxi-cortexa7_gcc-7.3.0_musl_eabi.Linux-x86_64.tar.xz)
- __busybox__:
  - [busybox-1.28.3.tar.bz2](https://busybox.net/downloads/busybox-1.28.3.tar.bz2)
  
- __set-up tftp server__:
  - [douban nodes](https://www.douban.com/note/668318511/)




XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
# 2. 编译ssh


## OPENSSL lib build
```


( :; LIBDEPS="${LIBDEPS:--L.. -lssl  -L.. -lcrypto }"; LDCMD="${LDCMD:-arm-openwrt-linux-gcc}"; LDFLAGS="${LDFLAGS:--DOPENSSL_THREADS  }"; LIBPATH=`for x in $LIBDEPS; do if echo $x | grep '^ *-L' > /dev/null 2>&1; then echo $x | sed -e 's/^ *-L//'; fi; done | uniq`; LIBPATH=`echo $LIBPATH | sed -e 's/ /:/g'`; LD_LIBRARY_PATH=$LIBPATH:$LD_LIBRARY_PATH ${LDCMD} ${LDFLAGS} -o ${APPNAME:=openssl} openssl.o verify.o asn1pars.o req.o dgst.o dh.o dhparam.o enc.o passwd.o gendh.o errstr.o ca.o pkcs7.o crl2p7.o crl.o rsa.o rsautl.o dsa.o dsaparam.o ec.o ecparam.o x509.o genrsa.o gendsa.o s_server.o s_client.o speed.o s_time.o apps.o s_cb.o s_socket.o app_rand.o version.o sess_id.o ciphers.o nseq.o pkcs12.o pkcs8.o spkac.o smime.o rand.o engine.o ocsp.o prime.o cms.o ${LIBDEPS} ) # LDCMD是没错的
```


- openssl 版本:  
[x] ~~master    bc624bd955 v3_purp.c: add locking to x509v3_cache_extensions() 宏定义不匹配😂~~   
[v] OpenSSL_1_0_1-stable    4675a56a3c apps/speed.c: Fix crash when config loading fails

- 编译openssl的可用命令 添加no-async以消除ld时的 ./libcrypto.so: undefined reference tosetcontext'错误
```
./Configure linux-generic32  no-asm no-shared no-async --prefix=/home/work/ssh/install/openssl   --cross-compile-prefix=arm-openwrt-linux-
```


- 头文件需要拷贝到chaintools目录下
```
root@bt:/work/ssh/source/openssl# locate openssl/opensslconf.h
/root/Desktop/Tina/openwrt-sdk-sunxi-cortexa7_gcc-7.3.0_musl_eabi.Linux-x86_64/staging_dir/host/include/openssl/opensslconf.h   #这个才是生效的目录
/root/Desktop/WorkBench/openssl/include/openssl/opensslconf.h.in
/usr/include/openssl/opensslconf.h
/usr/include/openssl/opensslconf.h.in
/usr/local/include/openssl/opensslconf.h
/usr/local/include/openssl/opensslconf.h.in
/work/ssh/source/openssl/include/openssl/opensslconf.h         #源目录
/work/ssh/source/openssl/include/openssl/opensslconf.h.in
```

- install dir:  /home/work/ssh/install/openssl/


- [!]git co br后， 需要手动make clean

=======================================



## zlib

- wget http://www.zlib.net/zlib1211.zip

- 先执行./configure 生成Makefile 然后手动修改Makefile中的gcc为arm-openwrt-linux-gcc
执行configure生成Makefile
CC=arm-openwrt-linux-gcc  AR=arm-openwrt-linux-ar RANLIB=arm-openwrt-linux-ranlib ./configure  --prefix=/work/ssh/install


- 然后make; make install

- install dir:
    - /work/ssh/install



## openssh
- ver: 
  - V_7_7                   45a573a0 Use includes.h instead of config.h.
```
./configure --host=arm-linux --with-libs --with-zlib=/work/ssh/install --with-ssl-dir=/home/work/ssh/install/openssl_1_0_1_stable/ --disable-etc-default-login CC=arm-openwrt-linux-gcc AR=arm-openwrt-linux-ar --prefix=/home/work/ssh/install/openssh
```


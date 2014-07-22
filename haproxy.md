#HAProxy Centos 6 설치

##HAProxy
haproxy는 오픈 소스 로드 벨런서로서 손쉽게 서비스 이중화가 가능하도록 합니다.  

로드 밸런서
 :	ㅇㄹㅇㄹ

##설치 환경
CentOS release 6.4 (Final)  
haproxy-1.5.2

## Link
- HAProxy home page  
http://www.haproxy.org/  
- Configuration Manual  
http://cbonte.github.io/haproxy-dconv/configuration-1.5.html


## 다운로드 및 압축 해제
1. yum install wget gcc gcc-c++ autoconf automake make openssl openssl-devel pcre-devel zlib
2. wget http://www.haproxy.org/download/1.5/src/haproxy-1.5.2.tar.gz
3. tar zxvf haproxy-1.5.2.tar.gz
4. cd haproxy-1.5.2
5. make TARGET=linux2628 USE_PCRE=1 USE_OPENSSL=1 USE_ZLIB=1
> 압축이 해제된 haproxy-1.5.2 디렉토리의 **README**　파일을 참고하여 설치합니다.  
> **target 옵션**
```
  - linux22     for Linux 2.2
  - linux24     for Linux 2.4 and above (default)
  - linux24e    for Linux 2.4 with support for a working epoll (> 0.21)
  - linux26     for Linux 2.6 and above
  - linux2628   for Linux 2.6.28, 3.x, and above (enables splice and tproxy)
  - solaris     for Solaris 8 or 10 (others untested)
  - freebsd     for FreeBSD 5 to 10 (others untested)
  - osx         for Mac OS/X
  - openbsd     for OpenBSD 3.1 to 5.2 (others untested)
  - aix51       for AIX 5.1
  - aix52       for AIX 5.2
  - cygwin      for Cygwin
  - generic     for any other OS or version.
  - custom      to manually adjust every setting
```
>`uname -a`로 현재 OS의 버전 확인.
```
>> uname -a   
Linux lvm01 2.6.32-358.el6.x86_64 #1 SMP Fri Feb 22 00:31:26 UTC 2013 x86_64 x86_64 x86_64 GNU/Linux
```
> **빌드 예제**  
> For example, I use this to build for Solaris 8 :  
> `$ make TARGET=solaris CPU=ultrasparc USE_STATIC_PCRE=1`  
>And I build it this way on OpenBSD or FreeBSD :   
> `$ gmake TARGET=freebsd USE_PCRE=1 USE_OPENSSL=1 USE_ZLIB=1`  
>And on a classic Linux with SSL and ZLIB support (eg: Red Hat 5.x) :  
>`$ make TARGET=linux26 USE_PCRE=1 USE_OPENSSL=1 USE_ZLIB=1`  
>And on a recent Linux >= 2.6.28 with SSL and ZLIB support :  
>`$ make TARGET=linux2628 USE_PCRE=1 USE_OPENSSL=1 USE_ZLIB=1`  
>In order to build a 32-bit binary on an x86_64 Linux system with SSL support  
>without support for compression but when OpenSSL requires ZLIB anyway :  
>`$ make TARGET=linux26 ARCH=i386 USE_OPENSSL=1 ADDLIB=-lz`
6. make install
>make install를 실행하면 아래와 같이 설치 위치가 출력됨.
```
install -d /usr/local/sbin
install haproxy /usr/local/sbin
install haproxy-systemd-wrapper /usr/local/sbin
install -d /usr/local/share/man/man1
install -m 644 doc/haproxy.1 /usr/local/share/man/man1
install -d /usr/local/doc/haproxy
for x in configuration architecture haproxy-en haproxy-fr; do \
		install -m 644 doc/$x.txt /usr/local/doc/haproxy ; \
	done
```
7. cp examples/haproxy.init /etc/init.d/haproxy
7. mkdir /etc/haproxy
8. cp ./examples.cfg  /etc/haproxy/haproxy.cfg
9. vi /etc/haproxy/haproxy.cfg
> 
```
lobal
	log 127.0.0.1	local0
	log 127.0.0.1	local1 notice
	#log loghost	local0 info
	maxconn 4096
	chroot /usr/share/haproxy
	uid 99
	gid 99
	daemon
	#debug
	#quiet
defaults
	log	global
	mode	http
	option	httplog
	option	dontlognull
    option forwardfor
	retries	3
	maxconn	2000
	option redispatch
    timeout connect      5000
    timeout client      50000
    timeout server      50000
listen	appli1-rewrite 0.0.0.0:80
	cookie	SERVERID rewrite
    cookie  JSESSIONID prefix
	balance	roundrobin
	server	webserver1 172.16.120.172:80 cookie webserver1 check inter 2000 rise 2 fall 5
	server	webserver2 172.16.120.173:80 cookie webserver2 check inter 2000 rise 2 fall 5
listen stats :4997
    mode http
    stats enable
    stats hide-version
    stats realm Haproxy\ Statistics
    stats uri /
    stats auth admin:pwd
```

## Remote Client 정보
http://www.stderr.net/apache/rpaf

    wget http://www.stderr.net/apache/rpaf/download/mod_rpaf-0.6.tar.gz




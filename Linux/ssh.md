
升级openssh9.7p1 需要openssl >=1.1.1
1. 升级openssl
```
yum install gcc openssl-devel zlib-devel


curl https://www.openssl.org/source/old/1.1.1/openssl-1.1.1w.tar.gz -o openssl-1.1.1w.tar.gz

tar xf openssl-1.1.1w.tar.gz && cd openssl-1.1.1w

./config --prefix=/usr/local/ssl --openssldir=/usr/local/ssl shared

make -j4 && make install
sh -c 'echo "/usr/local/ssl/lib" >> /etc/ld.so.conf.d/openssl-1.1.1.conf'

ldconfig -v |grep ssl

```
2. 升级openssh
```shell
yum install pam-devel -y
wget https://ftp.openbsd.org/pub/OpenBSD/OpenSSH/portable/openssh-9.7p1.tar.gz

cd openssh-9.7p1

./configure --prefix=/usr/local/openssh.97p1  --with-ssl-dir=/usr/local/ssl --with-pam

make -j4 && make install
/usr/local/openssh.97p1/bin/ssh -V
cp /usr/sbin/sshd /usr/sbin/sshd.old
ln -sv /usr/local/openssh.97p1/sbin/sshd  /usr/sbin/sshd


使用如下systemd配置文件
[Unit]
Description=OpenSSH server daemon
Documentation=man:sshd(8) man:sshd_config(5)
After=network.target sshd-keygen.service
Wants=sshd-keygen.service

[Service]
Type=simple
EnvironmentFile=/etc/sysconfig/sshd
ExecStart=/usr/sbin/sshd -D $OPTIONS
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target


systemctl daemon-reload
systemctl restart sshd
```
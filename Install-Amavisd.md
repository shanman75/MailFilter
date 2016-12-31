
Install AmavisD and ClamAV server
===========================

Install from Yum

```shell
yum install epel-release
yum --enablerepo=epel -y install amavisd-new clamav-server clamav-server-systemd
```

Enable the service for systemd

```shell
cp /usr/share/doc/clamav-server*/clamd.sysconfig /etc/sysconfig/clamd.amavisd
vi /etc/sysconfig/clamd.amavisd
```

```shell
# line 1, 2: uncomment and change
CLAMD_CONFIGFILE=/etc/clamd.d/amavisd.conf
CLAMD_SOCKET=/var/run/clamd.amavisd/clamd.sock
```

Enable tmpfile cleanup

```shell
vi /etc/tmpfiles.d/clamd.amavisd.conf
```

```shell
# create new
d /var/run/clamd.amavisd 0755 amavis amavis -
```

```shell
vi /usr/lib/systemd/system/clamd@.service
```

```shell
# add to the end
[Install]
WantedBy=multi-user.target
```

```shell
[root@mail ~]# systemctl start clamd@amavisd 
[root@mail ~]# systemctl enable clamd@amavisd 
```

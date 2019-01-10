
Install ClamAV
===========================

AWS Only
```shell
sudo amazon-linux-extras install epel
```
Continue:
```shell
yum --enablerepo=epel -y install clamav clamav-update
sed -i -e "s/^Example/#Example/" /etc/freshclam.conf
freshclam
```

```shell
vi /usr/lib/systemd/system/clam-freshclam.service
```

```shell
# Run the freshclam as daemon
[Unit]
Description = freshclam scanner
After = network.target

[Service]
Type = forking
ExecStart = /usr/bin/freshclam -d -c 4
Restart = on-failure
PrivateTmp = true

[Install]
WantedBy=multi-user.target
```

```shell
systemctl enable clam-freshclam.service
systemctl start clam-freshclam.service
```

Check Status
```shell
systemctl status clam-freshclam.service
```

```shell
? clam-freshclam.service - freshclam scanner
   Loaded: loaded (/usr/lib/systemd/system/clam-freshclam.service; enabled; vendor preset: disabled)
   Active: active (running) since Sat 2016-12-31 08:56:50 CST; 7s ago
  Process: 4000 ExecStart=/usr/bin/freshclam -d -c 4 (code=exited, status=0/SUCCESS)
 Main PID: 4001 (freshclam)
   CGroup: /system.slice/clam-freshclam.service
           +-4001 /usr/bin/freshclam -d -c 4

Dec 31 08:56:50 fructose.egarage.net systemd[1]: Starting freshclam scanner...
Dec 31 08:56:50 fructose.egarage.net freshclam[4001]: freshclam daemon 0.99.2 (OS: linux-gnu, ARCH: x86_64, CPU: x86_64)
Dec 31 08:56:50 fructose.egarage.net freshclam[4001]: ClamAV update process started at Sat Dec 31 08:56:50 2016
Dec 31 08:56:50 fructose.egarage.net systemd[1]: Started freshclam scanner.
Dec 31 08:56:50 fructose.egarage.net freshclam[4001]: main.cvd is up to date (version: 57, sigs: 4218790, f-level: 60, builder: amishhammer)
Dec 31 08:56:50 fructose.egarage.net freshclam[4001]: daily.cvd is up to date (version: 22810, sigs: 1214321, f-level: 63, builder: neo)
Dec 31 08:56:50 fructose.egarage.net freshclam[4001]: bytecode.cld is up to date (version: 285, sigs: 57, f-level: 63, builder: bbaker)
Dec 31 08:56:50 fructose.egarage.net freshclam[4001]: --------------------------------------
```

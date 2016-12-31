
Start the install of Postfix
===========================

```shell
[root@fructose ~]# yum -y install postfix pypolicyd-spf postgrey
```

```shell
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.compevo.com
 * extras: mirror.oss.ou.edu
 * updates: cosmos.illinois.edu
Package 2:postfix-2.10.1-6.el7.x86_64 already installed and latest version
Nothing to do
```

Add Configuration
----------------------------
```shell
postconf -e 'myhostname = fructose'
postconf -e 'mydomain = egarage.net'
postconf -e 'mydestination = $myhostname, $myhostname.$mydomain'
postconf -e 'inet_interfaces = all'
postconf -e 'inet_protocols = all'

postconf -e 'mynetworks = 127.0.0.0/8'
postconf -e 'message_size_limit = 30720000'

#configure maps/hashes
postconf -e 'alias_database = hash:/etc/aliases'
postconf -e 'relay_domains = $config_directory/domainmaps'
postconf -e 'relay_recipient_maps = hash:$config_directory/relay_recipients'
postconf -e 'transport_maps = hash:$config_directory/transport'


#sender restrictions
postconf -e 'smtpd_sender_restrictions = reject_non_fqdn_sender, reject_unknown_sender_domain, permit'

# HELO restrictions
postconf -e 'smtpd_delay_reject = yes'
postconf -e 'smtpd_helo_required = yes'
	

# Add AMAVISD-NEW filter
postconf -e 'content_filter = smtp-amavis:[127.0.0.1]:10024'


# Add hard reject messages for bad relay or local recipient
postconf -e 'unknown_local_recipient_reject_code = 551'
postconf -e 'unknown_relay_recipient_reject_code = 552'

# Update timings
postconf -e 'bounce_queue_lifetime = 4d'
postconf -e 'maximal_queue_lifetime = 4d'

# SPF
postconf -e 'spfpolicy_time_limit = 3600'


## TLS
postconf -e 'smtpd_use_tls = yes'
postconf -e 'smtpd_tls_key_file = /etc/postfix/egarage.net.privatekey'
postconf -e 'smtpd_tls_cert_file = /etc/postfix/egarage.net.crt'
postconf -e 'smtpd_tls_CAfile = /etc/postfix/godaddyCA.crt'
postconf -e 'smtpd_tls_loglevel = 0'
postconf -e 'smtpd_tls_received_header=yes'
postconf -e 'smtpd_tls_session_cache_timeout = 3600s'
postconf -e 'smtpd_tls_ask_ccert = yes'
postconf -e 'tls_random_source = dev:/dev/urandom'

postconf -e 'smtpd_tls_mandatory_protocols = !SSLv2,!SSLv3,!TLSv1,!TLSv1.1'
postconf -e 'smtpd_tls_protocols=!SSLv2,!SSLv3,!TLSv1,!TLSv1.1'
postconf -e 'smtpd_tls_mandatory_ciphers = medium'
postconf -e 'tls_medium_cipherlist = AES128+EECDH:AES128+EDH'
```

Enable Postgrey
```shell
systemctl start postgrey
systemctl enable postgrey
```


Install Maps
--------------------------------
```shell
scp shanman@source:"\
/etc/postfix/Makefile \
/etc/postfix/aliases \
/etc/postfix/domainmaps \
/etc/postfix/helo_access \
/etc/postfix/relay_recipients \
/etc/postfix/relay_recipients_rejects \
/etc/postfix/transport \
/etc/postfix/virtual \
" /etc/postfix/

scp shanman@source:"\
/etc/postfix/egarage.net.crt \
/etc/postfix/egarage.net.privatekey \
/etc/postfix/godaddyCA.crt \
" /etc/postfix/


```

Make Maps
---------------------------------------
```shell
cd /etc/postfix/
make
```

edit /etc/postfix/master.cf
```shell
policyd-spf  unix  -       n       n       -       0       spawn
                   user=nobody argv=/usr/libexec/postfix/policyd-spf
``` 

```shell
postfix reload
```
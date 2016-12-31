
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

```shell
vi /etc/amavisd/amavisd.conf
```

```shell
$mydomain = 'egarage.net';   # a convenient default for other settings

$TEMPBASE = "/tempspool/amavis/tmp";   # working directory, needs to exist, -T

$QUARANTINEDIR = "/tempspool/amavis/virusmails";

$myhostname = 'fructose.egarage.net';


$interface_policy{'10024'} = 'EXTERNAL';

$policy_bank{'EXTERNAL'} = {
  log_level => 2,
  forward_method => 'smtp:[127.0.0.1]:10025',
  notify_method => 'smtp:[127.0.0.1]:10025',
  smtpd_discard_ehlo_keywords => ['8BITMIME'],
  bypass_banned_checks_maps => [1],
  terminate_dsn_on_notify_success => 0,
  virus_admin_maps => ["spamcop\@$mydomain"],
  spam_admin_maps  => ["spamcop\@$mydomain"],
};



$sa_tag_level_deflt  = 0.0;  # add spam info headers if at, or above that level
$sa_tag2_level_deflt = 5.5;  # add 'spam detected' headers at that level
$sa_kill_level_deflt = 7.5;  # triggers spam evasive actions (e.g. blocks mail)
$sa_dsn_cutoff_level = 10;   # spam level beyond which a DSN is not sent
$sa_crediblefrom_dsn_cutoff_level = 18; # likewise, but for a likely valid From
# $sa_quarantine_cutoff_level = 25; # spam level beyond which quarantine is off
$penpals_bonus_score = 8;    # (no effect without a @storage_sql_dsn database)
$penpals_threshold_high = $sa_kill_level_deflt;  # don't waste time on hi spam
$bounce_killer_score = 100;  # spam score points to add for joe-jobbed bounces

$sa_mail_body_size_limit = 500*1024; # don't waste time on SA if mail is larger
$sa_local_tests_only = 0;    # only tests which do not require internet access?



$MAX_EXPANSION_QUOTA = 500*1024*1024;  # bytes  (default undef, not enforced)

$sa_spam_subject_tag = '***Spam*** ';
$sa_spam_modifies_subj = 1;
$defang_virus  = 1;  # MIME-wrap passed infected mail
$defang_banned = 1;  # MIME-wrap passed mail containing banned name
# for defanging bad headers only turn on certain minor contents categories:
$defang_by_ccat{CC_BADH.",3"} = 1;  # NUL or CR character in header
$defang_by_ccat{CC_BADH.",5"} = 1;  # header line longer than 998 characters
$defang_by_ccat{CC_BADH.",6"} = 1;  # header field syntax error

# sitewide senders
     # egarage extensions
     'enews2@midwiferytoday.com'              => -5.0,      # Added by SCR
     lc('TericaLynn@mail2world.com')            => -5.0,    # Added by SCR 11/4/04 for communications from->to buster
     lc('s-perry@comcast.net')                  => -5.0,    # Added by SCR 11/17/04 for communications to lisa@thebridwells.com
     lc('donotreply@fedex.com')                 => -9.0,    # Added by SCR 11/18/04 for FEDEX notification
     lc('email@news.tivo.com')                   => -7.0,    # Added by SCR 11/18/04 for TIVO mail
     lc('WebBoard@www.uorobot.com')             => -5.0,    # Added by SCR 11/23/04 for c3c1
     lc('ecrew@firehousesubs.net')              => -5.5,    # Added by SCR 12/4/04 for david@theclevelands.net
     lc('clearbot@clearmail2.etrade.com')       => -5.0,    # Added by SCR 02/03/05 for shanman@shanman.com's ETRADE alerts
     lc('alerts@notifications.ameritrade.com')  => -5.0,    # Added by SCR 02/04/05 for shanman@shanman.com AMeritrade alerts
     lc('webmaster@fastweb.com')                => -5.0,    # Added by SCR 02/04/05 for scholarship notifications
     lc('EQ@lopkop.com')                        => -5.0,    # Added by SCR 05/17/05 for notifications
     lc('LTBedell@aol.com')                     => -7.0,    # Added by SCR 05/27/05 for hallie@shanman.com
     lc('edithann@bright.net')                  => -7.0,    # Added by SCR 02/28/06 for buster@staci.net
     lc('jimmiller@comcast.net')                => -9.0,    # Added by c3  12/16/06 for theclevelands.
     lc('ronclev@comcast.net')                  => -9.0,    # Added by c3  12/16/06 for theclevelands.
     lc('olgrizzly@verizon.net')                => -5.0,    # Added by c3  12/16/06 for theclevelands.
     lc('cmille4@CLEMSON.EDU')                  => -5.0,    # Added by c3  12/16/06 for theclevelands.


$per_recip_whitelist_sender_lookup_tables = {
  'shanman@shanman.com' =>[qw( c3c1@c3c1.com root@egarage.net .grey.egarage.net root@curly.egarage.net alertingservice@citibank.com .aa.com .eds.com alerts@ameritradeinfo.com .tmomail.net)],
  'c3c1@c3c1.com' => [qw ( shanman@shanman.com root@egarage.net .grey.egarage.net .nmdfwvpn1.colemont.com .colemont.com)],
  'dan@kardell.org' => [qw ( .americanexpress.com .talkmatch.com)],
  'root@grey.egarage.net' => [qw (.grey.egarage.net)],
  'hallie@shanman.com' => [qw (.friscoisd.org)],
  'lisa@thebridwells.com' => [qw (jenniferewood@comcast.net)],
  'root@mo.egarage.net' => [qw (root@mo.egarage.net)],
  'root@curly.egarage.net' => [qw (root@curly.egarage.net)],
  'root@carbs.egarage.net' => [qw (root@carbs.egarage.net)],
  'DNSPing@egarage.net' => [qw (shanman@shanman.com c3c1@c3c1.com)],
};


```

```shell
mkdir /tempspool
mkdir /tempspool/amavis
mkdir /tempspool/amavis/tmp
mkdir /tempspool/amavis/virusmails
```

And, let her rip...Start your engines!

```shell
[root@mail ~]# systemctl start amavisd spamassassin 
[root@mail ~]# systemctl enable amavisd spamassassin
```


Add postfix link

Edit /etc/postfix/master.cf

```shell
smtp-amavis unix -    -    n    -    2 smtp
    -o smtp_data_done_timeout=1200
    -o smtp_send_xforward_command=yes
    -o disable_dns_lookups=yes
127.0.0.1:10025 inet n    -    n    -    - smtpd
    -o content_filter=
    -o local_recipient_maps=
    -o relay_recipient_maps=
    -o smtpd_restriction_classes=
    -o smtpd_client_restrictions=
    -o smtpd_helo_restrictions=
    -o smtpd_sender_restrictions=
    -o smtpd_recipient_restrictions=permit_mynetworks,reject
    -o mynetworks=127.0.0.0/8
    -o strict_rfc821_envelopes=yes
    -o smtpd_error_sleep_time=0
    -o smtpd_soft_error_limit=1001
    -o smtpd_hard_error_limit=1000
```
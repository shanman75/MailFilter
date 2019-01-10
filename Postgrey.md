Install Postgrey
===========================

Install module
```shell
yum install postgrey
```

Add the following to /etc/postfix/main.cf
```shell
smtpd_recipient_restrictions =
        permit_mynetworks,
#        check_recipient_access hash:$config_directory/recipient_access,
        check_recipient_access hash:/etc/postfix/relay_recipients_rejects,
#            permit_mynetworks
        reject_unauth_pipelining,
        reject_unauth_destination,
        reject_non_fqdn_recipient,
#SPF check
#        check_policy_service unix:private/spfpolicy,
        check_policy_service unix:private/policyd-spf,
        
#greylist
#        check_policy_service inet:127.0.0.1:10023,
        check_policy_service unix:/var/spool/postfix/postgrey/socket
```


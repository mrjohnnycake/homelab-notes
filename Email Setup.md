# Note on the note

I used to run Ubuntu on my VMs before switching to Debian. I found that Debian already had some packages installed and the steps were slightly different even though Ubuntu is based on Debian. If you're using Ubuntu I would suggest trying the Debian walkthrough and if it doesn't work then check out what is different and make those changes.


# Debian Based

Email Alerts Setup (Gmail)

1) Make sure you have `mailutils` and `postfix` installed

2. Enable 2FA for the gmail account that will be used by going to [security settings](https://myaccount.google.com/security)

3. Create app password for the account.
    1. Go to [App Passwords](https://security.google.com/settings/security/apppasswords)
    2. Select app: Mail
    3. Select device: Other
    4. Type in: Proxmox

***
## ==[[Default Debian VM (SeaBIOS)]] part 1==
- The next few steps have already been done in an attempt to save time so skip to the next section with a highlighted header below
***

Create the main postfix config:
```
sudo nano /etc/postfix/main.cf
```

Paste the following into the file:
```
# See /usr/share/postfix/main.cf.dist for a commented, more complete version

myhostname=Crow.local

# Debian specific:  Specifying a file name will cause the first
# line of that file to be used as the name.  The Debian default
# is /etc/mailname.
#myorigin = /etc/mailname

smtpd_banner = $myhostname ESMTP $mail_name (Ubuntu)
biff = no

# appending .domain is the MUA's job.
append_dot_mydomain = no

# Uncomment the next line to generate "delayed mail" warnings
#delay_warning_time = 4h

readme_directory = no

# See http://www.postfix.org/COMPATIBILITY_README.html -- default to 3.6 on
# fresh installs.
compatibility_level = 3.6

# TLS parameters
smtpd_tls_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
smtpd_tls_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
smtpd_tls_security_level=may
smtp_tls_CApath=/etc/ssl/certs
smtp_tls_security_level=may
# smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache

# this has to do with changing the from name... somehow
sender_canonical_maps = static:no-reply@gmail.com

smtpd_relay_restrictions = permit_mynetworks permit_sasl_authenticated defer_unauth_destination
# myhostname = websites.tailf3fc5.ts.net
alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases

mydestination = $myhostname, asd, websites, localhost.localdomain, localhost
# relayhost =
mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128
mailbox_size_limit = 0
recipient_delimiter = +
inet_interfaces = all
inet_protocols = all

relayhost = smtp.gmail.com:587
smtp_use_tls = yes
smtp_sasl_auth_enable = yes
smtp_sasl_security_options =
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_tls_CAfile = /etc/ssl/certs/Entrust_Root_Certification_Authority.pem
smtp_tls_session_cache_database = btree:/var/lib/postfix/smtp_tls_session_cache
smtp_tls_session_cache_timeout = 3600s
smtp_header_checks = regexp:/etc/postfix/header_checks
```

Write Gmail credentials to file and hash it. Again, make sure you are root.
```
sudo nano /etc/postfix/sasl_passwd
```

Add this, replacing with your password
```
smtp.gmail.com youremailname@gmail.com:your-password
```
- Password is in 1P

```
sudo chmod 600 /etc/postfix/sasl_passwd
```

***
## ==[[Default Debian VM (SeaBIOS)]] part 2==
***

Create a file where you can edit who the email is from:
```
cd /etc/postfix

sudo nano /etc/postfix/header_checks
```

Add this:
```
/^From:[[:space:]]+(.*)/ REPLACE From: "VM or Machine Name" <youremailname@gmail.com>
```

```
sudo postmap header_checks

sudo postmap hash:/etc/postfix/sasl_passwd
```

```
sudo systemctl enable postfix

sudo systemctl start postfix
```

Test to make sure everything is hunky-dory.
```
echo "sample message" | mail -s "sample subject" email@gmail.com
```

Reload with this if you need to:
```
sudo postfix reload
```



# Ubuntu Based #

Email Alerts Setup (Gmail)

1. SSH into proxmox node and become root user. Run the following commands to download extra software dependencies we'll need.

```
sudo apt update

sudo apt install libsasl2-modules mailutils postfix -y
```

* When the postfix install dialogue comes up, enter `1` for `No configuration`

Reboot to get all of the new services working
```
sudo reboot
```

2. Setup Gmail 2FA (if you haven't already)

- Enable 2FA for the gmail account that will be used (if you haven't already) by going to [Gmails's security settings](https://myaccount.google.com/security)

- Create app password for the account (if you haven't already).
    - Go to [App Passwords](https://security.google.com/settings/security/apppasswords)
    - Select app: Mail
    - Select device: Other
    - Type in: VMs or whatever describes your needs
  
3. Configure Postfix

Create the Postfix configuration file:
```
sudo vim /etc/postfix/main.cf
```

Append the following to the end of the file:
```sh
# See /usr/share/postfix/main.cf.dist for a commented, more complete version

# Debian specific:  Specifying a file name will cause the first
# line of that file to be used as the name.  The Debian default
# is /etc/mailname.
#myorigin = /etc/mailname

smtpd_banner = $myhostname ESMTP $mail_name (Ubuntu)
biff = no

# appending .domain is the MUA's job.
append_dot_mydomain = no

# Uncomment the next line to generate "delayed mail" warnings
#delay_warning_time = 4h

readme_directory = no

# See http://www.postfix.org/COMPATIBILITY_README.html -- default to 3.6 on
# fresh installs.
compatibility_level = 3.6

# TLS parameters
smtpd_tls_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
smtpd_tls_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
smtpd_tls_security_level=may
smtp_tls_CApath=/etc/ssl/certs
smtp_tls_security_level=may
# smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache

smtpd_relay_restrictions = permit_mynetworks permit_sasl_authenticated defer_unauth_destination
myhostname = websites.tailf3fc5.ts.net
alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases
mydestination = $myhostname, asd, websites, localhost.localdomain, localhost
# relayhost =
mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128
mailbox_size_limit = 0
recipient_delimiter = +
inet_interfaces = all
inet_protocols = all

relayhost = smtp.gmail.com:587
smtp_use_tls = yes
smtp_sasl_auth_enable = yes
smtp_sasl_security_options =
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_tls_CAfile = /etc/ssl/certs/Entrust_Root_Certification_Authority.pem
smtp_tls_session_cache_database = btree:/var/lib/postfix/smtp_tls_session_cache
smtp_tls_session_cache_timeout = 3600s
```
- Save

```
sudo vim /etc/postfix/sasl_passwd
```

Add the following line making sure to set the correct email and 2FA secure password from the step above:
```
smtp.gmail.com youremail@gmail.com:yourpassword
```

```
sudo chmod 600 /etc/postfix/sasl_passwd

sudo postmap hash:/etc/postfix/sasl_passwd
```

* That last line will generate /etc/postfix/sasl_passwd.db so that sasl_passwd is secure

```
sudo systemctl enable postfix

sudo systemctl start postfix
```

Reload if needed
```
sudo postfix reload
```

4. Test to make sure everything is working
```
sudo echo "sample message" | mail -s "sample subject" email@gmail.com
```

*IMPORTANT*
With updates to Postfix and SMTP and the rest, this configuration will likely need to be updated in the future

sa-secure-auditd
================
[![Build Status](https://travis-ci.org/softasap/sa-secure-auditd.svg?branch=master)](https://travis-ci.org/softasap/sa-secure-auditd)


Example of use: check box-example

Simple:

```YAML

custom_auditd_log_group: syslog # default root, change to other user if you plan to log messages

custom_auditd_props:
    - {regexp: "^log_group =*", line: "log_group = {{auditd_log_group}}"}

custom_auditd_rules:
  - "-D"  # clean rules
  - "-b 320"  # no of bufs for messages
  - "-f 1" # on failure 0 nothing, 1 dmesg, 2 kernel panic
  - "-a exit,always -S unlink -S rmdir"  # notify unlink rmdir
  - "-w /etc/group -p wa" # group modification
  - "-w /etc/passwd -p wa" # passwords modification
  - "-w /etc/shadow -p wa"
  - "-w /etc/sudoers -p wa"  
  - "-e 2" # prevent further changes to config

```

```YAML


     - {
         role: "sa-secure-auditd"
       }

```


Advanced:

```YAML


     - {
         role: "sa-secure-auditd",
         auditd_conf_props: "{{custom_auditd_props}}",
         auditd_log_group: custom_auditd_log_group  
       }


```


Hint: viewing auditd reports 
```bash
$ sudo journalctl --boot _TRANSPORT=audit
-- Logs begin at Thu 2016-01-05 09:20:01 CET. --
Jan 05 09:47:24 arsenic audit[3028]: USER_END pid=3028 uid=0 auid=1000 ses=3 subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 msg='op=PAM:session_close grantors=pam_keyinit,pam_limits,pam_keyinit,pam_limits,pam_systemd,pam_unix acct="root" exe="/usr/bin/sudo" hostname=? addr=? terminal=/dev/pts/4 res=success'
...
```

Or, perhaps

```bash
sudo journalctl -af _TRANSPORT=audit
```

Example of full audit.rules that correspond to
https://www.stigviewer.com/stig/red_hat_enterprise_linux_6/

```
## RHEL 6 Security Technical Implementation Guide

## Remove any existing rules
-D

## Buffer Size
-b 8192

## Failure Mode
-f 2

## Audit the audit logs
-w /var/log/audit/ -k auditlog

## Auditd configuration
-w /etc/audit/ -p wa -k auditconfig
-w /etc/libaudit.conf -p wa -k auditconfig
-w /etc/audisp/ -p wa -k audispconfig

## Monitor for use of audit management tools
-w /sbin/auditctl -p x -k audittools
-w /sbin/auditd -p x -k audittools

## Monitor AppArmor configuration changes
-w /etc/apparmor/ -p wa -k apparmor
-w /etc/apparmor.d/ -p wa -k apparmor

## Monitor usage of AppArmor tools
-w /sbin/apparmor_parser -p x -k apparmor_tools
-w /usr/sbin/aa-complain -p x -k apparmor_tools
-w /usr/sbin/aa-disable -p x -k apparmor_tools
-w /usr/sbin/aa-enforce -p x -k apparmor_tools

## Monitor Systemd configuration changes
-w /etc/systemd/ -p wa -k systemd
-w /lib/systemd/ -p wa -k systemd

## Monitor usage of systemd tools
-w /bin/systemctl -p x -k systemd_tools
-w /bin/journalctl -p x -k systemd_tools

## Special files
-a always,exit -F arch=b64 -S mknod -S mknodat -k specialfiles

## Mount operations
-a always,exit -F arch=b64 -S mount -S umount2 -k mount

## Changes to the time
-a always,exit -F arch=b64 -S adjtimex -S settimeofday -S clock_settime -k time

## Cron configuration & scheduled jobs
-w /etc/cron.allow -p wa -k cron
-w /etc/cron.deny -p wa -k cron
-w /etc/cron.d/ -p wa -k cron
-w /etc/cron.daily/ -p wa -k cron
-w /etc/cron.hourly/ -p wa -k cron
-w /etc/cron.monthly/ -p wa -k cron
-w /etc/cron.weekly/ -p wa -k cron
-w /etc/crontab -p wa -k cron
-w /var/spool/cron/crontabs/ -k cron

## User, group, password databases
-w /etc/group -p wa -k etcgroup
-w /etc/passwd -p wa -k etcpasswd
-w /etc/gshadow -k etcgroup
-w /etc/shadow -k etcpasswd
-w /etc/security/opasswd -k opasswd

## Monitor usage of passwd
-w /usr/bin/passwd -p x -k passwd_modification

## Monitor for use of tools to change group identifiers
-w /usr/sbin/groupadd -p x -k group_modification
-w /usr/sbin/groupmod -p x -k group_modification
-w /usr/sbin/addgroup -p x -k group_modification
-w /usr/sbin/useradd -p x -k user_modification
-w /usr/sbin/usermod -p x -k user_modification
-w /usr/sbin/adduser -p x -k user_modification

## Monitor module tools
-w /sbin/insmod -p x -k modules
-w /sbin/rmmod -p x -k modules
-w /sbin/modprobe -p x -k modules

## Login configuration and information
-w /etc/login.defs -p wa -k login
-w /etc/securetty -p wa -k login
-w /var/log/faillog -p wa -k login
-w /var/log/lastlog -p wa -k login
-w /var/log/tallylog -p wa -k login

## Network configuration
-w /etc/hosts -p wa -k hosts
-w /etc/network/ -p wa -k network

## System startup scripts
-w /etc/inittab -p wa -k init
-w /etc/init.d/ -p wa -k init
-w /etc/init/ -p wa -k init

## Library search paths
-w /etc/ld.so.conf -p wa -k libpath

## Local time zone
-w /etc/localtime -p wa -k localtime

## Time zone configuration
-w /etc/timezone -p wa -k timezone

## Kernel parameters
-w /etc/sysctl.conf -p wa -k sysctl

## Modprobe configuration
-w /etc/modprobe.conf -p wa -k modprobe
-w /etc/modprobe.d/ -p wa -k modprobe
-w /etc/modules -p wa -k modprobe

# Module manipulations.
-a always,exit -F arch=b64 -S init_module -S delete_module -k modules

## PAM configuration
-w /etc/pam.d/ -p wa -k pam
-w /etc/security/limits.conf -p wa -k pam
-w /etc/security/pam_env.conf -p wa -k pam
-w /etc/security/namespace.conf -p wa -k pam
-w /etc/security/namespace.init -p wa -k pam

## Postfix configuration
-w /etc/aliases -p wa -k mail
-w /etc/postfix/ -p wa -k mail

## SSH configuration
-w /etc/ssh/sshd_config -k sshd

## Changes to hostname
-a exit,always -F arch=b64 -S sethostname -k hostname

## Changes to issue
-w /etc/issue -p wa -k etcissue
-w /etc/issue.net -p wa -k etcissue

## Capture all failures to access on critical elements
-a exit,always -F arch=b64 -S open -F dir=/etc -F success=0 -k unauthedfileaccess
-a exit,always -F arch=b64 -S open -F dir=/bin -F success=0 -k unauthedfileaccess
-a exit,always -F arch=b64 -S open -F dir=/sbin -F success=0 -k unauthedfileaccess
-a exit,always -F arch=b64 -S open -F dir=/usr/bin -F success=0 -k unauthedfileaccess
-a exit,always -F arch=b64 -S open -F dir=/usr/sbin -F success=0 -k unauthedfileaccess
-a exit,always -F arch=b64 -S open -F dir=/var -F success=0 -k unauthedfileaccess
-a exit,always -F arch=b64 -S open -F dir=/home -F success=0 -k unauthedfileaccess
-a exit,always -F arch=b64 -S open -F dir=/root -F success=0 -k unauthedfileaccess
-a exit,always -F arch=b64 -S open -F dir=/srv -F success=0 -k unauthedfileaccess
-a exit,always -F arch=b64 -S open -F dir=/tmp -F success=0 -k unauthedfileaccess

## Monitor for use of process ID change (switching accounts) applications
-w /bin/su -p x -k priv_esc
-w /usr/bin/sudo -p x -k priv_esc
-w /etc/sudoers -p rw -k priv_esc

## Monitor usage of commands to change power state
-w /sbin/shutdown -p x -k power
-w /sbin/poweroff -p x -k power
-w /sbin/reboot -p x -k power
-w /sbin/halt -p x -k power

## Monitor admins accessing user files.
-a always,exit -F dir=/home/ -F uid=0 -C auid!=obj_uid -k admin_user_home

## Monitor changes and executions in /tmp and /var/tmp.
-w /tmp/ -p wxa -k tmp
-w /var/tmp/ -p wxa -k tmp

## Make the configuration immutable
-e 2
```

Copyright and license
---------------------

Code licensed under the [BSD 3 clause] (https://opensource.org/licenses/BSD-3-Clause) or the [MIT License] (http://opensource.org/licenses/MIT).

Subscribe for roles updates at [FB] (https://www.facebook.com/SoftAsap/)

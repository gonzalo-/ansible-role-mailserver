Ansible role for a Mailserver
=============================

Ansible role to create a Mailserver on OpenBSD (>=7.5 & -current) with OpenSMTPD, Dovecot and Rspamd.

Requirements
------------

OpenBSD, Python 3 (on client machine) and 10 minutes.

Notes
-----

This is still a WIP, so far, you need to create DKIM keys with rspamd (https://rspamd.com/doc/modules/dkim_signing.html),
new users and DNS entries. Also, you need to enable dovecot, smtpd and rspamd at boot.

You need to adjust your pf.conf (example bellow).

Also, you need to delete examples on /etc/mail/virtuals and /etc/mail/domains

Feedback is welcome.

Example pf.conf
---------------

For IMAPs I like to keep bruteforce people away with some tweaks on pf.conf, keep in mind that
lower numbers than that, can couse problems checking emails on huge directories like misc or mailing list.

```
...
pass in quick on egress proto tcp from any \
        to (egress) port imaps \
        flags S/SA modulate state \
        (max-src-conn 50, max-src-conn-rate 50/5, overload <bruteforce> flush global)
...
```

For SMTP I have pretty the same:

```
...
pass in quick log (to pflog1) proto tcp from any \
        to (egress) port smtp

pass in quick log (to pflog1) proto tcp from any \
        to (egress) port { submission, smtps } \
        flags S/SA modulate state \
        (max-src-conn 50, max-src-conn-rate 25/5, overload <bruteforce> flush global)
...
```

Example Ansible
---------------

This example is for a remote setup, so ,,test'' is your future mailserver, you
already put your ssh key on ,,test'' and this server already have python3.8
installed.

```
$ doas pkg_add ansible
...
$ cd /tmp && mkdir ansible && cd ansible
$ git clone https://github.com/gonzalo-/ansible-role-mailserver
...
...
...
$ mv ansible-role-mailserver gonzalo-.mailserver
$ cat hosts
test ansible_python_interpreter=/usr/local/bin/python3.8
$ cat mailserver.yml
---
- hosts: test
  roles:
     - role: gonzalo-.mailserver
  become: yes
  become_method: doas

  vars:
   domain: 'foobar.com'
   mail_dir: '/var/vmail'
   mail_user: 'gonzalo'
   release: '7.5'
   arch: 'amd64'
   installurl_mirror: 'https://cdn.openbsd.org/pub/OpenBSD/'
   pkg_path: 'https://cdn.openbsd.org/pub/OpenBSD/{{ release }}/packages/{{ arch }}/'
   packages_list:
    - dovecot
    - dovecot-pigeonhole
    - opensmtpd-extras
    - opensmtpd-filter-rspamd
    - opensmtpd-filter-senderscore
    - rspamd
$ ansible-playbook -i hosts mailserver.yml
...MAGIC...
$
```

Example Playbook
----------------
```
---
- hosts: test
  roles:
     - role: gonzalo-.mailserver
  become: yes
  become_method: doas

  vars:
   domain: 'foobar.com'
   mail_dir: '/var/vmail'
   mail_user: 'gonzalo'
   release: '7.5'
   arch: 'amd64'
   installurl_mirror: 'https://cdn.openbsd.org/pub/OpenBSD/'
   pkg_path: 'https://cdn.openbsd.org/pub/OpenBSD/{{ release }}/packages/{{ arch }}/'
   packages_list:
    - dovecot
    - dovecot-pigeonhole
    - opensmtpd-extras
    - opensmtpd-filter-rspamd
    - opensmtpd-filter-senderscore
    - rspamd
```

Enable Spam Learning with Dovecot Antispam
------------------------------------------
The rspamd system can be trained to learn spam (or ham) by checking if users move
their email from a folder in their Inbox to the Spam folder (to flag as spam), or
the reverse: move incorrectly flagged spam out of the Spam folder. This will incur
some overhead when it comes to moving messages, but it can enable the system to learn
spam/ham more efficiently.

Note: This ONLY works with IMAP

To enable, modify the following line in /etc/dovecot/conf.d/20-imap.conf:
```
mail_plugins = $mail_plugins imap_sieve
```

Also edit /etc/dovecot/conf.d/90-plugin.conf if you want to enable more logging
or to change the default Spam and Trash folders (if they are different on your system)
and then restart dovecot with: ```rcctl restart dovecot```


Author Information
------------------

https://x61.sh/

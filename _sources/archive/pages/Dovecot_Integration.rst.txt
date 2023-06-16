.. _dovecot_integration_with_ipa:

Dovecot Integration with IPA
============================

`Provided by Dale Macartney on the freeipa-users@redhat.com
list. <https://www.redhat.com/archives/freeipa-users/2012-January/msg00231.html>`__

| ``# Connect server to IPA domain (ensure DNS is working correctly otherwise this step will fail)``
| ``ipa-client-install -U -p admin -w mysecretpassword``

| ``# install postfix if necessary (installed by default in rhel6)``
| ``yum -y install postfix``

| ``# set postfix to start on boot``
| ``chkconfig postfix on``

| ``# configure postfix with hostname, domain and origin details``
| ``sed -i 's/#myhostname = host.domain.tld/myhostname = servername.example.com/g' /etc/postfix/main.cf``
| ``sed -i 's/#mydomain = domain.tld/mydomain = example.com/g' /etc/postfix/main.cf``
| ``sed -i 's/#myorigin = $mydomain/myorigin = $mydomain/g' /etc/postfix/main.cf``

| ``# configure postfix to listen on all interfaces``
| ``sed -i 's/#inet_interfaces = all/inet_interfaces = all/g' /etc/postfix/main.cf``
| ``sed -i 's/inet_interfaces = localhost/#inet_interfaces = localhost/g' /etc/postfix/main.cf``

| ``# apply postfix changes``
| ``service postfix restart``

| ``# Install dovecot``
| ``yum -y install dovecot``

| ``# set dovecot to start on boot``
| ``chkconfig dovecot on``

| ``# set dovecot to listen on imap and imaps only``
| ``sed -i 's/#protocols = imap pop3 lmtp/protocols = imap imaps/g' /etc/dovecot/dovecot.conf``

| ``# point dovecot to required mailbox directory (This is the section that was previously failing)``
| ``echo "mail_location = mbox:~/mail:INBOX=/var/mail/%u" >> /etc/dovecot/dovecot.conf``

| ``# reload dovecot to apply changes``
| ``service dovecot restart``

| ``# Apply working IPtables``
| ``cat > /etc/sysconfig/iptables << EOF``
| ``# Generated by iptables-save v1.4.7 on Tue Jan 10 12:17:41 2012``
| ``*filter``
| ``:INPUT ACCEPT [0:0]``
| ``:FORWARD ACCEPT [0:0]``
| ``:OUTPUT ACCEPT [29:4596]``
| ``- - -A INPUT -p tcp -m tcp --dport 25 -j ACCEPT``
| ``- - -A INPUT -p tcp -m tcp --dport 143 -j ACCEPT``
| ``- - -A INPUT -p tcp -m tcp --dport 993 -j ACCEPT``
| ``- - -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT``
| ``- - -A INPUT -p icmp -j ACCEPT``
| ``- - -A INPUT -i lo -j ACCEPT``
| ``- - -A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT``
| ``- - -A INPUT -j REJECT --reject-with icmp-host-prohibited``
| ``- - -A FORWARD -j REJECT --reject-with icmp-host-prohibited``
| ``COMMIT``
| ``# Completed on Tue Jan 10 12:17:41 2012``
| ``EOF``

With the above details, one gets a 100% working IPA authenticated mail
server, allowing IPA users to retrieve mail via imap/imaps.

Please note: This is using the default certificates created by the
dovecot installation. It is highly recommended you replace these with
your own valid certificates if you wish to run IMAPS in a production
environment.

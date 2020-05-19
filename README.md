LDAP & NFS & DNS
================

LDAP
----

### OpenLdap Server 구축

$ yum install -y openldap-servers openldap-clients nss-pam-ldapd
// 기본 패키지 인스톨
$ systemctl start slapd 
// 데몬시작 (389 포트)

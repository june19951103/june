LDAP & NFS & DNS
================

LDAP
----

### OpenLdap Server 구축

#### LDAP Server

* "$" -> "#" 으로 정정.

        $ yum install -y openldap-servers openldap-clients nss-pam-

// 기본 패키지 인스톨

        $ systemctl start slapd 

// 데몬시작 (389 포트)

$ vi /etc/hosts & ping sever

// 도메인 설정 및 핑 테스트

192.168.111.100 ldapserver.co.kr  server
192.168.111.200 ldapclient.co.kr  client
192.168.1111.250 mountserver.co.kr  mount

Openldap의 환경설정은 /etc/openldap/slapd.d/cn=config/olcDatabase=~.ldif를 읽어오는데, 
해당 파일을 수동편집하는게 아니라 Config 양식을 생성 후, ldapmodify 명령어로 업데이트를 하는 방식이다.

$ cp /usr/share/openldap-servers/slapd.ldif /etc/openldap/slapd.conf

// 환경설정 템플릿을 복사.

$ slappasswd 

// ldap 관리자 패스워드 설정 
// {SSHA}/TJ2S3gfbQyhVOIWNn3naK8OojDrhs06 해쉬로 변환

$ vi domain.ldif

dn:olcDatabase={2}hdb,cn=config
changetype:modify
replace:olcSuffix
olcSuffix:dc=ldapserver,dc=co,dc=kr

// dc 설정 (ldapserver.co.kr)

dn:olcDatabase={2}hdb,cn=config
changetype:modify
replace:olcRootDN
olcRootDN:cn=Manager,dc=ldapserver,dc=co,dc=kr

// dc 설정 (Manager 계정 = root)

dn:olcDatabase={2}hdb,cn=config
changetype:modify
replace:olcRootPW
olcRootPW:{SSHA}+69cxc4V3XQBQ4WY7T/HmjqVg/jJKcyx

// 원본설정 수정선언, RootPW 설정선언, 패스워드 기입

$ ldapmodify -Y EXTERNAL -H ldapi:/// -f domain.ldif

// LDAP Server에 환경설정 업데이트

#### LDAP 인증서 만들기

$ openssl req -new -x509 -out /etc/openldap/certs/ldapcert.pem -keyout /etc/openldap/certs/ldapkey.pem -days 365
// LDAP 서버에 대한 자체 서명된 인증서를 작성하자. 아래 명령은 "/etc/openldap/certs" 디렉토리에 인증서와 개인 키를 모두 생성

$ chown -R ldap:ldap /etc/openldap/certs/*.pem
$ ll /etc/openldap/certs/*.pem
// 권한을 ldap으로 변경 후 확인.

$ vi certs.ldif

dn:cn=config
changetype:modify
replace:olcTLSCertificateFile
olcTLSCertificateFile:/etc/openldap/certs/ldapcert.pem

dn:cn=config
changetype:modify
replace:olcTLSCertificateKeyFile
olcTLSCertificateKeyFile:/etc/openldap/certs/ldapkey.pem

$ ldapmodify -Y EXTERNAL -H ldapi:/// -f certs.ldif
// 업데이트 후 laptest -u 로 환경설정 적용확인.

$ cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG
// 유저DB 설정 탬플릿 복사.

$ chown -R ldap:ldap /var/lib/ldap
// 해당 폴더 전체 ldap 소유로 변경.

$ ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif
$ ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif
$ ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif
// 기본 디렉터리 DB 구조를 추가.

$ vi directory.ldif

dn:dc=ldapserver,dc=co,dc=kr
dc:ldapserver
objectClass:top
objectClass:domain
// ldapserver.co.kr 도메인의 LDAP Server Manager로 정의.

dn:cn=Manager,dc=ldapserver,dc=co,dc=kr
objectClass:organizationalRole
cn:Manager
description:LDAP Manager

dn:ou=People,dc=ldapserver,dc=co,dc=kr
objectClass:organizationalUnit
ou:People

dn:ou=Group,dc=ldapserver,dc=co,dc=kr
objectClass:organizationalUnit
ou:Group


$ ldapadd -x -W -D “cn=Manager,dc=ldapserver,dc=co,dc=kr” -f directory.ldif
// 환경설정에 구조 추가 

$ vi ldapuser1 

dn:uid=ldapuser1,ou=People,dc=ldapserver,dc=co,dc=kr
objectClass:top
objectClass:account
objectClass:posixAccount
objectClass:shadowAccount
cn:ldapuser1
uid:ldapuser1
uidNumber:9999
gidNumber:100
homeDirectory:/home/ldapuser1
loginShell:/bin/bash
gecos:ldapuser1[Admin (at) ldapserver]
userPassword:{crypt}x
shadowLastChange:17058
shadowMin:0
shadowMax:99999
shadowWarning:7
// ldapuser1 이라는 계정을 생성.

$ ldapadd -x -W -D “cn=Manager,dc=ldapserver,dc=co,dc=kr” -f ldapuser1.ldif 
// 계정정보 환경설정에 추가.

$ ldapsearch -x cn=ldapuser1 -b dc=ldapserver,dc=co,dc=kr
// 유저가 반영되었는지 확인.

#### LDAP Client

$ yum install -y openldap-client nss-pa-ldapd
// ldap 관련 패키지 설치.

$ authconfig –enableldap –enableldapauth –ldapserver=192.168.111.100 –ldapbasedn=”dc=ldapserver,dc=co,dc=kr” –enablemkhomedir –update
// 엘답서버에서 환경설정 가져오기.

$ systemctl restart nslcd
// 재시작

$ getent passwd ldapsuser1
// ldpauser1 계정 가져오기

$ su - ldapuser1
// 로그인 성공.

#### NFS Server

$ yum install nfs-utils
// NFS 패키지 설치

$ systemctl restart nfs-server

$ mkdir /test
// 마운트 할 디렉터리 생성.

$ vi /etc/exports
/test *(rw)
// "/etc/exports" 에서 권한설정.

$ exportfs -r
// 수정한 내용을 exportfs 명령으로 반환.

$ firewall-cmd –permanent -add-service=nfs
$ firewall-cmd --reload
// 방화벽 설정해제.

# showmount -e
# exportfs -v
// NFS 설정 잘 되었는지 확인.

#### 다시 LDAP Client

# yum install -y autofs
# systemctl start autofs
// NFS 패키지 설치 및 시작.

$ vi /etc/auto.master
/home/guest /etc/auto.guest
// 위 문구를 추가.

$ vi /etc/auto.guest 
ldapuser1 - rw, vers=3 192.168.111.250:/home/guest/ldapuser1
// 위 문구를 추가하여 NFS서버의 디렉터리를 마운트.

$ systemctl restart autofs
$ su - ldapuser1
$ df -h 
// 재시작 하여 로그인 후 마운트 성공 확인.

#### DNS서버 구축.

$yum -y install bind
// DNS 패키지 설치.

$ vi /etc/named.conf

// named.conf는 named 프로세스의 옵션 설정 파일인데 아래와 같이 수정.
// listen-on port 53 { any; } : 서비스할 포트와 IP대역을 정의한다
// allow-quey { any; } : 질의를 허용할 IP대역을 정의한다.
// rexursion no : 상위 질의 허용여부를 설정하며, yes 사용시 보안상 취약 할 수 있다.

$ vi /etc/named.rfc1912.conf 

zone "a.com" IN {
        type master;
        file "a.com.zone";
        allow-update { none; };
};
// 질의한 도메인의 DNS 정보를 가지는 파일을 zone 파일이라고 하며, named.rfc1912.conf 파일에서 질의할 도메인 옵션 및 zone 파일의 경로를 설정한다.

$ vi /var/named/a.com.zone

$TTL 1D
@       IN SOA  @ rname.invalid. (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
; Name server
        IN      NS      11.11.11.11.
; Host name & Information
www             IN      A       22.22.22.22


// zone 파일은 기본적으로 /var/named 경로에 저장하도록 설정되어 있으며, 위에서 설정한 파일 이름과 동일하게 생성 해야함. 
 < 레코드 설명 >
NS(Name Server) : zone을 풀이할 수 있는 DNS 서버의 목록을 가지고 있습니다.
A(Host) : 정규화된 도메인 이름/호스트명을 IPv4에 연결합니다.
CNAME(Canonical NAME) : 실제 호스트명과 연결되는 별칭을 정의합니다.
MX(Mail Exchange) : 메일서버에 도달할 수 있는 라우팅 정보를 제공합니다.
SOA(Start Of Authority) : DNS 영역의 주 DNS 서버 정의 및 refresh, retry 간격 등을 정의합니다.
PTR(PoinTeR) : 역방향 조회에서 A레코드를 가리킬 때 사용합니다.

$ systemctl start named
// 데몬 재시작.

$ netstat -nlpt 
// DNS 포트 LISTEN 확인.

$ nslookup 
// a.com의 질의결과를 확인성공.


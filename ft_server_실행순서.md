# 실습 실행 

<br/>

## 나의 환경

<br/>

### - wsl2를 사용 -

<br/>

## 1. Docker 설치

<br/>

<https://hub.docker.com/editions/community/docker-ce-desktop-windows>

<br/>

- General 탭에서 Use the WSL2 based engine 옵션 클릭
- Resource -> WSL Integration 페이지로 이동해서 설정을 확인

<br/>

## 2. Debian(buster) 사용

<br/>

- docker hub에서 debian:buster이미지 가져오기

```shell

$sudo docker pull debian:buster

```

<br/>

## 3. docker로 debian(buster)환경 실행 및 접속(debian bash 접속)

<br/>

```shell

$sudo docker run -it --name con_debian -p 80:80 -p 443:443 debian:buster 

```

```shell

#docker

root@324234:/# apt-get update

root@324234:/# apt-get upgrade 

```

<br/>

### + 옵션

<br/>

-i -> interactive(입 출력)
-t -> tty(터미널)활성화
--name [컨테이너 이름] -> 컨테이너 이름 지정
-p 컨테이너포트번호:호스트포트번호 -> 컨테이너의 포트를 개방한 뒤 호스트포트와 연결한다.

<br/>

## 4. debian buster에서 Nginx, cURL 설치

<br/>

cURL은 서버와 통신할 수 있는 commend 명령어 툴 
(url을 가지고 할 수 있는 것들을 다 할 수 있다.)
cURL -> <https://shutcoding.tistory.com/23>

<br/>

```shell

#docker

root@324234:/# apt-get -y install nginx curl 

```

<br/>

### +옵션

<br/>

-y -> 설치하겠냐는 물음에 y를 눌러주는 것

<br/>

## 5. Nginx 서버 에러확인/실행/확인/중지

<br/>

```shell

#docker

root@324234:/# nginx -t

root@324234:/# service nginx start

root@324234:/# service nginx status

root@324234:/# curl localhost 

root@324234:/# service nginx stop

```

<br/>

### +

<br/>

localhost가 확인 안되면 127.0.0.1를 입력해서 확인하자
아니면 history에서 캐시등을 다 지운 다음에 해보자

<br/>

## 5. self-signed SSL 인증서 만들기

<br/>

- SSL 위에서 돌아가는 HTTP => HTTPS를 실행하기 위한 인증서
- openssl이용(개인 키(.key), 서면요청파일(.csr), 인증서파일(.crt))

<br/>

```shell

#docker
root@324234:/# apt-get -y install openssl

root@324234:/# openssl req -x509 -newkey rsa:4096 -nodes -sha256 -keyout ft_server.key -out ft_server.crt -days 365 -subj "/C=KR/ST=SEOUL/L=SEOUL/O=42/OU=gon/CN=localhost"
# => ft_server.key, localhost.crt 생성

root@324234:/# mkdir /etc/nginx/ssl

root@324234:/# mv ft_server.key /etc/nginx/ssl/

root@324234:/# mv ft_server.crt /etc/nginx/ssl/

root@324234:/# chmod 600 /etc/nginx/ssl/*
#=> 보안 및 인증 관련 파일은 권한을 root만 허용

```

<br/>

### +옵션

<br/>

req: 인증서 요청 및 인증성 생성 유틸
-x509: 인증서 요청 대신 자체 서명된 인증서를 출력(테스트 인증서 or 자체 서명 루트 CA생성하는 데 사용)
-newkey: 개인키 생성
rea:4096: SSL/TLS에 가장 많이 쓰이는 공개키 암호화 알고리즘
-nodes: 개인키를 암호화하지 않기 위한 옵션(없을 시 4자리 이상 암호 입력)
-sha256: SHA-256 기반 인증서 생성
-keyout [key 파일 이름]: 키 파일 생성(키 파일이름 지정)
-out [인증서 이름]: 인증서 생성(인증서 이름 지정)
days 365: 인증서의 유효기간
-subj"": subject를 입력하기 위한 옵션 ex) -subj"CN=localhost" => CN입력

<br/>

## nginx ssl 설정하기(https로 리다이렉션 설정)

<br/>

- nginx 설치 후 디렉토리 구조
<https://wiki.debian.org/Nginx/DirectoryStructure>

<br/>

```shell

#docker

root@324234:/# apt-get -y install vim

root@324234:/# cd /etc/nginx/sites-available 

root@324234:/# vim default (수정)

```

<br/>

### +수정내용

<br/>

- 301(permanently moved)는 이 페이지는 영구적으로 이동되었으므로 이 주소로 연결하시오
- $host, $request_uri는 nginx의 변수로
(<http://localhost/wordpress> 접속하면 host = localhost, request_uri = /wordpress)
- 80번 포트로 들어오는 것은 <https://domain>으로 이동

```shell

#docker

root@324234:/# sudo service nginx reload

```

<br/>

## 7. <http://localhost>가 안전하지 않은 사이트여서 나오지 않을 경우

<br/>

- 1. NET::ERR_CERT_REVOKED 오른쪽 빈칸아무대나 마우스 좌클릭 
- 2. 이상한 것 많이 보인다
- 3. 그냥 키보드로 thisisunsafe 친다. (치는 것이 보이지 않는다.)

<br/>

## 8. php-fpm 설치 / Nginx 설정

<br/>

```shell
#docker

root@324234:/# apt-get -y install php-fpm

root@324234:/# cd /etc/nginx/sites-available 

root@324234:/# vim default (수정)

root@324234:/# service php7.3-fpm start

root@324234:/# service php7.3-fpm status

```

<br/>

### +수정내용

<br/>

#PHP 추가 부분

<br/>

## 9. Nginx autoindex 설정

<br/>

```shell

#docker

root@324234:/# cd /etc/nginx/sites-available 

root@324234:/# vim default (수정)

```

<br/>

## 10. mariadb(mysql 진화판)/ php-mysql / php-mbstring 설치

<br/>

- debian9 버전 이후부터 mariadb지원 하기 때문
- php-mysql은 php에서 mysql(데이터베이스)에 접근하도록 해주는 모듈
- php-mbstring은 2바이트 확장 문자를 읽도록하는 확장자(경고 방지용)

<br/>

```shell

#docker

root@324234:/# apt-get -y install mariadb-server

root@324234:/# apt-get -y install php-mysql

root@324234:/# apt-get -y install php-mbstring

```

<br/>

## 11. Wordpress 설치

<br/>

- 웹서버로부터 직접받아와야한다(wget) + apt-get으로 설치 X

<br/>

```shell

#docker

root@324234:/# apt-get install wget 

root@324234:/# wget https://wordpress.org/latest.tar.gz

root@324234:/# tar -xvf latest.tar.gz 

```

<br/>

### +옵션

<br/>

-x: 묶음 해제
-v: 묶음/해제 과정을 화면에 표시
-f: 파일 이름을 지정

<br/>

```shell
#docker

root@324234:/# cd wordpress

root@324234:/# cp -rp wp-confing-sample.php wp-config.php 

root@324234:/# vim wp-config.php(수정)

```

<br/>

### + 수정

<br/>

database_name_here -> wordpress
username_here -> gpaeng
password_here -> gpaeng

<br/>

```shell

#docker

root@324234:/# cd ..

root@324234:/# rm -rf latest.tar.gz

root@324234:/# mv wordpress/ /var/www/html/

root@324234:/# chown www-data:www-data /var/www/html/wordpress

```

<br/>

### +명령어

<br/>

chown: 권한 설정으로 wordpress의 유저, 그룹을 www-data로 설정

<br/>

## 12. Database 설정

<br/>

- Database에 wordpress 추가

<br/>

```shell
#docker, mysql

root@324234:/# service mysql start

root@324234:/# mysql 

MariaDB [(none)]> CREATE DATABASE wordpress;

MariaDB [(none)]> SHOW DATABASES;

MariaDB [(none)]> CREATE USER 'gpaeng'@'localhost' IDENTIFIED BY 'gpaeng'; 
# + 아이디 gpaeng 비밀번호 gpaeng 생성

MariaDB [(none)]> GRANT ALL PRIVILEGES ON wordpress.* TO 'gpaeng'@'localhost' WITH GRANT OPTION;
# + @'localhost'는 로컬 접속만 허용(권한 부여), @'%'이면 외부접속 허용

MariaDB [(none)]> USE wordpress;

MariaDB [wordpress]> SHOW TABLES;

MariaDB [wordpress]> exit;

root@324234:/# service nginx reload

root@324234:/# service php7.3-fpm restart

```

<br/>

## 13. phpmyadmin 설치

<br/>

```shell
#docker

root@324234:/# cd ..

root@324234:/# wget <https://files.phpmyadmin.net/phpMyAdmin/5.0.2/phpMyAdmin-5.0.2-all-languages.tar.gz>

root@324234:/# tar -xvf phpMyAdmin-5.0.2-all-languages.tar.gz 

root@324234:/# rm -rf phpMyAdmin-5.0.2-all-languages.tar.gz

root@324234:/# mv phpMyAdmin-5.0.2-all-languages phpmyadmin

root@324234:/# cp -rp config.sample.inc.php config.inc.php

root@324234:/# vim config.inc.php(수정)

```

### +

<br/>

암호생성 사이트
<https://phpsolved.com/phpmyadmin-blowfish-secret-generator/?g=%5Binsert_php%5Decho%20$code;%5B/insert_php%5D>
들어가면 config.inc.php를 어떻게 바꿔야 하는지 나옵니다.

<br/>

```shell
#docker

root@324234:/# mv phpmyadmin /var/www/html/

root@324234:/# service nginx reload

```

<br/>

## 14. Database 설정

<br/>

```shell

#docker 

root@324234:/# service mysql start 

root@324234:/# mysql < var/www/html/phpmayadmin/sql/create_tables.sql

```

<br/>
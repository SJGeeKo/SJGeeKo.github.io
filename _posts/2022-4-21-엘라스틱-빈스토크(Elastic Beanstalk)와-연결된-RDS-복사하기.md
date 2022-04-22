---
layout: post
title: 엘라스틱 빈스토크(Elastic Beanstalk)와 연결된 RDS(MySQL) 로컬 컴퓨터로 복사하기
---

## 개요

1. Elastic Beanstalk 인스턴스에 SSH 연결
2. RDS 접속
3. DB 데이터를 담은 sql 파일 생성
4. 로컬 컴퓨터로 해당 파일 전송
5. 로컬 DB에 sql 파일 붙여넣기

기존에는 DataGrip의 GUI를 이용하여 복사 붙여넣기를 무식하게 했는데 서비스를 운영하다보니 그렇게 하기엔 부담스러울 정도로 DB 데이터가 많아져서 이참에 좀 더 똑똑한 방법을 알아봤다.

MySQL의 mysqldump 명령어를 사용하면 된다(명령어에 dump가 들어가서 데이터를 갖다 버리는 건 아닐까 걱정했는데 아니고 복사 맞음)

### Elastic Beanstalk 인스턴스에 SSH 연결

1. EC2 페이지 접속
2. 좌측 메뉴 Network & Security > Key Pairs
3. Create key pair 버튼 클릭
4. 이름 자유롭게 정해주고 type은 RSA, file format은 .pem으로 설정.
5. 잘 생성됐으면 Elastic Beanstalk 페이지 접속
6. 좌측 메뉴 Configuration > Security
7. 방금 생성한 key pair를 선택하고 저장해줌.
8. key pair 경로로 찾아들어가서 아래 명령어 실행
    - `chmod 400 [키페어 이름]`
    - `ssh -i "[키페어 이름]" ec2-user@[Public IPv4 DNS]`
        - 보통 EC2에 SSH 연결하는 경우 `ec2-user@[Public IPv4 DNS]`가 아니라 `root@[Public IPv4 DNS]`로 하곤 했었는데 ELB는 root로 접속하려고 하는 경우 아래와 같은 에러메시지가 출력된다.
            - Please login as the user "ec2-user" rather than the user "root".
            - 에러메시지 말대로 ec2-user를 사용하면 문제 없더라.
9. 접속 성공

### RDS 접속

- ec2 접속에 성공했으면 아래 명령어 입력
    - `sudo yum install mysql`
    - `mysql -u [User] -p -h [RDS 엔드포인트]`
        - 비밀번호 입력하라그러면 설정한 비밀번호 입력하면 됨
        - RDS 엔드포인트는 RDS 페이지에서 확인 가능

### DB 데이터를 담은 sql 파일 생성(dump)

- `mysqldump -u [User] -p -h [RDS 엔드포인트] [DB(또는 스키마) 이름] > [원하는 파일명].sql`
    - 비밀번호 입력하라그러면 설정한 비밀번호 입력하면 됨
    - 원하는 파일명으로 .sql 파일이 생성된다.
    - 이제 이 파일을 원하는 DB로 집어넣자

### 로컬 컴퓨터로 해당 파일 전송

- `scp -i [키파일] ec2-user@[Public IPv4]:[파일 저장된 경로]/[저장한 파일명].sql [저장 원하는 경로(로컬)] [복사할 파일명].sql`

### 로컬 DB에 sql 파일 붙여넣기

- `mysql -u [User] -p [DB(또는 스키마) 이름] < [복사한 파일명].sql`
    - 부등호 방향이 sql 파일 생성시와 반대임을 확인할 수 있다.
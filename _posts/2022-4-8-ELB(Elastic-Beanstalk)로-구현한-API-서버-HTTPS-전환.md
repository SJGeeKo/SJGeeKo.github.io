---
layout: post
title: ELB(Elastic Beanstalk)로 구현한 API 서버 HTTPS 전환
---
잘 쓰던 앱에 커뮤니티 기능(사진 업로드, 글쓰기 등을 포함)이 추가된 버전을 업데이트를 하려고 심사를 넣었더니 **유저 관련 데이터 전송이 안전한 방식으로 이뤄지지 않고 있다며 HTTPS 등의 통신방식으로 바꾼 뒤 재심사** 하라는 메일을 받음.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a6117a83-6833-4407-8e07-1640daad2b5b/Untitled.png)

백엔드 서버를 API 방식으로 구현하여 사용하고 있었는데 이 과정에서 HTTPS가 아닌 HTTP 통신을 하고 있었다.

나는 ‘HTTPS는 HTTP보다 안전한 것'이라는 수준의 아주 얕고 단순한 지식 밖에  없었으므로 처음부터 공부하기로 함.

## 배움의 흐름

- HTTPS → SSL → ROUTE53 → 서브도메인 → ACM → 로드밸런서
- HTTPS와 SSL 공부 관련 기록은 [따로 정리]([https://sjgeeko.github.io/https와-ssl-인증서-개념-정리/](https://sjgeeko.github.io/https%EC%99%80-ssl-%EC%9D%B8%EC%A6%9D%EC%84%9C-%EA%B0%9C%EB%85%90-%EC%A0%95%EB%A6%AC/))함

## ACM(AWS Certificate Manager)란?

- SSL/TLS 인증서 구매, 업로드 및 갱신 자동화
- 인증서 무료 - 애플리케이션을 실행하기 위해 생성한 AWS 리소스에 대한 비용만 지불하면 됨
- HTTPS 통신을 위해 인증서는 필수이므로 우선 이 서비스를 이용하여 인증서를 발급받아야 함

다음으로 발급받은 인증서를 서버에 등록하는 방식을 찾아봤음. 내 API 서버는 바로 EC2에 올린 것이 아니라 ELB(Elastic Beanstalk)를 통해 올려져 있는 상태였기 때문에 왠지 특별한 방법이 있을 것 같아 관련 문서를 찾아봤음 → [이 문서]([https://kokohapps.tistory.com/entry/Elastic-Beanstalk-로-서버운영하기-2-도메인-연결-HTTPS-연결](https://kokohapps.tistory.com/entry/Elastic-Beanstalk-%EB%A1%9C-%EC%84%9C%EB%B2%84%EC%9A%B4%EC%98%81%ED%95%98%EA%B8%B0-2-%EB%8F%84%EB%A9%94%EC%9D%B8-%EC%97%B0%EA%B2%B0-HTTPS-%EC%97%B0%EA%B2%B0))의 도움을 받았다.

<aside>
💡 글에 포함된 스크린샷이 현재의 AWS UI와 약간 다르긴 한데 큰 어려움 없이 따라할 수 있었다.

</aside>

해당 문서의 ‘첫 부분에 Route 53에 내 도메인이 등록되어 있어야 한다’는 언급이 있는데 나는 Route 53이 뭔지조차 몰랐다.

또, Load Balancer에 대한 언급도 있는데 이 역시 처음 들어보는 단어라 기왕에 함께 찾아봤다.

## ROUTE 53이란?

- 아마존이 제공하는 도메인 네임 시스템.
    - 기존에 내가 사용하던 국내 서비스인 가비아와 사용방법 자체에서는 큰 차이 없는듯.
    - 다른 서비스(가비아 등)를 이용하여 구매한 도메인이 있다면 해당 도메인을 이쪽에서 사용하는 것도 가능.
- AWS의 다른 서비스와의 연동이 잘 되기 때문에 특별한 이유가 없다면 ROUTE 53을 쓰는 것이 적절해 보임.

## 로드밸런서란?

- 트래픽이 많아지면 하나의 서버로 감당하기 어려움 → 서버 증설 필요
- 단순히 서버 개수를 늘린다고 이 문제가 해결되지 않음 → 트래픽을 각 서버에 잘 분산시켜줘야 함.
    - 직원이 많아도 상사가 일을 효율적으로 분배하지 못하면 효율 저하되는 것과 비슷.
- 이처럼 부하(Load)를 여러 서버에 분산(Balance)시켜주는 장치 또는 기술이 로드밸런서.

이와 같은 이해를 바탕으로 위 블로그 글을 따라하니 잘 됐음. 사실 따라하는 것 자체는 얼마 걸리지 않았으나 맥락도 모르고 무작정 따라하는 것이 찜찜해서 공부하다보니 하루가 꼬박 걸렸다.

---
layout: post
title: s3 정적 호스팅 NoSuchKey 에러 해결법
---

# 문제상황

- vue로 만든 웹사이트를 aws s3를 이용해서 정적호스팅함.
- 해당 웹사이트에서 새로고침을 하면 NoSuchKey 라는 메시지와 함께 에러 발생.

# 해결법

- S3 정적 호스팅 properties에 있는 'index document'와 'error document'입력란 모두에다가 'index.html' 입력하면 해결됨.
- 참고: https://github.com/kriasoft/react-firebase-starter/issues/141#issuecomment-285772833
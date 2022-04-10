---
layout: post
title: \[Flutter\] iOS simulator 보이지 않는 문제 해결방법
---

## 문제상황
- Android Studio에서 잘 보이던 iOS simulator 항목이 어느 순간 갑자기 안보이기 시작함.

## 해결법
1. Xcode 최신 버전이 설치되어 있는지 확인
    - 없다면 설치
2. 터미널에 `sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer` 실행
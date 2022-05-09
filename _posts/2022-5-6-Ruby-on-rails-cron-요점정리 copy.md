---
layout: post
title: Ruby on Rails whenever gem(cron) 요점정리
---

## cron이란?

- UNIX 기반 운영체제(linux, ubuntu)에서 사용되는 자동화 툴
- 간단한 명령어를 이용하여 원하는 주기, 혹은 원하는 시점에 원하는 작업을 수행할 수 있도록 해준다.
- crontab이라는 명령어를 사용함.

## crontab 사용법

- `crontab -e`: 자동 실행 명령어를 작성(혹은 수정)할 수 있음.
- `crontab -l`: 작성한 자동 실행 명령어들을 보여줌. 당연히 처음엔 빈칸임.

## cron 작성 예시

- `30 21 * * * /Users/sjko/.pyenv/shims/python /Users/sjko/Desktop/code/word-counter/cron-test.py >> /Users/sjko/Desktop/cron_log.log 2>&1`
    - 매일 9시 30분에 python으로 cron-test.py를 실행시키고 cron_log.log 파일에 로그를 저장한다.
- cron 표현식에 대한 다양한 예시는 [이 페이지](https://docs.oracle.com/cd/E12058_01/doc/doc.1014/e12030/cron_expressions.htm)를 참고하자.

## Ruby on rails와 cron

- cron은 앞서 말했듯 UNIX 기반 운영체제에 기본제공되는 툴이라 Ruby on rails와 별개로 존재함.
- whenever gem은 단순히 ruby on rails로 작성한 코드를 cron 명령어로 손쉽게 변환해주는 역할을 할 뿐임.

## Whenever gem 사용법

- whenever gem 설치 - `gem install whenever`(혹은 Gemfile에 작성 후 `bundle`)
- 터미널에서 rails 프로젝트 루트경로로 이동
- `wheneverize .` - 이 명령어 실행하면 config 폴더 하위에 schedule.rb 파일 만들어짐. 여기에 자동화하고 싶은 명령어들을 작성한다.
    ```ruby
    set :output, "log/cron_log.log"
    
    ENV.each { |k, v| env(k, v) }

    env :PATH, ENV['PATH']

    every 1.day, at: '9:30 pm' do
        runner "LectureDate.sendOneDayNotice()"
    end
    ```
    - 사실, 이 파일에 코드를 어떻게 작성하면 되는지에 대해 schedule.rb에 주석으로 설명이 잘 적혀있기 때문에 잘 읽고 필요에 맞게 수정하면 된다.
- 작성할 거 다 작성했으면 터미널에 `whenever -i` 실행.
    - 이는 추후 schedule.rb 파일이 수정되는 경우 매번 실행해줘야 함.
    - 이 명령어가 schedule.rb 파일의 코드들을 cron 명령어들로 변경해주는 역할을 하기 때문임.
- 별 문제가 없다면 [write] crontab file updated 라는 메시지가 표시된다.
    - 보다 확실히 확인하고 싶다면 위에서 확인한 `crontab -l` 명령어를 이용하자.

## AWS EC2 등 서버에서 cron을 실행하고 싶다면?

- 위와 동일한 작업을 서버에서 한 번 더 해주면 됨.
- github 등 원격 저장소에 작업한 파일 업로드 > 서버에서 내려받음
- schedule.rb 파일은 이미 준비되어 있을 것이므로 `whenever -i` 실행
- `crontab -l`로 잘 반영됐는지 확인
- 가만히 놔두면 내가 설정한 주기(혹은 시점)로 작동함을 확인할 수 있다.
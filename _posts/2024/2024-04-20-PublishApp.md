---
layer: single
title: "안드로이드 모바일 앱 출시하기"
categories: Unity
tag: [C#, Unity, Game, Project]
toc: true
author_profile: false
sidebar: 
    nav: "docs"
---


# 앱 출시해보기

- 플레이스토어에 내가 만든 앱을 출시하려고 할 때 어떻게하면 되는지 찾아보고 정리한 글입니다.
- 크게 어렵지 않으나 몇가지 준비해야할 것이 있습니다.

## 구글 개발자 계정만들기

- 플레이스토어 앱 등록을 하기 위해서는 구글 개발자 계정이 필요합니다.
- [구글 개발자 콘솔사이트](https://play.google.com/console/developers) 에서 계정을 등록한 뒤 25$(한화 약 34000원)를 지불하면 됩니다.
- 결제는 최초 1회만 하면되며, 이후에는 자유롭게 앱 등록이 가능합니다.
- [구글 개발자 계정 등록 참조링크](https://wp.swing2app.co.kr/knowledgebase/google-developer/)


### 개발자 계정설정

- 계정을 만들고나면 개발자 계정설정을 해야합니다.
- 주소지 증빙서류를 포함한 본인인증이 필요하므로, 필요한 서류를 잘 확인하여 업로드 하시면됩니다.

![image](/images/2024/2024-04-20/capture_1.PNG)

- 서류까지 제출하고나면, 신원확인이 완료되었을 때 연락을 준다고 합니다.

![image](/images/2024/2024-04-20/capture_2.PNG)

- 1~2일 뒤에 메일로 신원확인 실패/성공여부를 알려줍니다.

## 구글 플레이용 빌드

- 프로젝트용 Keystore를 새로 생성하여 따로 보관해줍니다.
- 빌드할때 모바일 빌드와 똑같이 빌드를하면되는데, 아래 옵션을 선택한 뒤 빌드해주어야합니다.

![image](/images/2024/2024-04-20/capture_5.PNG)

- 성공적으로 빌드되면 폴더2개와 aab파일 하나가 생성된것을 볼 수 있습니다.

![image](/images/2024/2024-04-20/capture_6.PNG)

## 앱 등록하기

- 신원확인 및 본인인증을 마친 뒤 앱 출시를 할 수 있습니다.

![image](/images/2024/2024-04-20/capture_3.PNG)

- 앱 만들기 버튼을 누르게되면 앱 출시에 필요한 정보를 요구합니다.

![image](/images/2024/2024-04-20/capture_4.PNG)


### 앱 번들 등록하기

- 앱 출시를 위해 출시관리 - 앱 서명에 들어갑니다.
- 앱 서명에서 Java Keystore의 키 내보내기 및 업로드를 선택합니다.

![image](/images/2024/2024-04-20/capture_7.PNG)


- 암호화 공개 키와 PEPK 도구를 다운로드 한 뒤, 편하게 하기위해 빌드할 때 저장해둔 keystore의 위치에 두 파일을 넣어줍니다.

- 3.의 java -jar pepk.jar --keystore=foo.keystore --alias=foo --output=output.zip --include-cert --rsa-aes-encryption --encryption-key-path=/path/to/encryption_public_key.pem 를 메모장에 복사붙여넣기 합니다.

- 이제 내 경로, alias, 파일이름에 맞춰 수정해주어야합니다.

- java -jar /내경로/pepk.jar --keystore=내keystore명.keystore --alias=내alias --output=출력파일이름(아무거나).zip --include-cert --rsa-aes-encryption --encryption-key-path=/내경로/encryption_public_key.pem 과 같이 바꿔줍니다.

[앱 번들 등록하기](https://j2su0218.tistory.com/1321)

- 명령어창(cmd)을 열어 명령어를 넣어줍니다.
- 성공적으로 완료되었다면 새로운 압축파일이 생겨나고, 이것을 업로드해주면됩니다.

![image](/images/2024/2024-04-20/capture_8.PNG)



### 테스트 설정

- 앱 출시를 위해서 테스트 설정을 해야합니다.
- 테스트 종류에는 공개테스트, 비공개테스트, 내부테스트가 있습니다.
- 공개테스트는 Google Play에서 테스터에게 제공됩니다.
- 비공개테스트는 내가 지정한 테스터만 테스트가 가능합니다.
- 내부테스트는 이메일을 설정하여 테스터를 관리합니다.

![image](/images/2024/2024-04-20/capture_9.PNG)

- 이중에 내부테스트를 선택하였습니다.

- 먼저 테스트에 사용할 AppBundle를 구성합니다. 아까 등록해둔 앱 번들을 추가합니다.

![image](/images/2024/2024-04-20/capture_10.PNG)

![image](/images/2024/2024-04-20/capture_11.PNG)

- 내부 테스트 트랙설정을 완료해야하는데, 먼저 새 버전 만들기를 선택합니다.

![image](/images/2024/2024-04-20/capture_12.PNG)

- 출시명, 출시 노트를 입력하고 저장합니다.
- 이제, 테스터 설정을 해주어야합니다.

![image](/images/2024/2024-04-20/capture_13.PNG)

- 이메일을 추가하는 방식으로 테스터를 설정할 수 있습니다. 여기에 내 이메일을 넣어줬습니다.

![image](/images/2024/2024-04-20/capture_14.PNG)





---
title: "Shell Scripts 사용하기"
date: 2019-06-02T18:35:16+09:00
draft: false
categories: [
  "linux",
  "programming",
]
tags: [
  "shell",
]
---

# Intro
이번 포스트에서는 쉘 스크립트에 대해 알아보고, 사용 방법을 다룬다. 전반적으로 "이것이 우분투 리눅스다" 책을 참고하였고, 공부한 내용을 정리하였다.

***

# 1. 쉘(shell) 이란?

쉘은 사용자가 입력한 명령을 해석해서 커널에 전달하거나, 커널의 처리 결과를 사용자에게 전달하는 역할을 한다.

### 1-1. 쉘 명령문 형식

쉘 명령문의 형식은 다음과 같다.

```
{프롬프트} {명령어} {옵션} {인자}

ex) # cd ~
ex) # ls -al
ex) # rm -rf testdir
ex) # cat test.txt
```

### 1-2. 쉘 환경변수

쉘은 다양한 환경변수 값을 설정할 수 있는데, 리눅스의 주요 환경변수 목록은 아래와 같다.

환경변수 | 설명
--- | ---
HOME | 현재 사용자의 홈디렉토리
PATH | 실행 파일을 찾는 디렉토리 경로
LANG | 기본 지원 언어
PWD | 사용자의 현재 작업 디렉토리
TERM | 로그인 터미널 타입
SHELL | 로그인시 사용하는 쉘
USER | 현재 사용자의 이름
DISPLAY | X 디스플레이 이름
HOSTNAME | 호스트 이름
USERNAME | 현재 사용자 이름
OSTYPE | 운영체제 타입


현재 설정된 환경 변수는 `echo ${환경변수 이름}` 명령어를 통해 확인할 수 있다.
```
ex) echo $HOME
ex) echo $HOSTNAME
```

환경변수 설정 값을 변경하려면 `export {환경변수}={값}` 명령어를 실행하면 된다. 그 외의 환경변수는 `printenv` 명령어로 확인할 수 있다. 단, 일부 환경변수는 `printenv` 명령을 실행해도 출력되지 않는다.

***

# 2. 쉘 스크립트 사용해보기

쉘 스크립트도 일반적인 프로그래밍 언어와 비슷하게 변수, 반복문, 제어문 등을 사용할 수 있다.

### 2-1. 쉘 스크립트 작성하기

아래와 같이 간단한 쉘 스크립트를 작성하고 script.sh으로 저장해보자.

```
#!/bin/sh
echo "User Name: " $USER
echo "User Home Directory: " $HOME
exit 0
```

위의 코드를 간단히 설명하면 다음과 같다.
- 1행: bash를 사용하겠다는 의미로, 쉘 스크립트 작성시 첫 행에 반드시 입력해야한다.
- 2, 3행: echo 명령은 화면에 출력하는 명령이며, 먼저 문자를 출력하고 ${환경변수}에 해당하는 값을 출력한다.
- 4행: 종료 코드를 반환하며, 0은 쉘 스크립트 실행 성공을 의미한다.

### 2-2. 쉘 스크립트 실행하기

쉘 스크립트를 실행하는 방법에는 크게 두 가지가 있다.

*1) sh 명령으로 실행*

```
sh {실행파일 이름}
ex) sh script.sh

[실행 결과]
User Name:  rin_gu
User Home Directory:  /home/rin_gu
```

*2) 파일 권한을 실행 가능하게 변경하고 실행*

```
# chmod +x {실행파일 이름}
# ./{실행파일 경로}

ex)
# chmod +x script.sh
# ./script.sh

[실행 결과]
User Name:  rin_gu
User Home Directory:  /home/rin_gu
```

### 3. 변수

쉘 스크립트 변수 사용시 주의해야할 특징은 다음과 같다.
- 변수를 사용하기 전에 미리 선언하지 않으며, 처음 변수에 값이 할당되면 자동으로 변수가 생성된다.
- 변수에 넣는 모든 값을 '문자열'로 취급한다.
- 변수 이름의 대소문자를 구분한다.
- 변수를 대입할 때 '=' 좌우에 공백이 없어야 한다.

![](/images/20190602_shell/shell.png)  

### 3-1. 변수의 입출력

'$' 라는 문자가 들어간 글자를 출력하려면 ''로 묶어주거나 앞에 '\\'를 붙여줘야 한다.

```
#!/bin/sh
var="Hi Shell"
echo $var

echo '$var'
echo \$var
exit 0

[실행 결과]
Hi Shell
$var
$var
```

변수의 입력과 출력은 다음과 같이 사용할 수 있다.
```
#!/bin/sh
echo "값을 입력하세요: "
read var

echo "입력된 값은 " $var "입니다."
exit 0

[실행 결과]
값을 입력하세요:
shell script study
입력된 값은  shell script study 입니다.
```

### 3-2. 숫자 계산
변수에 넣은 값은 모두 문자열이 되므로, shell script에서 연산을 하기 위해서는 expr 키워드를 사용해야한다.
shell script에서 숫자 계산을 하기 위한 규칙은 다음과 같다.
- 수식과 함께 역따옴표로 묶어줘야 한다.
- 수식에 괄호를 사용하기 위해서는 그 앞에 역슬래시를 붙여줘야 한다.
- 곱하기 기호를 사용할 때도 그 앞에 역슬래시를 붙여줘야 한다.

```
#!/bin/sh
n1=10
n2=$n1+20
echo $n2

n3=`expr $n1 + 20`
echo $n3

n4=`expr \( $n1 + 20 \) / 10 \* 2`
echo $n4

exit 0

[실행 결과]
10+20
30 // ※ 각 단어를 띄어쓰기해야하는 것에 주의
6
```

### 3-3. 파라미터 변수
파라미터 변수는 $0, $1, $2 등의 형태를 가지며, 아래와 같이 매칭된다.

명령어 | 파라미터 1 | 파라미터 2 | 파라미터 n...
--- | --- | --- | ---
$0 | $1 | $2 | $n...

예를 들면 다음과 같다.

./script | 1 | 2
--- | --- | ---
$0 | $1 | $2

```
#!/bin/sh
echo "실행파일 이름: <$0>"
echo "첫 번째 파라미터: <$1>."
echo "두 번째 파라미터: <$2>."
echo "전체 파라미터는 <$*>이다."
exit 0

[실행 결과]
실행파일 이름: <./script.sh>
첫 번째 파라미터: <1>.
두 번째 파라미터: <2>.
전체 파라미터는 <1 2>이다.
```

# 4. if문

if문의 기본 사용법과 조건 연산자를 알아보자.

### 4-1. if문 기본 문법

if문의 기본 문법은 다음과 같다.

```
if [ 조건 ]
then
  참일 경우 실행
fi
```

이때 주의할 점은 [ 조건 ] 에서 각 단어에 모두 공백이 있어야 한다.

```
#!/bin/sh
if [ "rin_gu" = "rin_gu" ]
then
  echo "참 입니다."
fi
exit 0

[실행 결과]
참 입니다.
```

### 4-2. if-else문

if-else문의 기본 문법은 다음과 같다.

```
if [ 조건 ]
then
  참일 경우 실행
else문
  거짓인 경우 실행
fi
```

```
#!/bin/sh
if [ "rin_gu" = "shell script" ]
then
  echo "참 입니다."
else
  echo "거짓 입니다."
fi
exit 0

[실행 결과]
거짓 입니다.
```

### 4-3. 조건문에 사용되는 비교 연산자

조건문에 사용디는 비교 연산자에는 문자열 비교 연산자와 산술 비교 연산자가 있다.

**문자열 비교 연산자**

문자열 비교 | 내용 | 예시
--- | --- | ---
= | 같으면 참 | "문자열" = "문자열"
!= | 같지 않으면 참 | "문자열" != "문자열"
-n | 문자열이 NULL 값이 아니면 참 | -n "문자열"
-z | 문자열이 NULL 값이면 참 | -z "문자열"

**산술 비교 연산자**

산술 비교 | 내용 | 예시
--- | --- | ---
-eq | 두 수식이 같으면 참 | 수식 -eq 수식
-ne | 두 수식이 같지 않으면 참 | 수식 -ne 수식
-gt | 왼쪽 수식이 크면 참 | 수식 -gt 수식
-ge | 왼쪽 수식이 크거나 같으면 참 | 수식 -ge 수식
-lt | 왼쪽 수식이 작으면 참 | 수식 -lt 수식
-le | 왼쪽 수식이 작거나 같으면 참 | 수식 -le 수식
! | 수식이 거짓이라면 참 | !수식

```
#!/bin/sh
if [ 1024 -eq 2024 ]
then
  echo "1024과 2024는 같다."
else
  echo "1024과 2024는 다르다."
fi
exit 0

[실행 결과]
1024과 2024는 다르다.
```

### 4-4. 조건문에 사용되는 파일과 관련된 연산자

if문에서 파일을 처리하는 조건 연산자는 다음과 같다.

파일 조건 | 내용 | 예시
--- | ---
-d | 파일이 디렉토리면 참 | -d 파일명
-e | 파일이 존재하면 참 | -e 파일명
-f | 파일이 일반 파일이면 참 | -f 파일명
-g | 파일에 set-group-id가 설정되면 참 | -g 파일명
-r | 파일이 읽기 가능하면 참 | -r 파일명
-s | 파일 크기가 0이 아니면 참 | -s 파일명
-u | 파일에 set-user-id가 설정되면 참 | -u 파일명
-w | 파일이 쓰기 가능하면 참 | -w 파일명
-x | 파일이 실행 가능하면 참 | -x 파일명

***

# 5. case문

case문의 기본 사용법을 알아보자

### 5-1. case문 기본 문법

case문의 기본 문법은 다음과 같다.

```
case [ 변수 ] in
  변수값1)
    변수와 변수값1이 일치할 경우 실행
  변수값2)
    변수와 변수값2가 일치할 경우 실행
  ...
esac
```

### 5-2. case 문 사용해보기
(작성중)
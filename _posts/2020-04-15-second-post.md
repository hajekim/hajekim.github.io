---
title: "01 머신러닝 해보기: 시작하기 위한 준비물"
date: 2020-04-15 18:00:00 +0900
categories: jekyll 머신러닝
---
# 01 머신러닝 해보기 : 시작하기 위한 준비물

머신러닝(Machine Learning)
요즘 자주 듣고 있는 단어입니다.

기존에 업무를 하며 자주 듣던 단어는 오라클 미들웨어(Oracle Middleware), 웹 애플리케이션(Web Application)이었고, 퍼블릭 클라우드(Public Cloud)가 각광 받기 시작하며 아마존 웹 서비스(Amazon Web Services), 마이크로소프트 애저(Microsoft Azure), 구글 클라우드(Google Cloud)입니다.
그리고 지금은 머신러닝, 딥러닝(Deep Learning)인 듯 합니다.

빠르게 변화하는 기술 트렌드에 따라가기 위해 이 글을 작성합니다.

솔루션 엔지니어의 머신러닝 해보기입니다.
저와 비슷한 환경에 처하시거나 경험이 있으신 분들께 재미와 도움이 되었으면 좋겠습니다.



## 머신러닝

머신러닝에 대해서 역사를 따라가면 아서 사무엘과 톰 미쉘을 만날 수 있습니다.

1959년, IBM의 아서 사무엘(Arthur Lee Samuel)이 '머신러닝'이라는 단어를 대중화 하였습니다.
(https://en.wikipedia.org/wiki/Arthur_Samuel)

머신러닝 및 신경 과학 발전에 선구자적 공헌과 리더십의 톰 미첼 교수(Tom Mitchel, 카네기 멜론 대학)가 있습니다.

아서 사무엘이 정의한 머신러닝
```
"컴퓨터가 명시적으로 프로그래밍 없이, 학습할 수 있도록 능력을 제공 하는 연구 분야
The field of study that gives computers the ability to learn without being explicitly programmed"
```



톰 미첼 교수가 정의한 머신러닝

```
만약 작업(T)에 대해 성능 지표(P)에 의해 측정된 성능이 경험(E)에 따라 향상되었다면, 그 컴퓨터 프로그램은 작업(T)와 성능 지표(P)에 대해 경험(E)으로부터 학습했다고 말할 수 있다.
```



## 준비물

### Python

머신러닝을 하기 위해서는 주로 Python과 R이 사용됩니다. 물론 Java 같은 다른 언어로도 사용이 가능합니다만, 사용자 생태계가 잘되어 있는 것이 Python이라고 생각하므로, 저는 Python을 사용해서 작성합니다.

Python을 사용하기 위해서는 설치해야 합니다.
방법이 두 가지가 있습니다.

1. Python 패키지 설치
2. Anaconda 설치

Anaconda는 Python을 포함하여 자주 사용되는 패키지가 함께 제공되는 툴입니다.

솔루션 엔지니어의 경험으로는 '통합'이라는 말은 못미더우니 저는 1번 방법으로, Python을 설치하겠습니다.
Anaconda 설치 하실 분들께서는 https://www.anaconda.com/distribution/ 에서 설치하여 주세요.

#### Python 패키지 설치

제 환경은 macOS Catalina와 High Sierra입니다.
macOS의 패키지 관리자인 Homebrew를 이용해서 파이썬 패키지를 설치하고 사용할 예정입니다.
Homebrew를 통해 최신 버전의 Python을 사용할 수 있습니다.

//작성자 생각// Homebrew =  yum, apt-get on macOS

```
brew install python
```

저는 3.7.7이 설치 되었네요.

```
brew list python
/usr/local/Cellar/python/3.7.7/bin/2to3
/usr/local/Cellar/python/3.7.7/bin/2to3-3.7
/usr/local/Cellar/python/3.7.7/bin/easy_install-3.7
/usr/local/Cellar/python/3.7.7/bin/idle3
/usr/local/Cellar/python/3.7.7/bin/idle3.7
/usr/local/Cellar/python/3.7.7/bin/pip3
/usr/local/Cellar/python/3.7.7/bin/pip3.7
/usr/local/Cellar/python/3.7.7/bin/pydoc3
/usr/local/Cellar/python/3.7.7/bin/pydoc3.7
/usr/local/Cellar/python/3.7.7/bin/python3
/usr/local/Cellar/python/3.7.7/bin/python3-config
/usr/local/Cellar/python/3.7.7/bin/python3.7
/usr/local/Cellar/python/3.7.7/bin/python3.7-config
/usr/local/Cellar/python/3.7.7/bin/python3.7m
/usr/local/Cellar/python/3.7.7/bin/python3.7m-config
/usr/local/Cellar/python/3.7.7/bin/pyvenv
/usr/local/Cellar/python/3.7.7/bin/pyvenv-3.7
/usr/local/Cellar/python/3.7.7/bin/wheel3
/usr/local/Cellar/python/3.7.7/Frameworks/Python.framework/ (2827 files)
/usr/local/Cellar/python/3.7.7/IDLE 3.app/Contents/ (8 files)
/usr/local/Cellar/python/3.7.7/lib/pkgconfig/ (3 files)
/usr/local/Cellar/python/3.7.7/libexec/bin/ (7 files)
/usr/local/Cellar/python/3.7.7/libexec/pip/ (794 files)
/usr/local/Cellar/python/3.7.7/libexec/setuptools/ (365 files)
/usr/local/Cellar/python/3.7.7/libexec/wheel/ (60 files)
/usr/local/Cellar/python/3.7.7/Python Launcher 3.app/Contents/ (16 files)
/usr/local/Cellar/python/3.7.7/share/man/ (2 files)
```

만약, [Homebrew][homebrew]가 설치가 되지 않았다면 설치하고 Python을 설치 해주세요.

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```

Python 설치 고생하셨습니다.
이제 조금 쉬어 갈게요.

[homebrew]: https://brew.sh/
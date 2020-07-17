---
title: "Kubeflow 만들기"
date: 2020-07-04 18:00:00 +0900
categories: 머신러닝 MachineLearning Kubernetes 쿠버네티스트 Kubeflow
---
> **주의**  
> 빠르게 변화하는 기술 트렌드를 힘겨운 분들게 작은 도움이 되고자 이 글을 작성합니다.  
> 솔루션 엔지니어의 머신러닝 해보기입니다.  
> 저와 비슷한 경험 혹은 업무에 계신 분들께 도움과 재미가 되었으면 좋겠습니다.

'**머신러닝**'환경을 컨테이너로 만들어 봅시다.

사용 환경 및 설치 해야 할 아이들

- CentOS 7
- Kubernetes
- Kubeflow

---

## Kubeflow 소개

Kubeflow는 **클라우드 네이티브 머신러닝 플랫폼**입니다. 있어 보이게 얘기했지만, 클라우드 플랫폼에서 쓰기 쉬운 머신러닝 플랫폼입니다. 구글의 머신러닝 파이프라인을 기반으로 만들어졌습니다.  
Kubernetes를 통해서 실행되기 때문에, 자원 관리와 확장성 등의 장점을 그대로 사용할 수 있습니다.



## Install Kubernetes on CentOS 7

CentOS7
> 컴퓨터가 명시적으로 프로그래밍 없이, 학습할 수 있도록 능력을 제공 하는 연구 분야
>
> The field of study that gives computers the ability to learn without being explicitly programmed



톰 미첼 교수님께서 정의한 머신러닝

> 만약 작업(T)에 대해 성능 지표(P)에 의해 측정된 성능이 경험(E)에 따라 향상되었다면, 그 컴퓨터 프로그램은 작업(T)와 성능 지표(P)에 대해 경험(E)으로부터 **학습**했다고 말할 수 있다.
>
> A computer program is said to **learn** from experience *E* with respect to some class of tasks *T* and performance measure *P*, if its performance at tasks in *T*, as measured by *P*, improves with experience *E*.

정리하면, 머신러닝은 **가지고 있는 데이터의 패턴을 학습하여 궁금한 데이터를 예측하는 기술**이라 볼 수 있을 듯 합니다.

---

## 시작하기 위한 준비물

### Python

머신러닝을 하기 위해서는 주로 **Python**과 **R**이 사용됩니다. 물론 Java 같은 다른 언어로도 사용이 가능합니다만, 사용자 생태계가 잘되어 있는 것이 Python이라고 생각하므로, 저는 Python을 사용해서 작성합니다.

Python을 사용하기 위해서는 설치해야 합니다.  
방법이 두 가지가 있습니다.

1. Python 패키지 설치
2. Anaconda 설치

Anaconda는 Python을 포함하여 자주 사용되는 패키지가 함께 제공되는 툴입니다.

솔루션 엔지니어의 경험으로는 '통합'이라는 말은 못미더우니 저는 1번 방법으로, Python을 설치하겠습니다.

Hoxy, Homebrew가 불편하시거나, Windows를 사용 중이시라면 Python 인스톨 파일 다운로드해서 설치할 수 있어요.
[https://www.python.org/downloads/](https://www.python.org/downloads/)

Anaconda 설치를 원하신다면 [여기](https://www.anaconda.com/distribution/) 에 가서 다운로드 후 설치 해주세요.

#### Python 패키지 설치

제 환경은 macOS Catalina와 High Sierra입니다.  
macOS의 패키지 관리자인 Homebrew를 이용해서 파이썬 패키지를 설치하고 사용할 예정입니다.  
Homebrew를 통해 최신 버전의 Python을 사용할 수 있습니다.

> **작성자 생각**
> Homebrew는  macOS판 yum, apt-get

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
[^1]:Wikipedia Arthur Samuel, https://en.wikipedia.org/wiki/Arthur_Samuel/

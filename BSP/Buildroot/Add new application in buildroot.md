buildroot는 어쩄든 크로스컴파일러 환경 구축을 쉽게 하도록 도와주는 오픈소스이고, 이는 간단하게 rootfs를 생성 할 수 있다는 것을 의미한다

사용자 정의 앱인 custom application을 추가 할 떄가 있다. 방식은 2개이다

1. 기존 buildroot 패키지 시스템 활용하여 새로운 패키지 정의하는 방법
2. 외부 패키지 dir 만들어 app을 추가하는 방식

필수 파일들은 다음과 같다

⇒ buildroot 에서 패키지 구성 필수

- [Config.in](http://Config.in) ⇒ buildroot의 메뉴 구성에서 새로운 패키지를 선택할 수 있도록 하는 설정 파일

1. 다음과 같은 코드를 추가하면 내가 만든 디렉토리를 함께 실행 할 수 있다
    1. Config.in에서 추가해야한다

```c
config my_package 
	bool "Custom App"
	help
		사용자 정의 앱을 빌드한다.
```

1. custom_app.mk 파일을 작성해야한다

```c
touch custom_app.mk
vim custom_app.mk

CUSTOM_APP_VERSION = 1.0
CUSTOM_APP_SITE = $(TOPDIR)/package/custom_app/src 
CUSTOM_APP_SITE_METHOD = local

define CUSTOM_APP_BUILD_CMDS
	$(MAKE) -C $(@D)
endef 

define CUSTOM_APP_INSTALL_TARGET_CMDS
	$(INSTALL) -D -m 0755 $(@D)/custom_app $(TARGET_DIR)/usr/bin/custom_app
endef

$(eval $(generic-package))
```

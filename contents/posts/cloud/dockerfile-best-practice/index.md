---
title: 도커 파일 베스트 프랙티스
date: "2024-11-01"
tags:
  - Docker
  - Best Practice
  - 엘라
series: Docker
previewImage: docker_best.png
---

`베스트 프랙티스`란 목표 달성을 위해 가장 효과적인 해결책과 방식을 말합니다.
이번 글에서는 `도커 파일`을 작성할 때 베스트 프랙티스가 무엇이고 모행 프로젝트에서 작성한 도커 파일을 보면서 베스트 프랙티스와 비교해보고 정리하고자 합니다.

---

## 도커 파일의 베스트 프랙티스란?
도커 파일 베스트 프랙티스는 크게 `빌드 시간`, `이미지 크기`, `유지 보수성`, `재현성` 4가지로 볼 수 있습니다.

> 도커 파일 베스트 프랙티스는 [docker 공식 포스팅](https://www.docker.com/blog/intro-guide-to-dockerfile-best-practices/?fbclid=IwAR2IvfL95fHkckX1Hq9F0ARfOCbSjjpMgu7nPwNRsf3S8kz1QAvZfrXW3D8)을 참고하여 작성했습니다.



### 1. 빌드 시간
개발 중 코드 변경 시 Docker 이미지를 다시 빌드해야합니다. 이때 캐싱을 활용하면 반복되는 빌드 단계를 다시 실행하지 않아 빌드 시간을 줄일 수 있습니다. 여기서 Docker 캐싱은 이전에 빌드한 파일 시스템의 단계적 구조인 레이어를 캐시에 저장하여 동일한 레이어가 반복되면 해당 캐시를 활용해 이미지를 다시 빌드하지 않고 효율적으로 이미지를 구성할 수 있습니다.

```Dockerfile
FROM openjdk:22-slim

WORKDIR /app

# 실행권한 부여 & build.gradle 의존성 다운로드
RUN  apt-get update \
  && apt-get install -y dumb-init

# 소스코드 복사(COPY 명령어)
COPY run_server.sh .
RUN chmod +x run_server.sh

ENTRYPOINT ["/usr/bin/dumb-init", "--"]
CMD ["./run_server.sh"]
```
- **캐싱 순서**
위 코드는 모행 프로젝트에서 작성한 Dockerfile 입니다. 해당 코드를 보면 의존성 설치를 COPY 명령어 이전에 배치하여 의존성과 관련없는 파일이 변경되어도 의존성 설치 명령이 다시 실행되지 않도록 하였습니다. 
다음과 같이 소스 코드를 COPY하는 것 처럼 가장 많이 변하는 부분을 뒤로 배치하고 적게 변하는 부분을 먼저 배치하여 캐시 무효화를 방지합니다.

- **COPY 명령어를 구체적으로 작성**
`COPY .`와 같은 경우 현재 디렉토리 내의 모든 파일을 Docker 이미지 내의 작업 디렉토리(WORKDIR)로 복사하여 불필요한 파일도 복사될 수 있습니다. 이런 경우 프로젝트에서 작은 수정사항이 생겨도 전체를 복사하여 캐시 무효화가 발생합니다. 따라서 위 COPY 명령어와 같이 특정 파일만 구체적으로 지정하면 해당 파일이 변경되는 경우에만 다시 빌드되고 나머지 레이어는 캐시를 재사용하게 됩니다.

- **apt-get update & install과 같이 캐시 가능한 단위 식별**
RUN 명령어를 많이 사용하게되면 각 명령어마다 캐시 단위가 되어 불필요하게 캐시 단위가 많아지게 됩니다. 하지만 모든 명령어를 하나의 RUN 명령어 단위로 지정해도 캐시 무효화에 취약할 수 있습니다. 따라서 위 코드와 같이 apt-get update & install 등과 같은 캐시가 가능한 명령어끼리 RUN 명령어에 묶는 것이 좋습니다. 


### 2. 이미지 크기 줄이기
이미지 크기는 작으면 작을수록 배포 속도가 빨라지고 보안에도 유리합니다. 따라서 다음과 같은 방법으로 이미지 크기를 줄일 수 있습니다.

```Dockerfile
RUN	apt-get update \
	&& apt-get install -y dumb-init \
	&& npm install -g pnpm
```

- **불필요한 종속성 제거**
위 코드는 프론트엔드 Dockerfile 중 종속성 설치에 대한 부분입니다. 
현재는 apt 패키지 매니저로 소프트웨어 패키지 설치를 `apt-get install -y dumb-init` 다음과 같은 명령어로 dumb-init 패키지와 함께 추천 패키지도 자동으로 설치합니다. 따라서 불필요한 패키지를 설치하여 이미지 크기가 증가되어 이미지 다운로드 및 실행 시간이 증가될 수 있습니다.

```Dockerfile
RUN apt-get install --no-install-recommends -y dumb-init
```
위와 같이 `--no-install-recommends` 옵션을 따로 설정하게되면 추천 패키지를 설치하지 않아 불필요한 패키지가 없어 도커 이미지 크기를 줄일 수 있습니다. 따라서 도커 파일에서는 불필요한 종속성을 제거하고 디버깅 도구를 설치하지 않는 것이 이미지 크기를 줄이는데 효과적입니다.

- **패키지 매니저 캐시 제거**
위 코드에서 사용한 apt와 같이 패키지 매니저는 패키지를 설치할 때 설치에 필요한 정보와 메타데이터를 `/var/lib/apt/lists`와 같은 디렉토리에 캐시로 저장합니다. 이러한 패키지 매니저 캐시를 제거하는 것도 이미지 크기를 줄일 수 있는 방법입니다.
```Dockerfile
RUN apt-get update \
    && apt-get install -y --no-install-recommends dumb-init \
    && apt-get clean \  # 다운로드된 패키지 파일 제거
    && rm -rf /var/lib/apt/lists/*  # 패키지 목록 캐시 제거

```
도커에서는 `RUN`, `COPY`, `ADD` 등의 명령어가 실행될 때마다 새로운 레이어가 생성되고 이러한 레이어가 이미지의 파일 시스템을 구성하여 최종 이미지를 만들게 됩니다. 따라서 캐시를 제거하는 명령어를 다른 `RUN` 명령어에서 실행하면 독립적으로 실행되어 최종 이미지 크기에 영향을 주지 않아 위와 같이 같은 `RUN` 명령어(같은 레이어)에서 캐시를 제거해야합니다.

### 3. 유지 보수성

도커 파일에서 유지 보수성은 도커 이미지를 효율적으로 관리하고 업데이트하는 과정을 쉽게 만드는 것을 말합니다. 따라서 유지 보수성을 위해 다음과 같이 도커 이미지를 선택하고 설정하는 것이 중요합니다.

```Dockerfile
FROM openjdk:22-slim
```

- **공식 이미지 사용 권장**
다음과 같이 공식 이미지는 최적의 성능과 보안에 대한 모범 사례에 따라 이미 설정되어 있습니다. 따라서 공식 이미지를 사용하면 프로젝트의 신뢰성, 보안성, 유지 보수성이 향상되어 빠르게 개발 및 배포할 수 있습니다. 또한 여러 프로젝트에서 동일한 공식 이미지를 사용하면 레이어를 공유하여 일관된 환경을 제공할 수 있습니다. 

- **구체적인 태그 사용**
`openjdk:latest`와 같이 `:latest`태그를 사용하면 항상 이미지의 최신 버전을 사용할 수 있지만 변경사항이 발생할 수 있어 사용하는 것을 지양합니다. 따라서 위와 같이 구체적인 태크를 사용합니다. 

- **경량 이미지 선택**

| 태그                 | 설명                                                            |
|---------------------|----------------------------------------------------------------|
| `openjdk:22`        | - 기본 이미지로 JDK(Java Development Kit)와 JRE(Java Runtime Environment)가 모두 포함 |
| `openjdk:22-jre`    | - JRE버전만 포함된 이미지<br> - JDK는 포함되어 있지 않음 -> 더 작음|
| `openjdk:22-slim`   | - 슬림 버전 이미지로 기본 Debian 이미지를 기반으로 불필요한 요소가 제거되어 크기가 작음<br> - Debian은 표준 C 라이브러리인 GNU libc를 사용 -> 완벽한 호환 O |
| `openjdk:22-alpine` | - Alpine 이미지는 Alpine Linux 배포판을 기반으로 매우 가볍고 크기가 작음<br> - Alpine은 GNU libc보다 더 작은 musl libc을 사용 -> 완변한 호환 X |

<br>최소한의 기능을 제공하는 경량 이미지 선택하는 것이 이미지 크기를 줄여 최소한의 구성 요소만 관리하여 유지 보수성을 높일 수 있습니다. 다만 이미지의 호환성 등의 문제를 고려하여 선택해야합니다.

### 4. 재현성
도커 파일에서 재현성(Reproducibility)이란 항상 동일한 이미지를 생성하는 걸 보장한다는 것을 말합니다. 따라서 도커 파일을 청사진으로 생각하고 작성합니다. 도커 파일은 빌드 환경을 설명하고 올바른 버전의 빌드 도구들이 설치하여 환경 간의 불일치를 방지합니다. 또한 시스템 종속성이 있을 수 있습니다. 신뢰할 수 있는 정보는 빌드 아티팩트가 아니라 소스 코드입니다.

- **일관된 환경에서 소스 코드 빌드**
```Dockerfile
FROM openjdk:22-slim AS build

WORKDIR /app

COPY gradlew gradle build.gradle settings.gradle ${WORKDIR}
COPY gradle/wrapper /app/gradle/wrapper

COPY src /app/src

RUN chmod +x ./gradlew

RUN ./gradlew clean build

CMD ["sh", "-c", "java -jar -Dspring.profiles.active=prod *SNAPSHOT.jar"]
```
호스트 환경에서 JAR 파일을 빌드를 하는 것은 재현성과 거리가 멉니다. 호스트 시스템의 환경이 다르면 애플리케이션의 빌드 결과가 달라질 수 있기 때문에 빌드를 도커 컨테이너 내부에서 하는 것이 더 재현성을 지킬 수 있습니다.

- **별도의 단계에서 종속성 가져오기**
도커 파일에서 종속성을 가져오는 RUN 명령어를 소스 코드가 COPY된 이후에 실행하여 소스 코드 변경에 영향을 받지 않고 종속성을 캐시할 수 있습니다. 또한 이를 통해 각 단계가 명확히 구분되어 재현성에 효과적입니다.


- **멀티스테이지 빌드를 이용하여 빌드 종석성 제거(권장 도커 파일 작성법)**
```Dockerfile
# 첫 번째 빌드 단계
# AS로 이름 지정
FROM openjdk:22-slim AS build
...
RUN ./gradlew clean build

# 두 번째 빌드 단계
FROM openjdk:22-slim
# 빌더 단계에서 생성된 아티팩트 가져오기
COPY --from=build ${BUILDDIR} ${WORKDIR}
...
CMD ["sh", "-c", "java -jar -Dspring.profiles.active=prod *SNAPSHOT.jar"]
```
멀티스테이지 빌드는 도커 이미지를 관리하고 최적화하여 빌드 타임 종속성을 효과적으로 제거합니다. 여러 개의 FROM 문으로 멀티스테이지 빌드를 진행할 수 있습니다. 위 코드에서는 첫 번째 FROM 문 "빌드" 단계에서는 애플리케이션을 빌드하는 데 필요한 모든 종속성을 포함합니다. 두 번째 FROM 문은 최종 이미지를 정의하는 단계로 런타임에 필요한 최소한의 종속성만 포함합니다. 최종 이미지는 중간 빌더 단계에서 생성된 아티팩트를 가져옵니다. 중간 빌더 단계는 캐시되고 최종 이미지에는 나타나지 않습니다.

---
## 출처 및 참고자료
- [Intro Guide to Dockerfile Best Practices](https://www.docker.com/blog/intro-guide-to-dockerfile-best-practices/?fbclid=IwAR2IvfL95fHkckX1Hq9F0ARfOCbSjjpMgu7nPwNRsf3S8kz1QAvZfrXW3D8)
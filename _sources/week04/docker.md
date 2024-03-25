# Docker

## Docker의 핵심 구성 요소

### Docker 엔진

Docker 엔진은 Docker 플랫폼을 구동하는 핵심 구성 요소입니다. 컨테이너 생성, 관리 및 조율을 담당하며, 데몬 프로세스(dockerd), REST API, 명령줄 인터페이스(CLI)로 구성됩니다.

### Docker 이미지

Docker 이미지는 컨테이너를 생성하고 실행하는 데 필요한 애플리케이션 코드, 종속성, 라이브러리 및 구성 파일이 포함된 읽기 전용 템플릿입니다. 이미지는 Docker 레지스트리에 저장되며 컨테이너를 생성하기 위해 가져올 수 있습니다.

### Docker 컨테이너

Docker 컨테이너는 Docker 이미지로부터 생성된 경량의 독립적이고 실행 가능한 소프트웨어 패키지입니다. 애플리케이션 실행에 필요한 모든 것을 포함하여 서로 다른 환경에서도 일관성을 보장합니다.

### Dockerfile

Dockerfile은 Docker 이미지를 빌드하기 위한 명령어가 포함된 스크립트입니다. 베이스 이미지, 종속성, 구성 및 애플리케이션에 맞게 사용자 정의된 이미지를 생성하는 데 필요한 기타 설정을 지정합니다.

### Docker 레지스트리

Docker 레지스트리는 Docker 이미지를 위한 중앙 집중식 저장소입니다. Docker Hub는 Docker Inc.에서 유지 관리하는 공개 레지스트리지만, 사설 레지스트리를 생성하거나 타사 솔루션을 사용할 수도 있습니다.

## 설치

공식 설치 가이드에 따라 시스템에 Docker를 설치하세요:

- Linux: [https://docs.docker.com/engine/install/](https://docs.docker.com/engine/install/)
- MacOS: [https://docs.docker.com/docker-for-mac/install/](https://docs.docker.com/docker-for-mac/install/)
- Windows: [https://docs.docker.com/docker-for-windows/install/](https://docs.docker.com/docker-for-windows/install/)

## 기본 Docker 명령어

### 이미지 가져오기

레지스트리에서 이미지 다운로드:

```bash
docker pull <image-name>
```

### 이미지 목록 보기

로컬에서 사용 가능한 모든 이미지 표시:

```bash
docker images
```

### 컨테이너 생성 및 실행

이미지에서 컨테이너를 생성하고 시작:

```bash
docker run -it --name <container-name> <image-name>
```

### 컨테이너 목록 보기

실행 중인 모든 컨테이너 나열:

```bash
docker ps
```

중지된 컨테이너를 포함한 모든 컨테이너 나열:

```bash
docker ps -a
```

### 컨테이너 중지

실행 중인 컨테이너 중지:

```bash
docker stop <container-name>
```

### 컨테이너 제거

중지된 컨테이너 제거:

```bash
docker rm <container-name>
```

### 이미지 제거

사용하지 않는 이미지 제거:

```bash
docker rmi <image-name>
```

## Docker Compose

Docker Compose는 다중 컨테이너 Docker 애플리케이션을 정의하고 실행하기 위한 도구입니다. 단일 `docker-compose.yml` 파일에서 애플리케이션의 서비스, 네트워크 및 볼륨을 구성할 수 있어 여러 종속성이 있는 복잡한 애플리케이션 관리 프로세스를 간소화합니다.

### Docker Compose 설치

공식 설치 가이드를 따르세요:

- Linux: [https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/)
- MacOS & Windows: Docker Desktop에 Docker Compose가 사전 설치되어 있습니다.

### `docker-compose.yml` 파일 생성

프로젝트 디렉토리에 `docker-compose.yml` 파일을 생성하고 애플리케이션의 서비스, 네트워크 및 볼륨을 정의하세요.

`docker-compose.yml` 파일 예시:

```yaml
version: "3.8"

services:
  web:
    build: .
    ports:
      - "8080:8080"
  redis:
    image: "redis:alpine"
```

이 예시에서는 `web`과 `redis`의 두 서비스가 있습니다. `web` 서비스는 현재 디렉토리에서 빌드되며, 8080 포트가 호스트의 8080 포트에 매핑됩니다. `redis` 서비스는 Docker Hub의 공식 Redis 이미지를 사용합니다.

### 서비스 시작

`docker-compose.yml` 파일에 정의된 모든 서비스 시작:

```bash
docker-compose up
```

### 서비스 중지

모든 서비스 중지:

```bash
docker-compose down
```

## Docker 볼륨

Docker 볼륨은 Docker 컨테이너에 의해 생성되고 사용되는 데이터를 유지하는 데 사용됩니다. 애플리케이션의 데이터를 컨테이너의 수명주기와 분리하여 컨테이너가 제거되더라도 데이터가 그대로 유지되도록 합니다.

### 볼륨 생성

새 Docker 볼륨 생성:

```bash
docker volume create <volume-name>
```

### 컨테이너에서 볼륨 사용

컨테이너에서 볼륨을 사용하려면 `-v` 또는 `--mount` 플래그를 사용하여 마운트해야 합니다:

```bash
docker run -v <volume-name>:/data <image-name>
```

## Docker의 고급 기능

### 다단계 빌드 (Multi-stage Builds)

다단계 빌드는 Dockerfile에서 여러 단계를 정의하여 이미지 크기를 줄이고 빌드 프로세스를 최적화할 수 있는 Docker의 기능입니다. 각 단계는 이전 단계의 결과물을 사용할 수 있으며, 최종 이미지에는 필요한 아티팩트만 포함됩니다. 이는 불필요한 종속성과 도구를 제거하여 이미지 크기를 최소화하는 데 도움이 됩니다.

다단계 빌드 예시:

```dockerfile
# 빌드 단계
FROM golang:1.16 AS build
WORKDIR /app
COPY . .
RUN go build -o main .

# 런타임 단계
FROM alpine:3.14
WORKDIR /app
COPY --from=build /app/main .
CMD ["./main"]
```

### Docker 네트워크

Docker 네트워크는 컨테이너 간의 통신을 가능하게 합니다. Docker는 브리지, 호스트, 오버레이 등 다양한 네트워크 드라이버를 제공합니다. 사용자 정의 네트워크를 생성하여 컨테이너를 격리하고 서비스 디스커버리를 활성화할 수 있습니다.

네트워크 생성 예시:

```bash
docker network create my-network
```

### Docker 보안 Best Practices

Docker 컨테이너를 안전하게 실행하려면 다음과 같은 보안 모범 사례를 따르는 것이 좋습니다:

- 최소 권한 원칙 적용: 컨테이너에 필요한 최소한의 권한만 부여합니다.
- 신뢰할 수 있는 이미지 사용: 공식 이미지 또는 검증된 출처의 이미지를 사용합니다.
- 이미지 취약점 스캔: 정기적으로 이미지를 스캔하여 알려진 취약점을 식별하고 해결합니다.
- 시크릿 관리: Docker Swarm의 시크릿 관리 기능 또는 Hashicorp Vault와 같은 외부 도구를 사용하여 민감한 정보를 안전하게 저장하고 관리합니다.
- 네트워크 격리: 필요에 따라 컨테이너를 격리하고 최소한의 네트워크 액세스 권한을 부여합니다.

## 쿠버네티스 (Kubernetes)와의 통합

쿠버네티스는 컨테이너 오케스트레이션을 위한 오픈 소스 플랫폼으로, Docker 컨테이너를 대규모로 배포, 관리 및 확장할 수 있습니다. Docker와 쿠버네티스를 함께 사용하면 강력하고 유연한 컨테이너 기반 인프라를 구축할 수 있습니다.

쿠버네티스는 다음과 같은 주요 개념을 제공합니다:

- **파드 (Pod)**: 하나 이상의 컨테이너를 포함하는 쿠버네티스의 기본 배포 단위입니다.
- **서비스 (Service)**: 파드 집합에 대한 안정적인 네트워크 인터페이스를 제공합니다.
- **레플리카셋 (ReplicaSet)**: 지정된 수의 파드 복제본을 유지 관리합니다.
- **디플로이먼트 (Deployment)**: 파드와 레플리카셋에 대한 선언적 업데이트를 제공합니다.

Docker 컨테이너를 쿠버네티스에 배포하려면 먼저 Docker 이미지를 빌드한 다음, YAML 매니페스트 파일을 사용하여 쿠버네티스 리소스를 정의하고 배포합니다.

## 결론

Docker는 강력하고 유연한 컨테이너화 플랫폼으로, 애플리케이션 개발, 배포 및 관리 방식을 혁신하고 있습니다. 다단계 빌드, 네트워킹, 오케스트레이션 등의 고급 기능을 활용하고 보안 모범 사례를 따름으로써 효율적이고 안전한 컨테이너 기반 인프라를 구축할 수 있습니다. 또한 Docker와 쿠버네티스를 함께 사용하면 대규모 컨테이너 배포와 관리가 가능해집니다. Docker 기술 스택과 생태계가 계속 진화함에 따라 컨테이너화의 이점은 더욱 확대될 것입니다.

## 주요 용어 설명

- **오케스트레이션 (Orchestration)**: 컨테이너의 배포, 관리, 스케일링 및 네트워킹을 자동화하는 프로세스입니다.
- **서비스 디스커버리 (Service Discovery)**: 컨테이너화된 서비스를 자동으로 검색하고 통신할 수 있도록 하는 메커니즘입니다.
- **시크릿 (Secret)**: 암호, 토큰, 키 등 민감한 정보를 저장하고 관리하기 위한 메커니즘입니다.
- **매니페스트 (Manifest)**: 쿠버네티스에서 원하는 상태를 선언적으로 정의하는 YAML 또는 JSON 형식의 파일입니다.

## 최신 동향

- **Rootless 모드**: Docker는 이제 루트 권한 없이 컨테이너를 실행할 수 있는 Rootless 모드를 지원합니다. 이는 보안을 강화하고 컨테이너 실행과 관련된 잠재적인 위험을 줄이는 데 도움이 됩니다.
- **Docker 빌드킷 (Buildkit)**: Docker 19.03부터 도입된 빌드킷은 Dockerfile을 사용하여 이미지를 빌드하기 위한 새로운 백엔드입니다. 빌드 성능을 개선하고, 동시 빌드를 지원하며, 향상된 캐싱 메커니즘을 제공합니다.
- **확장된 빌드 컨텍스트**: Docker Buildkit은 이제 Docker 데몬의 컨텍스트 외부에 있는 파일과 디렉토리에 접근할 수 있어, 더 유연한 빌드 프로세스를 가능하게 합니다.
- **Compose 사양**: Docker Compose 파일 형식이 오픈 사양으로 발전하여, Compose가 Docker 이외의 플랫폼과 도구에서도 사용될 수 있게 되었습니다.
- **Kubernetes의 기본 컨테이너 런타임으로 containerd 채택**: 쿠버네티스는 Docker 대신 containerd를 기본 컨테이너 런타임으로 채택하기 시작했습니다. 이는 쿠버네티스의 성능과 안정성을 향상시킬 것으로 기대됩니다.
- **확장된 에코시스템**: Docker 기술 스택을 기반으로 하는 도구와 플랫폼이 계속 등장하고 있습니다. 예를 들어, Knative는 쿠버네티스 위에 서버리스 워크로드를 구축할 수 있게 해주며, Istio는 마이크로서비스 아키텍처를 위한 서비스 메시를 제공합니다.

Docker와 컨테이너 기술은 계속해서 발전하고 있으며, 클라우드 네이티브 애플리케이션 개발에 없어서는 안 될 중요한 역할을 하고 있습니다. DevOps 및 GitOps 워크플로우와 함께 Docker를 활용함으로써 조직은 애플리케이션 제공 속도를 높이고, 인프라 효율성을 개선하며, 운영 오버헤드를 줄일 수 있습니다. Docker의 미래는 더욱 자동화되고, 안전하며, 개발자 친화적인 컨테이너 기반 솔루션을 향해 나아갈 것입니다.

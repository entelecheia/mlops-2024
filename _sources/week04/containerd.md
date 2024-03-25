# `containerd`

## 소개

**Containerd**는 단순성, 견고성, 이식성에 중점을 둔 업계 표준 컨테이너 런타임입니다. Docker 생태계의 핵심 구성 요소이자 Cloud Native Computing Foundation (CNCF)의 독립 프로젝트로서 기능합니다. containerd의 주요 목표는 컨테이너 관리를 위한 경량의 독립 실행형 런타임을 제공하는 것입니다.

### 1. containerd 개요

- **미니멀리스트 디자인**: containerd는 컨테이너 생성, 시작, 중지, 모니터링 등 컨테이너 관리의 핵심 기능에 중점을 둔 간소화된 디자인을 강조합니다.
- **성능**: containerd는 Docker에 비해 성능 향상과 공간 축소를 위해 설계되었으며, 이는 리소스가 제한된 환경과 대규모 배포에 적합합니다.
- **모듈성**: containerd는 더 큰 컨테이너 플랫폼의 구성 요소로 사용되거나 다른 도구와 통합될 수 있어 유연성과 사용자 정의 옵션을 제공합니다.
- **OCI 호환성**: containerd는 OCI(Open Container Initiative) 런타임 및 이미지 사양과 호환되므로 다른 OCI 호환 도구 및 플랫폼과의 상호 운용성을 보장합니다.

### 2. containerd의 핵심 구성 요소

- **containerd 데몬**: 컨테이너 및 그 수명주기를 관리하는 containerd의 핵심 구성 요소입니다. 데몬은 gRPC API를 통해 클라이언트와 통신합니다.
- **ctr**: containerd 데몬과 상호 작용하기 위한 경량 커맨드라인 인터페이스(CLI) 도구로, 사용자가 컨테이너를 직접 관리할 수 있게 합니다.
- **runc**: containerd가 컨테이너를 실행하기 위해 사용하는 기본 OCI 호환 런타임입니다. 사용자는 kata-containers나 gVisor와 같은 대체 OCI 호환 런타임을 사용할 수도 있습니다.
- **스냅샷터**: 다양한 스토리지 백엔드 및 파일시스템에 대한 지원을 제공하여 컨테이너의 파일시스템 스냅샷을 관리하는 플러그형 구성 요소입니다.
- **콘텐츠 저장소**: 컨테이너 이미지와 바이너리 데이터를 안전하고 효율적으로 관리하기 위한 콘텐츠 주소 지정 가능한 스토리지 시스템입니다.

### 3. containerd 사용하기

- **containerd 설치**: containerd는 패키지 관리자를 통해 또는 사전 빌드된 바이너리를 다운로드하여 Linux 및 Windows와 같은 다양한 플랫폼에 설치할 수 있습니다.
- **컨테이너 관리**: 사용자는 `ctr` CLI 도구를 통해 또는 gRPC를 통해 containerd API에 직접 액세스하여 containerd와 상호 작용할 수 있습니다.
- **컨테이너 이미지**: containerd는 Docker Hub와 같은 OCI 호환 이미지 레지스트리에서 컨테이너 이미지를 가져오고 관리할 수 있습니다.
- **Kubernetes와의 통합**: containerd는 Kubernetes 클러스터의 컨테이너 런타임으로 사용될 수 있으므로 사용자는 Kubernetes 환경에서 성능과 모듈성의 이점을 활용할 수 있습니다.

### 4. 사용 사례 및 시나리오

- **엣지 컴퓨팅**: containerd의 경량 설계와 리소스 효율성은 제한된 리소스를 가진 엣지 컴퓨팅 환경에 적합합니다.
- **대규모 배포**: containerd의 성능과 단순성에 대한 중점은 리소스 활용과 관리 오버헤드가 중요한 관심사인 대규모 컨테이너 배포에 강력한 선택이 됩니다.
- **맞춤형 컨테이너 플랫폼**: containerd의 모듈식 설계를 통해 맞춤형 컨테이너 플랫폼 및 오케스트레이터와 통합할 수 있어 유연하고 사용자 정의 가능한 컨테이너 런타임 솔루션을 제공합니다.

### 5. containerd 보안

containerd는 컨테이너의 보안을 개선하기 위해 여러 기능과 모범 사례를 제공합니다:

- **Rootless 모드**: containerd는 루트 권한 없이 컨테이너를 실행할 수 있는 Rootless 모드를 지원하여, 잠재적인 보안 위험을 줄이고 컨테이너 실행을 보다 안전하게 만듭니다.
- **SELinux 및 AppArmor 지원**: containerd는 SELinux와 AppArmor 보안 모듈을 통합하여 컨테이너에 강화된 액세스 제어 및 격리 기능을 제공합니다.
- **Seccomp 프로파일**: containerd는 Seccomp (Secure Computing Mode) 프로파일을 사용하여 컨테이너 내에서 사용 가능한 시스템 호출을 제한함으로써 컨테이너 보안을 강화합니다.
- **취약점 스캐닝**: containerd는 Trivy나 Clair와 같은 취약점 스캐너와 통합되어 컨테이너 이미지의 알려진 취약점을 탐지하고 보고할 수 있습니다.

이러한 보안 기능을 활용함으로써 containerd 사용자는 컨테이너 환경의 전반적인 보안을 개선할 수 있습니다.

## Containerd 시작하기

이 섹션에서는 **containerd**를 설치하고 사용하여 컨테이너를 관리하는 과정을 안내합니다. 다음 주제를 다룰 것입니다:

1. containerd 설치
2. `ctr` 커맨드라인 도구 사용
3. 간단한 컨테이너 실행
4. 컨테이너 이미지 관리
5. 컨테이너 상세 정보 검사

### 1. containerd 설치

**Ubuntu에서:**

Ubuntu에 containerd를 설치하려면 다음 명령을 실행하세요:

```bash
sudo apt-get update
sudo apt-get install -y containerd
```

**CentOS에서:**

CentOS에 containerd를 설치하려면 먼저 EPEL 저장소를 활성화한 다음 containerd를 설치하세요:

```bash
sudo yum install -y epel-release
sudo yum install -y containerd
```

**Windows에서:**

[GitHub 릴리스 페이지](https://github.com/containerd/containerd/releases)에서 Windows용 최신 containerd 릴리스를 다운로드할 수 있습니다. 아카이브를 추출하고 `bin` 디렉토리를 시스템 PATH에 추가하세요.

설치 후 containerd 서비스를 시작하세요:

```bash
sudo systemctl start containerd
sudo systemctl enable containerd
```

### 2. `ctr` 커맨드라인 도구 사용

`ctr` 도구는 containerd와 상호 작용하기 위한 주요 커맨드라인 인터페이스입니다. 이를 사용하여 컨테이너, 이미지 및 기타 리소스를 관리할 수 있습니다. 사용 가능한 명령 목록을 보려면 `ctr --help`를 실행하세요.

### 3. 간단한 컨테이너 실행

이 예제에서는 containerd를 사용하여 Nginx 컨테이너를 실행합니다. 먼저 Nginx 이미지를 가져옵니다:

```bash
sudo ctr images pull docker.io/library/nginx:latest
```

다음으로 가져온 이미지를 사용하여 새 컨테이너를 생성합니다:

```bash
sudo ctr containers create docker.io/library/nginx:latest nginx-container
```

마지막으로 컨테이너를 시작합니다:

```bash
sudo ctr containers start nginx-container
```

Nginx 컨테이너가 이제 실행 중이어야 합니다. 상태를 확인하려면 다음 명령을 사용하세요:

```bash
sudo ctr containers list
```

### 4. 컨테이너 이미지 관리

containerd는 OCI 호환 이미지 레지스트리에서 컨테이너 이미지 관리를 지원합니다. `ctr` 도구를 사용하여 이미지를 가져오고, 나열하고, 제거할 수 있습니다.

- **이미지 가져오기**: `ctr images pull` 명령을 사용하여 컨테이너 이미지를 다운로드합니다. 예:

  ```bash
  sudo ctr images pull docker.io/library/alpine:latest
  ```

- **이미지 나열**: 다운로드한 이미지 목록을 보려면 다음 명령을 실행하세요:

  ```bash
  sudo ctr images list
  ```

- **이미지 제거**: 이미지를 제거하려면 `ctr images remove` 명령을 사용하세요:

  ```bash
  sudo ctr images remove docker.io/library/alpine:latest
  ```

### 5. 컨테이너 상세 정보 검사

`ctr` 도구를 사용하여 실행 중인 컨테이너의 구성, 환경 변수, 마운트 등 다양한 측면을 검사할 수 있습니다.

- **컨테이너 구성 검사**: 컨테이너의 구성을 보려면 다음 명령을 사용하세요:

  ```bash
  sudo ctr containers info nginx-container
  ```

- **컨테이너 프로세스 검사**: 컨테이너 내에서 실행 중인 프로세스를 보려면 `ctr containers exec` 명령을 사용하세요:

  ```bash
  sudo ctr containers exec --exec-id my-exec-id nginx-container ps aux
  ```

- **컨테이너 로그 검사**: 실행 중인 컨테이너의 로그를 보려면 `ctr containers logs` 명령을 사용하세요:

  ```bash
  sudo ctr containers logs nginx-container
  ```

### 6. containerd 모니터링

containerd는 Prometheus와 같은 모니터링 도구와 통합하여 컨테이너 및 런타임 메트릭을 수집하고 분석할 수 있습니다. containerd는 `/metrics` 엔드포인트를 통해 Prometheus 형식의 메트릭을 노출합니다.

containerd 메트릭을 수집하려면 다음 단계를 따르세요:

1. containerd 구성 파일(`/etc/containerd/config.toml`)에서 메트릭 엔드포인트를 활성화합니다:

   ```toml
   [metrics]
     address = "0.0.0.0:1338"
   ```

2. containerd를 다시 시작하여 변경 사항을 적용합니다:

   ```bash
   sudo systemctl restart containerd
   ```

3. Prometheus 구성 파일에 containerd 메트릭 엔드포인트를 추가합니다:

   ```yaml
   scrape_configs:
     - job_name: "containerd"
       static_configs:
         - targets: ["localhost:1338"]
   ```

4. Prometheus를 다시 시작하여 containerd 메트릭 수집을 시작합니다.

이제 Prometheus와 Grafana를 사용하여 containerd 메트릭을 시각화하고 분석할 수 있습니다. 이를 통해 컨테이너 리소스 사용량, 성능 병목 현상 및 기타 런타임 동작을 모니터링할 수 있습니다.

## 컨테이너 이미지 생성 방법

containerd를 위한 이미지 생성은 containerd 및 기타 OCI 호환 컨테이너 런타임에서 사용할 수 있는 OCI(Open Container Initiative) 호환 이미지를 빌드하는 것을 포함합니다. 이 가이드에서는 containerd와 함께 작동하는 컨테이너 이미지 빌드를 위한 고급 도구 키트인 BuildKit를 사용하여 컨테이너 이미지를 생성하는 과정을 안내합니다.

### 1. BuildKit 설치

먼저 BuildKit을 설치해야 합니다. [BuildKit 저장소](https://github.com/moby/buildkit)에서 다양한 플랫폼에 대한 설치 지침을 찾을 수 있습니다.

Ubuntu에서는 다음 명령을 사용하여 BuildKit을 설치할 수 있습니다:

```bash
sudo apt-get update
sudo apt-get install -y buildkit
```

CentOS에서는 다음 명령으로 BuildKit을 설치할 수 있습니다:

```bash
sudo yum install -y epel-release
sudo yum install -y buildkit
```

### 2. Dockerfile 생성

다음으로 빌드하려는 이미지를 설명하는 `Dockerfile`을 생성합니다. 예제로 `curl` 유틸리티를 포함하는 간단한 Alpine 기반 이미지를 생성해 보겠습니다:

```dockerfile
FROM alpine:latest
RUN apk add --no-cache curl
CMD ["curl", "--help"]
```

이 내용을 작업 디렉토리의 `Dockerfile`이라는 파일에 저장합니다.

### 3. BuildKit을 사용하여 이미지 빌드

BuildKit을 사용하여 이미지를 빌드하려면 먼저 `BUILDKIT_HOST` 환경 변수를 BuildKit 데몬을 가리키도록 설정합니다:

```bash
export BUILDKIT_HOST="unix:///var/run/buildkit/buildkitd.sock"
```

다음으로 `buildctl` 명령을 실행하여 이미지를 빌드합니다:

```bash
buildctl build \
  --frontend dockerfile.v0 \
  --local context=. \
  --local dockerfile=. \
  --output type=image,name=my-containerd-image,oci-mediatypes=true,push=false
```

이 명령에서:

- `--frontend dockerfile.v0`: BuildKit용 Dockerfile 프론트엔드를 지정합니다.
- `--local context=.`: 빌드 컨텍스트를 현재 디렉토리로 설정합니다.
- `--local dockerfile=.`: Dockerfile이 현재 디렉토리에 있음을 나타냅니다.
- `--output type=image,name=my-containerd-image,oci-mediatypes=true,push=false`: 출력을 `my-containerd-image`라는 OCI 호환 이미지로 구성하고 이미지를 레지스트리로 푸시하는 것을 비활성화합니다.

빌드가 완료되면 이미지를 containerd에서 로컬로 사용할 수 있습니다.

### 4. containerd에서 이미지 확인

이미지가 containerd에서 사용 가능한지 확인하려면 `ctr` 명령을 사용하세요:

```bash
sudo ctr images list | grep my-containerd-image
```

사용 가능한 이미지 목록에 `my-containerd-image`가 표시되어야 합니다.

이것으로 BuildKit을 사용하여 containerd용 이미지를 성공적으로 생성했습니다. 이제 이 이미지를 사용하여 containerd 또는 다른 OCI 호환 컨테이너 런타임으로 컨테이너를 생성하고 실행할 수 있습니다.

## containerd와 Kubernetes의 관계

Kubernetes는 컨테이너화된 애플리케이션의 배포, 스케일링 및 관리를 자동화하는 오픈 소스 컨테이너 오케스트레이션 플랫폼입니다. Container Runtime Interface (CRI)를 통해 Docker, containerd, CRI-O 등 다양한 컨테이너 런타임과 함께 작동합니다.

Kubernetes와 containerd의 관계는 다음과 같이 요약할 수 있습니다:

1. **컨테이너 런타임**: Containerd는 Kubernetes에서 지원하는 컨테이너 런타임 중 하나입니다. 컨테이너를 실행하고 수명주기를 관리하는 데 필요한 기능과 이미지 배포 및 스토리지를 제공합니다. Containerd는 Kubernetes 클러스터에서 Docker를 컨테이너 런타임으로 대체할 수 있으며, Kubernetes와 함께 사용하도록 맞춤화된 더 가벼운 초점을 가진 런타임을 제공합니다.

2. **Container Runtime Interface (CRI)**: Kubernetes는 CRI를 사용하여 containerd와 같은 컨테이너 런타임과 통신합니다. CRI는 기본 컨테이너 런타임을 Kubernetes 구성 요소에서 추상화하는 표준 API로, Kubernetes가 수정 없이 다른 컨테이너 런타임과 작동할 수 있도록 합니다. Containerd는 `containerd/cri` 플러그인을 통해 CRI를 구현하며, 이는 Kubernetes의 CRI API 호출을 containerd API 호출로 변환합니다.

3. **런타임 클래스**: Kubernetes는 RuntimeClass 기능을 통해 각 파드에 대한 컨테이너 런타임 선택을 지원합니다. 이를 통해 클러스터의 특정 파드에 대해 다른 컨테이너 런타임(예: Docker, containerd 또는 CRI-O) 중에서 선택할 수 있습니다. 적절한 RuntimeClass를 구성하여 Kubernetes에 특정 파드에 containerd를 사용하도록 지시할 수 있습니다.

4. **간소화된 스택**: Kubernetes 클러스터에서 Docker 대신 containerd를 컨테이너 런타임으로 사용하면 더 간단하고 효율적인 스택을 얻을 수 있습니다. 이는 containerd가 Docker에 비해 더 집중적이고 가벼운 런타임이기 때문입니다. Docker는 Kubernetes 환경에서 필요하지 않을 수 있는 추가 구성 요소(예: Docker Compose 또는 Docker Swarm)를 포함합니다. containerd를 사용하면 Kubernetes 노드의 리소스 공간과 복잡성을 잠재적으로 줄일 수 있습니다.

5. **CRI-O와의 비교**: CRI-O는 Kubernetes를 위해 특별히 설계된 또 다른 경량 컨테이너 런타임입니다. CRI-O는 containerd와 유사한 목표를 가지고 있지만, Kubernetes와의 더 긴밀한 통합에 중점을 둡니다. 반면에 containerd는 Kubernetes 이외의 사용 사례를 포함하여 더 일반적인 컨테이너 런타임으로 설계되었습니다.

## 결론

Containerd는 단순성, 성능 및 이식성을 갖춘 강력한 컨테이너 런타임으로, 컨테이너 기반 애플리케이션을 배포하고 관리하기 위한 효율적이고 안전한 기반을 제공합니다. OCI 표준 준수, Kubernetes와의 긴밀한 통합, 다양한 사용 사례에 대한 유연성 등의 주요 이점을 통해 containerd는 현대적인 컨테이너 인프라의 핵심 구성 요소로 자리매김했습니다.

Kubernetes 및 GitOps와 같은 다른 클라우드 네이티브 기술과 함께 containerd를 활용함으로써 조직은 효율성, 안정성 및 확장성 측면에서 상당한 이점을 누릴 수 있습니다. 또한 containerd의 간소화된 아키텍처와 모듈형 설계를 통해 사용자는 특정 요구 사항에 맞게 컨테이너 런타임을 사용자 정의하고 확장할 수 있습니다.

앞으로 containerd와 주변 생태계는 새로운 기능, 성능 최적화 및 보안 향상을 통해 계속 진화할 것입니다. CNCF의 주요 프로젝트로서 containerd는 클라우드 네이티브 및 컨테이너화된 애플리케이션의 미래를 형성하는 데 중요한 역할을 할 것입니다.

## 주요 용어 설명

- **OCI (Open Container Initiative)**: 컨테이너 런타임 및 이미지 형식을 위한 개방형 표준을 정의하는 프로젝트입니다. OCI 호환성은 다른 도구 및 플랫폼과의 상호 운용성을 보장합니다.
- **CRI (Container Runtime Interface)**: Kubernetes와 컨테이너 런타임 간의 통신을 위한 표준 API입니다. CRI를 통해 Kubernetes는 다양한 컨테이너 런타임과 원활하게 작동할 수 있습니다.
- **Rootless 모드**: 루트 권한 없이 컨테이너를 실행할 수 있는 containerd의 보안 기능입니다. 이는 컨테이너 실행과 관련된 잠재적인 위험을 줄이는 데 도움이 됩니다.
- **Seccomp (Secure Computing Mode)**: 컨테이너 내에서 사용 가능한 시스템 호출을 제한하여 보안을 강화하는 Linux 커널 기능입니다. Containerd는 Seccomp 프로파일을 통해 컨테이너의 공격 표면을 줄일 수 있습니다.

## 최신 동향

- **Rootless 컨테이너**: Rootless 모드에서 실행되는 컨테이너는 컨테이너 보안과 다중 테넌시를 개선하는 데 점점 더 많은 관심을 받고 있습니다. Rootless 컨테이너는 컨테이너와 호스트 시스템 간의 격리를 향상시킵니다.
- **Kata Containers와의 통합**: Kata Containers는 경량 가상 머신을 사용하여 컨테이너를 실행하는 보안 중심의 런타임입니다. Containerd는 Kata Containers와 통합되어 추가적인 격리 및 보안 계층을 제공할 수 있습니다.
- **SPIRE와의 통합**: SPIRE (SPIFFE Runtime Environment)는 컨테이너 및 마이크로서비스를 위한 ID 및 보안 프레임워크입니다. containerd는 SPIRE와의 통합을 통해 컨테이너의 인증, 권한 부여 및 암호화를 개선할 수 있습니다.
- **Window에서의 컨테이너 지원**: containerd는 Windows 컨테이너에 대한 지원을 지속적으로 개선하고 있습니다. 이를 통해 조직은 Windows 및 Linux 워크로드에 걸쳐 일관된 컨테이너 런타임을 활용할 수 있습니다.

containerd는 성능, 보안 및 유연성을 갖춘 효율적인 컨테이너 런타임을 제공함으로써 현대적인 애플리케이션 개발 및 배포에 필수적인 역할을 하고 있습니다. Kubernetes 및 클라우드 네이티브 기술과 함께 사용할 때 containerd는 대규모로 컨테이너화된 워크로드를 관리하기 위한 강력하고 안정적인 기반을 제공합니다.

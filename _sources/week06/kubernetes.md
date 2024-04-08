# 쿠버네티스

[쿠버네티스](https://kubernetes.io "쿠버네티스")는 컨테이너화된 시스템을 배포하고 확장하기 위한 가장 인기 있는 오케스트레이터입니다. 쿠버네티스를 사용하면 클라우드에서 애플리케이션을 안정적으로 구축하고 배포할 수 있습니다.

이 초보자 가이드에서는 쿠버네티스가 할 수 있는 일과 자신만의 컨테이너화된 솔루션을 실행하기 시작하는 방법을 배울 것입니다.

## 쿠버네티스란?

쿠버네티스는 컨테이너 배포 작업을 자동화하는 오픈 소스 시스템입니다. 원래 구글에서 개발되었지만 현재는 [클라우드 네이티브 컴퓨팅 재단(CNCF)](https://www.cncf.io)의 일부로 유지 관리되고 있습니다.

쿠버네티스가 두각을 나타내는 이유는 프로덕션에서 컨테이너를 사용할 때 발생하는 많은 과제를 해결하기 때문입니다. 무제한의 컨테이너 복제본을 쉽게 실행하고, 여러 물리적 호스트에 분산시키며, 사용자가 서비스에 접근할 수 있도록 네트워킹을 설정할 수 있습니다.

대부분의 개발자는 [도커](https://docker.com)로 컨테이너 여정을 시작합니다. 이는 포괄적인 도구이지만 비교적 저수준이며 한 번에 하나의 컨테이너와 상호 작용하는 CLI 명령에 의존합니다. 쿠버네티스는 협업할 수 있는 선언적 스키마를 사용하여 애플리케이션과 인프라를 정의하기 위한 훨씬 더 높은 수준의 추상화를 제공합니다.

## 쿠버네티스 기능

쿠버네티스는 컨테이너 및 관련 인프라를 실행하기 위한 전체 기능 범위를 포함하는 포괄적인 기능 세트를 가지고 있습니다:

- **자동화된 롤아웃, 스케일링 및 롤백** - 쿠버네티스는 지정된 수의 복제본을 자동으로 생성하고, 적합한 하드웨어에 분산시키며, 노드가 다운되면 컨테이너를 재조정하는 조치를 취합니다. CPU 사용량과 같은 변화하는 조건에 대응하여 복제본 수를 즉시 확장할 수 있습니다.
- **서비스 디스커버리, 로드 밸런싱 및 네트워크 인그레스** - 쿠버네티스는 내부 서비스 디스커버리와 공용 컨테이너 노출을 포함하는 완전한 네트워킹 솔루션을 제공합니다.
- **상태 비저장 및 상태 저장 애플리케이션** - 쿠버네티스는 초기에 상태 비저장 컨테이너에 중점을 두었지만 이제 상태 저장 앱을 나타내기 위한 [내장 객체](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset)도 가지고 있습니다. 쿠버네티스에서 모든 종류의 애플리케이션을 실행할 수 있습니다.
- **스토리지 관리** - 영구 스토리지는 클라우드, 네트워크 공유 또는 로컬 파일 시스템에 상관없이 제공업체 전반에 걸쳐 작동하는 일관된 인터페이스에 의해 추상화됩니다.
- **선언적 상태** - 쿠버네티스는 YAML 파일의 객체 매니페스트를 사용하여 클러스터에서 생성하려는 상태를 정의합니다. 매니페스트를 적용하면 쿠버네티스에 클러스터를 대상 상태로 자동 전환하도록 지시합니다. 원하는 변경 사항을 수동으로 스크립트로 작성할 필요가 없습니다.
- **환경 전반에 걸쳐 작동** - 쿠버네티스는 클라우드, 엣지 또는 개발자 워크스테이션에서 사용할 수 있습니다. 다양한 사용 사례에 맞는 많은 다른 배포판을 사용할 수 있습니다. AWS 및 Google Cloud와 같은 주요 클라우드 제공업체는 관리형 쿠버네티스 서비스를 제공하는 반면 [Minikube](https://minikube.sigs.k8s.io/docs) 및 [K3s](https://k3s.io)와 같은 단일 노드 배포판은 로컬 사용에 적합합니다.
- **고도로 확장 가능** - 쿠버네티스는 많은 기능을 제공하지만 확장을 사용하여 더 많은 기능을 추가할 수 있습니다. [사용자 정의 객체 유형](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources), 컨트롤러 및 운영자를 생성하여 자신만의 워크로드를 지원할 수 있습니다.

이렇게 많은 기능을 사용할 수 있기 때문에 쿠버네티스는 선언적 구성으로 컨테이너를 배포하려는 모든 상황에 이상적입니다.

## 쿠버네티스의 작동 방식

쿠버네티스는 여러 가지 이동 부품을 가지고 있기 때문에 복잡하다는 평판을 얻고 있습니다. 이들이 어떻게 조화를 이루는지 기본 사항을 이해하면 쿠버네티스 여정을 시작하는 데 도움이 될 것입니다.

쿠버네티스 환경을 **클러스터**라고 합니다. 여기에는 하나 이상의 **노드**가 포함됩니다. 노드는 단순히 컨테이너를 실행할 머신입니다. 물리적 하드웨어 또는 가상 머신일 수 있습니다.

노드뿐만 아니라 클러스터에는 **컨트롤 플레인**도 있습니다. 컨트롤 플레인은 전체 클러스터의 작업을 조정합니다. 새 컨테이너를 사용 가능한 노드에 예약하고 상호 작용하는 API 서버를 제공합니다. [여러 컨트롤 플레인 인스턴스](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability)로 클러스터를 실행하여 복원력이 더 뛰어난 고가용성 설정을 만들 수 있습니다.

가장 중요한 쿠버네티스 구성 요소는 다음과 같습니다:

- **kube-apiserver** - 컨트롤 플레인에서 API 서버를 실행하는 부분입니다. 실행 중인 쿠버네티스 클러스터와 상호 작용하는 유일한 방법입니다. [Kubectl CLI](https://kubernetes.io/docs/tasks/tools) 또는 HTTP 클라이언트를 사용하여 API 서버에 명령을 실행할 수 있습니다.
- **kube-controller-manager** - 컨트롤러 매니저는 쿠버네티스의 내장 컨트롤러를 시작하고 실행합니다. [컨트롤러](https://kubernetes.io/docs/concepts/architecture/controller)는 기본적으로 클러스터의 변경 사항 후에 작업을 적용하는 이벤트 루프입니다. API 요청이나 증가된 로드와 같은 이벤트에 대응하여 객체를 생성, 확장 및 삭제합니다.
- **kube-scheduler** - 스케줄러는 새 파드(컨테이너)를 클러스터의 노드에 할당합니다. 파드의 요구 사항을 충족할 수 있는 노드를 설정한 다음 성능과 안정성을 최대화하기 위해 가장 최적의 배치를 선택합니다.
- **kubelet** - 큐블릿은 각 노드에서 실행되는 작업자 프로세스입니다. 쿠버네티스 컨트롤 플레인과의 통신을 유지하여 지침을 받습니다. 큐블릿은 예약 요청에 대응하여 컨테이너 이미지를 가져오고 컨테이너를 시작할 책임이 있습니다.
- **kube-proxy** - 프록시는 개별 노드에서 찾을 수 있는 또 다른 구성 요소입니다. 트래픽이 클러스터의 서비스에 도달할 수 있도록 호스트의 네트워킹 시스템을 구성합니다.

[Kubectl](https://kubernetes.io/docs/tasks/tools)은 일반적으로 작동하는 쿠버네티스 환경의 최종 부분입니다. 이 CLI를 사용하여 클러스터 및 객체와 상호 작용해야 합니다. 클러스터가 설정되면 [공식 대시보드](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard) 또는 타사 솔루션을 설치하여 GUI에서 쿠버네티스를 제어할 수도 있습니다.

## 설치 및 설정

제공되는 배포판의 범위로 인해 쿠버네티스를 시작하는 방법은 매우 다양합니다. [공식 배포판을 사용하여 클러스터 생성](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm)은 비교적 번거로운 작업이므로 대부분의 사람들은 [Minikube](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm), [MicroK8s](https://microk8s.io), [K3s](https://k3s.io) 또는 [Kind](https://kind.sigs.k8s.io)와 같은 패키지 솔루션을 사용합니다.

이 튜토리얼에서는 K3s를 사용할 것입니다. 이는 모든 쿠버네티스 구성 요소를 단일 바이너리로 번들로 제공하는 초경량 쿠버네티스 배포판입니다. 다른 옵션과 달리 설치해야 할 종속성이나 실행할 무거운 VM이 없습니다. 또한 쿠버네티스 명령을 실행하는 데 사용할 Kubectl CLI도 포함되어 있습니다.

다음 명령을 실행하면 K3s가 머신에 설치됩니다:

```bash
curl -sfL https://get.k3s.io | sh -
```

```
[INFO]  systemd: Starting k3s
```

자동으로 사용 가능한 최신 쿠버네티스 릴리스를 다운로드하고 K3s에 대한 시스템 서비스를 등록합니다.

설치 후 다음 명령을 실행하여 자동 생성된 Kubectl 구성 파일을 .kube 디렉토리에 복사합니다:

```bash
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $USER:$USER ~/.kube/config
```

이제 다음 명령을 실행하여 K3s에 이 구성 파일을 사용하도록 지시합니다:

```bash
export KUBECONFIG=~/.kube/config
```

로그인 후 변경 사항을 자동으로 적용하려면 이 줄을 `~/.profile` 또는 `~/.bashrc` 파일에 추가할 수 있습니다.

다음으로 이 명령을 실행합니다:

```bash
kubectl get nodes
```

```
NAME       STATUS   ROLES                  AGE    VERSION
ubuntu22   Ready    control-plane,master   102s   v1.24.4+k3s1
```

머신의 호스트 이름으로 명명된 단일 노드가 나타나는 것을 볼 수 있습니다. 노드가 준비 상태로 표시되므로 이제 쿠버네티스 클러스터를 사용할 수 있습니다!

## 쿠버네티스 기본 용어 및 개념

클러스터가 실행 중이지만 무엇을 할 수 있을까요? 계속하기 전에 몇 가지 핵심 쿠버네티스 용어에 익숙해지는 것이 좋습니다.

### 노드

노드는 쿠버네티스 클러스터를 구성하는 물리적 머신을 나타냅니다. 생성한 컨테이너를 실행합니다. 물리적 하드웨어 또는 가상 머신일 수 있습니다.

쿠버네티스는 노드의 상태를 추적하고 각 노드를 객체로 노출합니다. 위의 예제에서 Kubectl을 사용하여 노드 목록을 검색했습니다.

새로운 클러스터에는 노드가 하나뿐이지만 쿠버네티스는 [최대 5,000개의 노드](https://kubernetes.io/docs/setup/best-practices/cluster-large)를 지원한다고 광고합니다. 이론적으로 [더 확장](https://openai.com/blog/scaling-kubernetes-to-7500-nodes)하는 것도 가능합니다.

### 네임스페이스

[네임스페이스](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces)는 다양한 리소스 그룹을 격리합니다. 리소스의 가시성을 범위로 지정하여 이름 충돌을 방지합니다.

동일한 네임스페이스 내에서 이름이 같은 두 객체를 생성하는 것은 금지되어 있습니다. 예를 들어 기본 네임스페이스에 있는 경우 데이터베이스라는 두 개의 파드를 만들 수 없습니다. 네임스페이스는 리소스의 논리적 분리를 제공하여 이 문제를 해결합니다. app-1과 app-2라는 두 개의 네임스페이스에는 각각 데이터베이스라는 파드가 포함될 수 있으며 충돌이 발생하지 않습니다.

네임스페이스는 유연하며 다양한 방식으로 사용할 수 있습니다. 클러스터의 각 워크로드에 대해 네임스페이스를 생성하는 것이 좋습니다. [역할 기반 액세스 제어](https://kubernetes.io/docs/reference/access-authn-authz/rbac)를 적용하여 사용자와 팀 간에 리소스를 나누는 데에도 네임스페이스를 사용할 수 있습니다.

### 파드

[파드](https://kubernetes.io/docs/concepts/workloads/pods)는 쿠버네티스의 기본 컴퓨팅 단위입니다. 파드는 컨테이너와 유사하지만 몇 가지 주요 차이점이 있습니다. 파드에는 여러 컨테이너가 포함될 수 있으며 각 컨테이너는 컨텍스트를 공유합니다. 전체 파드는 항상 동일한 노드에 예약됩니다. 파드 내의 컨테이너는 긴밀하게 결합되어 있으므로 API 및 데이터베이스와 같은 애플리케이션의 각 고유 부분에 대해 새 파드를 생성해야 합니다.

단순한 상황에서 파드는 일반적으로 애플리케이션이 실행하는 컨테이너와 일대일로 매핑됩니다. 더 고급 사례에서는 [초기화 컨테이너](https://kubernetes.io/docs/concepts/workloads/pods/init-containers)와 [임시 컨테이너](https://kubernetes.io/docs/concepts/workloads/pods/ephemeral-containers)를 사용하여 시작 동작을 사용자 정의하고 자세한 디버깅을 제공할 수 있습니다.

### 레플리카셋

[레플리카셋](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset)은 파드를 일관되게 복제하는 데 사용됩니다. 설정된 수의 복제본이 항상 실행 중일 것이라는 보장을 제공합니다. 노드가 오프라인 상태가 되거나 파드가 비정상이 되면 쿠버네티스는 자동으로 새 파드 인스턴스를 예약하여 지정된 복제본 수를 유지합니다.

### 디플로이먼트

[디플로이먼트](https://kubernetes.io/docs/concepts/workloads/controllers/deployment)는 선언적 업데이트 및 롤백 지원과 함께 레플리카셋을 래핑합니다. 이는 제어하기 쉬운 더 높은 수준의 추상화입니다.

디플로이먼트 객체를 사용하면 파드 집합의 원하는 상태를 지정할 수 있습니다. 여기에는 실행할 복제본 수가 포함됩니다. 디플로이먼트를 수정하면 필요한 변경 사항을 자동으로 감지하고 필요에 따라 레플리카셋의 크기를 조정합니다. [롤아웃을 일시 중지](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#pausing-and-resuming-a-deployment)하거나 이전 버전으로 되돌릴 수 있으며, 이는 일반 레플리카셋에서는 사용할 수 없는 기능입니다.

### 서비스

`쿠버네티스 서비스`는 파드를 네트워크에 노출하는 데 사용됩니다. 클러스터 내부 또는 외부에서 파드에 대한 정의된 액세스를 허용합니다.

[인그레스](https://kubernetes.io/docs/concepts/services-networking/ingress)는 밀접하게 관련된 객체입니다. 이들은 `로드 밸런서`를 통해 서비스에 대한 HTTP 경로를 설정하는 데 사용됩니다. 인그레스는 또한 TLS 인증서로 보호되는 HTTPS 트래픽도 지원합니다.

### 작업

쿠버네티스 [작업](https://kubernetes.io/docs/concepts/workloads/controllers/job)은 파드 집합을 생성하고 종료될 때까지 기다리는 객체입니다. 지정된 수의 파드가 성공적으로 종료될 때까지 실패한 파드를 재시도합니다. 그런 다음 작업이 완료된 것으로 표시됩니다.

작업은 클러스터 내에서 임시 작업을 실행하기 위한 메커니즘을 제공합니다. 쿠버네티스는 또한 [cron과 유사한](https://en.wikipedia.org/wiki/Cron) 예약 지원으로 작업을 래핑하는 `CronJobs`도 제공합니다. 이를 통해 배치 활동, 백업 및 애플리케이션에 필요한 기타 예약된 작업을 수용하기 위해 작업을 정기적으로 자동 실행할 수 있습니다.

### 볼륨

[볼륨](https://kubernetes.io/docs/concepts/storage/volumes)은 외부 파일 시스템 스토리지를 파드 내부에 마운트합니다. 서로 다른 클라우드 제공업체의 스토리지 구현 간의 차이점을 추상화합니다.

볼륨은 파드 간에 공유될 수 있습니다. 이를 통해 쿠버네티스는 파드가 종료되거나 재예약된 후에도 데이터를 보존해야 하는 상태 저장 애플리케이션을 실행할 수 있습니다. 클러스터에서 데이터베이스 또는 파일 서버를 실행할 때마다 볼륨을 사용해야 합니다.

### 시크릿 및 컨피그맵

`시크릿`은 API 키, 인증서 및 기타 종류의 자격 증명과 같은 중요한 데이터를 클러스터에 주입하는 데 사용됩니다. 환경 변수로 파드에 제공하거나 볼륨에 마운트된 파일로 제공할 수 있습니다.

`컨피그맵`은 비중요 정보에 대한 유사한 개념입니다. 이러한 객체는 앱에 필요한 모든 일반 설정을 저장해야 합니다.

### 데몬셋

`쿠버네티스 데몬셋`은 클러스터의 각 노드에서 파드의 복사본을 안정적으로 실행하는 데 사용됩니다. 새 노드가 추가되면 파드의 인스턴스가 자동으로 시작됩니다. 더 고급 상황에서는 [데몬셋 파드를 특정 노드에서만 실행되도록 제한](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/#running-pods-on-select-nodes)할 수 있습니다.

데몬셋은 클러스터에 전역 기능을 추가할 때 유용합니다. 데몬셋은 종종 모니터링 서비스와 로그 수집 에이전트를 실행하는 데 사용됩니다. 이러한 워크로드를 데몬셋에 배치하면 애플리케이션의 파드 옆에서 항상 실행 중일 것임을 보장합니다. 이는 파드가 예약된 노드에 관계없이 메트릭과 로그가 수집되도록 합니다.

### 네트워킹 정책

쿠버네티스는 `파드 간의 네트워크 트래픽 흐름을 제어하기 위한 정책 기반 시스템`을 지원합니다. 공격자가 인프라 전체로 이동하는 것을 방지하기 위해 중요한 파드를 다른 리소스로부터 격리할 수 있습니다.

네트워크 정책은 하나 이상의 일치하는 파드를 대상으로 하는 객체로 표현됩니다. 각 파드는 수신 및 송신 정책의 주체가 될 수 있습니다. 수신 정책은 들어오는 트래픽이 허용되는지 여부를 정의하는 반면 송신 규칙은 나가는 흐름에 영향을 줍니다. 두 파드 간의 통신은 어느 파드의 네트워킹 정책도 다른 파드에서의 수신 또는 송신을 거부하지 않는 경우에만 허용됩니다.

## Kubectl을 사용하여 쿠버네티스와 상호 작용

이제 기본 사항에 익숙해졌으므로 Kubectl을 사용하여 클러스터에 워크로드를 추가할 수 있습니다. 다음은 몇 가지 핵심 명령에 대한 빠른 참조입니다.

### 파드 나열

이 명령은 클러스터의 파드를 표시합니다:

```bash
kubectl get pods
```

```
No resources found in default namespace
```

`-n` 또는 `--namespace` 플래그를 사용하여 네임스페이스를 지정합니다:

```bash
ubectl get pods -n demo
```

```
No resources found in demo namespace
```

또는 `--all-namespaces`를 지정하여 모든 네임스페이스에서 파드를 가져옵니다:

```bash
kubectl get pods --all-namespaces
```

```
NAMESPACE     NAME                                      READY   STATUS      RESTARTS   AGE
kube-system   coredns-b96499967-4xdpg                   1/1     Running     0          114m
...
```

여기에는 쿠버네티스 시스템 구성 요소가 포함됩니다.

### 파드 생성

다음 명령을 사용하여 파드를 생성합니다:

```bash
kubectl run nginx --image nginx:latest
```

```
pod/nginx created
```

nginx:latest 컨테이너 이미지를 실행할 nginx라는 파드를 시작합니다.

### 디플로이먼트 생성

디플로이먼트를 생성하면 컨테이너의 여러 복제본을 확장할 수 있습니다:

```bash
kubectl create deployment nginx --image nginx:latest --replicas 3
```

```
deployment.apps/nginx created
```

nginx:latest 이미지를 실행하는 파드 3개가 생성된 것을 볼 수 있습니다:

```bash
kubectl get pods
```

```
NAME                     READY   STATUS    RESTARTS   AGE
nginx-7597c656c9-4qs55   1/1     Running   0          51s
nginx-7597c656c9-gdjl9   1/1     Running   0          51s
nginx-7597c656c9-7sxrc   1/1     Running   0          51s
```

### 디플로이먼트 스케일링

이제 이 명령을 사용하여 복제본 수를 늘립니다:

```bash
kubectl scale deployment nginx --replicas 5
```

```
deployment.apps/nginx scaled
```

쿠버네티스는 추가 용량을 제공하기 위해 두 개의 추가 파드를 생성했습니다:

```bash
kubectl get pods
```

```
NAME                     READY   STATUS    RESTARTS   AGE
nginx-7597c656c9-4qs55   1/1     Running   0          2m26s
nginx-7597c656c9-gdjl9   1/1     Running   0          2m26s
nginx-7597c656c9-7sxrc   1/1     Running   0          2m26s
nginx-7597c656c9-kwm6q   1/1     Running   0          2s
nginx-7597c656c9-nwf2s   1/1     Running   0          2s
```

### 서비스 노출

이제 이 NGINX 서버에 접근할 수 있도록 만들어 보겠습니다.

다음 명령을 실행하여 파드를 실행 중인 노드의 포트에 노출되는 서비스를 생성합니다:

```bash
kubectl expose deployment/nginx --port 80 --type NodePort
```

```
service/nginx exposed
```

다음 명령을 실행하여 할당된 포트를 확인합니다:

```bash
kubectl get services
```

```
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.43.0.1      <none>        443/TCP        121m
nginx        NodePort    10.43.149.39   <none>        80:30226/TCP   3s
```

포트는 30226입니다. 브라우저에서 <node-ip>:30226을 방문하면 기본 NGINX 랜딩 페이지가 표시됩니다.

이 튜토리얼에서 생성한 단일 노드 K3s 클러스터를 따라 하고 있다면 <node-ip>로 localhost를 사용할 수 있습니다. 그렇지 않으면 `get nodes 명령을 실행`하고 표시되는 INTERNAL-IP를 사용하세요.

```bash
kubectl get nodes -o wide
```

```
NAME       STATUS   ROLES                  AGE    VERSION        INTERNAL-IP
ubuntu22   Ready    control-plane,master   124m   v1.24.4+k3s1   192.168.122.210
```

### 포트 포워딩 사용

`Kubectl의 통합 포트 포워딩 기능`을 사용하면 노드 포트에 바인딩하지 않고도 서비스에 액세스할 수 있습니다. 첫 번째 서비스를 삭제하고 `--type` 플래그 없이 새 서비스를 생성합니다:

```bash
kubectl delete service nginx
```

```
service/nginx deleted
```

```bash
kubectl expose deployment/nginx –port 80
```

```
service/nginx exposed
```

이렇게 하면 클러스터 내부에서 내부 IP로 액세스할 수 있는 ClusterIP 서비스가 생성됩니다.

다음 명령을 실행하여 서비스의 세부 정보를 검색합니다:

```bash
kubectl get services
```

```
NAME     	TYPE    	CLUSTER-IP   	EXTERNAL-IP   PORT(S)   AGE
nginx   	ClusterIP   10.100.191.238   <none>    	80/TCP	2s
```

서비스는 클러스터 내에서 10.100.191.238:80으로 액세스할 수 있습니다.

다음 명령을 사용하여 로컬 머신에서 이 주소에 접근할 수 있습니다:

```bash
kubectl port-forward service/nginx 8080:80
```

브라우저에서 localhost:8080을 방문하면 NGINX 랜딩 페이지가 표시됩니다. Kubectl은 클러스터 내의 서비스로 트래픽을 리디렉션하고 있습니다. 완료되면 터미널에서 Ctrl+C를 눌러 포트 포워딩 세션을 중지할 수 있습니다.

포트 포워딩은 서비스 없이도 작동합니다. 다음 명령을 사용하여 디플로이먼트의 파드에 직접 연결할 수 있습니다:

```bash
kubectl port-forward deployment/nginx 8080:80
```

localhost:8080을 방문하면 다시 NGINX 랜딩 페이지가 표시되지만 이번에는 서비스를 거치지 않습니다.

### YAML 파일 적용

마지막으로 선언적 YAML 파일을 클러스터에 적용하는 방법을 살펴보겠습니다. 먼저 파드에 대한 `간단한 쿠버네티스 매니페스트`를 작성합니다:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx:latest
```

이 매니페스트를 nginx.yaml에 저장하고 `kubectl apply`를 실행하여 파드를 자동으로 생성합니다:

```bash
kubectl apply -f nginx.yaml
```

```
pod/nginx created
```

파일을 수정한 후 명령을 반복하여 클러스터에 변경 사항을 적용할 수 있습니다.

이제 Kubectl을 사용하여 쿠버네티스와 상호 작용하는 기본 사항에 익숙해졌습니다!

## 핵심 사항

쿠버네티스는 선도적인 컨테이너 오케스트레이터입니다. 이 기사에서는 쿠버네티스의 기능을 살펴보고, 작동 방식을 알아보았으며, 애플리케이션을 구성하는 데 사용할 가장 중요한 객체 유형을 다루었습니다.

이론을 넘어서 인기 있는 K3s 배포판으로 자신만의 쿠버네티스 클러스터를 시작하는 방법도 보았습니다. 이제 최소한의 수동 구성으로 자신만의 컨테이너화된 시스템을 대규모로 구축하고 실행할 수 있습니다.

**주요 용어 설명:**

- **컨트롤 플레인**: 클러스터의 상태를 관리하고 API 서버, 컨트롤러 매니저, 스케줄러를 실행하는 쿠버네티스의 핵심 구성 요소입니다.
- **kubectl**: 쿠버네티스 클러스터와 상호 작용하기 위한 명령줄 인터페이스 도구입니다.
- **오케스트레이션**: 컨테이너의 배포, 스케일링 및 관리를 자동화하는 프로세스입니다.
- **클러스터**: 쿠버네티스에 의해 관리되는 노드 그룹으로, 컨테이너화된 워크로드를 실행합니다.
- **매니페스트**: 쿠버네티스 객체의 원하는 상태를 선언적으로 정의하는 YAML 또는 JSON 파일입니다.

쿠버네티스는 지속적으로 발전하고 있으며, 새로운 기능과 개선 사항이 정기적으로 도입되고 있습니다. 쿠버네티스 생태계와 관련 프로젝트가 성장함에 따라 컨테이너 오케스트레이션의 가능성도 확대되고 있습니다.

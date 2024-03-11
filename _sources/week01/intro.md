# [MLOps 소개](https://lecture.entelecheia.ai/lectures/mlops/intro.html)

머신러닝 시스템 설계는 실제 문제를 해결하기 위해 머신러닝 모델과 알고리즘을 만들고 구현하는 과정입니다. 이는 소프트웨어 아키텍처 정의, 적절한 알고리즘 선택, 특정 요구사항을 충족하기 위한 인프라 설계 등을 포함합니다.

MLOps(Machine Learning Operations)는 머신러닝 시스템에 DevOps 원칙을 적용하는 관련 분야입니다. MLOps는 머신러닝 모델의 복잡성과 확장성을 처리할 수 있는 소프트웨어 아키텍처 개발, 모델 학습 및 배포에 필요한 데이터 및 인프라 관리, 효율성과 신뢰성 향상을 위한 프로세스 자동화에 초점을 맞춥니다.

```{mermaid}
:align: center

graph TD
A[1. 데이터 수집] --> B[2. 데이터 준비]
B --> C[3. 특성 선택]
C --> D[4. 모델 학습]
D --> E[5. 모델 검증]
E --> F[6. 모델 통합]
F --> G[7. 모니터링 & 유지보수]
G --> A

style A fill:#f9d5e5
style B fill:#eeac99
style C fill:#e06377
style D fill:#c83349
style E fill:#5b9aa0
style F fill:#d6e4aa
style G fill:#c9a690
```

## 머신러닝 시스템의 숨겨진 기술 부채

실제 머신러닝 시스템을 구축하는 것은 단순히 머신러닝 코드를 작성하는 것 이상을 포함합니다. 성공적인 머신러닝 시스템을 구축하려면 머신러닝 알고리즘과 모델, 데이터 엔지니어링, 소프트웨어 공학, DevOps 등 다양한 영역의 전문 지식이 필요합니다.

```{figure} figs/mlcode.png
---
width: 700px
name: fig-mlcode
---
Only a small fraction of real-world ML systems is composed of the ML code {cite:p}`Sculley:2015`
```

## 머신러닝 시스템의 과제

머신러닝 시스템 설계 및 개발에는 데이터 품질, 데이터 개인정보 보호 및 보안, 모델 선택 및 튜닝, 인프라 및 리소스 제약, 배포 및 통합, 모델 해석 가능성 및 투명성, 지속적인 개선 및 유지 관리 등 다양한 과제가 있습니다. 이러한 과제를 극복하려면 기계 학습 알고리즘 및 모델, 데이터 엔지니어링, 소프트웨어 엔지니어링, DevOps 등 다양한 영역에서 전문성이 필요합니다.

## DevOps와 MLOps 비교

DevOps와 MLOps는 소프트웨어 개발 및 배포 프로세스 개선에 중점을 둔 관련 분야이지만 구별되는 분야입니다. 주요 차이점은 다음과 같습니다:

- MLOps는 머신러닝 모델의 전체 수명 주기에 중점을 두는 반면 DevOps는 소프트웨어 개발 수명 주기에 중점을 둡니다.
- MLOps에는 머신러닝 알고리즘과 모델, 데이터 엔지니어링, 통계 분석 등 더 광범위한 기술과 전문 지식이 필요한 반면 DevOps는 소프트웨어 개발 및 IT 운영 전문 지식이 필요합니다.
- 머신러닝 모델은 전통적인 소프트웨어 애플리케이션보다 더 복잡하며 다른 테스트 및 유지 관리 전략이 필요합니다.

```{figure} figs/mlops-cycle.jpg
---
width: 500px
name: fig-mlops-cycle2
---
The MLOps cycle. Source: [NealAnalytics](https://nealanalytics.com/expertise/mlops/)
```

## 지속적인 제공 및 자동화 파이프라인

MLOps의 핵심 구성 요소 중 하나는 머신러닝 모델 구축 및 배포 프로세스를 간소화하기 위해 지속적인 제공 및 자동화 파이프라인을 사용하는 것입니다. Apache Airflow, TensorFlow, Kubernetes, Prometheus 등의 자동화 도구를 사용하여 데이터 수집, 준비, 기능 엔지니어링, 모델 교육 및 검증, 배포, 모니터링 및 유지 관리를 자동화할 수 있습니다.

## MLOps가 중요한 이유

MLOps는 머신러닝 프로젝트 구현 및 유지 관리가 어려울 수 있기 때문에 중요합니다. MLOps는 데이터 수집 및 전처리부터 모델 배포 및 유지 관리에 이르기까지 머신러닝 모델의 전체 수명 주기를 관리하기 위한 프레임워크를 제공함으로써 이러한 과제를 극복하는 데 도움이 됩니다.

## MLOps 구현 방법

MLOps 구현에는 조직의 요구 사항과 머신러닝 프로세스의 성숙도에 따라 다양한 방법이 있습니다. MLOps 구현은 현재 상태 평가, 요구사항 정의, 올바른 도구 선택, 프로세스 자동화, 모니터링 및 유지 관리, 머신러닝 모델의 지속적인 교육 및 테스트 수행 등의 단계가 포함됩니다.

구글에서 제안한 MLOps 성숙도 수준은 다음과 같습니다:

- MLOps 레벨 0: ML 시작 단계의 기업을 위한 수동적이고 대화식 프로세스
- MLOps 레벨 1: 생산에서 모델의 지속적인 교육을 달성하기 위해 ML 파이프라인 자동화
- MLOps 레벨 2: 강력한 자동화된 CI/CD 시스템을 포함하며 기술 기반 회사에 적합

## 종단간 MLOps 솔루션

다양한 MLOps 플랫폼과 도구가 있으며, 각각 장단점이 있습니다. 대표적인 MLOps 솔루션으로는 Algorithmia, Allegro.io, Cnvrg.io, Dataiku, Datarobot, H2O, Iguazio, Kubeflow, Pachyderm, Polyaxon, Valohai 등이 있습니다.

최근에는 MLOps 분야가 빠르게 발전하고 있으며, 클라우드 기반 MLOps 서비스가 각광받고 있습니다. 대표적으로 AWS의 SageMaker, GCP의 Vertex AI, Azure의 Azure Machine Learning 등이 있습니다. 이러한 서비스는 데이터 준비, 모델 학습, 배포, 모니터링 등 머신러닝 워크플로우의 전 과정을 통합하여 제공합니다.

또한 최근에는 MLOps와 데이터 품질 관리, 모델 해석 가능성, 공정성 등 책임감 있는 AI(Responsible AI)에 대한 관심도 높아지고 있습니다. 머신러닝 모델의 편향성을 최소화하고 설명 가능성을 높이기 위한 다양한 기술과 방법론이 연구되고 있습니다.

앞으로도 MLOps 분야는 지속적으로 발전할 것으로 보이며, 기업들의 AI/ML 프로젝트 성공을 위해 MLOps의 중요성은 더욱 커질 것으로 예상됩니다. MLOps를 효과적으로 적용하기 위해서는 조직 문화, 프로세스, 기술 스택 등을 종합적으로 고려해야 할 것입니다.

## References

```{bibliography}
:filter: docname in docnames
```

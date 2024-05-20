# LangChain

[LangChain](https://www.langchain.com "Opens in a new window")은 언어 모델의 능력을 활용하여 애플리케이션을 만드는 데 특화된 혁신적인 프레임워크입니다. LangChain은 컨텍스트를 이해하고 복잡한 추론 작업을 수행할 수 있는 애플리케이션을 개발하기 위해 개발자에게 필요한 도구를 제공합니다.

LangChain을 사용하면 애플리케이션이 프롬프트 지침이나 응답의 내용을 이해하고, 어떻게 응답하거나 어떤 행동을 취할지 결정하는 등의 복잡한 추론 작업을 수행할 수 있습니다. LangChain은 개념에서 실행까지의 과정을 단순화하는 다양한 구성 요소를 통해 지능형 애플리케이션 개발을 통합된 방식으로 지원합니다.

문서 데이터를 처리하는 LangChain의 유용성을 논의할 때, 워크플로우 자동화의 힘을 강조하는 것이 중요합니다. 스캔된 문서를 수동 작업 없이 실행 가능한 데이터로 변환한다고 상상해 보세요. 우리의 직관적인 플랫폼을 통해 **LangChain**을 다양한 앱과 서비스에 연결하여 프로세스를 간소화하고 귀중한 시간을 절약할 수 있습니다. 몇 분 안에 강력한 AI 강화 워크플로우를 구축하는 방법을 알아보세요.

## LangChain 이해하기

LangChain은 단순한 프레임워크 이상으로, 여러 필수 요소로 구성된 완전한 생태계입니다.

- **LangChain Libraries**: 파이썬과 자바스크립트로 제공되는 이 라이브러리는 LangChain의 중추 역할을 하며, 다양한 구성 요소를 위한 인터페이스와 통합을 제공합니다. 이러한 라이브러리는 구성 요소를 일관된 체인과 에이전트로 결합하는 기본 런타임을 제공하며, 즉시 사용할 수 있는 구현을 포함합니다.
- **LangChain Templates**: 다양한 작업에 맞춘 배포 가능한 참조 아키텍처 모음입니다. 챗봇을 만들거나 복잡한 분석 도구를 구축하든, 이 템플릿은 견고한 시작점을 제공합니다.
- **LangServe**: LangChain 체인을 REST API로 배포할 수 있는 다목적 라이브러리입니다. 이 도구는 LangChain 프로젝트를 접근 가능하고 확장 가능한 웹 서비스로 전환하는 데 필수적입니다.
- **LangSmith**: 개발자 플랫폼으로, LLM 프레임워크로 구축된 체인을 디버그, 테스트, 평가 및 모니터링하도록 설계되었습니다. LangChain과의 원활한 통합을 통해 개발자가 애플리케이션을 정제하고 완성하는 데 있어 필수적인 도구입니다.

이러한 구성 요소들은 개발, 배포, 운영을 쉽게 만들어 줍니다. LangChain을 사용하면 라이브러리를 사용하여 애플리케이션을 작성하고, 템플릿을 참조하여 가이드를 받을 수 있습니다. LangSmith를 사용하면 체인을 검사, 테스트 및 모니터링하여 애플리케이션이 지속적으로 개선되고 배포 준비가 되도록 할 수 있습니다. 마지막으로 LangServe를 사용하면 어떤 체인이든 API로 쉽게 전환하여 배포할 수 있습니다.

다음 섹션에서는 LangChain을 설정하고 지능형 언어 모델 기반 애플리케이션을 만드는 여정을 시작하는 방법에 대해 자세히 알아보겠습니다.

## 설치 및 설정

LangChain의 세계로 들어갈 준비가 되셨나요? 설정은 간단하며, 이 가이드에서는 단계별로 과정을 안내합니다.

LangChain 여정의 첫 번째 단계는 설치입니다. 이를 쉽게 하기 위해 pip 또는 conda를 사용할 수 있습니다. 터미널에서 다음 명령을 실행하세요:

```
pip install langchain
```

---

_최신 기능을 선호하고 모험을 즐기는 분들은 LangChain을 소스에서 직접 설치할 수 있습니다. 저장소를 클론하고 `langchain/libs/langchain` 디렉토리로 이동한 후 다음 명령을 실행하세요:_

```
pip install -e .
```

_실험적 기능이 필요한 경우, `langchain-experimental`을 설치하는 것이 좋습니다. 이는 연구 및 실험 목적으로 설계된 최신 코드를 포함하고 있습니다. 다음 명령으로 설치할 수 있습니다:_

```
pip install langchain-experimental
```

_LangChain CLI는 LangChain 템플릿과 LangServe 프로젝트를 다루기 위한 편리한 도구입니다. LangChain CLI를 설치하려면 다음 명령을 사용하세요:_

```
pip install langchain-cli
```

_LangServe는 LangChain 체인을 REST API로 배포하는 데 필수적입니다. LangChain CLI와 함께 설치됩니다._

---

LangChain은 종종 모델 제공자, 데이터 스토어, API 등과의 통합이 필요합니다. 이번 예제에서는 OpenAI의 모델 API를 사용할 것입니다. OpenAI Python 패키지를 설치하려면 다음 명령을 실행하세요:

```
pip install openai
```

API에 접근하려면 OpenAI API 키를 환경 변수로 설정하세요:

```
export OPENAI_API_KEY="your_api_key"
```

또는 파이썬 환경에서 직접 키를 전달할 수 있습니다:

```
import os
os.environ['OPENAI_API_KEY'] = 'your_api_key'
```

LangChain은 모듈을 통해 언어 모델 애플리케이션을 생성할 수 있습니다. 이 모듈은 독립적으로 사용될 수도 있고, 복잡한 용도에 맞게 조합될 수도 있습니다. 이러한 모듈은 다음과 같습니다:

- **Model I/O**: 다양한 언어 모델과의 상호작용을 촉진하고, 입력과 출력을 효율적으로 처리합니다.
- **Retrieval**: 애플리케이션 특정 데이터를 접근하고 상호작용할 수 있게 하여, 동적 데이터 활용을 가능하게 합니다.
- **Agents**: 고수준 지시를 기반으로 적절한 도구를 선택하여 의사 결정 능력을 강화합니다.
- **Chains**: 애플리케이션 개발을 위한 구성 요소로서, 사전에 정의된 재사용 가능한 구성 요소를 제공합니다.
- **Memory**: 여러 체인 실행 간에 애플리케이션 상태를 유지하여 컨텍스트 인식 상호작용을 가능하게 합니다.

각 모듈은 특정 개발 요구 사항을 타겟으로 하여, LangChain을 사용하면 고급 언어 모델 애플리케이션을 만들 수 있는 종합적인 도구 모음을 제공합니다.

또한, **LangChain Expression Language (LCEL)**는 모듈을 쉽게 조합할 수 있는 선언적 방법을 제공하며, 이는 보편적인 Runnable 인터페이스를 사용하여 구성 요소를 체인화할 수 있습니다.

LCEL은 다음과 같이 보입니다:

```python
from langchain.chat_models import ChatOpenAI
from langchain.prompts import ChatPromptTemplate
from langchain.schema import BaseOutputParser

# 예제 체인
chain = ChatPromptTemplate() | ChatOpenAI() | CustomOutputParser()
```

이제 기본 사항을 다루었으므로, 다음을 계속 진행하겠습니다:

- 각 LangChain 모듈을 자세히 살펴봅니다.
- LangChain Expression Language를 사용하는 방법을 배웁니다.
- 일반적인 사용 사례를 탐구하고 이를 구현합니다.
- LangServe를 사용하여 끝에서 끝까지 애플리케이션을 배포합니다.
- LangSmith를 확인하여 디버그, 테스트 및 모니터링합니다.

시작해 봅시다!

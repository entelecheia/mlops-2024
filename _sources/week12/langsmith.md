# LangSmith 소개

LangChain은 LLM 애플리케이션 및 에이전트를 프로토타입하는 데 매우 유용하지만, 이러한 애플리케이션을 실제 환경에 배포하는 것은 매우 어렵습니다. 고품질의 제품을 만들기 위해 프롬프트, 체인, 기타 구성 요소를 많이 수정하고 반복해야 할 것입니다.

이 과정을 돕기 위해 LangSmith가 도입되었습니다. LangSmith는 LLM 애플리케이션을 디버깅, 테스트 및 모니터링할 수 있는 통합 플랫폼입니다.

언제 유용할까요? 새로운 체인, 에이전트 또는 도구 세트를 빠르게 디버깅하거나, 구성 요소(체인, LLM, 검색 도구 등)가 어떻게 연결되고 사용되는지를 시각화하거나, 단일 구성 요소에 대해 다양한 프롬프트와 LLM을 평가하거나, 데이터 세트에 대해 주어진 체인을 여러 번 실행하여 일정한 품질을 유지하거나, 사용 흔적을 캡처하고 LLM 또는 분석 파이프라인을 사용하여 인사이트를 생성하려는 경우에 유용할 수 있습니다.

## 전제 조건

1. LangSmith 계정을 만들고 API 키를 생성합니다(왼쪽 하단 모서리 참조).
2. 문서를 살펴보며 플랫폼에 익숙해집니다.

이제 시작해봅시다!

먼저, LangChain이 추적 로그를 기록하도록 환경 변수를 설정합니다. 이를 위해 `LANGCHAIN_TRACING_V2` 환경 변수를 true로 설정합니다. LangChain이 로그를 기록할 프로젝트를 설정하려면 `LANGCHAIN_PROJECT` 환경 변수를 설정합니다(이 값이 설정되지 않으면 기본 프로젝트에 로그가 기록됩니다). 프로젝트가 존재하지 않으면 자동으로 생성됩니다. 또한 `LANGCHAIN_ENDPOINT`와 `LANGCHAIN_API_KEY` 환경 변수를 설정해야 합니다.

참고: Python의 컨텍스트 매니저를 사용하여 추적 로그를 기록할 수도 있습니다:

```python
from langchain.callbacks.manager import tracing_v2_enabled

with tracing_v2_enabled(project_name="My Project"):
    agent.run("How many people live in canada as of 2023?")
```

그러나 이 예제에서는 환경 변수를 사용할 것입니다.

```python
%pip install openai tiktoken pandas duckduckgo-search --quiet

import os
from uuid import uuid4

unique_id = uuid4().hex[0:8]
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_PROJECT"] = f"Tracing Walkthrough - {unique_id}"
os.environ["LANGCHAIN_ENDPOINT"] = "https://api.smith.langchain.com"
os.environ["LANGCHAIN_API_KEY"] = "<YOUR-API-KEY>"  # 자신의 API 키로 업데이트

# 이 튜토리얼에서 에이전트가 사용할 것입니다
os.environ["OPENAI_API_KEY"] = "<YOUR-OPENAI-API-KEY>"
```

LangSmith 클라이언트를 생성하여 API와 상호작용합니다:

```python
from langsmith import Client

client = Client()
```

LangChain 구성 요소를 생성하고 실행을 플랫폼에 기록합니다. 이 예제에서는 일반 검색 도구(DuckDuckGo)에 액세스할 수 있는 ReAct 스타일 에이전트를 생성할 것입니다. 에이전트의 프롬프트는 여기에서 확인할 수 있습니다:

```python
from langchain import hub
from langchain.agents import AgentExecutor
from langchain.agents.format_scratchpad import format_to_openai_function_messages
from langchain.agents.output_parsers import OpenAIFunctionsAgentOutputParser
from langchain.chat_models import ChatOpenAI
from langchain.tools import DuckDuckGoSearchResults
from langchain.tools.render import format_tool_to_openai_function

# 최신 버전의 프롬프트를 가져옵니다
prompt = hub.pull("wfh/langsmith-agent-prompt:latest")

llm = ChatOpenAI(
    model="gpt-3.5-turbo-16k",
    temperature=0,
)

tools = [
    DuckDuckGoSearchResults(
        name="duck_duck_go"
    ),  # DuckDuckGo를 사용하는 일반 인터넷 검색
]

llm_with_tools = llm.bind(functions=[format_tool_to_openai_function(t) for t in tools])

runnable_agent = (
    {
        "input": lambda x: x["input"],
        "agent_scratchpad": lambda x: format_to_openai_function_messages(
            x["intermediate_steps"]
        ),
    }
    | prompt
    | llm_with_tools
    | OpenAIFunctionsAgentOutputParser()
)

agent_executor = AgentExecutor(
    agent=runnable_agent, tools=tools, handle_parsing_errors=True
)
```

우리는 지연을 줄이기 위해 여러 입력에 대해 에이전트를 동시에 실행하고 있습니다. 실행은 백그라운드에서 LangSmith에 기록되므로 실행 지연에 영향을 주지 않습니다:

```python
inputs = [
    "What is LangChain?",
    "What's LangSmith?",
    "When was Llama-v2 released?",
    "What is the langsmith cookbook?",
    "When did langchain first announce the hub?",
]

results = agent_executor.batch([{"input": x} for x in inputs], return_exceptions=True)

results[:2]
```

환경을 성공적으로 설정한 경우, 에이전트 추적이 앱의 프로젝트 섹션에 표시됩니다. 축하합니다!

그러나 에이전트가 도구를 효과적으로 사용하고 있지 않은 것 같습니다. 이를 평가하여 기준선을 확인해보겠습니다.

LangSmith는 실행 로그를 기록하는 것 외에도 LLM 애플리케이션을 테스트하고 평가할 수 있습니다.

이 섹션에서는 LangSmith를 사용하여 벤치마크 데이터 세트를 만들고 에이전트에 AI 지원 평가자를 실행합니다. 몇 가지 단계를 거쳐 수행합니다:

## LangSmith 데이터 세트 생성

아래에서는 LangSmith 클라이언트를 사용하여 위의 입력 질문과 답변 목록에서 데이터 세트를 생성합니다. 이후 새로운 에이전트의 성능을 측정하는 데 사용할 것입니다. 데이터 세트는 테스트 사례로 사용할 수 있는 입력-출력 쌍의 모음입니다:

```python
outputs = [
    "LangChain is an open-source framework for building applications using large language models. It is also the name of the company building LangSmith.",
    "LangSmith is a unified platform for debugging, testing, and monitoring language model applications and agents powered by LangChain",
    "July 18, 2023",
    "The langsmith cookbook is a github repository containing detailed examples of how to use LangSmith to debug, evaluate, and monitor large language model-powered applications.",
    "September 5, 2023",
]

dataset_name = f"agent-qa-{unique_id}"

dataset = client.create_dataset(
    dataset_name,
    description="An example dataset of questions over the LangSmith documentation.",
)

for query, answer in zip(inputs, outputs):
    client.create_example(
        inputs={"input": query}, outputs={"output": answer}, dataset_id=dataset.id
    )
```

## 새 에이전트 초기화 및 벤치마크

LangSmith는 어떤 LLM, 체인, 에이전트 또는 사용자 지정 함수도 평가할 수 있게 합니다. 대화형 에이전트는 상태를 가지고 있으므로(메모리를 가짐) 데이터 세트 실행 간 상태가 공유되지 않도록 체인 팩토리(생성자) 함수를 전달하여 각 호출에 대해 초기화합니다:

```python
# 체인은 상태를 가질 수 있으므로(예: 메모리를 가질 수 있음), 각 데이터 세트 행에 대해 새 체인을 초기화하는 방법을 제공합니다.
# 이는 각 행에 대해 새 체인을 반환하는 팩토리 함수를 전달함으로써 수행됩니다.
def agent_factory(prompt):
    llm_with_tools = llm.bind(
        functions=[format_tool_to_openai_function(t) for t in tools]
    )
    runnable_agent = (
        {
            "input": lambda x: x["input"],
            "agent_scratchpad": lambda x: format_to_openai_function_messages(
                x["intermediate_steps"]
            ),
        }
        | prompt
        | llm_with_tools
        | OpenAIFunctionsAgentOutputParser()
    )
    return AgentExecutor(agent=runnable_agent, tools=tools, handle_parsing_errors=True)
```

## 평가 구성

UI에서 체인의 결과를 수동으로 비교하는 것도 효과적이지만, 시간이 많이 걸릴 수 있습니다. 자동 메트릭 및 AI 지원 피드백을 사용하여 구성 요소의 성능을 평가하는 것이 도움이 될 수 있습니다:

```python
from langchain.evaluation import EvaluatorType
from langchain.smith import RunEvalConfig

evaluation_config = RunEvalConfig(
    evaluators=[
        EvaluatorType.QA,
        EvaluatorType.EMBEDDING_DISTANCE,
        RunEvalConfig.LabeledCriteria("helpfulness"),
        RunEvalConfig.LabeledScoreString(
            {
                "accuracy": """
Score 1: The answer is completely unrelated to the reference.
Score 3: The answer has minor relevance but does not align with the reference.
Score 5: The answer has moderate relevance but contains inaccuracies.
Score 7: The answer aligns with the reference but has minor errors or omissions.
Score 10: The answer is completely accurate and aligns perfectly with the reference."""
            },
            normalize_by=10,
        ),
    ],
    custom_evaluators=[],
)
```

## 에이전트 및 평가자 실행

run_on_dataset (또는 비동기 arun_on_dataset) 함수를 사용하여 모델을 평가합니다. 이 함수는 다음 작업을 수행합니다:

1. 지정된 데이터 세트에서 예제 행을 가져옵니다.
2. 각 예제에 대해 에이전트(또는 사용자 지정 함수)를 실행합니다.
3. 결과 실행 추적 및 해당 참조 예제에 평가자를 적용하여 자동 피드백을 생성합니다.

결과는 LangSmith 앱에서 확인할 수 있습니다:

```python
chain_results = run_on_dataset(
    dataset_name=dataset_name,
    llm_or_chain_factory=functools.partial(agent_factory, prompt=prompt),
    evaluation=evaluation_config,
    verbose=True,
    client=client,
    project_name=f"runnable-agent-test-5d466cbc-{unique_id}",
    tags=[
        "testing-notebook",
        "prompt:5d466cbc",
    ],
)
```

이제 테스트 실행 결과가 있으므로 에이전트를 변경하고 벤치마크를 다시 실행할 수 있습니다. 다른 프롬프트로 다시 시도해보고 결과를 확인해봅시다:

```python
candidate_prompt = hub.pull("wfh/langsmith-agent-prompt:39f3bbd0")

chain_results = run_on_dataset(
    dataset_name=dataset_name,
    llm_or_chain_factory=functools.partial(agent_factory, prompt=candidate_prompt),
    evaluation=evaluation_config,
    verbose=True,
    client=client,
    project_name=f"runnable-agent-test-39f3bbd0-{unique_id}",
    tags=[
        "testing-notebook",
        "prompt:39f3bbd0",
    ],
)
```

LangSmith는 데이터에 액세스하여 CSV 또는 JSONL과 같은 일반 형식으로 내보낼 수 있습니다. 클라이언트를 사용하여 실행을 가져와 추가 분석을 위해 자체 데이터베이스에 저장하거나 다른 사람과 공유할 수도 있습니다. 평가 실행에서 실행 추적을 가져옵니다:

```python
runs = client.list_runs(project_name=chain_results["project_name"], execution_order=1)

# 시간이 지나면 데이터가 채워집니다.
client.read_project(project_name=chain_results["project_name"]).feedback_stats
```

이것은 빠르게 시작하기 위한 가이드였습니다. 하지만 LangSmith를 활용하여 개발 속도를 높이고 더 나은 결과를 도출하는 방법은 많습니다.

LangSmith를 최대한 활용하는 방법에 대한 자세한 정보는 LangSmith 문서를 참조하십시오.

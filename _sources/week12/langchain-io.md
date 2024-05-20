# LangChain - Model I/O

LangChain에서 애플리케이션의 핵심 요소는 언어 모델입니다. 이 모듈은 언어 모델과 효과적으로 인터페이스하는 데 필요한 기본 구성 요소를 제공하여 원활한 통합과 통신을 보장합니다.

## Model I/O의 주요 구성 요소

1. **LLM 및 채팅 모델 (상호 교환 가능)**:

   - **LLM**:
     - **정의**: 순수 텍스트 완성 모델입니다.
     - **입출력**: 텍스트 문자열을 입력받아 텍스트 문자열을 출력합니다.
   - **채팅 모델**:
     - **정의**: 언어 모델을 기반으로 하지만 입력 및 출력 형식이 다릅니다.
     - **입출력**: 채팅 메시지 목록을 입력으로 받아 채팅 메시지를 출력합니다.

2. **프롬프트**: 모델 입력을 템플릿화하고 동적으로 선택 및 관리합니다. 이는 언어 모델의 응답을 유도하는 유연하고 컨텍스트 특정 프롬프트를 생성할 수 있게 합니다.
3. **출력 파서**: 모델 출력에서 정보를 추출하고 형식화합니다. 이는 언어 모델의 원시 출력을 애플리케이션이 필요로 하는 구조화된 데이터나 특정 형식으로 변환하는 데 유용합니다.

## LLMs

LangChain은 OpenAI, Cohere, Hugging Face 등 다양한 대형 언어 모델(LLM)과의 통합을 제공합니다. LangChain 자체는 LLM을 호스팅하지 않지만 다양한 LLM과 상호 작용할 수 있는 일관된 인터페이스를 제공합니다.

다음은 OpenAI LLM 래퍼를 사용하는 예입니다. 이 방법은 다른 LLM 유형에도 적용될 수 있습니다. 우리는 이미 "시작하기" 섹션에서 이를 설치했습니다. LLM을 초기화해 봅시다.

```python
from langchain.llms import OpenAI
llm = OpenAI()
```

- LLM은 [Runnable 인터페이스](https://python.langchain.com/docs/expression_language/interface "Opens in a new window")를 구현하며, 이는 [LangChain Expression Language (LCEL)](https://python.langchain.com/docs/expression_language/ "Opens in a new window")의 기본 구성 요소입니다. 이를 통해 `invoke`, `ainvoke`, `stream`, `astream`, `batch`, `abatch`, `astream_log` 호출을 지원합니다.
- LLM은 **문자열**을 입력으로 받거나 `List[BaseMessage]` 및 `PromptValue`를 포함한 문자열 프롬프트로 강제 변환할 수 있는 객체를 입력으로 받습니다.

몇 가지 예를 살펴보겠습니다.

```python
response = llm.invoke("List the seven wonders of the world.")
print(response)
```

텍스트 응답을 스트리밍하기 위해 `stream` 메서드를 호출할 수도 있습니다.

```python
for chunk in llm.stream("Where were the 2012 Olympics held?"):
    print(chunk, end="", flush=True)
```

## 채팅 모델

LangChain의 채팅 모델 통합은 인터랙티브 채팅 애플리케이션을 만드는 데 필수적입니다. 채팅 모델은 내부적으로 언어 모델을 사용하지만, 입력 및 출력을 중심으로 한 별도의 인터페이스를 제공합니다. 다음은 LangChain에서 OpenAI의 채팅 모델을 사용하는 방법에 대한 자세한 설명입니다.

```python
from langchain.chat_models import ChatOpenAI
chat = ChatOpenAI()
```

LangChain의 채팅 모델은 `AIMessage`, `HumanMessage`, `SystemMessage`, `FunctionMessage`, `ChatMessage`(임의의 역할 매개변수를 가진)와 같은 다양한 메시지 유형과 함께 작동합니다. 일반적으로 `HumanMessage`, `AIMessage`, `SystemMessage`가 가장 자주 사용됩니다.

채팅 모델은 주로 `List[BaseMessage]`를 입력으로 받습니다. 문자열은 `HumanMessage`로 변환할 수 있으며 `PromptValue`도 지원됩니다.

```python
from langchain.schema.messages import HumanMessage, SystemMessage

messages = [
    SystemMessage(content="You are Micheal Jordan."),
    HumanMessage(content="Which shoe manufacturer are you associated with?"),
]

response = chat.invoke(messages)
print(response.content)
```

## 프롬프트

프롬프트는 언어 모델이 관련 있고 일관된 출력을 생성하도록 유도하는 데 필수적입니다. 프롬프트는 간단한 지침부터 복잡한 예시까지 다양합니다. LangChain에서는 몇 가지 전용 클래스와 함수를 통해 프롬프트 처리를 매우 간소화할 수 있습니다.

LangChain의 `PromptTemplate` 클래스는 문자열 프롬프트를 생성하는 다목적 도구입니다. 이는 Python의 `str.format` 구문을 사용하여 동적 프롬프트 생성을 가능하게 합니다. 플레이스홀더가 있는 템플릿을 정의하고 필요한 값으로 채울 수 있습니다.

```python
from langchain.prompts import PromptTemplate

# 플레이스홀더가 있는 간단한 프롬프트
prompt_template = PromptTemplate.from_template(
    "Tell me a {adjective} joke about {content}."
)

# 플레이스홀더를 채워 프롬프트 생성
filled_prompt = prompt_template.format(adjective="funny", content="robots")
print(filled_prompt)
```

채팅 모델의 경우 프롬프트는 다양한 역할을 가진 메시지를 포함하여 더 구조화되어 있습니다. 이를 위해 LangChain은 `ChatPromptTemplate`을 제공합니다.

```python
from langchain.prompts import ChatPromptTemplate

# 다양한 역할을 가진 채팅 프롬프트 정의
chat_template = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a helpful AI bot. Your name is {name}."),
        ("human", "Hello, how are you doing?"),
        ("ai", "I'm doing well, thanks!"),
        ("human", "{user_input}"),
    ]
)

# 채팅 프롬프트 형식화
formatted_messages = chat_template.format_messages(name="Bob", user_input="What is your name?")
for message in formatted_messages:
    print(message)
```

이 접근 방식은 동적 응답이 있는 인터랙티브한 챗봇을 만드는 데 유용합니다.

`PromptTemplate`과 `ChatPromptTemplate`은 LangChain Expression Language (LCEL)와 원활하게 통합되어 더 큰 복잡한 워크플로의 일부가 될 수 있습니다. 이에 대해서는 이후에 더 자세히 논의할 것입니다.

특정 형식이나 지침이 필요한 작업의 경우 사용자 지정 프롬프트 템플릿이 필수적일 수 있습니다. 사용자 지정 프롬프트 템플릿을 만들려면 입력 변수를 정의하고 사용자 지정 형식화 메서드를 정의해야 합니다. 이러한 유연성은 LangChain이 다양한 애플리케이션 요구 사항을 충족할 수 있게 합니다. [자세히 알아보기](https://python.langchain.com/docs/modules/model_io/prompts/prompt_templates/custom_prompt_template "Opens in a new window")

LangChain은 또한 few-shot 프롬프트를 지원하여 모델이 예시에서 학습할 수 있게 합니다. 이 기능은 맥락적 이해나 특정 패턴이 필요한 작업에 중요합니다. few-shot 프롬프트 템플릿은 예시 집합에서 빌드하거나 Example Selector 객체를 사용하여 만들 수 있습니다. [자세히 알아보기](https://python.langchain.com/docs/modules/model_io/prompts/prompt_templates/few_shot_examples "Opens in a new window")

## 출력 파서

출력 파서는 LangChain에서 중요한 역할을 하며, 사용자가 언어 모델이 생성한 응답을 구조화할 수 있게 합니다. 이 섹션에서는 LangChain의 PydanticOutputParser, SimpleJsonOutputParser, CommaSeparatedListOutputParser, DatetimeOutputParser, XMLOutputParser를 사용한 코드 예제를 제공하겠습니다.

**PydanticOutputParser**

LangChain은 Pydantic 데이터 구조로 응답을 구문 분석하기 위해 PydanticOutputParser를 제공합니다. 아래는 그 사용 방법에 대한 단계별 예제입니다:

```python
from typing import List
from langchain.llms import OpenAI
from langchain.output_parsers import PydanticOutputParser
from langchain.prompts import PromptTemplate
from langchain.pydantic_v1 import BaseModel, Field, validator

# 언어 모델 초기화
model = OpenAI(model_name="text-davinci-003", temperature=0.0)

# 원하는 데이터 구조를 Pydantic을 사용하여 정의
class Joke(BaseModel):
    setup: str = Field(description="question to set up a joke")
    punchline: str = Field(description="answer to resolve the joke")

    @validator("setup")
    def question_ends_with_question_mark(cls, field):
        if field[-1] != "?":
            raise ValueError("Badly formed question!")
        return field

# PydanticOutputParser 설정
parser = PydanticOutputParser(pydantic_object=Joke)

# 형식 지침이 포함된 프롬프트 생성
prompt = PromptTemplate(
    template="Answer the user query.\n{format_instructions}\n{query}\n",
    input_variables=["query"],
    partial_variables={"format_instructions": parser.get_format_instructions()},
)

# 언어 모델에 대한 쿼리 정의
query = "Tell me a joke."

# 프롬프트, 모델, 파서를 결합하여 구조화된 출력 얻기
prompt_and_model = prompt | model
output = prompt_and_model.invoke({"query": query})

# 파서를 사용하여 출력 구문 분석
parsed_result = parser.invoke(output)

# 결과는 구조화된 객체
print(parsed_result)
```

**SimpleJsonOutputParser**

LangChain의 SimpleJsonOutputParser는 JSON과 유사한 출력을 구문 분석할 때 사용됩니다. 다음은 예제입니다:

```python
from langchain.output_parsers.json import SimpleJsonOutputParser

# JSON 프롬프트 생성
json_prompt = PromptTemplate.from_template(
    "Return a JSON object with `birthdate` and `birthplace` key that answers the following question: {question}"
)

# JSON 파서 초기화
json_parser = SimpleJsonOutputParser()

# 프롬프트, 모델, 파서가 포함된 체인 생성
json_chain = json_prompt | model | json_parser

# 결과를 스트리밍
result_list = list(json_chain.stream({"question": "When and where was Elon Musk born?"}))

# 결과는 JSON과 유사한 사전 목록
print(result_list)
```

**CommaSeparatedListOutputParser**

CommaSeparatedListOutputParser는 모델 응답에서 쉼표로 구분된 목록을 추출할 때 유용합니다. 다음은 예제입니다:

```python
from langchain.output_parsers import CommaSeparatedListOutputParser
from langchain.prompts import PromptTemplate
from langchain.llms import OpenAI

# 파서 초기화
output_parser = CommaSeparatedListOutputParser()

# 형식 지침 생성
format_instructions = output_parser.get_format_instructions()

# 목록을 요청하는 프롬프트 생성
prompt = PromptTemplate(
    template="List five {subject}.\n{format_instructions}",
    input_variables=["subject"],
    partial_variables={"format_instructions": format_instructions}
)

# 모델에 대한 쿼리 정의
query = "English Premier League Teams"

# 출력 생성
output = model(prompt.format(subject=query))

# 파서를 사용하여 출력 구문 분석
parsed_result = output_parser.parse(output)

# 결과는 항목 목록
print(parsed_result)
```

**DatetimeOutputParser**

LangChain의 DatetimeOutputParser는 날짜 및 시간 정보를 구문 분석하도록 설계되었습니다. 사용 방법은 다음과 같습니다:

```python
from langchain.prompts import PromptTemplate
from langchain.output_parsers import DatetimeOutputParser
from langchain.chains import LLMChain
from langchain.llms import OpenAI

# DatetimeOutputParser 초기화
output_parser = DatetimeOutputParser()

# 형식 지침이 포함된 프롬프트 생성
template = """
Answer the user's question:
{question}
{format_instructions}
"""

prompt = PromptTemplate.from_template(
    template,
    partial_variables={"format_instructions": output_parser.get_format_instructions()},
)

# 프롬프트와 언어 모델이 포함된 체인 생성
chain = LLMChain(prompt=prompt, llm=OpenAI())

# 모델에 대한 쿼리 정의
query = "when did Neil Armstrong land on the moon in terms of GMT?"

# 체인 실행
output = chain.run(query)

# 날짜 및 시간 파서를 사용하여 출력 구문 분석
parsed_result = output_parser.parse(output)

# 결과는 datetime 객체
print(parsed_result)
```

이 예제들은 LangChain의 출력 파서를 사용하여 다양한 유형의 모델 응답을 구조화하는 방법을 보여줍니다. 출력 파서는 LangChain에서 언어 모델 출력의 사용성과 해석 가능성을 향상시키는 귀중한 도구입니다.

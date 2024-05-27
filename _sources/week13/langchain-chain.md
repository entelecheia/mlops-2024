# LangChain 모듈 - 체인 (Chains)

LangChain은 복잡한 애플리케이션에서 대형 언어 모델(LLM)을 활용하기 위해 설계된 도구입니다. LLM과 다른 유형의 구성 요소를 포함하는 구성 요소 체인을 생성하기 위한 프레임워크를 제공합니다. 두 가지 주요 프레임워크는 다음과 같습니다:

- LangChain 표현 언어 (LangChain Expression Language, LCEL)
- 레거시 체인 인터페이스 (Legacy Chain Interface)

LangChain 표현 언어(LCEL)는 체인의 직관적인 구성을 가능하게 하는 구문입니다. 스트리밍, 비동기 호출, 배치 처리, 병렬화, 재시도, 대체 및 추적과 같은 고급 기능을 지원합니다. 예를 들어, 다음 코드와 같이 LCEL에서 프롬프트, 모델 및 출력 파서를 구성할 수 있습니다:

```python
from langchain.prompts import ChatPromptTemplate
from langchain.schema import StrOutputParser

model = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)
prompt = ChatPromptTemplate.from_messages([
    ("system", "You're a very knowledgeable historian who provides accurate and eloquent answers to historical questions."),
    ("human", "{question}")
])
runnable = prompt | model | StrOutputParser()

for chunk in runnable.stream({"question": "What are the seven wonders of the world"}):
    print(chunk, end="", flush=True)
```

또는 LLMChain은 구성 요소를 구성하는 LCEL과 유사한 옵션입니다. LLMChain 예시는 다음과 같습니다:

```python
from langchain.chains import LLMChain

chain = LLMChain(llm=model, prompt=prompt, output_parser=StrOutputParser())
chain.run(question="What are the seven wonders of the world")
```

LangChain의 체인은 Memory 객체를 통합하여 상태를 유지할 수도 있습니다. 이를 통해 호출 간에 데이터 지속성을 제공합니다. 다음 예시에서 볼 수 있듯이:

```python
from langchain.chains import ConversationChain
from langchain.memory import ConversationBufferMemory

conversation = ConversationChain(llm=chat, memory=ConversationBufferMemory())
conversation.run("Answer briefly. What are the first 3 colors of a rainbow?")
conversation.run("And the next 4?")
```

LangChain은 또한 구조화된 출력을 얻고 체인 내에서 함수를 실행하는 데 유용한 OpenAI의 함수 호출 API와의 통합을 지원합니다. 구조화된 출력을 얻기 위해 Pydantic 클래스 또는 JsonSchema를 사용하여 출력을 지정할 수 있습니다. 예시:

```python
from langchain.pydantic_v1 import BaseModel, Field
from langchain.chains.openai_functions import create_structured_output_runnable
from langchain.chat_models import ChatOpenAI
from langchain.prompts import ChatPromptTemplate

class Person(BaseModel):
    name: str = Field(..., description="The person's name")
    age: int = Field(..., description="The person's age")
    fav_food: Optional[str] = Field(None, description="The person's favorite food")

llm = ChatOpenAI(model="gpt-4", temperature=0)
prompt = ChatPromptTemplate.from_messages([
    # Prompt messages here
])

runnable = create_structured_output_runnable(Person, llm, prompt)
runnable.invoke({"input": "Sally is 13"})
```

구조화된 출력을 위해 LLMChain을 사용하는 레거시 접근 방식도 있습니다:

```python
from langchain.chains.openai_functions import create_structured_output_chain

class Person(BaseModel):
    name: str = Field(..., description="The person's name")
    age: int = Field(..., description="The person's age")

chain = create_structured_output_chain(Person, llm, prompt, verbose=True)
chain.run("Sally is 13")
```

LangChain은 OpenAI 함수를 활용하여 추출, 태깅, OpenAPI, 인용 QA 등 다양한 목적을 위한 특정 체인을 생성합니다.

추출의 맥락에서 이 과정은 구조화된 출력 체인과 유사하지만 정보 또는 엔티티 추출에 중점을 둡니다. 태깅의 경우, 문서에 감정, 언어, 스타일, 다루는 주제 또는 정치적 경향과 같은 클래스를 레이블링하는 것이 목적입니다.

LangChain에서 태깅이 어떻게 작동하는지는 Python 코드로 시연할 수 있습니다. 필요한 패키지를 설치하고 환경을 설정하는 것으로 시작합니다:

```python
pip install langchain openai
# Set env var OPENAI_API_KEY or load from a .env file:
# import dotenv
# dotenv.load_dotenv()

from langchain.chat_models import ChatOpenAI
from langchain.prompts import ChatPromptTemplate
from langchain.chains import create_tagging_chain, create_tagging_chain_pydantic
```

태깅을 위한 스키마를 정의하고, 속성과 예상되는 유형을 지정합니다:

```python
schema = {
    "properties": {
        "sentiment": {"type": "string"},
        "aggressiveness": {"type": "integer"},
        "language": {"type": "string"},
    }
}

llm = ChatOpenAI(temperature=0, model="gpt-3.5-turbo-0613")
chain = create_tagging_chain(schema, llm)
```

다양한 입력으로 태깅 체인을 실행한 예시는 모델이 감정, 언어 및 공격성을 해석하는 능력을 보여줍니다:

```python
inp = "Estoy increiblemente contento de haberte conocido! Creo que seremos muy buenos amigos!"
chain.run(inp)
# {'sentiment': 'positive', 'language': 'Spanish'}

inp = "Estoy muy enojado con vos! Te voy a dar tu merecido!"
chain.run(inp)
# {'sentiment': 'enojado', 'aggressiveness': 1, 'language': 'es'}
```

더 세밀한 제어를 위해 가능한 값, 설명 및 필수 속성을 포함하여 스키마를 더 구체적으로 정의할 수 있습니다. 이러한 향상된 제어의 예시는 다음과 같습니다:

```python
schema = {
    "properties": {
        # Schema definitions here
    },
    "required": ["language", "sentiment", "aggressiveness"],
}

chain = create_tagging_chain(schema, llm)
```

Pydantic 스키마를 사용하여 태깅 기준을 정의할 수도 있습니다. 이는 필수 속성과 유형을 지정하는 파이썬스러운 방법을 제공합니다:

```python
from enum import Enum
from pydantic import BaseModel, Field

class Tags(BaseModel):
    # Class fields here

chain = create_tagging_chain_pydantic(Tags, llm)
```

또한 LangChain의 메타데이터 태거 문서 변환기를 사용하여 LangChain 문서에서 메타데이터를 추출할 수 있습니다. 이는 태깅 체인과 유사한 기능을 제공하지만 LangChain 문서에 적용됩니다.

LangChain의 또 다른 기능은 OpenAI 함수를 사용하여 텍스트에서 인용을 추출하는 인용 검색입니다. 다음 코드에서 이를 시연합니다:

```python
from langchain.chains import create_citation_fuzzy_match_chain
from langchain.chat_models import ChatOpenAI

llm = ChatOpenAI(temperature=0, model="gpt-3.5-turbo-0613")
chain = create_citation_fuzzy_match_chain(llm)
# Further code for running the chain and displaying results
```

LangChain에서 대형 언어 모델(LLM) 애플리케이션의 체이닝은 일반적으로 프롬프트 템플릿과 LLM, 그리고 선택적으로 출력 파서를 결합하는 것을 포함합니다. 이를 수행하는 권장 방법은 LangChain 표현 언어(LCEL)를 통하는 것이지만, 레거시 LLMChain 접근 방식도 지원됩니다.

LCEL을 사용하면 BasePromptTemplate, BaseLanguageModel 및 BaseOutputParser가 모두 Runnable 인터페이스를 구현하고 서로 쉽게 연결될 수 있습니다. 다음 예제에서 이를 시연합니다:

```python
from langchain.prompts import PromptTemplate
from langchain.chat_models import ChatOpenAI
from langchain.schema import StrOutputParser

prompt = PromptTemplate.from_template(
    "What is a good name for a company that makes {product}?"
)
runnable = prompt | ChatOpenAI() | StrOutputParser()
runnable.invoke({"product": "colorful socks"})
# Output: 'VibrantSocks'
```

LangChain의 라우팅을 통해 이전 단계의 출력이 다음 단계를 결정하는 비결정적 체인을 생성할 수 있습니다. 이는 LLM과의 상호 작용에서 일관성을 유지하고 구조화하는 데 도움이 됩니다. 예를 들어, 서로 다른 유형의 질문에 최적화된 두 개의 템플릿이 있는 경우 사용자 입력에 따라 템플릿을 선택할 수 있습니다.

다음은 RunnableBranch를 사용하여 LCEL로 이를 구현하는 방법입니다. RunnableBranch는 (조건, 실행 가능) 쌍 목록과 기본 실행 가능으로 초기화됩니다:

```python
from langchain.chat_models from ChatOpenAI
from langchain.schema.output_parser import StrOutputParser
from langchain.schema.runnable import RunnableBranch
# Code for defining physics_prompt and math_prompt

general_prompt = PromptTemplate.from_template(
    "You are a helpful assistant. Answer the question as accurately as you can.\n\n{input}"
)
prompt_branch = RunnableBranch(
    (lambda x: x["topic"] == "math", math_prompt),
    (lambda x: x["topic"] == "physics", physics_prompt),
    general_prompt,
)

# More code for setting up the classifier and final chain
```

최종 체인은 주제 분류기, 프롬프트 분기, 출력 파서와 같은 다양한 구성 요소를 사용하여 입력의 주제에 따라 흐름을 결정합니다:

```python
from operator import itemgetter
from langchain.schema.output_parser import StrOutputParser
from langchain.schema.runnable import RunnablePassthrough

final_chain = (
    RunnablePassthrough.assign(topic=itemgetter("input") | classifier_chain)
    | prompt_branch
    | ChatOpenAI()
    | StrOutputParser()
)

final_chain.invoke(
    {
        "input": "What is the first prime number greater than 40 such that one plus the prime number is divisible by 3?"
    }
)
# Output: Detailed answer to the math question
```

이 접근 방식은 복잡한 쿼리를 처리하고 입력에 따라 적절하게 라우팅하는 LangChain의 유연성과 강력함을 보여줍니다.

언어 모델의 영역에서는 초기 호출 후에 일련의 후속 호출을 수행하는 것이 일반적인 관행입니다. 이때 한 호출의 출력을 다음 호출의 입력으로 사용합니다. 이 순차적 접근 방식은 이전 상호 작용에서 생성된 정보를 기반으로 구축하려는 경우에 특히 유용합니다. LangChain 표현 언어(LCEL)가 이러한 시퀀스를 생성하는 데 권장되는 방법이지만, SequentialChain 메서드는 여전히 이전 버전과의 호환성을 위해 문서화되어 있습니다.

이를 설명하기 위해 먼저 연극 시놉시스를 생성한 다음 해당 시놉시스를 기반으로 리뷰를 생성하는 시나리오를 고려해 보겠습니다. Python의 `langchain.prompts`를 사용하여 두 개의 `PromptTemplate` 인스턴스를 생성합니다: 하나는 시놉시스용이고 다른 하나는 리뷰용입니다. 다음은 이러한 템플릿을 설정하는 코드입니다:

```python
from langchain.prompts import PromptTemplate

synopsis_prompt = PromptTemplate.from_template(
    "You are a playwright. Given the title of play, it is your job to write a synopsis for that title.\n\nTitle: {title}\Playwright: This is a synopsis for the above play:"
)

review_prompt = PromptTemplate.from_template(
    "You are a play critic from the New York Times. Given the synopsis of play, it is your job to write a review for that play.\n\nPlay Synopsis:\n{synopsis}\nReview from a New York Times play critic of the above play:"
)
```

LCEL 접근 방식에서는 이러한 프롬프트를 `ChatOpenAI` 및 `StrOutputParser`와 함께 체이닝하여 먼저 시놉시스를 생성한 다음 리뷰를 생성하는 시퀀스를 만듭니다. 코드 스니펫은 다음과 같습니다:

```python
from langchain.chat_models import ChatOpenAI
from langchain.schema import StrOutputParser

llm = ChatOpenAI()
chain = (
    {"synopsis": synopsis_prompt | llm | StrOutputParser()}
    | review_prompt
    | llm
    | StrOutputParser()
)
chain.invoke({"title": "Tragedy at sunset on the beach"})
```

시놉시스와 리뷰가 모두 필요한 경우 `RunnablePassthrough`를 사용하여 각각에 대해 별도의 체인을 만든 다음 결합할 수 있습니다:

```python
from langchain.schema.runnable import RunnablePassthrough

synopsis_chain = synopsis_prompt | llm | StrOutputParser()
review_chain = review_prompt | llm | StrOutputParser()
chain = {"synopsis": synopsis_chain} | RunnablePassthrough.assign(review=review_chain)
chain.invoke({"title": "Tragedy at sunset on the beach"})
```

보다 복잡한 시퀀스의 경우 `SequentialChain` 메서드가 사용됩니다. 이를 통해 여러 입력과 출력을 처리할 수 있습니다. 연극의 제목과 시대를 기반으로 시놉시스가 필요한 경우를 고려해 보겠습니다. 다음은 이를 설정하는 방법입니다:

```python
from langchain.llms import OpenAI
from langchain.chains import LLMChain, SequentialChain
from langchain.prompts import PromptTemplate

llm = OpenAI(temperature=0.7)

synopsis_template = "You are a playwright. Given the title of play and the era it is set in, it is your job to write a synopsis for that title.\n\nTitle: {title}\nEra: {era}\nPlaywright: This is a synopsis for the above play:"
synopsis_prompt_template = PromptTemplate(input_variables=["title", "era"], template=synopsis_template)
synopsis_chain = LLMChain(llm=llm, prompt=synopsis_prompt_template, output_key="synopsis")

review_template = "You are a play critic from the New York Times. Given the synopsis of play, it is your job to write a review for that play.\n\nPlay Synopsis:\n{synopsis}\nReview from a New York Times play critic of the above play:"
prompt_template = PromptTemplate(input_variables=["synopsis"], template=review_template)
review_chain = LLMChain(llm=llm, prompt=prompt_template, output_key="review")

overall_chain = SequentialChain(
    chains=[synopsis_chain, review_chain],
    input_variables=["era", "title"],
    output_variables=["synopsis", "review"],
    verbose=True,
)

overall_chain({"title": "Tragedy at sunset on the beach", "era": "Victorian England"})
```

체인 전체에서 컨텍스트를 유지하거나 체인의 후반부에서 사용하려는 경우 `SimpleMemory`를 사용할 수 있습니다. 이는 복잡한 입력/출력 관계를 관리하는 데 특히 유용합니다. 예를 들어, 연극의 제목, 시대, 시놉시스, 리뷰를 기반으로 소셜 미디어 게시물을 생성하려는 시나리오에서 `SimpleMemory`는 이러한 변수를 관리하는 데 도움이 될 수 있습니다:

```python
from langchain.memory import SimpleMemory
from langchain.chains import SequentialChain

template = "You are a social media manager for a theater company. Given the title of play, the era it is set in, the date, time and location, the synopsis of the play, and the review of the play,

 it is your job to write a social media post for that play.\n\nHere is some context about the time and location of the play:\nDate and Time: {time}\nLocation: {location}\n\nPlay Synopsis:\n{synopsis}\nReview from a New York Times play critic of the above play:\n{review}\n\nSocial Media Post:"
prompt_template = PromptTemplate(input_variables=["synopsis", "review", "time", "location"], template=template)
social_chain = LLMChain(llm=llm, prompt=prompt_template, output_key="social_post_text")

overall_chain = SequentialChain(
    memory=SimpleMemory(memories={"time": "December 25th, 8pm PST", "location": "Theater in the Park"}),
    chains=[synopsis_chain, review_chain, social_chain],
    input_variables=["era", "title"],
    output_variables=["social_post_text"],
    verbose=True,
)

overall_chain({"title": "Tragedy at sunset on the beach", "era": "Victorian England"})
```

순차적 체인 외에도 문서 작업을 위한 특수 체인이 있습니다. 이러한 각 체인은 문서 결합부터 반복적 문서 분석을 기반으로 답변 세분화, 요약을 위한 문서 내용 매핑 및 축소, 점수가 매겨진 응답을 기반으로 재순위 지정에 이르기까지 다양한 목적을 수행합니다. 이러한 체인은 추가적인 유연성과 사용자 정의를 위해 LCEL로 재생성될 수 있습니다.

- `StuffDocumentsChain`은 문서 목록을 LLM에 전달되는 단일 프롬프트로 결합합니다.
- `RefineDocumentsChain`은 문서가 모델의 컨텍스트 용량을 초과하는 작업에 적합하며, 각 문서에 대해 답변을 반복적으로 업데이트합니다.
- `MapReduceDocumentsChain`은 각 문서에 개별적으로 체인을 적용한 다음 결과를 결합합니다.
- `MapRerankDocumentsChain`은 문서 기반 응답마다 점수를 매기고 가장 높은 점수의 응답을 선택합니다.

다음은 LCEL을 사용하여 `MapReduceDocumentsChain`을 설정하는 방법의 예시입니다:

```python
from functools import partial
from langchain.chains.combine_documents import collapse_docs, split_list_of_docs
from langchain.schema import Document, StrOutputParser
from langchain.schema.prompt_template import format_document
from langchain.schema.runnable import RunnableParallel, RunnablePassthrough

llm = ChatAnthropic()
document_prompt = PromptTemplate.from_template("{page_content}")
partial_format_document = partial(format_document, prompt=document_prompt)

map_chain = (
    {"context": partial_format_document}
    | PromptTemplate.from_template("Summarize this content:\n\n{context}")
    | llm
    | StrOutputParser()
)

map_as_doc_chain = (
    RunnableParallel({"doc": RunnablePassthrough(), "content": map_chain})
    | (lambda x: Document(page_content=x["content"], metadata=x["doc"].metadata))
).with_config(run_name="Summarize (return doc)")

def format_docs(docs):
    return "\n\n".join(partial_format_document(doc) for doc in docs)

collapse_chain = (
    {"context": format_docs}
    | PromptTemplate.from_template("Collapse this content:\n\n{context}")
    | llm
    | StrOutputParser()
)

reduce_chain = (
    {"context": format_docs}
    | PromptTemplate.from_template("Combine these summaries:\n\n{context}")
    | llm
    | StrOutputParser()
).with_config(run_name="Reduce")

map_reduce = (map_as_doc_chain.map() | collapse | reduce_chain).with_config(run_name="Map reduce")
```

이러한 구성을 통해 LCEL과 기본 언어 모델의 장점을 활용하여 문서 내용에 대한 상세하고 포괄적인 분석이 가능합니다.

# LangChain 표현 언어 (LCEL)

자연어 처리와 기계 학습 세계에서 복잡한 연산 체인을 구성하는 것은 어려운 작업일 수 있습니다. 다행히도 LangChain 표현 언어(LCEL)가 이를 해결해 주며, 정교한 언어 처리 파이프라인을 구축하고 배포하는 선언적이고 효율적인 방법을 제공합니다. LCEL은 체인 구성 프로세스를 단순화하도록 설계되어 프로토타이핑에서 프로덕션으로 쉽게 전환할 수 있습니다. 이 블로그에서는 LCEL이 무엇이고 왜 사용하는지 살펴보고, 그 기능을 보여주는 실제 코드 예제와 함께 설명하겠습니다.

LCEL 또는 LangChain 표현 언어는 언어 처리 체인을 구성하기 위한 강력한 도구입니다. 프로토타이핑에서 프로덕션으로 원활하게 전환하도록 설계되었으며, 광범위한 코드 변경이 필요하지 않습니다. 간단한 "프롬프트 + LLM" 체인을 구축하든 수백 개의 단계가 있는 복잡한 파이프라인을 구축하든 LCEL이 적용될 수 있습니다.

LCEL을 언어 처리 프로젝트에 사용해야 하는 이유는 다음과 같습니다:

1. 빠른 토큰 스트리밍: LCEL은 언어 모델에서 출력 파서로 실시간으로 토큰을 전달하여 응답성과 효율성을 향상시킵니다.
2. 다양한 API: LCEL은 프로토타이핑 및 프로덕션 사용을 위한 동기 및 비동기 API를 지원하여 여러 요청을 효율적으로 처리합니다.
3. 자동 병렬화: LCEL은 가능한 경우 병렬 실행을 최적화하여 동기 및 비동기 인터페이스 모두에서 대기 시간을 줄입니다.
4. 신뢰할 수 있는 구성: 규모에 맞게 재시도 및 대체를 구성하여 체인 신뢰성을 높이고 개발 중 스트리밍을 지원합니다.
5. 중간 결과 스트리밍: 처리 중 중간 결과에 액세스하여 사용자 업데이트 또는 디버깅 목적으로 사용할 수 있습니다.
6. 스키마 생성: LCEL은 입력 및 출력 유효성 검사를 위해 Pydantic 및 JSONSchema 스키마를 생성합니다.
7. 포괄적인 추적: LangSmith는 복잡한 체인의 모든 단계를 자동으로 추적하여 관찰 가능성과 디버깅을 제공합니다.
8. 쉬운 배포: LCEL로 생성된 체인을 LangServe를 사용하여 쉽게 배포할 수 있습니다.

이제 LCEL의 강력함을 보여주는 실제 코드 예제를 살펴보겠습니다. LCEL이 빛나는 일반적인 작업과 시나리오를 탐구해 보겠습니다.

## 프롬프트 + LLM

가장 기본적인 구성은 프롬프트와 언어 모델을 결합하여 사용자 입력을 받아 프롬프트에 추가하고 모델에 전달한 다음 원시 모델 출력을 반환하는 체인을 만드는 것입니다. 다음은 그 예시입니다:

```python
from langchain.prompts import ChatPromptTemplate
from langchain.chat_models import ChatOpenAI

prompt = ChatPromptTemplate.from_template("tell me a joke about {foo}")
model = ChatOpenAI()
chain = prompt | model

result = chain.invoke({"foo": "bears"})
print(result)
```

이 예제에서 체인은 곰에 대한 농담을 생성합니다.

체인에 중지 시퀀스를 첨부하여 텍스트 처리 방식을 제어할 수 있습니다. 예를 들면 다음과 같습니다:

```python
chain = prompt | model.bind(stop=["\n"])
result = chain.invoke({"foo": "bears"})
print(result)
```

이 구성은 줄 바꿈 문자를 만나면 텍스트 생성을 중지합니다.

LCEL은 체인에 함수 호출 정보를 첨부하는 것을 지원합니다. 다음은 그 예시입니다:

```python
functions = [
    {
        "name": "joke",
        "description": "A joke",
        "parameters": {
            "type": "object",
            "properties": {
                "setup": {"type": "string", "description": "The setup for the joke"},
                "punchline": {
                    "type": "string",
                    "description": "The punchline for the joke",
                },
            },
            "required": ["setup", "punchline"],
        },
    }
]
chain = prompt | model.bind(function_call={"name": "joke"}, functions=functions)
result = chain.invoke({"foo": "bears"}, config={})
print(result)
```

이 예제는 농담을 생성하기 위해 함수 호출 정보를 첨부합니다.

## 프롬프트 + LLM + 출력 파서

원시 모델 출력을 보다 작업하기 쉬운 형식으로 변환하기 위해 출력 파서를 추가할 수 있습니다. 이렇게 할 수 있는 방법은 다음과 같습니다:

```python
from langchain.schema.output_parser import StrOutputParser

chain = prompt | model | StrOutputParser()
result = chain.invoke({"foo": "bears"})
print(result)
```

이제 출력이 문자열 형식이므로 다운스트림 작업에 더 편리합니다.

반환할 함수를 지정할 때 LCEL을 사용하여 직접 구문 분석할 수 있습니다. 예를 들면 다음과 같습니다:

```python
from langchain.output_parsers.openai_functions import JsonOutputFunctionsParser

chain = (
    prompt
    | model.bind(function_call={"name": "joke"}, functions=functions)
    | JsonOutputFunctionsParser()
)
result = chain.invoke({"foo": "bears"})
print(result)
```

이 예제는 "joke" 함수의 출력을 직접 구문 분석합니다.

이것들은 LCEL이 복잡한 언어 처리 작업을 단순화하는 방법의 몇 가지 예일 뿐입니다. 챗봇을 구축하든, 콘텐츠를 생성하든, 복잡한 텍스트 변환을 수행하든 LCEL은 워크플로우를 간소화하고 코드를 더 유지 관리하기 쉽게 만들 수 있습니다.

## RAG (검색 증강 생성)

LCEL은 검색과 언어 생성 단계를 결합하는 검색 증강 생성 체인을 생성하는 데 사용할 수 있습니다. 다음은 그 예시입니다:

```python
from operator import itemgetter

from langchain.prompts import ChatPromptTemplate
from langchain.chat_models import ChatOpenAI
from langchain.embeddings import OpenAIEmbeddings
from langchain.schema.output_parser import StrOutputParser
from langchain.schema.runnable import RunnablePassthrough, RunnableLambda
from langchain.vectorstores import FAISS

# Create a vector store and retriever
vectorstore = FAISS.from_texts(
    ["harrison worked at kensho"], embedding=OpenAIEmbeddings()
)
retriever = vectorstore.as_retriever()

# Define templates for prompts
template = """Answer the question based only on the following context:
{context}

Question: {question}
"""
prompt = ChatPromptTemplate.from_template(template)

model = ChatOpenAI()

# Create a retrieval-augmented generation chain
chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | prompt
    | model
    | StrOutputParser()
)

result = chain.invoke("where did harrison work?")
print(result)
```

이 예제에서 체인은 컨텍스트에서 관련 정보를 검색하고 질문에 대한 응답을 생성합니다.

## 대화형 검색 체인

대화 기록을 체인에 쉽게 추가할 수 있습니다. 다음은 대화형 검색 체인의 예시입니다:

```python
from langchain.schema.runnable import RunnableMap
from langchain.schema import format_document

from langchain.prompts.prompt import PromptTemplate

# Define templates for prompts
_template = """Given the following conversation and a follow up question, rephrase the follow up question to be a standalone question, in its original language.

Chat History:
{chat_history}
Follow Up Input: {question}
Standalone question:"""
CONDENSE_QUESTION_PROMPT = PromptTemplate.from_template(_template)

template = """Answer the question based only on the following context:
{context}

Question: {question}
"""
ANSWER_PROMPT = ChatPromptTemplate.from_template(template)

# Define input map and context
_inputs = RunnableMap(
    standalone_question=RunnablePassthrough.assign(
        chat_history=lambda x: _format_chat_history(x["chat_history"])
    )
    | CONDENSE_QUESTION_PROMPT
    | ChatOpenAI(temperature=0)
    | StrOutputParser(),
)
_context = {
    "context": itemgetter("standalone_question") | retriever | _combine_documents,
    "question": lambda x: x["standalone_question"],
}
conversational_qa_chain = _inputs | _context | ANSWER_PROMPT | ChatOpenAI()

result = conversational_qa_chain.invoke(
    {
        "question": "where did harrison work?",
        "chat_history": [],
    }
)
print(result)
```

이 예제에서 체인은 대화 컨텍스트 내에서 후속 질문을 처리합니다.

## 메모리 사용 및 소스 문서 반환

LCEL은 메모리 사용과 소스 문서 반환도 지원합니다. 다음은 체인에서 메모리를 사용하는 방법입니다:

```python
from operator import itemgetter
from langchain.memory import ConversationBufferMemory

# Create a memory instance
memory = ConversationBufferMemory(
    return_messages=True, output_key="answer", input_key="question"
)

# Define steps for the chain
loaded_memory = RunnablePassthrough.assign(
    chat_history=RunnableLambda(memory.load_memory_variables) | itemgetter("history"),
)

standalone_question = {
    "standalone_question": {
        "question": lambda x: x["question"],
        "chat_history": lambda x: _format_chat_history(x["chat_history"]),
    }
    | CONDENSE_QUESTION_PROMPT
    | ChatOpenAI(temperature=0)
    | StrOutputParser(),
}

retrieved_documents = {
    "docs": itemgetter("standalone_question") | retriever,
    "question": lambda x: x["standalone_question"],
}

final_inputs = {
    "context": lambda x: _combine_documents(x["docs"]),
    "question": itemgetter("question"),
}

answer = {
    "answer": final_inputs | ANSWER_PROMPT | ChatOpenAI(),
    "docs": itemgetter("docs"),
}

# Create the final chain by combining the steps
final_chain = loaded_memory | standalone_question | retrieved_documents | answer

inputs = {"question": "where did harrison work?"}
result = final_chain.invoke(inputs)
print(result)
```

이 예제에서 메모리는 대화 기록과 소스 문서를 저장하고 검색하는 데 사용됩니다.

## 다중 체인

Runnables를 사용하여 여러 체인을 연결할 수 있습니다. 다음은 분기 및 병합의 예시입니다:

```python
from operator import itemgetter

from langchain.chat_models import ChatOpenAI
from langchain.prompts import ChatPromptTemplate
from langchain.schema import StrOutputParser

prompt1 = ChatPromptTemplate.from_template("what is the city {person} is from?")
prompt2 = ChatPromptTemplate.from_template(
    "what country is the city {city} in? respond in {language}"
)

model = ChatOpenAI()

chain1 = prompt1 | model | StrOutputParser()

chain2 = (
    {"city": chain1, "language": itemgetter("language")}
    | prompt2
    | model
    | StrOutputParser()
)

result = chain2.invoke({"person": "obama", "language": "spanish"})
print(result)
```

이 예제에서는 두 개의 체인이 결합되어 도시에 대한 정보와 지정된 언어로 해당 국가에 대한 정보를 생성합니다.

## 분기 및 병합

LCEL을 사용하면 RunnableMaps를 사용하여 체인을 분할하고 병합할 수 있습니다. 다음은 분기 및 병합의 예시입니다:

```python
from operator import itemgetter

from langchain.chat_models import ChatOpenAI
from langchain.prompts import ChatPromptTemplate
from langchain.schema import StrOutputParser

planner = (
    ChatPromptTemplate.from_template("Generate an argument about: {input}")
    | ChatOpenAI()
    | StrOutputParser()
    | {"base_response": RunnablePassthrough()}
)

arguments_for = (
    ChatPromptTemplate.from_template(
        "List the pros or positive aspects of {base_response}"
    )
    | ChatOpenAI()
    | StrOutputParser()
)
arguments_against = (
    ChatPromptTemplate.from_template(
        "List the cons or negative aspects of {base_response}"
    )
    | ChatOpenAI()
    | StrOutputParser()
)

final_responder = (
    ChatPromptTemplate.from_messages(
        [
            ("ai", "{original_response}"),
            ("human", "Pros:\n{results_1}\n\nCons:\n{results_2}"),
            ("system", "Generate a final response given the critique"),
        ]
    )
    | ChatOpenAI()
    | StrOutputParser()
)

chain = (
    planner
    | {
        "results_1": arguments_for,
        "results_2": arguments_against,
        "original_response": itemgetter("base_response"),
    }
    | final_responder
)

result = chain.invoke({"input": "scrum"})
print(result)
```

이 예제에서 분기 및 병합 체인은 인수를 생성하고 최종 응답을 생성하기 전에 장단점을 평가하는 데 사용됩니다.

### LCEL을 사용하여 Python 코드 작성

LangChain 표현 언어(LCEL)의 강력한 응용 프로그램 중 하나는 사용자 문제를 해결하기 위해 Python 코드를 작성하는 것입니다. 다음은 LCEL을 사용하여 Python 코드를 작성하는 방법의 예시입니다:

````python
from langchain.chat_models import ChatOpenAI
from langchain.prompts import ChatPromptTemplate
from langchain.schema.output_parser import StrOutputParser
from langchain_experimental.utilities import PythonREPL

template = """Write some python code to solve the user's problem.

Return only python code in Markdown format, e.g.:

```python
....
```"""
prompt = ChatPromptTemplate.from_messages([("system", template), ("human", "{input}")])

model = ChatOpenAI()

def _sanitize_output(text: str):
    _, after = text.split("```python")
    return after.split("```")[0]

chain = prompt | model | StrOutputParser() | _sanitize_output | PythonREPL().run

result = chain.invoke({"input": "what's 2 plus 2"})
print(result)
````

이 예제에서 사용자는 입력을 제공하고 LCEL은 문제를 해결하기 위해 Python 코드를 생성합니다. 그런 다음 Python REPL을 사용하여 코드를 실행하고 Markdown 형식으로 결과 Python 코드를 반환합니다.

Python REPL을 사용하면 임의의 코드를 실행할 수 있으므로 주의해서 사용하시기 바랍니다.

### 체인에 메모리 추가

메모리는 많은 대화형 AI 애플리케이션에서 필수적입니다. 다음은 임의의 체인에 메모리를 추가하는 방법입니다:

```python
from operator import itemgetter
from langchain.chat_models import ChatOpenAI
from langchain.memory import ConversationBufferMemory
from langchain.schema.runnable import RunnablePassthrough, RunnableLambda
from langchain.prompts import ChatPromptTemplate, MessagesPlaceholder

model = ChatOpenAI()
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a helpful chatbot"),
        MessagesPlaceholder(variable_name="history"),
        ("human", "{input}"),
    ]
)

memory = ConversationBufferMemory(return_messages=True)

# Initialize memory
memory.load_memory_variables({})

chain = (
    RunnablePassthrough.assign(
        history=RunnableLambda(memory.load_memory_variables) | itemgetter("history")
    )
    | prompt
    | model
)

inputs = {"input": "hi, I'm Bob"}
response = chain.invoke(inputs)
response

# Save the conversation in memory
memory.save_context(inputs, {"output": response.content})

# Load memory to see the conversation history
memory.load_memory_variables({})
```

이 예제에서 메모리는 대화 기록을 저장하고 검색하는 데 사용되어 챗봇이 컨텍스트를 유지하고 적절하게 응답할 수 있도록 합니다.

### Runnables와 외부 도구 사용

LCEL을 사용하면 외부 도구와 Runnables를 매끄럽게 통합할 수 있습니다. 다음은 DuckDuckGo 검색 도구를 사용한 예시입니다:

```python
from langchain.chat_models import ChatOpenAI
from langchain.prompts import ChatPromptTemplate
from langchain.schema.output_parser import StrOutputParser
from langchain.tools import DuckDuckGoSearchRun

search = DuckDuckGoSearchRun()

template = """Turn the following user input into a search query for a search engine:

{input}"""
prompt = ChatPromptTemplate.from_template(template)

model = ChatOpenAI()

chain = prompt | model | StrOutputParser() | search

search_result = chain.invoke({"input": "I'd like to figure out what games are tonight"})
print(search_result)
```

이 예제에서 LCEL은 DuckDuckGo 검색 도구를 체인에 통합하여 사용자 입력에서 검색 쿼리를 생성하고 검색 결과를 검색할 수 있습니다.

LCEL의 유연성으로 인해 다양한 외부 도구와 서비스를 언어 처리 파이프라인에 쉽게 통합할 수 있어 기능과 기능이 향상됩니다.

### LLM 애플리케이션에 중재 추가

LLM 애플리케이션이 콘텐츠 정책을 준수하고 중재 안전 장치를 포함하도록 하려면 체인에 중재 확인을 통합할 수 있습니다. LangChain을 사용하여 중재를 추가하는 방법은 다음과 같습니다:

```python
from langchain.chains import OpenAIModerationChain
from langchain.llms import OpenAI
from langchain.prompts import ChatPromptTemplate

moderate = OpenAIModerationChain()

model = OpenAI()
prompt = ChatPromptTemplate.from_messages([("system", "repeat after me: {input}")])

chain = prompt | model

# Original response without moderation
response_without_moderation = chain.invoke({"input": "you are stupid"})
print(response_without_moderation)

moderated_chain = chain | moderate

# Response after moderation
response_after_moderation = moderated_chain.invoke({"input": "you are stupid"})
print(response_after_moderation)
```

이 예제에서 `OpenAIModerationChain`은 LLM에 의해 생성된 응답에 중재를 추가하는 데 사용됩니다. 중재 체인은 OpenAI의 콘텐츠 정책을 위반하는 내용이 있는지 응답을 확인합니다. 위반이 발견되면 그에 따라 응답에 플래그를 지정합니다.

### 의미론적 유사성에 의한 라우팅

LCEL을 사용하면 사용자 입력의 의미론적 유사성을 기반으로 사용자 지정 라우팅 논리를 구현할 수 있습니다. 다음은 사용자 입력을 기반으로 체인 논리를 동적으로 결정하는 방법의 예시입니다:

```python
from langchain.chat_models import ChatOpenAI
from langchain.embeddings import OpenAIEmbeddings
from langchain.prompts import PromptTemplate
from langchain.schema.output_parser import StrOutputParser
from langchain.schema.runnable import RunnableLambda, RunnablePassthrough
from langchain.utils.math import cosine_similarity

physics_template = """You are a very smart physics professor. \
You are great at answering questions about physics in a concise and easy to understand manner. \
When you don't know the answer to a question you admit that you don't know.

Here is a question:
{query}"""

math_template = """You are a very good mathematician. You are great at answering math questions. \
You are so good because you are able to break down hard problems into their component parts, \
answer the component parts, and then put them together to answer the broader question.

Here is a question:
{query}"""

embeddings = OpenAIEmbeddings()
prompt_templates = [physics_template, math_template]
prompt_embeddings = embeddings.embed_documents(prompt_templates)

def prompt_router(input):
    query_embedding = embeddings.embed_query(input["query"])
    similarity = cosine_similarity([query_embedding], prompt_embeddings)[0]
    most_similar = prompt_templates[similarity.argmax()]
    print("Using MATH" if most_similar == math_template else "Using PHYSICS")
    return PromptTemplate.from_template(most_similar)

chain = (
    {"query": RunnablePassthrough()}
    | RunnableLambda(prompt_router)
    | ChatOpenAI()
    | StrOutputParser()
)

print(chain.invoke({"query": "What's a black hole"}))
print(chain.invoke({"query": "What's a path integral"}))
```

이 예제에서 `prompt_router` 함수는 사용자 입력과 물리학 및 수학 질문에 대한 미리 정의된 프롬프트 템플릿 사이의 코사인 유사성을 계산합니다. 유사성 점수에 따라 체인은 가장 관련성이 높은 프롬프트 템플릿을 동적으로 선택하여 챗봇이 사용자의 질문에 적절하게 응답하도록 합니다.

### 에이전트 및 Runnables 사용

LangChain을 사용하면 Runnables, 프롬프트, 모델 및 도구를 결합하여 에이전트를 생성할 수 있습니다. 다음은 에이전트를 구축하고 사용하는 방법의 예시입니다:

```python
from langchain.agents import XMLAgent, tool, AgentExecutor
from langchain.chat_models import ChatAnthropic

model = ChatAnthropic(model="claude-2")

@tool
def search(query: str) -> str:
    """Search things about current events."""
    return "32 degrees"

tool_list = [search]

# Get prompt to use
prompt = XMLAgent.get_default_prompt()

# Logic for going from intermediate steps to a string to pass into the model
def convert_intermediate_steps(intermediate_steps):
    log = ""
    for action, observation in intermediate_steps:
        log += (
            f"<tool>{action.tool}</tool><tool_input>{action.tool_input}"
            f"</tool_input><observation>{observation}</observation>"
        )
    return log

# Logic for converting tools to a string to go in the prompt
def convert_tools(tools):
    return "\n".join([f"{tool.name}: {tool.description}" for tool in tools])

agent = (
    {
        "question": lambda x: x["question"],
        "intermediate_steps": lambda x: convert_intermediate_steps(
            x["intermediate_steps"]
        ),
    }
    | prompt.partial(tools=convert_tools(tool_list))
    | model.bind(stop=["</tool_input>", "</final_answer>"])
    | XMLAgent.get_default_output_parser()
)

agent_executor = AgentExecutor(agent=agent, tools=tool_list, verbose=True)

result = agent_executor.invoke({"question": "What's the weather in New York?"})
print(result)
```

이 예제에서 모델, 도구, 프롬프트 및 중간 단계와 도구 변환을 위한 사용자 지정 논리를 결합하여 에이전트를 생성합니다. 그런 다음 에이전트를 실행하여 사용자의 쿼리에 대한 응답을 제공합니다.

### SQL 데이터베이스 쿼리

LangChain을 사용하여 SQL 데이터베이스를 쿼리하고 사용자 질문을 기반으로 SQL 쿼리를 생성할 수 있습니다. 다음은 그 예시입니다:

```python
from langchain.prompts import ChatPromptTemplate

template = """Based on the table schema below, write a SQL query that would answer the user's question:
{schema}

Question: {question}
SQL Query:"""
prompt = ChatPromptTemplate.from_template(template)

from langchain.utilities import SQLDatabase

# Initialize the database (you'll need the Chinook sample DB for this example)
db = SQLDatabase.from_uri("sqlite:///./Chinook.db")

def get_schema(_):
    return db.get_table_info()

def run_query(query):
    return db.run(query)

from langchain.chat_models import ChatOpenAI
from langchain.schema.output_parser import StrOutputParser
from langchain.schema.runnable import RunnablePassthrough

model = ChatOpenAI()

sql_response = (
    RunnablePassthrough.assign(schema=get_schema)
    | prompt
    | model.bind(stop=["\nSQLResult:"])
    | StrOutputParser()
)

result = sql_response.invoke({"question": "How many employees are there?"})
print(result)

template = """Based on the table schema below, question, SQL query, and SQL response, write a natural language response:
{schema}

Question: {question}
SQL Query: {query}
SQL Response: {response}"""
prompt_response = ChatPromptTemplate.from_template(template)

full_chain = (
    RunnablePassthrough.assign(query=sql_response)
    | RunnablePassthrough.assign(
        schema=get_schema,
        response=lambda x: db.run(x["query"]),
    )
    | prompt_response
    | model
)

response = full_chain.invoke({"question": "How many employees are there?"})
print(response)
```

이 예제에서 LangChain은 사용자 질문을 기반으로 SQL 쿼리를 생성하고 SQL 데이터베이스에서 응답을 검색하는 데 사용됩니다. 프롬프트와 응답은 데이터베이스와의 자연어 상호 작용을 제공하기 위해 포맷됩니다.

LangChain Expression Language(LCEL)은 복잡한 언어 처리 파이프라인을 구축하고 배포하는 데 필요한 유연성과 표현력을 제공합니다. 프롬프트, 언어 모델, 출력 파서를 결합하는 기본적인 것부터 검색 증강 생성, 대화형 검색, 메모리 사용, 분기 및 병합, Python 코드 생성, 에이전트 생성, SQL 데이터베이스 쿼리에 이르기까지 다양한 사용 사례와 시나리오를 다룹니다.

LCEL의 주요 이점 중 일부는 다음과 같습니다:

1. 직관적인 구문: LCEL의 선언적 구문은 복잡한 파이프라인을 쉽게 표현할 수 있게 해줍니다.
2. 모듈성: LCEL의 구성 요소 기반 아키텍처를 통해 개발자는 재사용 가능한 구성 요소를 만들고 다양한 사용 사례에 적용할 수 있습니다.
3. 확장성: LCEL은 분산 환경에서 병렬 처리 및 스케일아웃을 지원하여 대규모 워크로드를 처리할 수 있습니다.
4. 통합: LCEL은 외부 도구, 서비스 및 데이터 소스와의 원활한 통합을 가능하게 하여 언어 처리 파이프라인의 기능을 확장합니다.
5. 관찰 가능성: LCEL의 포괄적인 추적 및 모니터링 기능을 통해 개발자는 복잡한 파이프라인의 동작을 이해하고 디버그할 수 있습니다.

LCEL은 자연어 처리와 기계 학습의 빠르게 발전하는 분야에서 강력하고 유연한 도구입니다. 개발자가 정교한 언어 처리 애플리케이션을 빌드하고 배포할 수 있는 직관적이고 표현력 있는 방법을 제공함으로써 LCEL은 대화형 AI, 콘텐츠 생성, 의미 검색 등 다양한 사용 사례를 가능하게 합니다.

개발자는 LCEL을 사용하여 최첨단 언어 모델과 고급 검색 및 메모리 기술을 활용하는 강력한 언어 기반 애플리케이션을 만들 수 있습니다. 또한 LCEL의 모듈식 아키텍처와 확장 가능한 설계를 통해 개발자는 특정 사용 사례와 요구 사항에 맞게 파이프라인을 쉽게 사용자 정의하고 최적화할 수 있습니다.

LCEL의 통합 기능을 통해 개발자는 SQL 데이터베이스, 검색 엔진, 외부 API 등 다양한 데이터 소스와 서비스를 원활하게 통합할 수 있습니다. 이를 통해 언어 처리 파이프라인을 풍부한 데이터와 도메인 전문 지식으로 강화하여 더욱 정확하고 관련성 있는 결과를 얻을 수 있습니다.

종합적으로 LangChain Expression Language는 현대 언어 기반 애플리케이션을 구축하기 위한 강력하고 유연한 프레임워크를 제공합니다. 직관적인 구문, 모듈식 설계, 확장성을 갖춘 LCEL은 자연어 처리와 기계 학습 분야에서 개발자가 혁신적인 솔루션을 만드는 데 있어 귀중한 도구입니다.

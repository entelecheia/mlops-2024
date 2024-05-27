# LangChain 모듈 - 메모리

LangChain에서 메모리는 대화형 인터페이스의 근본적인 측면으로, 시스템이 과거의 상호작용을 참조할 수 있게 합니다. 이는 정보를 저장하고 조회하는 과정을 통해 이루어지며, 주로 읽기와 쓰기의 두 가지 주요 작업이 포함됩니다. 메모리 시스템은 실행 중에 체인과 두 번 상호작용하며, 사용자 입력을 증강하고 입력 및 출력을 저장하여 향후 참조할 수 있게 합니다.

## 시스템에 메모리 구축하기

1. **채팅 메시지 저장:** LangChain 메모리 모듈은 메모리 목록에서 데이터베이스에 이르기까지 다양한 방법을 통합하여 채팅 메시지를 저장합니다. 이를 통해 모든 채팅 상호작용이 기록되어 나중에 참조할 수 있습니다.
2. **채팅 메시지 조회:** 메시지를 저장하는 것 외에도, LangChain은 이러한 메시지를 유용하게 보기 위해 데이터 구조 및 알고리즘을 사용합니다. 간단한 메모리 시스템은 최근 메시지를 반환할 수 있지만, 더 고급 시스템은 과거 상호작용을 요약하거나 현재 상호작용에서 언급된 엔티티에 집중할 수 있습니다.

LangChain에서 메모리 사용을 보여주기 위해 `ConversationBufferMemory` 클래스를 예로 들어보겠습니다. 이는 채팅 메시지를 버퍼에 저장하는 간단한 메모리 형태입니다. 다음은 예제 코드입니다:

```python
from langchain.memory import ConversationBufferMemory

memory = ConversationBufferMemory()
memory.chat_memory.add_user_message("Hello!")
memory.chat_memory.add_ai_message("How can I assist you?")
```

메모리를 체인에 통합할 때, 메모리에서 반환된 변수를 이해하고 이를 체인에서 어떻게 사용하는지 이해하는 것이 중요합니다. 예를 들어, `load_memory_variables` 메서드는 메모리에서 읽어온 변수를 체인의 기대와 맞추는 데 도움이 됩니다.

**LangChain을 사용한 엔드투엔드 예제**

`ConversationBufferMemory`를 `LLMChain`에서 사용하는 예제를 살펴보겠습니다. 체인과 적절한 프롬프트 템플릿 및 메모리가 결합되어 일관되고 맥락을 이해하는 대화 경험을 제공합니다. 다음은 간단한 예제입니다:

```python
from langchain.llms import OpenAI
from langchain.prompts import PromptTemplate
from langchain.chains import LLMChain
from langchain.memory import ConversationBufferMemory

llm = OpenAI(temperature=0)
template = "Your conversation template here..."
prompt = PromptTemplate.from_template(template)
memory = ConversationBufferMemory(memory_key="chat_history")
conversation = LLMChain(llm=llm, prompt=prompt, memory=memory)

response = conversation({"question": "What's the weather like?"})
```

이 예제는 LangChain의 메모리 시스템이 체인과 어떻게 통합되어 일관되고 맥락을 이해하는 대화 경험을 제공하는지 보여줍니다.

## LangChain의 메모리 유형

LangChain은 AI 모델과의 상호작용을 향상시키기 위해 다양한 메모리 유형을 제공합니다. 각 메모리 유형은 고유한 매개변수와 반환 유형을 가지고 있어, 다양한 시나리오에 적합합니다. LangChain에서 사용할 수 있는 몇 가지 메모리 유형과 코드 예제를 살펴보겠습니다.

### 대화 버퍼 메모리 (Conversation Buffer Memory)

이 메모리 유형은 대화에서 메시지를 저장하고 추출할 수 있게 합니다. 대화의 역사를 문자열 또는 메시지 목록으로 추출할 수 있습니다.

```python
from langchain.memory import ConversationBufferMemory

memory = ConversationBufferMemory()
memory.save_context({"input": "hi"}, {"output": "whats up"})
memory.load_memory_variables({})

# 역사(history)를 문자열로 추출
{'history': 'Human: hi\nAI: whats up'}

# 역사(history)를 메시지 목록으로 추출
{'history': [
  HumanMessage(content='hi', additional_kwargs={}),
  AIMessage(content='whats up', additional_kwargs={})
]}
```

Conversation Buffer Memory는 채팅과 같은 상호작용을 위해 체인에 사용할 수도 있습니다.

### 대화 버퍼 창 메모리 (Conversation Buffer Window Memory)

이 메모리 유형은 최근 상호작용 목록을 유지하며, 마지막 K개의 상호작용을 사용합니다. 이를 통해 버퍼가 너무 커지는 것을 방지합니다.

```python
from langchain.memory import ConversationBufferWindowMemory

memory = ConversationBufferWindowMemory(k=1)
memory.save_context({"input": "hi"}, {"output": "whats up"})
memory.save_context({"input": "not much you"}, {"output": "not much"})
memory.load_memory_variables({})

{'history': 'Human: not much you\nAI: not much'}
```

대화 버퍼 메모리와 마찬가지로, 이 메모리 유형도 채팅과 같은 상호작용을 위해 체인에 사용할 수 있습니다.

### 대화 엔티티 메모리 (Conversation Entity Memory)

이 메모리 유형은 대화에서 특정 엔티티에 대한 사실을 기억하고 LLM을 사용하여 정보를 추출합니다.

```python
from langchain.memory import ConversationEntityMemory
from langchain.llms import OpenAI

llm = OpenAI(temperature=0)
memory = ConversationEntityMemory(llm=llm)
_input = {"input": "Deven & Sam are working on a hackathon project"}
memory.load_memory_variables(_input)
memory.save_context(
    _input,
    {"output": " That sounds like a great project! What kind of project are they working on?"}
)
memory.load_memory_variables({"input": 'who is Sam'})

{'history': 'Human: Deven & Sam are working on a hackathon project\nAI:  That sounds like a great project! What kind of project are they working on?',
 'entities': {'Sam': 'Sam is working on a hackathon project with Deven.'}}
```

### 대화 지식 그래프 메모리 (Conversation Knowledge Graph Memory)

이 메모리 유형은 지식 그래프를 사용하여 메모리를 재구성합니다. 메시지에서 현재 엔티티와 지식 삼중항을 추출할 수 있습니다.

```python
from langchain.memory import ConversationKGMemory
from langchain.llms import OpenAI

llm = OpenAI(temperature=0)
memory = ConversationKGMemory(llm=llm)
memory.save_context({"input": "say hi to sam"}, {"output": "who is sam"})
memory.save_context({"input": "sam is a friend"}, {"output": "okay"})
memory.load_memory_variables({"input": "who is sam"})

{'history': 'On Sam: Sam is friend.'}
```

이 메모리 유형은 대화 기반 지식 검색을 위해 체인에 사용할 수도 있습니다.

### 대화 요약 메모리 (Conversation Summary Memory)

이 메모리 유형은 시간에 따라 대화의 요약을 생성하며, 긴 대화에서 정보를 요약하는 데 유용합니다.

```python
from langchain.memory import ConversationSummaryMemory
from langchain.llms import OpenAI

llm = OpenAI(temperature=0)
memory = ConversationSummaryMemory(llm=llm)
memory.save_context({"input": "hi"}, {"output": "whats up"})
memory.load_memory_variables({})

{'history': '\nThe human greets the AI, to which the AI responds.'}
```

### 대화 요약 버퍼 메모리 (Conversation Summary Buffer Memory)

이 메모리 유형은 대화 요약과 버퍼를 결합하여 최근 상호작용과 요약 간의 균형을 유지합니다. 토큰 길이를 사용하여 상호작용을 플러시할 때를 결정합니다.

```python
from langchain.memory import ConversationSummaryBufferMemory
from langchain.llms import OpenAI

llm = OpenAI()
memory = ConversationSummaryBufferMemory(llm=llm, max_token_limit=10)
memory.save_context({"input": "hi"}, {"output": "whats up"})
memory.save_context({"input": "not much you"}, {"output": "not much"})
memory.load_memory_variables({})

{'history': 'System: \nThe human says "hi", and the AI responds with "whats up".\nHuman: not much you\nAI: not much'}
```

이러한 메모리 유형을 사용하여 LangChain에서 AI 모델과의 상호작용을 향상시킬 수 있습니다. 각 메모리 유형은 특정 목적에 맞게 설계되었으며, 요구 사항에 따라 선택할 수 있습니다.

### 대화 토큰 버퍼 메모리 (Conversation Token Buffer Memory)

Conversation Token Buffer Memory는 최근 상호작용을 메모리에 저장하는 또 다른 메모리 유형입니다. 이전의 메모리 유형들이 상호작용의 수에 초점을 맞춘 것과 달리, 이 메모리는 토큰 길이를 사용하여 상호작용을 플러시할 때를 결정합니다.

LLM과 함께 사용하는 예제:

```python
from langchain.memory import ConversationTokenBufferMemory
from langchain.llms import OpenAI

llm = OpenAI()

memory = ConversationTokenBufferMemory(llm=llm, max_token_limit=10)
memory.save_context({"input": "hi"}, {"output": "whats up"})
memory.save_context({"input": "not much you"}, {"output": "not much"})

memory.load_memory_variables({})

{'history': 'Human: not much you\nAI: not much'}
```

이 예제에서는 메모리가 상호작용의 수가 아닌 토큰 길이에 따라 상호작용을 제한하도록 설정됩니다.

메시지 목록으로 역사를 가져오는 방법:

```python
memory = ConversationTokenBufferMemory(
    llm=llm, max_token_limit=10, return_messages=True
)
memory.save_context({"input": "hi"}, {"output": "whats up"})
memory.save_context({"input": "not much you"}, {"output": "not much"})
```

체인에서 사용하는 예제:

ConversationTokenBufferMemory를 체인에서 사용하여 AI 모델과의 상호작용을 관리할 수 있습니다.

```python
from langchain.chains import ConversationChain

conversation_with_summary = ConversationChain(
    llm=llm,
    # 테스트 목적으로 max_token_limit을 매우 낮게 설정
    memory=ConversationTokenBufferMemory(llm=OpenAI(), max_token_limit=60),
    verbose=True,
)
conversation_with_summary.predict(input="Hi, what's up?")
```

이 예제에서 ConversationTokenBufferMemory는 ConversationChain에서 사용되어 대화를 관리하고 토큰 길이에 따라 상호작용을 제한합니다.

### VectorStoreRetrieverMemory

**VectorStoreRetrieverMemory**는 메모리를 벡터 스토어에 저장하고, 호출될 때마다 가장 "중요한" 상위 K개의 문서를 쿼리합니다. 이 메모리 유형은 상호작용의 순서를 명시적으로 추적하지 않고 벡터 검색을 사용하여 관련 메모리를 가져옵니다.

#### 코드 예제

```python
from datetime import datetime
from langchain.embeddings.openai import OpenAIEmbeddings
from langchain.llms import OpenAI
from langchain.memory import VectorStoreRetrieverMemory
from langchain.chains import ConversationChain
from langchain.prompts import PromptTemplate

# 벡터 스토어 초기화
import faiss
from langchain.docstore import InMemoryDocstore
from langchain.vectorstores import FAISS

embedding_size = 1536  # OpenAIEmbeddings의 차원 수
index = faiss.IndexFlatL2(embedding_size)
embedding_fn = OpenAIEmbeddings().embed_query
vectorstore = FAISS(embedding_fn, index, InMemoryDocstore({}), {})

# VectorStoreRetrieverMemory 생성
retriever = vectorstore.as_retriever(search_kwargs=dict(k=1))
memory = VectorStoreRetrieverMemory(retriever=retriever)

# 컨텍스트와 관련 정보를 메모리에 저장
memory.save_context({"input": "My favorite food is pizza"}, {"output": "that's good to know"})
memory.save_context({"input": "My favorite sport is soccer"}, {"output": "..."})
memory.save_context({"input": "I don't like the Celtics"}, {"output": "ok"})

# 쿼리를 기반으로 메모리에서 관련 정보 검색
print(memory.load_memory_variables({"prompt": "what sport should i watch?"})["history"])
```

이 예제에서 **VectorStoreRetrieverMemory**는 대화에서 중요한 정보를 벡터 스토어에 저장하고, 쿼리할 때 관련 메모리를 검색합니다. 이를 통해 순서를 추적하지 않고도 중요한 정보를 효율적으로 찾을 수 있습니다.

#### 체인에서 사용

**VectorStoreRetrieverMemory**를 체인에서 사용하여 대화 기반 지식 검색을 강화할 수 있습니다.

```python
from langchain.chains import ConversationChain

conversation_with_summary = ConversationChain(
    llm=llm,
    # 테스트 목적으로 max_token_limit을 매우 낮게 설정
    memory=VectorStoreRetrieverMemory(llm=OpenAI(), max_token_limit=60),
    verbose=True,
)
conversation_with_summary.predict(input="Hi, what's up?")
```

이 예제에서 **VectorStoreRetrieverMemory**는 **ConversationChain**에서 사용되어 대화를 관리하고 토큰 길이에 따라 상호작용을 제한합니다.

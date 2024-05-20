# LangServe로 배포하기

LangServe는 LangChain 런너블(runnable) 및 체인을 REST API로 배포할 수 있도록 돕는 도구입니다. 이 라이브러리는 FastAPI와 통합되어 있으며, 데이터 검증을 위해 Pydantic을 사용합니다. 또한, 서버에 배포된 런너블을 호출할 수 있는 클라이언트와 LangChainJS에서 사용할 수 있는 JavaScript 클라이언트를 제공합니다.

## 특징

- 입력 및 출력 스키마가 LangChain 객체에서 자동으로 추론되어 각 API 호출 시 엄격하게 적용됩니다. 이와 함께 풍부한 에러 메시지가 제공됩니다.
- JSONSchema와 Swagger를 포함한 API 문서 페이지가 제공됩니다.
- /invoke, /batch, /stream 엔드포인트는 단일 서버에서 많은 동시 요청을 처리할 수 있도록 효율적으로 설계되었습니다.
- /stream_log 엔드포인트를 통해 체인/에이전트의 모든 중간 단계를 스트리밍할 수 있습니다.
- /playground 페이지에서 스트리밍 출력과 중간 단계를 사용하여 런너블을 구성하고 호출할 수 있습니다.
- (선택 사항으로) LangSmith로의 내장 추적 기능을 추가할 수 있습니다. API 키만 추가하면 됩니다.
- FastAPI, Pydantic, uvloop, asyncio와 같은 오픈 소스 Python 라이브러리를 사용하여 안정성이 보장됩니다.

## 한계

- 서버에서 발생하는 이벤트에 대해 클라이언트 콜백을 지원하지 않습니다.
- Pydantic V2를 사용할 때 OpenAPI 문서가 생성되지 않습니다. FastAPI는 pydantic v1과 v2 네임스페이스를 혼용하는 것을 지원하지 않습니다. 자세한 내용은 아래 섹션을 참조하십시오.

## LangServe 프로젝트 부트스트랩

LangChain CLI를 사용하여 LangServe 프로젝트를 빠르게 부트스트랩할 수 있습니다. 이를 위해 최신 버전의 langchain-cli가 설치되어 있는지 확인하십시오. 설치는 다음 명령어를 사용합니다:

```
pip install -U langchain-cli
```

LangChain CLI를 사용하여 LangServe 인스턴스를 빠르게 시작할 수 있습니다. 다음 명령어를 사용하여 새로운 LangServe 프로젝트를 생성합니다:

```
langchain app new ./path/to/directory
```

다음 예제는 OpenAI 채팅 모델, Anthropic 채팅 모델, 그리고 주제에 대해 농담을 하는 체인을 배포하는 서버입니다.

```python
#!/usr/bin/env python
from fastapi import FastAPI
from langchain.prompts import ChatPromptTemplate
from langchain.chat_models import ChatAnthropic, ChatOpenAI
from langserve import add_routes

app = FastAPI(
  title="LangChain Server",
  version="1.0",
  description="A simple api server using Langchain's Runnable interfaces",
)

add_routes(
    app,
    ChatOpenAI(),
    path="/openai",
)

add_routes(
    app,
    ChatAnthropic(),
    path="/anthropic",
)

model = ChatAnthropic()
prompt = ChatPromptTemplate.from_template("tell me a joke about {topic}")
add_routes(
    app,
    prompt | model,
    path="/chain",
)

if __name__ == "__main__":
    import uvicorn

    uvicorn.run(app, host="localhost", port=8000)
```

서버를 배포한 후, 생성된 OpenAPI 문서를 다음 명령어로 확인할 수 있습니다:

```
curl localhost:8000/docs
```

/docs 접미사를 추가해야 합니다.

다음은 다양한 엔드포인트를 사용하는 Python 코드입니다:

```python
from langchain.schema import SystemMessage, HumanMessage
from langchain.prompts import ChatPromptTemplate
from langchain.schema.runnable import RunnableMap
from langserve import RemoteRunnable

openai = RemoteRunnable("http://localhost:8000/openai/")
anthropic = RemoteRunnable("http://localhost:8000/anthropic/")
joke_chain = RemoteRunnable("http://localhost:8000/chain/")

joke_chain.invoke({"topic": "parrots"})

# or async
await joke_chain.ainvoke({"topic": "parrots"})

prompt = [
    SystemMessage(content='Act like either a cat or a parrot.'),
    HumanMessage(content='Hello!')
]

# Supports astream
async for msg in anthropic.astream(prompt):
    print(msg, end="", flush=True)

prompt = ChatPromptTemplate.from_messages(
    [("system", "Tell me a long story about {topic}")]
)

# Can define custom chains
chain = prompt | RunnableMap({
    "openai": openai,
    "anthropic": anthropic,
})

chain.batch([{"topic": "parrots"}, {"topic": "cats"}])
```

JavaScript에서는 다음과 같이 사용할 수 있습니다(버전 0.0.166 이상의 LangChain.js 필요):

```javascript
import { RemoteRunnable } from "langchain/runnables/remote";

const chain = new RemoteRunnable({
  url: `http://localhost:8000/chain/invoke/`,
});
const result = await chain.invoke({
  topic: "cats",
});
```

Python의 requests를 사용하여 호출할 수도 있습니다:

```python
import requests
response = requests.post(
    "http://localhost:8000/chain/invoke/",
    json={'input': {'topic': 'cats'}}
)
response.json()
```

curl을 사용한 예제:

```shell
curl --location --request POST 'http://localhost:8000/chain/invoke/' \
    --header 'Content-Type: application/json' \
    --data-raw '{
        "input": {
            "topic": "cats"
        }
    }'
```

다음 코드는 서버에 엔드포인트를 추가합니다:

```python
...
add_routes(
  app,
  runnable,
  path="/my_runnable",
)
```

이 코드는 다음 엔드포인트를 서버에 추가합니다:

- POST /my_runnable/invoke - 단일 입력에 대해 런너블 호출
- POST /my_runnable/batch - 입력 배치에 대해 런너블 호출
- POST /my_runnable/stream - 단일 입력에 대해 호출하고 출력 스트리밍
- POST /my_runnable/stream_log - 단일 입력에 대해 호출하고 생성된 모든 중간 단계를 포함하여 출력 스트리밍
- GET /my_runnable/input_schema - 런너블의 입력에 대한 JSON 스키마
- GET /my_runnable/output_schema - 런너블의 출력에 대한 JSON 스키마
- GET /my_runnable/config_schema - 런너블의 설정에 대한 JSON 스키마

런너블에 대한 playground 페이지는 /my_runnable/playground에서 확인할 수 있습니다. 이 페이지에서는 스트리밍 출력과 중간 단계를 사용하여 런너블을 구성하고 호출할 수 있습니다.

클라이언트 및 서버 모두 다음 명령어로 설치할 수 있습니다:

```shell
pip install "langserve[all]"
```

또는 클라이언트 코드용으로는 `pip install "langserve[client]"`, 서버 코드용으로는 `pip install "langserve[server]"`를 사용합니다.

서버에 인증을 추가해야 하는 경우, FastAPI의 보안 문서와 미들웨어 문서를 참조하십시오.

GCP Cloud Run에 배포하려면 다음 명령어를 사용하십시오:

```shell
gcloud run deploy [your-service-name] --source . --port 8001 --allow-unauthenticated --region us-central1 --set-env-vars=OPENAI_API_KEY=your_key
```

LangServe는 Pydantic 2를 일부 제한 사항과 함께 지원합니다. Pydantic V2를 사용할 때 invoke/batch/stream/stream_log에 대한 OpenAPI 문서가 생성되지 않습니다. FastAPI는 pydantic v1과 v2 네임스페이스를 혼용하는 것을 지원하지 않습니다. LangChain은 Pydantic v2에서 v1 네임스페이스를 사용합니다. 이 제한 사항 외에는 API 엔드포인트, playground, 기타 기능이 예상대로 작동해야 합니다.

LLM 애플리케이션은 종종 파일을 처리합니다. 파일 처리를 구현하기 위한 다양한 아키텍처가 있습니다. 높은 수준에서 다음과 같은 방법이 있습니다:

- 파일을 전용 엔드포인트를 통해 서버에 업로드하고 별도의 엔드포인트를 사용하여 처리합니다.
- 파일을 값(파일의 바이트) 또는 참조(e.g., s3 URL로 파일 내용)로 업로드할 수 있습니다.
- 처리 엔드포인트는 블로킹 또는 논블로킹일 수 있습니다.
- 상당한 처리가 필요한 경우 전용 프로세스 풀로 처리를 오프로드할 수 있습니다.

애플리케이션에 적합한 아키텍처를 결정해야 합니다. 현재 값으로 파일을 업로드하려면 base64 인코딩을 사용하여 런너블로 업로드합니다(multipart/form-data는 아직 지원되지 않습니다).

다음은 base64 인코딩을 사용하여 파일을 원격 런너블에 보내는 방법을 보여주는 예제입니다:

```python
try:
    from pydantic.v1 import Field
except ImportError:
    from pydantic import Field

from langserve import CustomUserType

# CustomUserType을 상속받지 않고 BaseModel을 상속받으면 서버는 이를 dict로 디코딩합니다.
class FileProcessingRequest(CustomUserType):
    """base64로 인코딩된 파일을 포함하는 요청입니다."""

    # 추가 필드는 playground UI용 위젯을 지정하는 데 사용됩니다.
    file: str = Field(..., extra={"widget": {"type": "base64file"}})
    num_chars: int = 100
```

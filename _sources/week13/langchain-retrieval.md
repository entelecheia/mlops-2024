# LangChain 모듈 - 검색

LangChain에서의 검색(Retrieval)은 모델의 학습 데이터셋에 포함되지 않은 사용자 특정 데이터가 필요한 애플리케이션에서 중요한 역할을 합니다. 이 과정을 Retrieval Augmented Generation (RAG)이라고 하며, 외부 데이터를 가져와 언어 모델의 생성 과정에 통합하는 것을 포함합니다. LangChain은 간단한 애플리케이션부터 복잡한 애플리케이션까지 이 과정을 용이하게 하는 포괄적인 도구 및 기능 세트를 제공합니다.

LangChain은 여러 컴포넌트를 통해 검색을 수행하며, 이를 하나씩 살펴보겠습니다.

## 문서 로더 (Document Loaders)

LangChain의 문서 로더를 사용하면 다양한 소스에서 데이터를 추출할 수 있습니다. 100개 이상의 로더가 사용 가능하며, 다양한 문서 유형, 앱 및 소스(프라이빗 S3 버킷, 공개 웹사이트, 데이터베이스)를 지원합니다.

요구 사항에 따라 문서 로더를 선택할 수 있습니다. 이러한 모든 로더는 데이터를 **Document** 클래스로 수집합니다. Document 클래스로 수집된 데이터를 사용하는 방법은 나중에 배우게 될 것입니다.

### 텍스트 파일 로더 (Text File Loader)

간단한 `.txt` 파일을 문서로 로드합니다.

```python
from langchain.document_loaders import TextLoader

loader = TextLoader("./sample.txt")
document = loader.load()
```

### CSV 로더 (CSV Loader)

CSV 파일을 문서로 로드합니다.

```python
from langchain.document_loaders.csv_loader import CSVLoader

loader = CSVLoader(file_path='./example_data/sample.csv')
documents = loader.load()
```

필드 이름을 지정하여 파싱을 커스터마이징할 수도 있습니다.

```python
loader = CSVLoader(file_path='./example_data/mlb_teams_2012.csv', csv_args={
    'delimiter': ',',
    'quotechar': '"',
    'fieldnames': ['MLB Team', 'Payroll in millions', 'Wins']
})
documents = loader.load()
```

### PDF 로더 (PDF Loaders)

LangChain의 PDF 로더는 PDF 파일에서 컨텐츠를 파싱하고 추출하기 위한 다양한 방법을 제공합니다. 각 로더는 서로 다른 요구 사항에 맞춰져 있고 다른 기본 라이브러리를 사용합니다. 아래는 각 로더에 대한 자세한 예시입니다.

#### PyPDFLoader

기본적인 PDF 파싱에 사용됩니다.

```python
from langchain.document_loaders import PyPDFLoader

loader = PyPDFLoader("example_data/layout-parser-paper.pdf")
pages = loader.load_and_split()
```

#### MathPixLoader

수학 컨텐츠와 다이어그램 추출에 이상적입니다.

```python
from langchain.document_loaders import MathpixPDFLoader

loader = MathpixPDFLoader("example_data/math-content.pdf")
data = loader.load()
```

#### PyMuPDFLoader

빠르고 상세한 메타데이터 추출을 포함합니다.

```python
from langchain.document_loaders import PyMuPDFLoader

loader = PyMuPDFLoader("example_data/layout-parser-paper.pdf")
data = loader.load()

# PyMuPDF의 get_text() 호출에 추가 인자 전달 가능
data = loader.load(option="text")
```

#### PDFMinerLoader

텍스트 추출에 대한 보다 세분화된 제어에 사용됩니다.

```python
from langchain.document_loaders import PDFMinerLoader

loader = PDFMinerLoader("example_data/layout-parser-paper.pdf")
data = loader.load()
```

#### AmazonTextractPDFParser

OCR 및 기타 고급 PDF 파싱 기능을 위해 AWS Textract를 활용합니다.

```python
from langchain.document_loaders import AmazonTextractPDFLoader

# AWS 계정 및 구성 필요
loader = AmazonTextractPDFLoader("example_data/complex-layout.pdf")
documents = loader.load()
```

#### PDFMinerPDFasHTMLLoader

의미론적 파싱을 위해 PDF에서 HTML을 생성합니다.

```python
from langchain.document_loaders import PDFMinerPDFasHTMLLoader

loader = PDFMinerPDFasHTMLLoader("example_data/layout-parser-paper.pdf")
data = loader.load()
```

#### PDFPlumberLoader

상세한 메타데이터를 제공하고 페이지당 하나의 문서를 지원합니다.

```python
from langchain.document_loaders import PDFPlumberLoader

loader = PDFPlumberLoader("example_data/layout-parser-paper.pdf")
data = loader.load()
```

## 통합 로더 (Integrated Loaders)

LangChain은 Slack, Sigma, Notion, Confluence, Google Drive 등의 앱 및 데이터베이스에서 데이터를 직접 로드하여 LLM 애플리케이션에 사용할 수 있는 다양한 맞춤형 로더를 제공합니다.

아래는 이를 설명하기 위한 몇 가지 예시입니다.

### 예시 I - Slack

널리 사용되는 인스턴트 메시징 플랫폼인 Slack은 LLM 워크플로와 애플리케이션에 통합될 수 있습니다.

1. Slack 작업 공간 관리 페이지로 이동하세요.
2. `{your_slack_domain}.slack.com/services/export`로 이동하세요.
3. 원하는 날짜 범위를 선택하고 내보내기를 시작하세요.
4. 내보내기가 완료되면 Slack에서 이메일과 DM을 통해 알려줍니다.
5. 내보내기는 다운로드 폴더 또는 지정된 다운로드 경로에 `.zip` 파일을 생성합니다.
6. 다운로드한 `.zip` 파일의 경로를 `LOCAL_ZIPFILE`에 할당하세요.
7. `langchain.document_loaders` 패키지에서 `SlackDirectoryLoader`를 사용하세요.

```python
from langchain.document_loaders import SlackDirectoryLoader

SLACK_WORKSPACE_URL = "https://xxx.slack.com" # Slack URL로 바꾸세요
LOCAL_ZIPFILE = "" # Slack zip 파일의 경로

loader = SlackDirectoryLoader(LOCAL_ZIPFILE, SLACK_WORKSPACE_URL)
docs = loader.load()
print(docs)
```

이렇게 Slack 데이터를 로드하여 LLM 애플리케이션에서 활용할 수 있습니다. LangChain은 문서 로더를 통해 다양한 소스의 데이터를 쉽게 통합할 수 있도록 지원하며, 이는 RAG 프로세스에서 핵심적인 역할을 합니다.

## 문서 변환기 (Document Transformers)

LangChain의 문서 변환기는 이전 섹션에서 생성한 문서를 조작하는데 필수적인 도구입니다.

이들은 긴 문서를 더 작은 청크로 분할하고, 결합하고, 필터링하는 등의 작업에 사용되며, 이는 문서를 모델의 컨텍스트 윈도우에 맞추거나 특정 애플리케이션 요구사항을 충족시키는 데 중요합니다.

이러한 도구 중 하나는 RecursiveCharacterTextSplitter로, 문자 목록을 사용하여 분할하는 다재다능한 텍스트 분할기입니다. 청크 크기, 오버랩, 시작 인덱스와 같은 매개변수를 허용합니다. 다음은 Python에서 사용하는 예시입니다:

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

state_of_the_union = "Your long text here..."

text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=100,
    chunk_overlap=20,
    length_function=len,
    add_start_index=True,
)

texts = text_splitter.create_documents([state_of_the_union])
print(texts[0])
print(texts[1])
```

또 다른 도구는 CharacterTextSplitter로, 지정된 문자를 기준으로 텍스트를 분할하고 청크 크기와 오버랩을 제어할 수 있습니다:

```python
from langchain.text_splitter import CharacterTextSplitter

text_splitter = CharacterTextSplitter(
    separator="\n\n",
    chunk_size=1000,
    chunk_overlap=200,
    length_function=len,
    is_separator_regex=False,
)

texts = text_splitter.create_documents([state_of_the_union])
print(texts[0])
```

HTMLHeaderTextSplitter는 HTML 컨텐츠를 헤더 태그를 기준으로 분할하여 의미 구조를 유지하도록 설계되었습니다:

```python
from langchain.text_splitter import HTMLHeaderTextSplitter

html_string = "Your HTML content here..."
headers_to_split_on = [("h1", "Header 1"), ("h2", "Header 2")]

html_splitter = HTMLHeaderTextSplitter(headers_to_split_on=headers_to_split_on)
html_header_splits = html_splitter.split_text(html_string)
print(html_header_splits[0])
```

HTMLHeaderTextSplitter와 다른 분할기(예: Pipelined Splitter)를 결합하여 더 복잡한 조작을 수행할 수 있습니다:

```python
from langchain.text_splitter import HTMLHeaderTextSplitter, RecursiveCharacterTextSplitter

url = "https://example.com"
headers_to_split_on = [("h1", "Header 1"), ("h2", "Header 2")]
html_splitter = HTMLHeaderTextSplitter(headers_to_split_on=headers_to_split_on)
html_header_splits = html_splitter.split_text_from_url(url)

chunk_size = 500
text_splitter = RecursiveCharacterTextSplitter(chunk_size=chunk_size)
splits = text_splitter.split_documents(html_header_splits)
print(splits[0])
```

LangChain은 Python Code Splitter 및 JavaScript Code Splitter와 같이 다양한 프로그래밍 언어에 특화된 분할기도 제공합니다:

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter, Language

python_code = """
def hello_world():
    print("Hello, World!")
hello_world()
"""

python_splitter = RecursiveCharacterTextSplitter.from_language(
    language=Language.PYTHON, chunk_size=50
)
python_docs = python_splitter.create_documents([python_code])
print(python_docs[0])

js_code = """
function helloWorld() {
  console.log("Hello, World!");
}
helloWorld();
"""

js_splitter = RecursiveCharacterTextSplitter.from_language(
    language=Language.JS, chunk_size=60
)
js_docs = js_splitter.create_documents([js_code])
print(js_docs[0])
```

토큰 제한이 있는 언어 모델에 유용한 토큰 수를 기준으로 텍스트를 분할하려면 TokenTextSplitter를 사용합니다:

```python
from langchain.text_splitter import TokenTextSplitter

text_splitter = TokenTextSplitter(chunk_size=10)
texts = text_splitter.split_text(state_of_the_union)
print(texts[0])
```

마지막으로 LongContextReorder는 긴 컨텍스트로 인한 모델의 성능 저하를 방지하기 위해 문서를 재정렬합니다:

```python
from langchain.document_transformers import LongContextReorder

reordering = LongContextReorder()
reordered_docs = reordering.transform_documents(docs)
print(reordered_docs[0])
```

이러한 도구들은 LangChain에서 문서를 변환하는 다양한 방법을 보여줍니다. 단순한 텍스트 분할부터 복잡한 재정렬 및 언어 특화 분할까지 다룹니다. 보다 심층적이고 구체적인 사용 사례를 위해서는 LangChain 문서의 통합 섹션을 참조하는 것이 좋습니다.

우리의 예제에서는 로더가 이미 청크 문서를 생성했으며, 이 부분은 이미 처리되었습니다.

## 텍스트 임베딩 모델 (Text Embedding Models)

LangChain의 텍스트 임베딩 모델은 OpenAI, Cohere, Hugging Face와 같은 다양한 임베딩 모델 제공업체를 위한 표준화된 인터페이스를 제공합니다. 이러한 모델은 텍스트를 벡터 표현으로 변환하여 벡터 공간에서 텍스트 유사성을 통한 의미론적 검색과 같은 작업을 가능하게 합니다.

텍스트 임베딩 모델을 시작하려면 일반적으로 특정 패키지를 설치하고 API 키를 설정해야 합니다. 우리는 이미 OpenAI에 대해 이를 수행했습니다.

LangChain에서는 `embed_documents` 메서드를 사용하여 여러 텍스트를 임베딩하고 벡터 표현 목록을 제공합니다. 예를 들면:

```python
from langchain.embeddings import OpenAIEmbeddings

# 모델 초기화
embeddings_model = OpenAIEmbeddings()

# 텍스트 목록 임베딩
embeddings = embeddings_model.embed_documents(
    ["Hi there!", "Oh, hello!", "What's your name?", "My friends call me World", "Hello World!"]
)
print("Number of documents embedded:", len(embeddings))
print("Dimension of each embedding:", len(embeddings[0]))
```

단일 텍스트(예: 검색 쿼리)를 임베딩하려면 `embed_query` 메서드를 사용합니다. 이는 쿼리를 문서 임베딩 세트와 비교할 때 유용합니다. 예를 들면:

```python
from langchain.embeddings import OpenAIEmbeddings

# 모델 초기화
embeddings_model = OpenAIEmbeddings()

# 단일 쿼리 임베딩
embedded_query = embeddings_model.embed_query("What was the name mentioned in the conversation?")
print("First five dimensions of the embedded query:", embedded_query[:5])
```

이러한 임베딩을 이해하는 것이 중요합니다. 각 텍스트는 벡터로 변환되며, 그 차원은 사용된 모델에 따라 달라집니다. 예를 들어, OpenAI 모델은 일반적으로 1536차원 벡터를 생성합니다. 이러한 임베딩은 관련 정보를 검색하는 데 사용됩니다.

LangChain의 임베딩 기능은 OpenAI에 국한되지 않고 다양한 제공업체와 작동하도록 설계되었습니다. 설정 및 사용 방법은 제공업체에 따라 약간 다를 수 있지만, 텍스트를 벡터 공간에 임베딩하는 핵심 개념은 동일합니다. 고급 구성 및 다양한 임베딩 모델 제공업체와의 통합을 포함한 자세한 사용법은 LangChain 문서의 통합 섹션이 유용한 리소스입니다.

### 벡터 저장소 (Vector Stores)

LangChain의 벡터 저장소는 텍스트 임베딩의 효율적인 저장과 검색을 지원합니다. LangChain은 50개 이상의 벡터 저장소와 통합되어 사용 편의성을 위한 표준화된 인터페이스를 제공합니다.

**예시: 임베딩 저장 및 검색**

텍스트를 임베딩한 후에는 `Chroma`와 같은 벡터 저장소에 저장하고 유사성 검색을 수행할 수 있습니다:

```python
from langchain.vectorstores import Chroma

db = Chroma.from_texts(embedded_texts)
similar_texts = db.similarity_search("search query")
```

대안으로 FAISS 벡터 저장소를 사용하여 문서의 인덱스를 생성할 수 있습니다:

```python
from langchain.embeddings import OpenAIEmbeddings
from langchain.vectorstores import FAISS

pdfstore = FAISS.from_documents(pdfpages,
            embedding=OpenAIEmbeddings())

airtablestore = FAISS.from_documents(airtabledocs,
            embedding=OpenAIEmbeddings())
```

### 검색기 (Retrievers)

LangChain의 검색기는 비정형 쿼리에 대응하여 문서를 반환하는 인터페이스입니다. 이들은 벡터 저장소보다 더 일반적이며, 저장보다는 검색에 중점을 둡니다. 벡터 저장소를 검색기의 백본으로 사용할 수 있지만, 다른 유형의 검색기도 있습니다.

Chroma 검색기를 설정하려면 먼저 `pip install chromadb`를 사용하여 설치합니다. 그런 다음 일련의 Python 명령을 사용하여 문서를 로드, 분할, 임베딩 및 검색합니다. 다음은 Chroma 검색기를 설정하는 코드 예시입니다:

```python
from langchain.embeddings import OpenAIEmbeddings
from langchain.text_splitter import CharacterTextSplitter
from langchain.vectorstores import Chroma

full_text = open("state_of_the_union.txt", "r").read()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=100)
texts = text_splitter.split_text(full_text)

embeddings = OpenAIEmbeddings()
db = Chroma.from_texts(texts, embeddings)
retriever = db.as_retriever()

retrieved_docs = retriever.invoke("What did the president say about Ketanji Brown Jackson?")
print(retrieved_docs[0].page_content)
```

MultiQueryRetriever는 사용자 입력 쿼리에 대해 여러 쿼리를 생성하고 결과를 결합하여 프롬프트 튜닝을 자동화합니다. 다음은 간단한 사용 예시입니다:

```python
from langchain.chat_models from ChatOpenAI
from langchain.retrievers.multi_query from MultiQueryRetriever

question = "What are the approaches to Task Decomposition?"
llm = ChatOpenAI(temperature=0)
retriever_from_llm = MultiQueryRetriever.from_llm(
    retriever=db.as_retriever(), llm=llm
)

unique_docs = retriever_from_llm.get_relevant_documents(query=question)
print("Number of unique documents:", len(unique_docs))
```

LangChain의 맥락 압축(Contextual Compression)은 쿼리의 맥락을 사용하여 검색된 문서를 압축하여 관련 정보만 반환되도록 합니다. 이는 컨텐츠 감소와 덜 관련된 문서 필터링을 포함합니다. 다음 코드 예제는 Contextual Compression Retriever 사용 방법을 보여줍니다:

```python
from langchain.llms import OpenAI
from langchain.retrievers import ContextualCompressionRetriever
from langchain.retrievers.document_compressors import LLMChainExtractor

llm = OpenAI(temperature=0)
compressor = LLMChainExtractor.from_llm(llm)
compression_retriever = ContextualCompressionRetriever(base_compressor=compressor, base_retriever=retriever)

compressed_docs = compression_retriever.get_relevant_documents("What did the president say about Ketanji Jackson Brown")
print(compressed_docs[0].page_content)
```

EnsembleRetriever는 더 나은 성능을 위해 다양한 검색 알고리즘을 결합합니다. BM25와 FAISS Retriever를 결합하는 예시는 다음 코드에 나와 있습니다:

```python
from langchain.retrievers import BM25Retriever, EnsembleRetriever
from langchain.vectorstores import FAISS

bm25_retriever = BM25Retriever.from_texts(doc_list).set_k(2)
faiss_vectorstore = FAISS.from_texts(doc_list, OpenAIEmbeddings())
faiss_retriever = faiss_vectorstore.as_retriever(search_kwargs={"k": 2})

ensemble_retriever = EnsembleRetriever(
    retrievers=[bm25_retriever, faiss_retriever], weights=[0.5, 0.5]
)

docs = ensemble_retriever.get_relevant_documents("apples")
print(docs[0].page_content)
```

MultiVector Retriever는 문서당 여러 벡터를 사용하여 문서를 쿼리할 수 있게 해주어, 문서 내의 다양한 의미론적 측면을 포착하는데 유용합니다. 여러 벡터를 생성하는 방법에는 더 작은 청크로 분할하거나, 요약하거나, 가상의 질문을 생성하는 방법 등이 있습니다. 문서를 더 작은 청크로 분할하려면 다음과 같은 Python 코드를 사용할 수 있습니다:

```python
from langchain.retrievers.multi_vector import MultiVectorRetriever
from langchain.vectorstores import Chroma
from langchain.embeddings import OpenAIEmbeddings
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.storage import InMemoryStore
from langchain.document_loaders from TextLoader
import uuid

loaders = [TextLoader("file1.txt"), TextLoader("file2.txt")]
docs = [doc for loader in loaders for doc in loader.load()]
text_splitter = RecursiveCharacterTextSplitter(chunk_size=10000)
docs = text_splitter.split_documents(docs)

vectorstore = Chroma(collection_name="full_documents", embedding_function=OpenAIEmbeddings())
store = InMemoryStore()
id_key = "doc_id"
retriever = MultiVectorRetriever(vectorstore=vectorstore, docstore=store, id_key=id_key)

doc_ids = [str(uuid.uuid4()) for _ in docs]
child_text_splitter = RecursiveCharacterTextSplitter(chunk_size=400)
sub_docs = [sub_doc for doc in docs for sub_doc in child_text_splitter.split_documents([doc])]
for sub_doc in sub_docs:
    sub_doc.metadata[id_key] = doc_ids[sub_docs.index(sub_doc)]

retriever.vectorstore.add_documents(sub_docs)
retriever.docstore.mset(list(zip(doc_ids, docs)))
```

더 집중된 컨텐츠 표현으로 인한 더 나은 검색을 위해 요약을 생성하는 것도 또 다른 방법입니다. 다음은 요약 생성의 예시입니다:

```python
from langchain.chat_models import ChatOpenAI
from langchain.prompts import ChatPromptTemplate
from langchain.schema.output_parser import StrOutputParser
from langchain.schema.document import Document

chain = (lambda x: x.page_content) | ChatPromptTemplate.from_template("Summarize the following document:\n\n{doc}") | ChatOpenAI(max_retries=0) | StrOutputParser()
summaries = chain.batch(docs, {"max_concurrency": 5})

summary_docs = [Document(page_content=s, metadata={id_key: doc_ids[i]}) for i, s in enumerate(summaries)]
retriever.vectorstore.add_documents(summary_docs)
retriever.docstore.mset(list(zip(doc_ids, docs)))
```

LLM을 사용하여 각 문서와 관련된 가상의 질문을 생성하는 것도 또 다른 접근 방식입니다. 다음 코드로 이를 수행할 수 있습니다:

```python
functions = [{"name": "hypothetical_questions", "parameters": {"questions": {"type": "array", "items": {"type": "string"}}}}]
from langchain.output_parsers.openai_functions import JsonKeyOutputFunctionsParser

chain = (lambda x: x.page_content) | ChatPromptTemplate.from_template("Generate 3 hypothetical questions:\n\n{doc}") | ChatOpenAI(max_retries=0).bind(functions=functions, function_call={"name": "hypothetical_questions"}) | JsonKeyOutputFunctionsParser(key_name="questions")
hypothetical_questions = chain.batch(docs, {"max_concurrency": 5})

question_docs = [Document(page_content=q, metadata={id_key: doc_ids[i]}) for i, questions in enumerate(hypothetical_questions) for q in questions]
retriever.vectorstore.add_documents(question_docs)
retriever.docstore.mset(list(zip(doc_ids, docs)))
```

Parent Document Retriever는 작은 청크를 저장하고 더 큰 상위 문서를 검색하여 임베딩 정확성과 컨텍스트 유지 사이의 균형을 맞추는 또 다른 검색기입니다. 구현은 다음과 같습니다:

```python
from langchain.retrievers import ParentDocumentRetriever

loaders = [TextLoader("file1.txt"), TextLoader("file2.txt")]
docs = [doc for loader in loaders for doc in loader.load()]

child_splitter = RecursiveCharacterTextSplitter(chunk_size=400)
vectorstore = Chroma(collection_name="full_documents", embedding_function=OpenAIEmbeddings())
store = InMemoryStore()
retriever = ParentDocumentRetriever(vectorstore=vectorstore, docstore=store, child_splitter=child_splitter)

retriever.add_documents(docs, ids=None)

retrieved_docs = retriever.get_relevant_documents("query")
```

자체 쿼리 검색기(Self-querying Retriever)는 자연어 입력에서 구조화된 쿼리를 구성하고 이를 기본 VectorStore에 적용합니다. 구현은 다음 코드에 나와 있습니다:

```python
from langchain.chat_models from ChatOpenAI
from langchain.chains.query_constructor.base from AttributeInfo
from langchain.retrievers.self_query.base from SelfQueryRetriever

metadata_field_info = [AttributeInfo(name="genre", description="...", type="string"), ...]
document_content_description = "Brief summary of a movie"
llm = ChatOpenAI(temperature=0)

retriever = SelfQueryRetriever.from_llm(llm, vectorstore, document_content_description, metadata_field_info)

retrieved_docs = retriever.invoke("query")
```

WebResearchRetriever는 주어진 쿼리를 기반으로 웹 검색을 수행합니다:

```python
from langchain.retrievers.web_research import WebResearchRetriever

# Initialize components
llm = ChatOpenAI(temperature=0)
search = GoogleSearchAPIWrapper()
vectorstore = Chroma(embedding_function=OpenAIEmbeddings())

# Instantiate WebResearchRetriever
web_research_retriever = WebResearchRetriever.from_llm(vectorstore=vectorstore, llm=llm, search=search)

# Retrieve documents
docs = web_research_retriever.get_relevant_documents("query")
```

이제 검색기를 쿼리할 수 있습니다. 쿼리의 출력은 쿼리와 관련된 문서 객체가 될 것입니다.

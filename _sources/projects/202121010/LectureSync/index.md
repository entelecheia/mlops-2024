# LectureSync 프로젝트 보고서

## 개요

LectureSync는 비디오나 영상을 업로드하면 STT (Speech-to-Text) 기술을 통해 텍스트를 추출하고, 추가로 PDF 및 텍스트 파일을 업로드하면 영상에 대한 스크립트와 추가 파일들을 RAG (Retrieval-Augmented Generation) 시스템을 통해 통합하여 RAG 챗봇을 구현하는 프로젝트입니다. 이 프로젝트는 LangChain, RAG, Map-Reduce 기술을 활용하여 구현되었습니다.

## 사용된 기술과 설명

### LangChain

- LangChain은 텍스트 데이터 처리를 위한 라이브러리로, 자연어 처리 작업을 더 쉽게 할 수 있도록 다양한 유틸리티와 인터페이스를 제공합니다.LectureSync에서는 텍스트 데이터를 처리하고 관리하는 데 사용됩니다.

### RAG (Retrieval-Augmented Generation)

- RAG는 검색과 생성 기능을 결합하여 질문에 대한 답변을 생성하는 시스템입니다. LectureSync에서는 업로드된 스크립트와 추가 파일들을 활용하여 사용자의 질문에 대해 더 정확하고 풍부한 답변을 제공합니다.

### Map-Reduce

- Map-Reduce는 대용량 데이터 처리를 위한 프로그래밍 모델입니다. LectureSync에서는 대량의 텍스트 데이터를 병렬로 처리하여 효율적으로 분석하는 데 사용됩니다.

## 주요 코드 설명

### 음성 파일 STT(Speech-To-Text) - stt.py

speech-to-text를 위한 코드입니다. stt 기술은 gcs api를 통해 진행했습니다.

- `convert_audio_to_mono`함수는 input 파일이 wav 형식을 지원하기 때문에 mp4,mp3 형식의 파일을 wav파일로 변환해주는 함수입니다.
- `transcribe_audio` 함수를 통해 wave로 변환된 파일에 대한 stt를 진행합니다. 여기서 output 형식은 `trnascript: sentences`와 문장의 시작과 끝나는 시간을 리턴합니다.

```python
from google.cloud import speech_v1p1beta1 as speech
from google.cloud import storage
import io
import os
from pydub import AudioSegment
from dotenv import load_dotenv

load_dotenv()

# 환경 변수로 서비스 계정 키 파일 설정
google_credentials_path = os.getenv("GOOGLE_APPLICATION_CREDENTIALS")

def upload_to_gcs(bucket_name, source_file_name, destination_blob_name):
    """Uploads a file to Google Cloud Storage."""
    storage_client = storage.Client()
    bucket = storage_client.bucket(bucket_name)
    blob = bucket.blob(destination_blob_name)
    blob.upload_from_filename(source_file_name)
    return f'gs://{bucket_name}/{destination_blob_name}'

def convert_audio_to_mono(file_path):
    """Convert audio file to mono WAV format."""
    audio = AudioSegment.from_file(file_path)
    audio = audio.set_channels(1)  # 모노로 변환
    wav_file = file_path.rsplit('.', 1)[0] + '_mono.wav'
    audio.export(wav_file, format='wav')
    return wav_file

def transcribe_audio(speech_file, output_file, output_file_sentences):
    """Transcribe the given audio file using Google Cloud Speech-to-Text API."""
    if speech_file.endswith('.mp4') or speech_file.endswith('.mp3'):
        speech_file = convert_audio_to_mono(speech_file)
    # Google Cloud Storage 버킷 이름
    bucket_name = 'lecturesync-stt'
    # 업로드할 GCS 경로
    gcs_path = 'audio-files/test_video.wav'

    # GCS에 파일 업로드
    gcs_uri = upload_to_gcs(bucket_name, speech_file, gcs_path)
    # Google Cloud 클라이언트 생성
    client = speech.SpeechClient()

    audio = speech.RecognitionAudio(uri=gcs_uri)
    config = speech.RecognitionConfig(
        encoding=speech.RecognitionConfig.AudioEncoding.LINEAR16,
        sample_rate_hertz=44100,
        language_code="ko-KR",
        enable_word_time_offsets=True
    )

    # 음성 인식 요청
    operation = client.long_running_recognize(config=config, audio=audio)

    print("Waiting for operation to complete...")
    response = operation.result(timeout=700)

    # 결과를 파일에 저장
    with open(output_file, 'w') as f:
        for result in response.results:
            alternative = result.alternatives[0]
            f.write("Transcript: {}\n".format(alternative.transcript))
            for word_info in alternative.words:
                word = word_info.word
                start_time = word_info.start_time
                end_time = word_info.end_time
                f.write(f"Word: {word}, start_time: {start_time.total_seconds()}, end_time: {end_time.total_seconds()}\n")

    # 문장 단위로 시작과 끝 시간 저장
    with open(output_file_sentences, 'w') as f:
        for result in response.results:
            alternative = result.alternatives[0]
            start_time = alternative.words[0].start_time.total_seconds()
            end_time = alternative.words[-1].end_time.total_seconds()
            f.write(f"Transcript: {alternative.transcript}\n")
            f.write(f"Start time: {start_time}, End time: {end_time}\n")

# 오디오 파일 경로
speech_file = "/home/a202121010/workspace/projects/LectureSync/data/doc_data/video_data/test_video.mp4"
output_file = '/home/a202121010/workspace/projects/LectureSync/data/doc_data/summary_txt_data/output2.txt'
output_file_sentences = '/home/a202121010/workspace/projects/LectureSync/data/doc_data/stt_txt_data/sentences.txt'
# 음성 인식 수행
transcribe_audio(speech_file, output_file, output_file_sentences)
```

### 문서 요약 - summarize.py

이 모듈은 PDF 및 텍스트 파일을 읽고 요약하여 RAG 시스템에 입력 데이터로 사용합니다. 주요 클래스와 함수들의 역할을 설명합니다.

#### DocumentSummarizer 클래스

이 클래스는 문서를 요약하는데 필요한 모든 기능을 포함하고 있습니다. 주요 속성과 메서드는 다음과 같습니다.

#### 속성

- `pdf_path`: PDF 파일 경로 리스트
- `txt_path`: 텍스트 파일 경로 리스트
- `model_url`: 모델 URL (기본값: http://172.16.229.33:11436)
- `model_name`: 모델 이름 (기본값: EEVE-Korean-Instruct-10.8B)
- `temperature`: 모델의 온도 설정 (기본값: 0.3)
- `chunk_size`: 문서를 나눌 때 사용되는 청크 크기 (기본값: 2000)
- `chunk_overlap`: 청크 간의 중첩 크기 (기본값: 200)

#### 메서드

- `__init__`: 초기화 메서드로, 속성을 설정하고 환경 변수를 설정합니다.
- `load_documents`: PDF 및 텍스트 파일을 읽어 문서 리스트를 반환합니다.
- `split_documents`: 문서를 청크 단위로 분할합니다.
- `create_llm`: 언어 모델(LLM)을 생성합니다.
- `create_map_reduce_chain`: Map-Reduce 체인을 생성합니다.
- `create_stuff_chain`: Stuff 체인을 생성합니다.
- `summarize`: 문서를 요약합니다.

```python
import os
from langchain.chains.combine_documents.stuff import StuffDocumentsChain
from langchain.chains.llm import LLMChain
from langchain_core.prompts import PromptTemplate
from langchain_community.document_loaders import PyPDFLoader, TextLoader
from langchain.text_splitter import CharacterTextSplitter
from langchain_core.output_parsers import StrOutputParser
from langchain.chains.combine_documents.map_reduce import MapReduceDocumentsChain, ReduceDocumentsChain
from langchain_community.chat_models import ChatOllama
from langchain_community.chat_models import ChatOpenAI

os.environ["OPNEAI_API_KEY"] = os.environ.get("OPENAI_API_KEY")

class DocumentSummarizer:
    def __init__(self,
                 pdf_path=None,
                 txt_path=None,
                 model_url='http://172.16.229.33:11436',
                 model_name='EEVE-Korean-Instruct-10.8B',
                 temperature=0.3,
                 chunk_size=2000,
                 chunk_overlap=200):
        self.pdf_path = pdf_path
        self.txt_path = txt_path
        self.model_url = model_url
        self.model_name = model_name
        self.temperature = temperature
        self.chunk_size = chunk_size
        self.chunk_overlap = chunk_overlap

        # 설정된 환경 변수
        os.environ["TRANSFORMERS_CACHE"] = "./data/llm_model/"
        os.environ["HF_HOME"] = "./data/llm_model/"

    def load_documents(self):
        docs = []
        if self.pdf_path:
            for file_path in self.pdf_path:
                pdf_loader = PyPDFLoader(file_path)
                docs.extend(pdf_loader.load())

        if self.txt_path:
            for file_path in self.txt_path:
                txt_loader = TextLoader(file_path)
                docs.extend(txt_loader.load())
        return docs

    def split_documents(self, docs):
        text_splitter = CharacterTextSplitter.from_tiktoken_encoder(
            separator="\n\n",
            chunk_size=self.chunk_size,
            chunk_overlap=self.chunk_overlap,
        )
        return text_splitter.split_documents(docs)

    def create_llm(self):
        # return ChatOllama(base_url=self.model_url,
        #               model=self.model_name,
        #               temperature=self.temperature)
        return ChatOpenAI(temperature=0.1,
                          model_name='gpt-4o')

    def create_map_reduce_chain(self, llm):
        # Define map prompt
        map_prompt_template = """다음은 문서(CONTEXT) 중 일부 내용입니다. 이 문서 목록을 기반으로 주요 내용을 한국어(Korean)로 요약해:
        너는 다음과 같은 규칙을 따라야돼.
        - 특수기호 제외
        - 문장에 대한 연관성이 있음
        - 핵심 내용에 대한 언급
        - 글자만 생성해
        - 최대한 많은 정보를 포함해
        CONTEXT:
        {map}

        답변:"""
        map_prompt = PromptTemplate.from_template(map_prompt_template)

        # Define reduce prompt
        reduce_template = """다음은 요약의 집합입니다. 이것(CONTEXT)들을 바탕으로 통합된 요약을 자연스러운 문장으로 한국말(Korean)로 만들어.
        너는 다음과 같은 규칙을 따라야돼.
        - 특수기호 제외
        - 문장에 대한 연관성이 있음
        - 핵심 내용에 대한 언급
        - 글자만 생성해
        - 최대한 많은 정보를 포함해
        CONTEXT:
        {pages}

        답변:"""
        reduce_prompt = PromptTemplate.from_template(reduce_template)

        # Define chains
        map_chain = LLMChain(llm=llm, prompt=map_prompt)
        reduce_chain = LLMChain(llm=llm, prompt=reduce_prompt)

        document_prompt = PromptTemplate(
            input_variables=["page_content"],
            template="{page_content}"
        )

        combine_documents_chain = StuffDocumentsChain(
            llm_chain=map_chain,
            document_prompt=document_prompt,
            document_variable_name="map"
        )

        reduce_documents_chain = ReduceDocumentsChain(
            combine_documents_chain=combine_documents_chain,
            collapse_documents_chain=combine_documents_chain,
            token_max=8000,
        )

        return MapReduceDocumentsChain(
            llm_chain=reduce_chain,
            reduce_documents_chain=reduce_documents_chain,
            document_variable_name="pages",
        )
    def create_stuff_chain(self, llm):
        stuff_template = """다음 문서(CONTEXT)를 기반으로 주요 내용을 한국어(Korean)로 요약해:
        너는 다음과 같은 규칙을 따라야돼.
        - 특수기호 제외
        - 문장에 대한 연관성이 있음
        - 핵심 내용에 대한 언급
        - 글자만 생성
        - 최대한 많은 정보를 포함
        CONTEXT:
        {pages}

        답변:"""
        stuff_prompt = PromptTemplate.from_template(stuff_template)
        stuff_chain = stuff_prompt | llm | StrOutputParser()

        return stuff_chain

    def summarize(self, type=None):
        docs = self.load_documents()
        splits = self.split_documents(docs)
        print(f'총 분할된 도큐먼트 수: {len(splits)}')

        llm = self.create_llm()
        if type == 'stuff':
            stuff_chain = self.create_stuff_chain(llm)
            result = stuff_chain.invoke({"pages":splits})
            print(result)
            return result

        if type == 'map_reduce':
            map_reduce_chain = self.create_map_reduce_chain(llm)
            result = map_reduce_chain.invoke({"input_documents":splits}, return_only_outputs=True)
            print(result)
            return result['output_text']
```

### RAG 시스템 구현 - rag.py

이 모듈은 RAG (Retrieval-Augmented Generation) 시스템을 구현합니다. 주된 역할은 PDF와 텍스트 파일에서 데이터를 로드하고, 이를 기반으로 챗봇을 통해 사용자의 질문에 답변하는 것입니다. 주요 클래스와 함수들의 역할을 설명합니다.

#### Chatbot 클래스

이 클래스는 문서를 로드하고, 문서의 내용을 기반으로 질문에 답변하는 챗봇을 구현합니다. 주요 속성과 메서드는 다음과 같습니다.

#### 속성

- `pdf_path`: PDF 파일 경로 리스트
- `txt_path`: 텍스트 파일 경로 리스트
- `stt_txt_path`: STT (Speech-to-Text) 결과 텍스트 파일 경로 리스트
- `chunk_size`: 문서를 나눌 때 사용되는 청크 크기 (기본값: 2000)
- `chunk_overlap`: 청크 간의 중첩 크기 (기본값: 200)
- `docs_pdf`: PDF 파일에서 로드된 문서 리스트
- `docs_stt`: STT 결과에서 로드된 문서 리스트
- `docs`: 텍스트 파일에서 로드된 문서 리스트
- `vectorstore`: 벡터 스토어 객체
- `retriever`: 문서 검색 객체
- `chat_history`: 챗봇의 대화 히스토리 리스트
- `prompt`: 챗봇 프롬프트 템플릿
- `llm`: 언어 모델 객체
- `sentences_data`: STT 결과에서 추출된 문장 데이터 리스트
- `corpus`: STT 결과 문장 텍스트 리스트
- `corpus_embeddings`: STT 결과 문장 임베딩 리스트

#### 메서드

- `__init__`: 초기화 메서드로, 속성을 설정하고 문서를 로드하여 벡터 스토어를 생성합니다.
- `parse_transcription_file`: STT 결과 파일을 파싱하여 문장 데이터를 추출합니다.
- `find_sentence_time`: 질문과 일치하는 문장을 찾아 해당 문장의 시작 및 종료 시간을 반환합니다.
- `format_docs`: 문서 리스트를 포맷팅하여 문자열로 반환합니다.
- `add_to_history`: 질문과 응답을 대화 히스토리에 추가합니다.
- `create_chain`: RAG 체인을 생성합니다.
- `chat`: 사용자의 질문에 대한 답변을 생성합니다.
- `search_sentence`: 질문과 일치하는 문장을 검색하여 해당 문장의 시간을 반환합니다.

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain.text_splitter import RecursiveCharacterTextSplitter, CharacterTextSplitter
from langchain_community.document_loaders import PyPDFLoader, TextLoader
from langchain_community.vectorstores import FAISS
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough
from langchain_community.embeddings import HuggingFaceBgeEmbeddings
from langchain_community.chat_models import ChatOllama
from langchain_community.chat_models import ChatOpenAI
from sentence_transformers import SentenceTransformer, util
import re
import torch


class Chatbot:
    def __init__(self, pdf_path=None, txt_path=None, stt_txt_path=None, chunk_size=2000, chunk_overlap=200):
        self.pdf_path = pdf_path
        self.txt_path = txt_path
        self.stt_txt_path = stt_txt_path
        self.chunk_size = chunk_size
        self.chunk_overlap = chunk_overlap
        self.docs_pdf = []
        self.docs_stt = []
        self.docs = []

        if pdf_path:
            for file_path in self.pdf_path:
                pdf_loader = PyPDFLoader(file_path)
                self.docs_pdf += pdf_loader.load()

        if txt_path:
            for file_path in self.txt_path:
                txt_loader = TextLoader(file_path)
                self.docs += txt_loader.load()

        if stt_txt_path:
            for file_path in self.stt_txt_path:
                txt_loader = TextLoader(file_path)
                self.docs_stt += txt_loader.load()

        self.vectorstore = None
        self.retriever = None

        all_docs = []
        if self.docs_pdf:
            text_splitter = CharacterTextSplitter.from_tiktoken_encoder(
                separator="\n\n",
                chunk_size=self.chunk_size,
                chunk_overlap=self.chunk_overlap,
            )
            self.splits_pdf = text_splitter.split_documents(self.docs_pdf)
            all_docs.extend(self.splits_pdf)

        if self.docs:
            text_splitter = CharacterTextSplitter.from_tiktoken_encoder(
                separator="\n\n",
                chunk_size=self.chunk_size,
                chunk_overlap=self.chunk_overlap,
            )
            self.splits_doc = text_splitter.split_documents(self.docs)
            all_docs.extend(self.splits_doc)
        if self.docs_stt:
            text_splitter = CharacterTextSplitter.from_tiktoken_encoder(
                separator="\n\n",
                chunk_size=self.chunk_size,
                chunk_overlap=self.chunk_overlap,
            )
            self.splits_stt = text_splitter.split_documents(self.docs_stt)
            all_docs.extend(self.splits_stt)

        if all_docs:
            self.vectorstore = FAISS.from_documents(documents=all_docs, embedding=HuggingFaceBgeEmbeddings())
            self.retriever = self.vectorstore.as_retriever(search_kwargs={"k": 3})

        self.chat_history = []

        self.prompt = ChatPromptTemplate.from_messages([
            ("system", """친절한 챗봇으로서 상대방의 요청에 최대한 자세하고 친절하게 답하자.
             대답은 다음 '{context}' 기반으로 대답해. 모든 대답은 한국어(Korean)으로 대답해줘.
             사용자가 질문하고 너가 답한 내용은 '{chat_history}' 이러한 내용이야.
             만약 답변할 정보가 없거나 확실하지 않으면 '정보를 찾을 수 없습니다' 또는 '확실하지 않습니다'라고 대답해."""),
            ("human", "{question}")
        ])

        # self.llm = ChatOllama(base_url='http://172.16.229.33:11436',
        #                       model='EEVE-Korean-Instruct-10.8B',
        #                       temperature=0.4)
        self.llm = ChatOpenAI(temperature=0.1,
                              model_name='gpt-4o')

        # STT 관련 변수 초기화
        self.sentences_data = self.parse_transcription_file(self.stt_txt_path[0] if self.stt_txt_path else None)
        if self.sentences_data:
            self.corpus = [sentence["text"] for sentence in self.sentences_data]
            self.corpus_embeddings = HuggingFaceBgeEmbeddings().embed_documents(self.corpus)
        else:
            self.corpus = []
            self.corpus_embeddings = None

    @staticmethod
    def parse_transcription_file(file_path):
        if not file_path:
            return None
        with open(file_path, 'r', encoding='utf-8') as file:
            content = file.read()

        sentence_pattern = re.compile(r"Transcript: (.*?)\nStart time: ([\d.]+), End time: ([\d.]+)\n", re.DOTALL)
        sentences_info = sentence_pattern.findall(content)

        sentences_data = []
        for sentence, start_time, end_time in sentences_info:
            sentences_data.append({
                "text": sentence.strip(),
                "start_time": float(start_time),
                "end_time": float(end_time)
            })

        return sentences_data

    def find_sentence_time(self, query):
        if not self.corpus_embeddings:
            return "문장 검색이 불가능합니다."
        query_embedding = HuggingFaceBgeEmbeddings().embed_documents([query])
        query_embedding = torch.tensor(query_embedding)
        hits = util.semantic_search(query_embedding, torch.tensor(self.corpus_embeddings), top_k=3)
        hits = hits[0]

        response = []
        for hit in hits:
            sentence_info = self.sentences_data[hit['corpus_id']]
            response.append(f"Sentence: {sentence_info['text']}, Start Time: {sentence_info['start_time']}, End Time: {sentence_info['end_time']}")
        return "\n".join(response)

    @staticmethod
    def format_docs(docs):
        return "\n\n".join(doc.page_content for doc in docs)

    def add_to_history(self, question, response):
        self.chat_history.append(("human", question))
        self.chat_history.append(("assistant", response))

    def create_chain(self):
        if self.retriever:
            rag_chain = (
                {"context": self.retriever | self.format_docs, "question": RunnablePassthrough(), "chat_history": RunnablePassthrough()}
                | self.prompt
                | self.llm
                | StrOutputParser()
            )
        else:
            rag_chain = (
                {"context": RunnablePassthrough(), "question": RunnablePassthrough(), "chat_history": RunnablePassthrough()}
                | self.prompt
                | self.llm
                | StrOutputParser()
            )
        return rag_chain

    def chat(self, question):
        # chat_history = ''
        # if self.chat_history is not None:
        chat_history = "\n".join([f"{role}: {text}" for role, text in self.chat_history])
        inputs = {"question": question, "chat_history": chat_history}
        chain = self.create_chain()
        response = chain.invoke(question)
        self.add_to_history(question, response)
        return response

    def search_sentence(self, question):

        sentence_time_info = self.find_sentence_time(question)

        return sentence_time_info
```

### Streamlit 프론트엔드 - streamlit_frontend.py

이 모듈은 Streamlit을 사용하여 LectureSync 프로젝트의 웹 인터페이스를 구현합니다. 주요 기능은 사용자가 파일을 업로드하고, 이를 처리하여 챗봇을 통해 질문에 답변하거나 문장을 검색할 수 있도록 하는 것입니다. 주요 함수와 그 역할을 설명합니다.

#### 주요 구성 요소

- Chatbot 인스턴스 생성 및 설정
- 파일 업로드 및 처리
- LLM 응답 생성
- 문장 검색

```python
from rag_copy import Chatbot
from summarize_copy import DocumentSummarizer
import streamlit as st
from pydub import AudioSegment
import os
from io import BytesIO
import chardet
import datetime

# 챗봇 인스턴스 생성

UPLOAD_DIR = "data/doc_data"
UPLOAD_TXT_DIR = "data/doc_data/summary_txt_data/"
UPLOAD_STT_DIR = "data/doc_data/stt_txt_data/"
# Streamlit 페이지 설정
st.set_page_config(page_title="LectureSync")
with st.sidebar:
    st.title('LectureSync ChatBot')

# Function for generating LLM response
def generate_response(input):
    if 'rag_bot' in st.session_state:
        result = st.session_state.rag_bot.chat(input)
        print(result)
    else:
        result = "Bot is not defined. Please upload files first."
    return result

def search_sentence(query):
    if 'rag_bot' in st.session_state:
        result = st.session_state.rag_bot.search_sentence(query)
        print(result)
    else:
        result = "Bot is not defined. Please upload files first."
    return result

def summary_doc(files):
    if 'summarizer' in st.session_state:
        result = st.session_state.summarizer.summarize(type='map_reduce')
    else:
        result = "Summarizer is not defined. Please upload files first."
    return result

def save_summary(txt):
    current_time = datetime.datetime.now().strftime("%Y%m%d_%H%M%S")
    file_name = f"text_{current_time}.txt"
    file_txt_path = UPLOAD_TXT_DIR + file_name
    with open(file_txt_path, 'w', encoding='utf-8') as file:
        file.write(txt)
    return file_txt_path

def save_stt():
    current_time = datetime.datetime.now().strftime("%Y%m%d_%H%M%S")
    file_name = f"text_{current_time}.txt"
    file_txt_path = UPLOAD_STT_DIR + file_name
    return file_txt_path

# Function to handle file upload and conversion
def handle_audio_video_upload(uploaded_file):
    # Save uploaded file to a specified directory
    file_extension = uploaded_file.name.split('.')[-1]
    temp_file_path = os.path.join(UPLOAD_DIR, uploaded_file.name)

    with open(temp_file_path, 'wb') as temp_file:
        temp_file.write(uploaded_file.getbuffer())

    # Convert to WAV if necessary
    audio = AudioSegment.from_file(temp_file_path)
    wav_file_path = temp_file_path + ".wav"
    audio.export(wav_file_path, format="wav")

    return wav_file_path

def handle_pdf_upload(uploaded_file):
    # Save uploaded PDF file to a specified directory
    temp_file_path = os.path.join(UPLOAD_DIR, uploaded_file.name)

    with open(temp_file_path, 'wb') as temp_file:
        temp_file.write(uploaded_file.getbuffer())

    return temp_file_path

# Store LLM generated responses
if "messages" not in st.session_state.keys():
    st.session_state.messages = [{"role": "assistant", "content": "안녕하세요 LectureSync ChatBot 입니다. 강의 음성 파일 또는 강의 자료를 업로드 해주세요."}]

# Display chat messages
for message in st.session_state.messages:
    with st.chat_message(message["role"]):
        st.write(message["content"])

# File uploader for video, audio, or PDF
uploaded_files = st.file_uploader("Upload a video, audio, or PDF file", type=[ "pdf"], accept_multiple_files=True)

file_type_list = ["mp4", "mp3", "wav"]
audio_files = []
pdf_files = []
txt_files = []
if uploaded_files:
    # Handle file upload and conversion
    for uploaded_file in uploaded_files:
        file_type = uploaded_file.type.split('/')[-1]

        if file_type == 'audio' or file_type == 'video':
            audio_data = handle_audio_video_upload(uploaded_file)
            audio_files.append(audio_data)
            for audio_file in audio_files:
                stt_file = transcribe_audio(audio_file)
                stt_path = save_stt(stt_file)
                stt_files.append(stt_path)
            st.session_state.messages.append({"role": "user", "content": f"Uploaded audio/video file: {uploaded_file.name}"})
            with st.chat_message("user"):
                st.write(f"Uploaded audio/video file: {uploaded_file.name}")

        elif uploaded_file.type == 'application/pdf':
            pdf_data = handle_pdf_upload(uploaded_file)
            pdf_files.append(pdf_data)
            st.session_state.messages.append({"role": "user", "content": f"Uploaded PDF file: {uploaded_file.name}"})
            with st.chat_message("user"):
                st.write(f"Uploaded PDF file: {uploaded_file.name}")

    # Add a button to process the uploaded files
    if st.button("Process Files"):
        if pdf_files:
            st.session_state.messages.append({"role": "assistant", "content": "Processing uploaded files."})
            with st.chat_message("assistant"):
                with st.spinner("Processing your files..."):
                    model_url = 'http://172.16.229.33:11436'
                    model_name = 'EEVE-Korean-Instruct-10.8B'
                    st.session_state.summarizer = DocumentSummarizer(pdf_path=pdf_files, model_url=model_url, model_name=model_name)
                    summary = summary_doc(pdf_files)
                    response = f"요약이 끝났어요! 요약한 내용은 다음과 같아요: {summary}"
                    st.write(response)
                    txt_file = save_summary(response)
                    txt_files.append(txt_file)
                    stt_txt_path = ['data/doc_data/stt_txt_data/sentences.txt']
            st.session_state.rag_bot = Chatbot(pdf_path=pdf_files, txt_path=txt_files, stt_txt_path = stt_txt_path)

        # Clear the uploaded files
        st.session_state.uploaded_files = []
else:
    st.session_state.rag_bot = Chatbot()

# User-provided prompt
if input := st.chat_input():
    st.session_state.messages.append({"role": "user", "content": input})
    with st.chat_message("user"):
        st.write(input)

# Generate a new response if last message is not from assistant
if st.session_state.messages[-1]["role"] != "assistant" and 'input' in locals():
    with st.chat_message("assistant"):
        with st.spinner("Getting your answer from mystery stuff.."):
            response = generate_response(input)
            st.write(response)
    message = {"role": "assistant", "content": response}
    st.session_state.messages.append(message)

# 문장 검색을 트리거하는 버튼
query = st.text_input("검색할 문장을 입력하세요:")
if st.button("문장 검색"):
    if query:
        with st.spinner("문장을 검색하는 중..."):
            search_results = search_sentence(query)
            st.write(search_results)
```

## 결론

LectureSync 프로젝트는 강의 비디오와 PDF, 텍스트 파일을 통해 강의 자료를 요약하고 이를 기반으로 사용자의 질문에 답변하는 챗봇을 구현하는 강력한 학습 도구입니다. 이 프로젝트는 LangChain, RAG, Map-Reduce 기술을 활용하여 대용량의 텍스트 데이터를 효율적으로 처리하고, 사용자가 필요한 정보를 쉽게 얻을 수 있도록 도와줍니다.

### 주요 기능 및 성과

- STT 모듈을 통해 강의 비디오의 음성을 텍스트로 변환하여 텍스트 데이터로 활용합니다.
- RAG 시스템을 통해 다양한 자료를 통합하고, 이를 기반으로 사용자의 질문에 대한 정확하고 상세한 답변을 제공합니다.
- PDF 및 텍스트 파일 요약 모듈을 통해 대량의 자료를 요약하여 핵심 정보를 추출합니다.
- Streamlit 프론트엔드를 통해 사용자 친화적인 웹 인터페이스를 제공하여 손쉽게 자료를 업로드하고, 질문을 입력하며, 필요한 정보를 검색할 수 있습니다.

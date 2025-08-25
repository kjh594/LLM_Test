# LLM 기반 유전변이 해석 시스템

## 1. 개요

### 1.1 목적
본 프로젝트는 유전변이 해석을 위한 거대언어모델(LLM)의 개발, 구축, 운영에 필요한 표준 절차를 마련하고,
이를 통해 일관성과 안정성을 갖춘 AI 시스템을 구현하는 것을 목표로 합니다.

### 1.2 용어 정의
- **LLM (Large Language Model)**: 거대언어모델, 대규모 텍스트 데이터로 학습된 언어 처리 AI 모델  
- **Fine-tuning**: 사전 훈련된 모델을 특정 도메인에 맞게 추가 학습시키는 과정  
- **LoRA (Low-Rank Adaptation)**: 기존 모델의 가중치를 고정하고 저차원 행렬만 학습하는 효율적인 파인튜닝 기법  
- **RAG (Retrieval-Augmented Generation)**: 검색 증강 생성, 관련 문서를 검색하여 답변 생성에 활용하는 기법  
- **NER (Named Entity Recognition)**: 개체명 인식, 텍스트에서 특정 개체를 식별하는 기술  
- **ChromaDB**: Python 기반 벡터 데이터베이스, 임베딩 벡터 저장 및 유사도 검색 지원  
- **HGVS**: Human Genome Variation Society, 국제 표준 유전변이 명명법  
- **Prompt Engineering**: 모델의 응답을 최적화하기 위한 입력 프롬프트 설계 기법  
- **Variant**: 유전변이, 기존 게놈 서열과 다른 DNA 서열  
- **Annotation**: 주석, 유전변이의 의학적 의미를 해석한 설명  


## 1.3 전체 구성도

![전체 구성도](https://raw.githubusercontent.com/kjh594/LLM_Test/main/image/diagram.png)

---

## 2. 데이터 준비 절차

### 2.1 모델 실행 환경

#### 2.1.1 Ollama 설치 및 LLaMA3 모델 준비
- [Ollama 공식 페이지](https://ollama.com/)에서 설치  
- 설치 후 모델 다운로드:

```bash
ollama pull llama3
```

- 로컬 API 서버 실행:

```bash
ollama serve
```

- 동작 확인(예시):

```bash
curl http://localhost:11434/api/generate -d '{
  "model":"llama3",
  "prompt":"hello"
}'
```

- 테스트 호출

```bash
ollama run llama3
```

#### 2.1.2 환경변수 설정

파이프라인 실행 전 다음 API 키를 환경변수로 설정해야 합니다:

- OpenAI API 키 설정 (필수)

```bash
export OPENAI_API_KEY="your-openai-api-key-here"
```

- ~/.bashrc 또는 ~/.zshrc에 추가

```bash
echo 'export OPENAI_API_KEY="your-openai-api-key-here"' >> ~/.bashrc
source ~/.bashrc
```
- API 키 확인

```bash
echo $OPENAI_API_KEY
```

---

## 3. 단계별 프로세스


### 3-1. PubMed 논문 수집

이 단계에서는 **NCBI Entrez API**를 활용하여 체계적인 논문 수집을 수행합니다.  
총 **584개의 의학 분야 저널**을 대상으로 하며, 분야별 예시는 다음과 같습니다:

- Oncology: 99개
- Genetics: (수치 기입 필요)
- Immunology: (수치 기입 필요)
- Gastroenterology: 41개

또한, **53개의 유전변이 관련 키워드**를 사용하여 지정한 연도에 출판된 논문을 검색합니다.

---

- 실행 방법

#### 1a) PubMed ID 검색 (필수 매개변수만)
```bash
python 01_pubmed_fetching/scripts/fetch_pubmed_ids.py   --year 2023   --keywords ./01_pubmed_fetching/configs/ref/keyword_list.csv   --journal_list ./01_pubmed_fetching/configs/ref/Oncology_journal_list.csv   --email your.email@example.com   --test   # 선택사항: 테스트 모드
```

#### 1b) Medline 다운로드 (필수 매개변수만)
```bash
python 01_pubmed_fetching/scripts/download_medline.py   --year 2023   --journal_category oncology   --email your.email@example.com
```

---

- 결과 구조 (자동 생성)

```
output/2023/
├── ids/
│   └── oncology_2023_study-id_n_20.txt   # 자동 생성
└── medline/
    └── oncology/
        ├── pmid_12345678.medline
        ├── pmid_87654321.medline
        └── ...
```

- PubMed 검색 쿼리 예시

```bash
def build_search_query(keyword: str, journal: str, year: int) -> str:
    return (f'({keyword}) AND ({journal})[Journal] AND '
            f'({year}/01/01[Entrez Date] : {year}/12/31[Entrez Date])')
```

- API 속도 제한 준수

```bash
def respect_api_limits():
    time.sleep(0.34)  # 초당 3회 제한 (1/3 = 0.33초)
```

### 3-2. 텍스트 추출

이 단계에서는 **BioPython의 Medline 파서**를 사용하여 다운로드된 파일에서 **초록(Abstract) 텍스트만 추출**합니다.  

---

- 실행 명령어

모든 연도의 Medline 파일을 자동으로 찾아 처리합니다 (매개변수 없음):

```bash
python 02_text_extraction/extract_abstracts.py
```
- 결과 구조

```bash
output/2023/abstracts/
├── pmid_12345678.txt
├── pmid_87654321.txt
├── pmid_11111111.txt
└── ...
```

- python 코드 예시

```bash
def extract_abstract(medline_data: dict) -> str:
    """Medline 데이터에서 초록 추출"""
    abstract = medline_data.get('AB')
    if isinstance(abstract, list):
        return ' '.join(abstract)
    return abstract if abstract else ""

# 처리 통계 추적
stats = {
    'total_files': 0,
    'processed': 0,
    'abstracts_found': 0,
    'no_abstract': 0,
    'errors': 0,
    'skipped': 0
}
```

### 3-3. NER 처리

이 단계에서는 **OpenAI GPT-4o-mini** 모델을 활용하여 의학 논문 초록에서 세 가지 핵심 개체를 추출합니다:

- **Disease**: 질병명  
- **Gene**: 유전자명  
- **Variant**: 유전변이명  

---

- 실행 명령어

모든 연도의 초록 파일을 자동으로 찾아 **NER 처리**를 수행합니다:

```bash
python 03_ner_processing/extract_entities.py \
  --model gpt-4o-mini \   # 선택사항: 모델 지정
  --resume                # 선택사항: 중단된 작업 재개
```

- 결과 파일 형식(JSON 예시)


```bash
{
  "disease": ["melanoma", "colorectal cancer"],
  "gene": ["KRAS", "TP53", "BRAF"],
  "variant": ["G12D", "R273H", "V600E"]
}

# NER 프롬프트 형식
NER_PROMPT = """
아래 의학 논문 초록을 읽고 Named Entity Recognition을 수행해주세요.

추출할 개체명 유형:
- Disease: 질병명
- Gene: 유전자명
- Variant: 유전변이명

출력 형식: JSON (텍스트 없이)
{"disease": [], "gene": [], "variant": []}

초록: {abstract}
"""
```

### 3-4. 한국어 번역

이 단계에서는 **ChatGPT (GPT-4o-mini 모델)**를 활용하여 필터링된 영문 초록을 
의학적 정확성을 유지하면서 한국어로 번역 및 요약합니다.  

---

- 실행 명령어

모든 연도의 필터링된 변이 데이터를 자동으로 찾아 한국어 번역 수행:

```bash
python 05_korean_annotation/generate_annotations.py \
  --model gpt-4o-mini \   # 선택사항: 모델 지정
  --resume                # 선택사항: 중단된 작업 재개

```

- 결과 구조

```bash
output/2023/korean/
├── 12345678.txt     # 한국어 요약 (PMID 기반 파일명)
├── 87654321.txt
├── 11111111.txt
└── ...
```


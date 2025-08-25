# 1. 개요

## 1.1 목적
본 매뉴얼은 유전변이 해석에 특화된 거대언어모델(LLM)의 개발, 구축, 운영에 대한 표준 절차를 제시하여  
일관성 있고 안정적인 AI 시스템 구축을 목표로 합니다.

## 1.2 용어 정의
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

![전체 구성도](https://raw.githubusercontent.com/kjh594/LLM_Test/main/images/diagram.png)

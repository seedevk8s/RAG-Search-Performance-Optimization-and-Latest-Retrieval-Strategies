# Contextual Compression 상세 분석 - Mermaid 다이어그램

## 1. 전체 시스템 아키텍처

```mermaid
graph TD
    A[documents.csv - 30개 원본 문서] --> B[GPT-4o-mini 압축]
    B --> C[압축된 문서 30개]
    
    A --> D[원본 Pinecone 인덱스]
    C --> E[압축 Pinecone 인덱스]
    
    F[queries.csv - 30개 질의] --> G[검색 실행]
    D --> G
    E --> G
    
    G --> H[성능 비교]
    H --> I[P@5, R@5, MRR, MAP]
    
    J[환경 설정] --> B
    J --> D
    J --> E
```

## 2. 문서 압축 워크플로우

```mermaid
sequenceDiagram
    participant Doc as 원본 문서
    participant Chain as Summarization Chain
    participant GPT as GPT-4o-mini
    participant Comp as 압축 문서
    participant Vec as 벡터 스토어
    
    loop 30개 문서 각각
        Doc->>Chain: 문서 내용 입력
        Chain->>GPT: 프롬프트 + 문서
        Note over GPT: temperature=0.3으로<br/>일관된 요약 생성
        GPT-->>Chain: 요약된 텍스트
        Chain-->>Comp: 압축 문서 생성
        Note over Chain,Comp: 1초 대기<br/>(API 제한 관리)
        Comp->>Vec: 압축 인덱스에 저장
    end
```

## 3. 평가 메트릭 계산 과정

```mermaid
graph TD
    subgraph "질의별 평가"
        A[질의 입력] --> B[원본 인덱스 검색<br/>top-k=5]
        A --> C[압축 인덱스 검색<br/>top-k=5]
        
        B --> D[검색 결과<br/>doc_ids 리스트]
        C --> E[검색 결과<br/>doc_ids 리스트]
        
        F[관련성 데이터<br/>relevant_doc_ids<br/>형식: doc1=등급;doc2=등급] --> G[parse_relevant 함수<br/>딕셔너리 변환]
    end
    
    subgraph "지표 계산"
        D --> H[compute_metrics 함수]
        E --> I[compute_metrics 함수]
        G --> H
        G --> I
        
        H --> J["P@5 = hits/5<br/>R@5 = hits/total_relevant<br/>MRR = 1/first_relevant_rank<br/>MAP = mean(precisions)"]
        I --> K["P@5 = hits/5<br/>R@5 = hits/total_relevant<br/>MRR = 1/first_relevant_rank<br/>MAP = mean(precisions)"]
    end
    
    subgraph "전체 평가"
        J --> L[30개 질의 평균]
        K --> M[30개 질의 평균]
        
        L --> N[원본 최종 점수]
        M --> O[압축 최종 점수]
        
        N --> P[성능 비교 분석]
        O --> P
    end
```

## 4. 성능 비교 결과 상세

```mermaid
graph LR
    subgraph "Precision@5 (정확도)"
        A1[원본: 0.260000<br/>26.0%] 
        A2[압축: 0.266667<br/>26.7%]
        A3["+2.6% 향상<br/>✅ 개선"]
        A1 --> A3
        A2 --> A3
    end
    
    subgraph "Recall@5 (재현율)"
        B1[원본: 0.916667<br/>91.7%]
        B2[압축: 0.927778<br/>92.8%] 
        B3["+1.2% 향상<br/>✅ 개선"]
        B1 --> B3
        B2 --> B3
    end
    
    subgraph "MRR (평균 역순위)"
        C1[원본: 0.983333<br/>98.3%]
        C2[압축: 0.966667<br/>96.7%]
        C3["-1.7% 감소<br/>⚠️ 소폭 저하"]
        C1 --> C3
        C2 --> C3
    end
    
    subgraph "MAP (평균 정밀도)"
        D1[원본: 0.959444<br/>95.9%]
        D2[압축: 0.950185<br/>95.0%]
        D3["-1.0% 감소<br/>⚠️ 소폭 저하"]
        D1 --> D3
        D2 --> D3
    end
```

## 5. 기술 스택 상세

```mermaid
mindmap
  root((Contextual Compression))
    LLM
      GPT-4o-mini
        temperature: 0.3
        API 호출 제한 관리
        일관된 요약 품질
    Vector DB
      Pinecone
        Serverless 아키텍처
        1536 차원 임베딩
        코사인 유사도
        자동 스케일링
    Embeddings
      OpenAI text-embedding
        문서 임베딩
        질의 임베딩
        의미적 유사도 계산
    Framework
      LangChain
        체인 구성
        프롬프트 템플릿
        출력 파서
        벡터 스토어 통합
    Data Processing
      Pandas
        CSV 데이터 처리
        결과 분석
        DataFrame 조작
    Visualization
      Matplotlib
        한글 폰트 설정
        성능 그래프
        비교 차트
```

## 6. 실험 설계 검증 포인트

```mermaid
graph TD
    subgraph "데이터 품질"
        A[30개 문서<br/>적절한 샘플 크기] --> A1[✅ 통계적 유의성]
        B[30개 질의<br/>다양한 질의 유형] --> B1[✅ 포괄적 평가]
        C[관련성 판단<br/>등급 기반 평가] --> C1[✅ 정확한 기준]
    end
    
    subgraph "기술적 구현"
        D[동일한 임베딩 모델<br/>OpenAI embeddings] --> D1[✅ 공정한 비교]
        E[동일한 검색 파라미터<br/>top-k=5] --> E1[✅ 일관된 조건]
        F[체계적 평가 함수<br/>4가지 지표] --> F1[✅ 다각도 분석]
    end
    
    subgraph "실험 통제"
        G[API 호출 제한 관리<br/>1초 대기] --> G1[✅ 안정적 실행]
        H[재현 가능한 코드<br/>환경 변수 관리] --> H1[✅ 실험 재현성]
        I[시각화 포함<br/>한글 지원] --> I1[✅ 결과 해석]
    end
```

## 7. 실제 구현 세부사항

```mermaid
graph TD
    A[환경 변수 로드] --> B[API 키 설정]
    B --> C[데이터 로딩]
    
    C --> D[documents.csv 30개]
    C --> E[queries.csv 30개]
    
    D --> F[GPT-4o-mini 압축 체인]
    F --> G[압축된 문서 생성]
    
    G --> H[Pinecone 압축 인덱스 생성]
    D --> I[Pinecone 원본 인덱스]
    
    E --> J[검색 실행]
    H --> J
    I --> J
    
    J --> K[원본 검색 결과]
    J --> L[압축 검색 결과]
    
    K --> M[평가 메트릭 계산]
    L --> M
    
    M --> N[성능 비교 시각화]
```

이 상세한 mermaid 다이어그램들은 실험의 모든 단계와 기술적 세부사항을 명확하게 보여줍니다!

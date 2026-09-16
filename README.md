# Korean AI Overview activation dataset

한국어 검색 검색어에 대한 AI 검색 요약 활성화 여부와 4축 라벨(형태·의도·주제·확신도)을 담은 데이터셋입니다.

## 파일

- `KoAIO.csv` — 배포용 최종 데이터셋 (23,276행)

## 데이터 구성 (3개 소스)

| 출처 | 설명 | 고유 검색어 | 행 수(구글+네이버 쌍) |
|---|---|---:|---:|
| `ORCAS` | ORCAS 검색로그에서 샘플링, 한국어로 번역 | 3,265 | 6,530 |
| `AIO` | 공개 벤치마크(ELI5/NQ/Debate/Amazon Retail/ORCAS 등 하위 subset), 한국어로 번역 | 4,022 | 8,044 |
| `Naver_Kin` | 네이버 지식iN 질문 제목, 원문 대신 `docId`/`url`로 배포 | 4,351 | 8,702 |

## 컬럼

| 컬럼 | 타입 | 설명 |
|---|---|---|
| `데이터 출처` | `ORCAS`/`AIO`/`Naver_Kin` | 검색어 출처 |
| `AIO_Benchmark 하위출처` | 문자열, `AIO`에만 해당 | AIO 소스 내 세부 벤치마크명 (ELI5/NQ/Debate/Amazon Retail/ORCAS 등) `AIO_Benchmark="ORCAS"`는 최상위 `source="ORCAS"`와 다른 그룹입니다 — AIO Benchmark 내부에 포함된 ORCAS 유래 서브셋을 가리키며, 두 그룹 간 중복 검색어는 이미 제거되었습니다 |
| `검색어` | 한국어 텍스트, `ORCAS`/`AIO`에만 해당 | GPT-5.6 Terra로 번역한 검색 검색어 |
| `지식인_doc_id` | 문자열, `Naver_Kin`에만 해당 | 네이버 지식iN 게시물 고유 ID |
| `지식인_url` | URL, `Naver_Kin`에만 해당 | 위 게시물의 원문 링크 |
| `검색엔진` | `google`/`naver` | 어느 엔진에서 측정했는지 |
| `AI 검색 요약 활성화` | `True`/`False` | 해당 검색엔진이 이 검색어에 AI 검색 요약을 노출했는지 |
| `문법적 형태` | 범주형(10종) | 문법적 형태 (명사구/의문문/평서문 등) |
| `검색 의도` | 범주형(5종) | 검색 의도 (이동형/거래형/상업형/정보형/분류 불가). |
| `주제` | 범주형(20종) | Google Trends 19개 카테고리 + 분류불가 = 총 20 카테고리 기준 주제 분류 |
| `LLM 확신도` | `high`/`medium`/`low` | 위 3개 라벨(form/intent/category)에 대한 LLM 자체 확신도 |

## 라벨링 방법

- 모델: `openai/gpt-5.6-terra` (OpenRouter)
- 형태·의도·카테고리 3축을 하나의 구조화 출력(JSON schema)으로 동시 라벨링
- `label_confidence`는 LLM이 자체 판단한 확신도이며, 사람 검증(100건, 어노테이터 2인) 대조 결과:
  - `검색 의도`: high 신뢰도군이 medium 대비 사람과의 일치율이 약 25%p 높음 
  - `주제`: high가 medium 대비 약 11.5%p 높음 
  - `문법적 형태`: high/medium 간 차이 거의 없음 

## 필터링 파이프라인 (원본 검색어 → 최종 배포본)

1. 원본 수집 검색어: 33,379건 (ORCAS/AIO 4개 폴더 × 4,685 + Naver_Kin 9,954)
2. 한국어 번역 가능성 판정(LLM 기반 보수적 필터) — 표현이 부자연스럽거나(BROKEN_GRAMMAR 등) 한국 맥락에 무관한(LOCALE, FOREIGN_SERVICE 등) 검색어 제외
3. AI 검색 요약 활성화 여부 측정 (Google/Naver 각각 크롤)
4. LLM 통해 문법적 형태 / 검색 의도 / 주제에 대한 라벨링 

## 개인정보/저작권 관련 처리

- Naver_Kin은 검색어 원문 대신 `docId`/`url`만 배포 — 게시물 텍스트를 직접 재배포하지 않음

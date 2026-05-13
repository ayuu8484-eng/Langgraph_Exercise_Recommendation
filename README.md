# AI 운동 추천 멀티에이전트 시스템

> **LangGraph + LangChain + Ollama**를 활용한 로컬 LLM 기반 다단계 멀티에이전트 파이프라인

---

## 프로젝트 개요

사용자의 자연어 질문에서 신체 증상을 자동으로 추출하고, 맞춤형 운동을 추천하는 **멀티에이전트 AI 시스템**
세 개의 전문화된 에이전트가 순차적으로 협력하여 입력된 건강 고민을 분석하고 구체적인 운동 솔루션을 제공

```
사용자 질문 → [추출 에이전트] → [매칭 에이전트] → [답변 에이전트] → 최종 추천
```

---

### 상태 흐름 (AgentState)

| 상태 키 | 타입 | 설명 |
|---|---|---|
| `query` | str | 사용자의 원본 질문 |
| `symptoms` | str | 추출된 신체 증상/상태 |
| `exercise_candidates` | str | 추천 운동 후보 목록 |
| `result` | str | 최종 생성된 답변 |

---

## 에이전트 구성

### 1. Extractor Agent — 증상 추출

```python
extractor_prompt = PromptTemplate.from_template("""
사용자의 질문에서 신체적 상태나 증상에 해당하는 단어 또는 구를 추출하세요.
결과는 쉼표로 구분된 문자열로 출력하세요.
질문: {query}
""")
```

**역할**: 사용자의 자연어 입력에서 건강 관련 키워드(증상, 신체 상태)만 정확하게 추출  
**입력**: `query` (사용자 질문)  
**출력**: `symptoms` (쉼표 구분 증상 문자열)  

**예시**
```
입력: "체력이 안좋고, 살이 계속 찌는데 어떤 운동을 할까?"
출력: "체력 저하, 체중 증가"
```

---

### 2. Matcher Agent — 운동 후보 매칭

```python
matcher_prompt = PromptTemplate.from_template("""
다음 사용자의 상태(증상)를 해결하거나 개선하는 데 도움이 되는 운동 3가지를 추천하세요.
결과는 운동 이름만 쉼표로 구분하여 출력하세요.
상태/증상: {symptoms}
""")
```

**역할**: 추출된 증상을 기반으로 적합한 운동 3가지를 선별  
**입력**: `symptoms` (증상 목록)  
**출력**: `exercise_candidates` (운동 이름 목록)  

**예시**
```
입력: "체력 저하, 체중 증가"
출력: "걷기, 수영, 자전거 타기"
```

---

### 3. Answer Agent — 최종 답변 생성

```python
answer_prompt = PromptTemplate.from_template("""
사용자의 상태와 추천 운동 리스트를 바탕으로 최종 답변을 작성하세요.
[상태]: {symptoms}
[추천 운동]: {exercise_candidates}
""")
```

**역할**: 증상 분석 결과와 운동 후보를 종합하여 사용자 친화적인 최종 답변 생성  
**입력**: `symptoms` + `exercise_candidates`  
**출력**: `result` (완성된 운동 추천 답변)  

---

## LangGraph 파이프라인

```python
workflow = StateGraph(AgentState)

# 노드 등록
workflow.add_node("extractor", extractor_agent)
workflow.add_node("matcher", matcher_agent)
workflow.add_node("answer", answer_agent)

# 실행 순서 정의
workflow.set_entry_point("extractor")
workflow.add_edge("extractor", "matcher")
workflow.add_edge("matcher", "answer")
workflow.add_edge("answer", END)

app = workflow.compile()
```

LangGraph의 `StateGraph`를 통해 각 에이전트를 **노드**로 등록하고, **엣지**로 실행 순서를 명시적으로 연결합니다.  
에이전트 간 데이터는 `AgentState` TypedDict를 통해 공유되며, 각 노드는 상태를 변환해 다음 노드로 전달합니다.

<img width="108" height="432" alt="image" src="https://github.com/user-attachments/assets/b0b623bc-ac2c-4177-9ab4-241e80a7e4d0" />

---

## 기술 스택

| 분류 | 기술 | 설명 |
|---|---|---|
| **LLM 프레임워크** | [LangChain](https://www.langchain.com/) | 프롬프트 템플릿, LLM 체인 구성 |
| **멀티에이전트 오케스트레이션** | [LangGraph](https://langchain-ai.github.io/langgraph/) | 상태 기반 에이전트 그래프 관리 |
| **로컬 LLM 서버** | [Ollama](https://ollama.com/) | 로컬 환경에서 LLM 실행 |
| **LLM 모델** | [EXAONE 3.5 2.4B](https://www.lgresearch.ai/) | LG AI Research의 한국어 특화 소형 언어모델 |
| **언어** | Python 3.10+ | 전체 파이프라인 구현 |
| **실행 환경** | Jupyter Notebook | 개발 및 시각화 |

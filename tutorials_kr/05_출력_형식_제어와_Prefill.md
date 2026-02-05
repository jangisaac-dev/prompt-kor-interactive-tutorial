[Thumbnail Prompt: A sleek, modern split-screen interface. On the left, a developer enters a raw JSON bracket '{'. On the right, Claude instantly generates a perfectly formatted JSON structure without any introductory text. Digital art style, vibrant cyan and purple accents, high tech atmosphere.]

# Anthropic Prompt Engineering 101 - Part 5: 출력 형식 제어와 Claude 대신 말하기 (Prefill)

LLM을 서비스에 연동해 본 개발자라면 누구나 한 번쯤 이런 경험이 있을 겁니다. API는 깔끔한 JSON 데이터만 원하는데, Claude는 친절하게도 "네, 요청하신 내용을 JSON으로 정리해 드립니다!"라며 불필요한 서두를 붙이는 상황 말이죠. 이 사소한 '잡담(chitchat)'은 파싱 에러의 주범이 되고, 전체 시스템의 안정성을 위협합니다.

이번 가이드에서는 Claude를 완벽하게 통제하여 여러분이 원하는 **정확한 형식**으로만 답하게 만드는 두 가지 핵심 기술, **출력 형식 제어(Formatting Output)**와 **대신 말하기(Prefill)**를 다룹니다. 이 기술들을 마스터하면 Claude는 더 이상 단순한 챗봇이 아니라, 여러분의 백엔드 코드와 완벽하게 맞물려 돌아가는 정교한 데이터 처리 엔진이 됩니다.

## 1. Claude 대신 말하기: Prefill의 마법

**Prefill(사전 채우기)**은 Anthropic API가 제공하는 가장 강력한 기능 중 하나입니다. 보통의 대화는 사용자가 묻고 모델이 답하는 형식이지만, Prefill을 사용하면 **Claude가 답변을 시작할 첫 몇 단어(또는 문장)를 우리가 미리 작성해서 보낼 수 있습니다.**

Claude는 마치 자기가 그 말을 막 뱉은 것처럼 착각하고, 그 뒤를 자연스럽게 이어갑니다. 이는 단순히 서두를 생략하는 수준을 넘어, 모델의 말투, 논리적 방향성, 심지어는 특정 형식으로의 전환을 강제하는 '치트키'가 됩니다.

### Prefill의 세 가지 핵심 효과
1. **잡담 원천 차단**: "Certainly!", "Here is the..." 같은 불필요한 텍스트를 생략하고 바로 본론(JSON/XML)으로 들어가게 합니다.
2. **논리적 유도**: 모델이 이미 특정 결론을 내린 것처럼 답변을 시작하게 하여, 그 결론을 뒷받침하는 근거를 생성하도록 유도합니다.
3. **복잡한 구조 강제**: XML 태그나 JSON의 키 값을 미리 열어줌으로써 형식이 깨질 확률을 0%에 가깝게 만듭니다.

---

## 2. 심화 예제: Multiple Variables와 스타일 제어

프롬프트에 여러 변수를 삽입하고, Prefill을 결합하면 매우 동적인 텍스트 생성이 가능해집니다. 이메일을 특정 스타일로 재작성하는 예제를 통해 살펴봅시다.

**프롬프트 설정:**
- **Variable {{EMAIL}}**: 사용자의 원문 이메일
- **Variable {{ADJECTIVE}}**: 재작성할 스타일 (예: "olde english")

**User Message:**
```text
Please rewrite the following email in a {{ADJECTIVE}} style.

<email>
{{EMAIL}}
</email>
```

**Assistant Message (Prefill):**
```text
Certainly! Here is the email rewritten in a {{ADJECTIVE}} style:
```

이렇게 Prefill을 설정하면 Claude는 "이메일을 어떻게 고쳐드릴까요?" 같은 질문 없이 바로 변환된 내용을 출력합니다.

**실제 실행 결과 (ADJECTIVE = "olde english"):**
> Certainly! Here is the email rewritten in a **olde english** style:
> 
> "Hark! I doth write to thee regarding the shipment that hath failed to arrive at my manor..."

이처럼 Prefill 문구 안에 변수({{ADJECTIVE}})를 다시 포함시키면, Claude는 자신이 어떤 스타일로 글을 써야 하는지 한 번 더 상기하게 되어 정확도가 비약적으로 상승합니다.

---

## 3. 완벽한 JSON을 얻는 단계적 Prefill 전략

개발자들이 가장 많이 사용하는 JSON 출력 제어 기법입니다. Claude에게 JSON을 요청할 때, 단순히 "JSON으로 줘"라고 말하는 것보다 Prefill을 어떻게 활용하느냐에 따라 파싱 안정성이 달라집니다.

### 1단계: 기본 시작 괄호 (`{`)
가장 흔한 방법입니다. Assistant 메시지를 `{`로 시작합니다.

- **Prefill**: `{`
- **Claude**: `"name": "Apple", "color": "Red"}`

이 방식만으로도 앞뒤의 불필요한 잡담은 99% 사라집니다.

### 2단계: 특정 키 강제 (`{"name":`)
만약 특정 필드가 반드시 처음에 와야 하거나, 모델이 자꾸 엉뚱한 키를 생성한다면 더 구체적으로 Prefill을 채울 수 있습니다.

- **Prefill**: `{"name":`
- **Claude**: `"Claude-3-Sonnet", "version": "3.5"}`

이 기법은 **스키마 준수**가 생명인 API 연동 환경에서 파싱 로직을 보호하는 가장 확실한 방어선이 됩니다.

---

## 4. Exercise 5.3: 두 마리 동물의 하이쿠 (멀티 데이터 제어)

여러 개의 데이터를 동시에 생성하고 각각을 개별 태그로 분리해야 할 때 XML과 Prefill의 조합은 빛을 발합니다.

**입력 변수:**
- `ANIMAL1`: 고양이
- `ANIMAL2`: 강아지

**User Message:**
```text
{{ANIMAL1}}과 {{ANIMAL2}}에 대해 각각 하이쿠를 한 편씩 지어주세요.
첫 번째 시는 <haiku1> 태그에, 두 번째 시는 <haiku2> 태그에 넣어주세요.
```

**Assistant Message (Prefill):**
```text
<haiku1>
```

**Claude의 이어지는 응답 (실제 출력):**
```xml
부드러운 발
햇살 아래 잠드네
오후의 낮잠
</haiku1>
<haiku2>
마당을 뛰네
꼬리 흔들며 반겨
충직한 친구
</haiku2>
```

우리가 `<haiku1>` 태그를 미리 열어주었기 때문에, Claude는 자연스럽게 첫 번째 시를 완성하고 닫는 태그(`</haiku1>`)를 붙인 뒤, 스스로 `<haiku2>`를 열어 두 번째 시를 작성합니다. 이 패턴을 활용하면 하나의 요청으로 여러 개의 정제된 데이터를 안전하게 추출할 수 있습니다.

---

## 5. Stop Sequences: 생산 라인의 비상 정지 버튼

구조화된 출력을 사용할 때 성능과 비용을 최적화하는 마지막 퍼즐 조각은 `stop_sequences`입니다.

Claude는 요청받은 형식을 다 작성한 뒤에도 간혹 "도움이 되셨나요?" 같은 맺음말을 덧붙이려 시도합니다. 이는 불필요한 토큰 소비(비용 발생)와 응답 지연(Latency)을 초래합니다.

**API 요청 설정 예시:**
```python
response = client.messages.create(
    model="claude-3-5-sonnet-20241022",
    max_tokens=1024,
    messages=[
        {"role": "user", "content": "...하이쿠를 지어주세요..."},
        {"role": "assistant", "content": "<haiku>"}
    ],
    stop_sequences=["</haiku>"] # 핵심 설정!
)
```

### Stop Sequences의 효과
- **비용 절약**: 닫는 태그(`</haiku>`)가 생성되는 즉시 모델이 멈추므로, 그 이후에 나올 수 있는 모든 '잡담' 토큰 비용을 아낄 수 있습니다.
- **파싱 안정성**: 답변의 끝이 명확해지므로, 정규표현식이나 파서가 데이터를 읽어올 때 혼선이 생기지 않습니다.
- **속도 향상**: 불필요한 문장을 생성하는 시간을 제거하여 사용자에게 더 빠른 응답을 전달합니다.

---

## 6. 실전 사용기 (시뮬레이션)

> ⚠️ 아래 내용은 실제 실행 결과가 아닌 텍스트 시뮬레이션입니다.

**상황**: 사용자의 리뷰 데이터를 분석하여 `긍정/부정` 점수와 `핵심 키워드`를 추출하는 자동화 파이프라인을 구축 중입니다.

### 시나리오 A: Prefill 미사용 (실패 사례)
- **User**: "이 리뷰 분석해서 JSON으로 줘: '배송은 진짜 빠른데, 제품 마감이 좀 아쉽네요.'"
- **Claude**: "네, 분석해 드렸습니다. 아래는 요청하신 JSON 데이터입니다. ```json { "sentiment": 0.6, "keywords": ["배송", "마감"] } ``` 도움이 되셨길 바랍니다!"
- **결과**: 🚨 **JSONDecodeError** 발생. (앞뒤 설명 텍스트 때문에 파서가 작동하지 않음)

### 시나리오 B: Prefill + Stop Sequences 사용 (성공 사례)
- **User**: "리뷰 분석 JSON 추출: '배송은 진짜 빠른데, 제품 마감이 좀 아쉽네요.'"
- **Assistant (Prefill)**: `{`
- **Stop Sequences**: `}`
- **Claude**: `"sentiment": 0.6, "keywords": ["배송", "마감"]}`
- **결과**: ✅ **성공**. 파이프라인이 즉시 `{...}` 데이터만 수신하여 DB에 저장. 잡담 토큰 0개 사용.

---

## 7. 결론

Claude를 단순한 챗봇으로 대하면 그 잠재력의 50%밖에 쓰지 못하는 것입니다. **Prefill**과 **출력 형식 제어**, 그리고 **Stop Sequences**를 조합하면 Claude는 여러분의 애플리케이션 안에서 완벽하게 예측 가능한 '함수'처럼 작동합니다.

- **Prefill**로 모델의 입을 열고,
- **XML/JSON**으로 형식을 고정하며,
- **Stop Sequences**로 깔끔하게 마무리지으세요.

이 세 가지 도구만 있으면 LLM의 고질적인 문제인 '수다'와 '불확실성'을 통제 가능한 '강점'으로 바꿀 수 있습니다.

## 8. TL;DR (핵심 요약)

*   **Prefill (대신 말하기)**: `assistant` 답변의 시작을 미리 작성하면 Claude가 그 뒤를 자연스럽게 이어갑니다.
*   **Multiple Variables**: Prefill 문구 안에 변수를 다시 포함하면 스타일 변환의 정확도가 극대화됩니다.
*   **JSON 제어**: `{` 또는 `{"key":`로 답변을 시작하게 하여 잡담을 차단하고 스키마를 강제하세요.
*   **XML 태그 활용**: `<tag>`로 데이터를 감싸면 여러 항목을 동시에 추출할 때 매우 유리합니다.
*   **Stop Sequences**: 닫는 태그(`</tag>`, `}`)를 중단 시퀀스로 설정하여 토큰 비용과 응답 시간을 아끼세요.

---
참고 링크
- [Anthropic Documentation - Prefill Claudes response](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/prefill-claudes-response)
- [Anthropic Documentation - Control output format](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/control-output-format)

Anthropic, PromptEngineering, Claude, Prefill, JSON, XML, LLM개발, StopSequences, 앤스로픽, 프롬프트엔지니어링

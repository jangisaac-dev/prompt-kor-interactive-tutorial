# Anthropic Prompt Engineering 101 - Part 10: 고급 기법 (Chaining, Tool Use, 그리고 그 너머)

> **Thumbnail Prompt:** A digital robot hand connecting puzzle pieces that represent different software tools, with a glowing chain linking them together, futuristic tech aesthetic, 4k resolution, cinematic lighting.

## 1. 프롬프트는 '문장'이 아니라 '로직'입니다

프롬프트 하나로 모든 문제를 해결하려는 시도는 마치 복잡한 프로그램을 `main()` 함수 하나에 다 몰아넣는 것과 같습니다. Anthropic의 엔지니어들은 프롬프트를 단순한 명령어가 아닌, 서로 연결되고 도구를 조작하는 **'소프트웨어 로직'**으로 취급합니다.

우리는 지금까지 긴 여정을 함께했습니다. 기본적인 명확성(Clarity)부터 XML 태그, 단계적 사고(CoT), 그리고 환각 방지 기법까지 살펴보았습니다. 이제 여러분은 단순히 "AI와 대화하는 법"을 넘어, **"복잡한 시스템의 뇌로 작동하는 AI"**를 설계할 준비가 되었습니다.

이번 마지막 장에서는 단일 프롬프트의 한계를 뛰어넘는 **시스템 통합(System Integration)** 기술을 다룹니다. AI가 스스로 실수를 교정하게 만드는 **Chaining**, 외부 세계와 소통하며 기능을 실행하는 **Tool Use**, 그리고 지식을 확장하는 **RAG**까지. 이것은 프롬프트 엔지니어링이 본격적인 소프트웨어 엔지니어링으로 진화하는 지점입니다.

## 2. 프롬프트 체이닝 (Chaining Prompts): 스스로 교정하는 AI

복잡한 작업을 한 번에 처리하려고 하면 사람도 실수합니다. AI도 마찬가지입니다. **프롬프트 체이닝**은 하나의 거대한 작업을 여러 단계로 쪼개고, 앞 단계의 출력을 다음 단계의 입력으로 사용하는 기법입니다.

특히 이 기법은 **"생성(Generation)"과 "검증(Verification)"을 분리**할 때 강력한 힘을 발휘합니다.

### 2.1. 실전 예시: 자가 교정(Self-Correction) 워크플로

'ab'로 끝나는 단어 목록을 생성하는 과제를 생각해 봅시다. LLM은 토큰 단위로 사고하기 때문에, 철자 기반의 제약 조건에 약한 모습을 보일 때가 있습니다. 이를 3단계 체인으로 해결해 보겠습니다.

**Step 1: 목록 생성 (Generation)**
먼저 모델에게 후보군을 생성하게 합니다.
- **Prompt:** `"ab"로 끝나는 영어 단어 10개를 나열해 줘.`
- **Output:** `flab, grab, crab, slab, blab, stab, scab, drab, zlab, jab` (여기서 'zlab'은 실제 단어가 아닙니다.)

**Step 2: 검증 (Verification)**
Step 1의 결과를 입력으로 주고, 모델에게 '검수자' 역할을 맡깁니다.
- **Prompt:** 
  ```text
  아래는 "ab"로 끝나는 단어 목록이야. 
  1. 목록을 검토하고 실제로 존재하지 않는 단어가 있다면 제거하거나 올바른 단어로 교체해 줘.
  2. 만약 모든 단어가 실제 단어라면, 원래 목록을 그대로 반환해.

  <list>
  [Step 1의 출력 결과]
  </list>
  ```
- **Output:** `zlab은 실제 단어가 아닙니다. 대신 'cab'으로 교체합니다.`

**Step 3: 최종 출력 (Final Output)**
검증된 내용을 바탕으로 최종 결과물을 정제합니다.

### 2.2. 실전 예시: 스토리 개선 (Critique & Rewrite)

창작 영역에서도 체이닝은 빛을 발합니다. "한 번에 완벽한 글을 써라"고 하기보다, 다음과 같은 **비판적 루프**를 설계하세요.

1.  **Write**: "고양이에 대한 짧은 이야기를 써줘."
2.  **Critique**: "방금 쓴 이야기의 페이싱과 캐릭터 묘사를 비판적으로 검토해 줘. 개선할 점 3가지를 알려줘."
3.  **Rewrite**: "비판 내용을 반영하여 이야기를 훨씬 더 풍부하게 다시 써줘."

이처럼 생성과 분석의 인지 부하를 분리하면, 모델은 각 단계에서 자신의 '뇌'를 100% 활용할 수 있게 됩니다.

## 3. 도구 사용 (Tool Use): AI를 시스템 라우터로 활용하기

Claude와 같은 최신 LLM은 단순히 텍스트를 생성하는 것을 넘어, **외부 도구(Tool)를 조작할 줄 압니다.** AI가 직접 계산을 하거나 DB를 조회하는 대신, "이 작업을 수행하려면 계산기나 SQL이 필요해"라고 판단하고 시스템에 요청하는 방식입니다.

### 3.1. 시스템 프롬프트: 도구 스펙(Schema) 정의

도구 사용을 위해 가장 먼저 해야 할 일은 시스템 프롬프트에 사용 가능한 도구 목록을 **XML 스키마** 형태로 명확히 정의하는 것입니다.

```xml
너는 도움을 주는 AI 비서야. 너는 아래와 같은 도구를 사용할 수 있어.
도구가 필요한 경우 <function_calls> 태그를 사용하여 호출해 줘.

<tools>
    <tool_description>
        <tool_name>do_pairwise_arithmetic</tool_name>
        <description>두 수에 대해 사칙연산을 수행합니다.</description>
        <parameters>
            <parameter><name>first_operand</name><type>number</type></parameter>
            <parameter><name>second_operand</name><type>number</type></parameter>
            <parameter><name>operator</name><type>string</type><description>+, -, *, / 중 하나</description></parameter>
        </parameters>
    </tool_description>
</tools>
```

### 3.2. 도구 사용 워크플로 (The Loop)

도구 사용은 다음과 같은 순환 구조로 이루어집니다.

1.  **Claude의 판단**: 사용자 질문이 도구를 필요로 한다면, Claude는 답변 대신 `<function_calls>`를 출력합니다.
2.  **Stop Sequence**: 이때 시스템은 `</function_calls>`를 **Stop Sequence**로 설정하여 모델이 호출 직후 생성을 멈추게 합니다.
3.  **시스템 실행**: 백엔드(Python 등)에서 실제 도구 로직(예: 계산기 실행)을 수행합니다.
4.  **결과 주입 (Injection)**: 시스템이 결과를 `<function_results>` 태그에 담아 Claude에게 다시 전달합니다.
5.  **최종 답변**: Claude는 주입된 결과를 보고 사용자에게 자연스러운 언어로 최종 답변을 내놓습니다.

**예시: 계산기 활용 시나리오**
> ⚠️ 아래 내용은 실제 실행 결과가 아닌 텍스트 시뮬레이션입니다.

- **User**: "120에 45를 곱하면 얼마야?"
- **Claude**: `<function_calls><tool_code>do_pairwise_arithmetic(first_operand=120, second_operand=45, operator="*")</tool_code></function_calls>`
- **System**: `<function_results>5400</function_results>`
- **Claude**: "120에 45를 곱한 값은 5,400입니다."

이 기법을 응용하여 **SQL 데이터베이스**에서 유저 정보를 조회하거나, 최신 뉴스를 검색하는 등의 기능을 완벽하게 구현할 수 있습니다.

## 4. 검색 증강 생성 (RAG) - 지식의 확장

RAG(Retrieval-Augmented Generation)는 본질적으로 **Retrieval(검색 도구) + Contextual Answering(문맥 기반 답변)**의 결합입니다.

우리가 8편에서 배운 **"문맥에 근거해 답하기"** 기술과 10편의 **"Tool Use"** 기술이 만나면 RAG가 완성됩니다. 모델이 외부 지식 베이스(Vector DB 등)를 검색 도구로 사용하여 실시간으로 필요한 정보를 가져오고, 그 정보를 기반으로 정확한 답변을 내놓는 것이죠.

## 5. 시리즈를 마치며: 프롬프트 엔지니어링은 '반복적 과학'입니다

이것으로 총 10편에 걸친 **Anthropic Prompt Engineering 101** 시리즈를 마칩니다. 우리는 단순히 "AI에게 잘 말하는 법"을 넘어, AI의 사고 과정을 설계하고 제어하는 시스템적 접근법을 배웠습니다.

1.  **Clarity**: 명확함이 모든 프롬프트의 기초입니다.
2.  **Persona & Context**: 역할을 부여하고 충분한 배경 정보를 제공하세요.
3.  **XML Tags**: 데이터를 구조화하여 모델의 이해도를 높이세요.
4.  **CoT**: 복잡한 문제는 '단계별'로 생각하게 만드세요.
5.  **Prefill & Formatting**: 원하는 출력 형식을 강제하세요.
6.  **Chaining & Tools**: 여러 프롬프트를 엮고 외부 도구를 연결하세요.

프롬프트 엔지니어링은 한 번에 정답을 맞히는 문학이 아니라, **작성-테스트-실패-수정**을 반복하는 **실험 과학**입니다. 끊임없는 반복(Iteration)만이 최상의 결과를 만듭니다.

이제 여러분의 차례입니다. IDE를 켜고, Claude에게 말을 걸어보세요. 여러분이 설계할 새로운 지능형 시스템을 기대하겠습니다.

---

## TL;DR
- **Chaining**: 생성과 검수 단계를 분리하여 모델의 자가 교정 능력을 극대화하세요.
- **Tool Use**: Claude를 외부 시스템과 연결하는 스마트한 라우터로 활용하세요.
- **RAG**: 검색 도구와 문맥 기반 답변 기술을 결합하여 지식을 확장할 수 있습니다.
- **Iteration**: 프롬프트 엔지니어링은 반복적인 실험을 통해 완성되는 과학입니다.

## 참고 링크
- [Anthropic Cookbook - Function Calling](https://github.com/anthropic-inc/anthropic-cookbook)
- [Anthropic Documentation - Tool Use](https://docs.anthropic.com/claude/docs/tool-use)

AnthropicPromptEngineering, PromptEngineering, Claude3, LLM, ChainingPrompts, ToolUse, RAG, AI개발, 프롬프트최적화, 시스템엔지니어링

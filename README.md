# Anthropic 프롬프트 엔지니어링 인터랙티브 튜토리얼 (한국어 심화 가이드)

이 저장소는 [Anthropic의 공식 프롬프트 엔지니어링 튜토리얼](https://github.com/anthropics/prompt-eng-interactive-tutorial)을 기반으로, 한국어 사용자들을 위해 **심화된 설명과 실전적인 예제**를 보강하여 재구성한 가이드입니다.

단순한 번역을 넘어, 한국어 뉘앙스를 고려한 프롬프트 작성법과 최신 Claude 3 모델의 특성을 반영한 Best Practice를 담았습니다.

## 📚 튜토리얼 목차

각 챕터는 단계별로 구성되어 있으며, 순서대로 학습하는 것을 권장합니다.

1. **[제1장: 환경 설정 및 시작하기](tutorials_kr/01_환경설정_및_시작하기.md)**
   - API Key 발급부터 기본 Python 환경 설정까지
   - 첫 번째 API 호출 실습

2. **[제2장: 프롬프트 기초와 명확성](tutorials_kr/02_프롬프트_기초와_명확성.md)**
   - "명확하다"는 것의 진짜 의미
   - 모호함을 없애는 구체적인 지시 기법

3. **[제3장: 페르소나와 역할 부여](tutorials_kr/03_페르소나와_역할_부여.md)**
   - Claude에게 역할을 부여하여 응답 품질 높이기
   - 상황에 맞는 어조와 스타일 설정

4. **[제4장: 데이터와 지시의 분리 (XML 태그 활용)](tutorials_kr/04_데이터와_지시_분리.md)**
   - XML 태그를 활용해 입력 데이터를 구조화하는 방법
   - 프롬프트 주입 공격 방지 및 정확도 향상

5. **[제5장: 출력 형식 제어와 Prefill](tutorials_kr/05_출력_형식_제어와_Prefill.md)**
   - JSON, 마크다운 등 원하는 포맷으로 출력 유도하기
   - Prefill(사전 입력)을 통한 강력한 제어권 확보

6. **[제6장: 단계적 사고 (CoT)](tutorials_kr/06_단계적_사고.md)**
   - 복잡한 추론을 위한 Chain of Thought 기법
   - `<thinking>` 태그를 활용한 논리적 도약 방지

7. **[제7장: 예시를 통한 학습 (Few-Shot Prompting)](tutorials_kr/07_예시를_통한_학습.md)**
   - 예제 제공을 통해 일관된 스타일과 포맷 유지하기
   - 효과적인 예제 선정과 배치 전략

8. **[제8장: 환각(Hallucination) 줄이기](tutorials_kr/08_Claude_환각_줄이기.md)**
   - Claude가 모르는 것을 "모른다"고 답하게 만들기
   - RAG(검색 증강 생성)의 기초와 신뢰성 확보

9. **[제9장: 복잡한 프롬프트 설계 (메타 프롬프트)](tutorials_kr/09_복잡한_프롬프트_설계.md)**
   - 긴 문맥을 다루는 요령과 주의점
   - 복합적인 지시사항을 체계적으로 전달하는 법

10. **[제10장: 고급 기법 - Chaining & Tool Use](tutorials_kr/10_고급_기법_Chaining_ToolUse.md)**
    - 작업을 여러 단계로 나누어 연결하기 (Prompt Chaining)
    - 외부 도구(Function Calling) 활용 입문

---

## 🔗 원본 출처 및 크레딧

이 튜토리얼은 [Anthropic](https://www.anthropic.com/)에서 제공하는 [Prompt Engineering Interactive Tutorial](https://github.com/anthropic-inc/prompt-eng-interactive-tutorial)을 참고하여 작성되었습니다.

Original Content © Anthropic.
Korean Adaptation & Deep Dive Guide by Isaac Jang.

# NeuroAI Evidence Explorer

## Problem

NeuroAI를 공부하는 학습자가 하나의 연구 질문과 관련된 논문,
핵심 근거, 한계, 후속 질문을 빠르게 파악하기 어렵다.

## V0

연구 질문을 입력하면 arXiv에서 관련 논문 5편을 검색하고
제목, 저자, 초록, 링크를 보여주는 Python CLI를 만든다.

## Example Input

How does hippocampal replay relate to experience replay in AI?

## Expected Output

- Title
- Authors
- Published date
- Abstract
- Paper URL

## V0 Acceptance Criteria

- 명령행에서 연구 질문을 입력할 수 있다.
- arXiv API에 실제 요청을 보낸다.
- 논문 5편을 가져온다.
- 결과에 제목과 URL이 포함된다.
- API 오류가 나면 이해 가능한 메시지를 출력한다.
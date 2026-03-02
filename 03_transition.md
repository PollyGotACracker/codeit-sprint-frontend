# Transition

## Transition 관련 이벤트

- `transitionrun`: transition 생성 시 발생(지연 시간과 관계없음)
- `transitionend`: transition이 끝날 때 발생
- `transitioncancel`: transition 취소 시 발생
  - hover transition 진행 중 마우스가 영역에서 벗어날 때
  - transition 대상 요소가 사라질 때(display: none 등)
- `transitionstart`: transition이 실제로 시작될 때(지연 시간 경과 후)

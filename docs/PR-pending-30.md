# 선행 Issue 완료 알림

## 관련 Issue

- [Issue #30](https://github.com/sys-arg/arg-ubuntu/issues/30)

## 구현한 기능

### 후속 Issue 담당자 자동 알림

- 동작 — 선행 Issue가 `Completed`로 닫히면, 해당 번호가 필수 선행 목록에 있는 열린 후속 Issue를 확인한다. 목록의 모든 Issue가 완료됐으면 GitHub에 지정된 담당자를 댓글로 멘션한다.
- 사용 기술 — GitHub Actions `issues.closed` 이벤트, `actions/github-script@v8`, GitHub Issues REST API를 사용한다. 워크플로에는 `issues: write` 권한을 지정한다.
- 핵심 코드 — [issue-ready-notification.yml](../.github/workflows/issue-ready-notification.yml#L24)의 `requiredIssues()`와 [후속 Issue 조건 검사](../.github/workflows/issue-ready-notification.yml#L59).

### 중복 방지 및 재실행

- 동작 — 워크플로 실행을 순서대로 처리하고 이전 봇 알림을 확인한다. 일부 댓글 쓰기가 실패하면 해당 실패를 기록하고 다른 후속 Issue를 계속 처리한다. 재실행하면 작성된 알림을 건너뛴다.
- 사용 기술 — GitHub Actions의 동시 실행 큐, Issue 댓글의 봇 작성자와 HTML 주석 식별 문자열을 사용한다.
- 핵심 코드 — [동시 실행 설정](../.github/workflows/issue-ready-notification.yml#L10)과 [댓글 중복 확인 및 등록](../.github/workflows/issue-ready-notification.yml#L67).

## 주요 변경 사항

- [.github/workflows/issue-ready-notification.yml](../.github/workflows/issue-ready-notification.yml)을 추가했다. Issue 본문의 `### 필수 선행 이슈`에서 같은 저장소의 `- #번호` 항목을 읽는다.

## 검증 결과

- macOS에서 Ruby YAML로 트리거, 권한, 큐와 완료 조건의 문법을 확인했다.
- 현재 GitHub Issue 구조를 읽어 온 API 응답의 복제본으로 Node.js 모의 검증 6개 묶음을 실행했다. #9 완료 시 #10·#11·#13·#16, #10·#11 완료 시 #14가 알림 대상이 됐다.
- 일부만 완료, 미계획 종료, 누락·잘못된 선행 관계, 담당자 미지정, 닫힌 후속 Issue, 다중 담당자, 재실행, 개별 댓글 실패 후 재시도를 검증했다.
- 실제 GitHub Actions 실행 및 담당자 알림 수신은 기본 브랜치 반영 후 새 완료 이벤트로 확인한다.

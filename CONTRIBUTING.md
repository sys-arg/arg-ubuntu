# 기여 가이드

이슈를 등록한 뒤 `dev`에서 작업 브랜치를 만들고, 변경 사항을 `dev` 대상 PR로 보냅니다. 릴리즈 PR은 최신 `main`에서 만든 `release/vX.Y.Z`를 `main`으로 보냅니다. 병합 전에 대상 브랜치에서 요구하는 리뷰와 검사를 완료합니다.

| 항목 | 형식 | 예시 |
| --- | --- | --- |
| 이슈 제목 | `[TYPE] 설명` | `[FEAT] 로그인 구현` |
| 작업 브랜치 | `type/이슈번호-설명` | `feature/12-login` |
| 릴리즈 브랜치 | `release/vX.Y.Z` | `release/v1.0.0` |
| 커밋 제목 | `type: 설명` | `feat: 로그인 구현` |
| 작업 PR 제목 | `type: 설명 (#이슈번호)` | `feat: 로그인 구현 (#12)` |
| 작업 PR 본문 | `Related to #이슈번호` | `Related to #12` |

커밋과 PR 유형은 `feat`, `fix`, `refactor`, `docs`, `style`, `test`, `chore`, `ci`, `build`, `perf`를 사용합니다. 이슈 제목의 유형은 대문자로 씁니다. 기존 Task 템플릿의 `[TASK]`도 사용할 수 있습니다. 기능 브랜치에는 `feature/`를 사용하고, 나머지 접두어는 커밋 유형과 같습니다. 브랜치 설명에는 영문 소문자와 숫자를 사용하고 단어 사이를 하이픈으로 연결합니다.

작업 브랜치, PR 제목, PR 본문에는 같은 저장소의 동일한 이슈 번호를 씁니다. `Related to #12`는 PR 템플릿의 `Related Issue` 섹션에 한 줄로 작성합니다. 이슈는 팀의 완료 기준에 따라 닫습니다.

## 자동 검사

- `Validate PR`은 `dev` 대상 PR에서 브랜치 이름, 새로 들어오는 커밋 제목, PR 제목과 본문의 이슈 연결을 검사합니다. 연결한 이슈의 존재 여부와 제목 형식도 확인합니다.
- `Validate Issue`는 이슈를 열거나 제목을 수정할 때 제목을 검사합니다. 기본 브랜치에 워크플로가 반영되면 이슈 이벤트에 반응합니다.
- `Validate submission`은 `main` 대상 PR의 전체 병합 후보에서 릴리즈 브랜치 형식과 개발 테스트 경로를 검사합니다. 제품 파일이 있는 후보는 Ubuntu 24.04에서 `make clean && make`와 `./arg --help`를 확인합니다.

## 릴리즈

1. Ubuntu 24.04의 깨끗한 `dev` 작업 디렉터리에서 `make clean && make && make -f tests/Makefile test`를 실행하고 검증한 커밋 SHA를 기록합니다. `main`에 먼저 들어간 제품 수정이 있다면 `dev`에 반영한 뒤 검증합니다.
2. 깨끗한 작업 디렉터리에서 최신 `origin/main`을 기준으로 릴리즈 브랜치를 만듭니다. 아래 SHA와 버전을 실제 값으로 바꿉니다. 검증한 `dev` 커밋에서 제출 파일의 추가·수정·삭제를 가져옵니다.

   ```bash
   (
     set -e
     git fetch origin
     dev_commit=검증을-마친-dev-커밋-SHA
     git switch -c release/v1.0.0 origin/main
     git restore --source="$dev_commit" --staged --worktree -- . \
       ':(top,exclude)tests' ':(top,exclude).github/workflows/dev-tests.yml'
     git rm -r --ignore-unmatch -- tests .github/workflows/dev-tests.yml
     git diff --cached --check
     git diff --cached --stat
     git commit -m "chore: v1.0.0 릴리즈 준비"
     git push -u origin release/v1.0.0
   )
   ```

3. `main`으로 PR을 열고 기준 `dev` SHA와 개발 테스트 결과를 남깁니다. `Validate submission`과 리뷰 결과를 확인합니다. 제출용 작업 디렉터리에서 Ubuntu 일반 사용자로 `./arg`를 실행해 예상 초기 화면을 확인하고 결과를 PR에 기록합니다. 승인 후 Merge commit으로 병합합니다.
4. 실제 머지된 `main` 커밋을 대상으로 GitHub에서 `vX.Y.Z` 태그와 Release를 수동으로 생성합니다. 해당 태그의 소스 아카이브는 제출 파일을 담습니다.

첫 릴리즈 PR에 검사 결과가 생성되면 `main` 보호 규칙에 `Validate submission`을 필수 상태 검사로 등록합니다.

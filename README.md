# Gerrit → Jira Comment (Chrome Extension)

Chrome Extension (Manifest V3)으로, Gerrit change 페이지에서 툴바 버튼 한 번으로 Jira 이슈에 코멘트를 자동 추가합니다.

---

## 기능

| 기능 | 설명 |
|------|------|
| 원클릭 댓글 | Gerrit change 페이지에서 확장프로그램 아이콘 클릭 → Jira 이슈에 ADF 형식 댓글 자동 추가 |
| 이슈키 자동 추출 | 커밋 메시지의 `jira: TF-123` 태그 우선, 없으면 제목에서 bare key 탐색 |
| 커스텀 댓글 템플릿 | 8개 플레이스홀더로 댓글 형식을 자유롭게 설정, 미설정 시 기본값 사용 |
| 연결 테스트 | 옵션 페이지에서 Jira 인증을 미리 확인 가능 |
| 토스트 알림 | 성공/실패 결과를 페이지 우상단에 4.5초간 표시 |

---

## 설치 방법

> Chrome Web Store에 등록되지 않은 로컬 설치(개발자 모드)입니다.

1. 이 저장소를 클론하거나 ZIP으로 다운로드합니다.
2. Chrome 주소창에 `chrome://extensions` 를 입력합니다.
3. 우상단 **개발자 모드** 토글을 켭니다.
4. **압축해제된 확장프로그램을 로드합니다** 클릭 → 이 폴더(`gerrit-jira-automation/`)를 선택합니다.
5. 확장프로그램 아이콘이 툴바에 표시됩니다.

---

## 초기 설정

### 1. Jira 인증 설정

1. 툴바의 확장프로그램 아이콘을 **우클릭** → **옵션** 을 선택합니다.
   (또는 `chrome://extensions` → 이 확장프로그램의 **세부정보** → **확장 프로그램 옵션**)
2. **Jira 이메일**과 **Jira API 토큰**을 입력합니다.
   - API 토큰 발급: [https://id.atlassian.com/manage-profile/security/api-tokens](https://id.atlassian.com/manage-profile/security/api-tokens)
3. **저장** 클릭.
4. (권장) **연결 테스트** 클릭 → `연결 성공 (200 OK)` 확인.

### 2. 댓글 템플릿 설정 (선택)

옵션 페이지 하단 **Jira 댓글 템플릿** 섹션에서 Jira에 남길 댓글 형식을 자유롭게 작성할 수 있습니다.

**사용 가능한 플레이스홀더:**

| 플레이스홀더 | 치환 내용 | 출처 |
|---|---|---|
| `{title}` | 커밋 제목 (첫 번째 줄) | DOM |
| `{body}` | 커밋 메시지 본문 (`jira:` 라인 제거) | DOM |
| `{branch}` | 타겟 브랜치명 | DOM |
| `{change_num}` | Gerrit change 번호 (예: `12345`) | URL |
| `{project}` | 저장소 경로 (예: `platform/myapp`) | URL |
| `{owner}` | change 작성자 이름 | DOM |
| `{date}` | 반영 일시 (`YYYY-MM-DD HH:MM`) | JS |
| `{url}` | Gerrit change 링크 (Jira에서 클릭 가능) | URL |

> URL 출처(`{change_num}`, `{project}`, `{url}`)는 페이지 URL을 파싱하므로 항상 정확합니다.
> DOM 출처는 Gerrit 버전에 따라 빈 값이 될 수 있습니다.

**기본 템플릿** (미설정 시 자동 적용):

```
{title}

{body}

브랜치: {branch}
반영 일시: {date}
Gerrit: {url}
```

**커스텀 예시:**

```
[{project}#{change_num}] {title}

{body}

작성자: {owner} | 브랜치: {branch}
반영 일시: {date}
Gerrit: {url}
```

**초기화:** 템플릿을 비우거나 **기본값으로 초기화** 버튼을 누르면 기본 템플릿이 적용됩니다.

---

## 사용 방법

1. Gerrit change 상세 페이지로 이동합니다.
   예: `http://gerrit.thinkfree.com/c/myproject/+/12345`
2. 툴바의 **Gerrit → Jira Comment** 아이콘을 클릭합니다.
3. 이슈키가 감지되면 설정된 템플릿으로 Jira에 댓글이 추가되고, 우상단 토스트로 결과가 표시됩니다.

---

## 이슈키 추출 규칙

| 순위 | 소스 | 예시 |
|------|------|------|
| **1** | 커밋 메시지의 `jira:` 태그 (명시적·권장) | `jira: TF-123` |
| **2** | change 제목의 bare key | `[TF-123] Fix bug`, `TF-123: Fix bug` |
| — | 없으면 | warn 토스트 표시 후 종료 |

> 제목보다 커밋 메시지 태그를 우선하는 이유: 제목에 `[OOXML]` 같은 비-이슈 태그가 포함된 경우 오탐을 방지하기 위함입니다.

---

## 토스트 메시지 유형

| 상황 | 타입 | 메시지 |
|------|------|--------|
| 성공 | 녹색 | `Jira 댓글 추가 완료: TF-123` |
| 이슈키 미발견 | 주황 | `이슈 키를 찾지 못했습니다 (예: TF-123)...` |
| 비 Gerrit 탭 | 주황 | `Gerrit change 페이지에서만 사용할 수 있습니다.` |
| 인증 미설정 | 빨강 | `Jira 이메일/토큰이 설정되지 않았습니다...` |
| 401 인증 실패 | 빨강 | `인증 실패 (401): 이메일 또는 API 토큰을 확인하세요.` |
| 403 권한 없음 | 빨강 | `권한 없음 (403): 해당 이슈에 코멘트 권한이 없습니다.` |
| 404 이슈 없음 | 빨강 | `이슈를 찾을 수 없음 (404): 이슈 키를 확인하세요.` |
| 네트워크 오류 | 빨강 | `네트워크 오류가 발생했습니다.` |

---

## 보안 설계

| 항목 | 구현 |
|------|------|
| 토큰 저장 | `chrome.storage.local` 전용 — `chrome.storage.sync` 미사용 |
| 로그 노출 | 토큰·이메일·Authorization 헤더 등 민감 정보를 `console.log` 포함 어떤 로그에도 출력하지 않음 |
| 네트워크 요청 | 모든 Jira fetch는 background service worker에서만 수행 — content script는 DOM 읽기 전용 |
| host_permissions | `http://gerrit.thinkfree.com/*`, `https://gerrit.thinkfree.com/*`, `https://thinkfree.atlassian.net/*` 최소 고정 |
| 이슈키 검증 | service worker에서 정규식 allowlist로 재검증 후 URL 경로에 삽입 (path-traversal 방지) |
| 변경 URL 검증 | content script가 반환한 URL도 service worker에서 Gerrit 도메인 allowlist 재검증 |
| 응답 본문 | Jira API 응답 본문은 읽지 않음 — HTTP 상태 코드만 사용 |
| 토큰 입력 UI | `type="password"` 필드 사용 |

---

## 아키텍처

```
manifest.json          MV3 설정, 최소 권한
service_worker.js      chrome.action.onClicked 처리
                       템플릿 렌더링 + ADF 변환
                       Jira API fetch (POST /comment, GET /myself)
                       chrome.storage.local 읽기
content_script.js      Gerrit DOM 파싱 (subject, issueKey, branch, owner,
                         commitBody, changeNum, project)
                       Shadow DOM 탐색 (queryShadow / queryShadowAll)
                       토스트 표시
options.html / .js     이메일·토큰 저장 UI
                       댓글 템플릿 편집 + 플레이스홀더 레퍼런스
                       연결 테스트 (service worker 경유)
```

### 메시지 흐름

```
[action click] ──► service_worker
                        │
                        ├─► EXTRACT_INFO ──► content_script (DOM 추출)
                        │       ◄─────────── { subject, issueKey, url,
                        │                     branch, body, changeNum,
                        │                     project, owner }
                        │
                        ├─► chrome.storage.local
                        │     (jiraEmail, jiraToken, commentTemplate)
                        │
                        ├─► renderTemplate() → textToAdf()
                        │
                        ├─► fetch POST /rest/api/3/issue/{key}/comment
                        │
                        └─► SHOW_TOAST ──► content_script (토스트 표시)

[options page] ──► chrome.runtime.sendMessage TEST_CONNECTION
                        │
                        └─► service_worker: fetch GET /rest/api/3/myself
                                ◄─── { status } (body discarded)
```

---

## 지원 환경

- Gerrit: `http://gerrit.thinkfree.com/*` (HTTPS도 지원)
- Jira Cloud: `https://thinkfree.atlassian.net`
- Chrome 109+ (MV3 service worker 안정)

---

## 라이선스

Internal use only.

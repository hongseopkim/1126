# 1126 · SCK Work Guide

개인별 행사 업무를 시간순으로 안내하고, Google 스프레드시트 전체 탭을 검색하는 모바일 우선 웹앱입니다.

## 핵심 동작

- Google 계정 OAuth 로그인 후 사용자 계정의 스프레드시트 읽기 권한으로 직접 연결
- `전체시나리오 및 업무분장` 탭에서 담당자 이름을 자동 추출
- 현재 시각과 행사일(2026-11-26)을 기준으로 현재/다음/완료 업무 판정
- A안(현재 업무 집중형)과 B안(전체 타임라인형) 전환
- 이전/다음 업무 이동, 장소 및 원본 주의사항 표시
- 모든 스프레드시트 탭 검색: 내빈, 주차, 물품, 행사장 셋팅, 업체, 케이터링 등
- 연락처/전화번호/이메일 열과 패턴은 검색 결과에서 제외 또는 마스킹
- 앱을 열어 둔 동안 60초마다 자동 새로고침, 수동 새로고침 제공
- 모바일 홈 화면 추가를 위한 PWA manifest 및 정적 리소스 캐시

## 개인정보 및 보안 설계

- 이 저장소에는 행사 참석자 명단, 연락처, 이메일, 업무분장 데이터가 포함되지 않습니다.
- 앱은 공개 정적 파일만 GitHub Pages에서 제공합니다.
- 실제 시트 데이터는 사용자가 Google OAuth로 인증한 뒤 Google Sheets API에서 직접 읽습니다.
- Google OAuth 토큰과 스프레드시트 데이터는 브라우저 메모리에서만 사용하며 앱 서버/DB에 저장하지 않습니다.
- 허용 이메일 도메인은 `config.js`의 `allowedEmailDomains`에서 제한합니다.
- Google Sheets API scope는 `spreadsheets.readonly`이며 원본 수정 기능은 없습니다.
- 연락처와 이메일은 기본 검색 결과에 표시하지 않습니다.

## 최초 배포 설정

### 1. Google Cloud 설정

1. Google Cloud Console에서 프로젝트를 만들거나 기존 프로젝트를 선택합니다.
2. **APIs & Services → Library**에서 **Google Sheets API**를 활성화합니다.
3. **OAuth consent screen**을 설정합니다. Google Workspace 조직에서만 사용할 경우 `Internal` 유형을 우선 검토합니다. 외부 유형을 쓰는 경우 테스트 사용자에 사용할 회사 계정을 추가해야 할 수 있습니다.
4. **Credentials → Create Credentials → OAuth client ID → Web application**을 선택합니다.
5. Authorized JavaScript origins에 아래 주소를 추가합니다.

   `https://hongseopkim.github.io`

6. 발급된 OAuth 2.0 Client ID를 복사합니다. Client Secret은 필요하지 않으며, 절대 저장소에 올리지 않습니다.

### 2. 앱 설정

`config.js`에서 아래 값을 설정합니다.

```js
window.SCK_1126_CONFIG = {
  googleClientId: "발급받은_WEB_CLIENT_ID.apps.googleusercontent.com",
  spreadsheetId: "1R1kg4K9uqjdAfAiRJ9lmeK-EJAHzI4keIXkYD4vVnTQ",
  eventDate: "2026-11-26",
  eventDateLabel: "2026년 11월 26일(목)",
  timezone: "Asia/Seoul",
  allowedEmailDomains: ["sc.or.kr"],
  autoRefreshMs: 60000
};
```

`googleClientId`는 브라우저용 공개 식별자입니다. 비밀키나 Client Secret을 `config.js`에 입력하지 마세요.

### 3. 스프레드시트 권한

앱을 이용할 각 담당자의 Google 계정에 원본 스프레드시트의 최소 읽기 권한(뷰어)을 부여해야 합니다. 사용자는 본인에게 Google Sheets에서 허용된 범위의 데이터만 읽을 수 있습니다.

### 4. GitHub Pages 활성화

1. GitHub 저장소 `hongseopkim/1126`의 **Settings → Pages**로 이동합니다.
2. Build and deployment의 Source를 **GitHub Actions**로 설정합니다.
3. `main` 브랜치에 변경사항이 올라오면 `.github/workflows/pages.yml`이 정적 파일을 배포합니다.
4. 배포 주소는 `https://hongseopkim.github.io/1126/`입니다.

배포 후 Google OAuth Client ID의 Authorized JavaScript origins는 위의 `https://hongseopkim.github.io`와 일치해야 합니다. OAuth 동의 화면의 게시 상태/조직 정책에 따라 Workspace 관리자 승인이 필요할 수 있습니다.

## 데이터 갱신 방식

- 앱 최초 로그인 시 시트 메타데이터와 각 탭의 표시값을 읽습니다.
- 앱이 열린 동안 기본 60초마다 최신 데이터를 다시 읽습니다.
- 사용자는 새로고침 버튼으로 즉시 갱신할 수 있습니다.
- Google Sheets API 호출 권한 또는 네트워크가 끊기면 마지막으로 불러온 화면을 유지하고 오류 상태를 표시합니다.

이 방식은 Google Sheets를 원본으로 유지하는 주기적 실시간 동기화입니다. Google Sheets 변경 이벤트를 즉시 푸시하는 방식은 아니며, 자동 갱신 주기(기본 60초)와 수동 새로고침으로 최신 상태를 반영합니다.

## 개발 메모

- 순수 HTML/CSS/JavaScript 정적 앱이며 프레임워크/빌드 단계가 없습니다.
- 외부 백엔드 DB를 두지 않아 참석자 데이터의 별도 복제본을 만들지 않습니다.
- Google OAuth Client ID가 설정되지 않은 상태에서는 로그인 버튼이 비활성화됩니다.
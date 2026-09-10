# [PRD] 사내 실시간 메신저 프로토타입 웹 애플리케이션

## 1. 프로젝트 개요 (Overview)
- **프로젝트명**: 사내 실시간 메신저 웹 애플리케이션 (프로토타입)
- **목적**: 사내 구성원 간 실시간 커뮤니케이션을 위한 메신저 인터페이스 및 핵심 흐름(로그인, 채팅, 탭 이동) 검증
- **기술 제약 조건**:
  - **단일 파일 구현**: `index.html` 단 하나의 파일 안에 HTML, CSS(`<style>`), JavaScript(`<script>`)를 모두 포함.
  - **무설치 / 노빌드**: 외부 라이브러리 빌드 도구(Webpack, Vite, npm 등) 없이 브라우저에서 더블 클릭만으로 즉시 실행 가능해야 함.
  - **백엔드 서버 없음**: 현재 단계에서는 외부 서버 통신 없이 순수 프론트엔드 JavaScript(In-Memory 상태)로 모든 동작을 시뮬레이션함.
  - **미래 확장성 (TODO 주석)**: 추후 Supabase 및 Google OAuth 연동 시 교체해야 할 핵심 코드 지점마다 `// TODO: Supabase 연동` 주석을 명시함.

---

## 2. 디자인 시스템 및 컬러 팔레트 (Design System)

| 구분 | 역할 | 추천 HEX 코드 | 설명 |
| :--- | :--- | :--- | :--- |
| **Primary Navy** | 사이드바 배경 | `#0F172A` (Slate 900) | 깊고 안정감 있는 다크 남색 |
| **Secondary Navy** | 사이드바 메뉴 활성/호버 | `#1E293B` (Slate 800) | 선택된 메뉴 하이라이트 배경색 |
| **Accent Blue** | 포인트 컬러 & 내 말풍선 | `#2563EB` (Blue 600) | 브랜드 포인트, 전송 버튼, 본인 메시지 말풍선 |
| **Accent Blue Hover** | 버튼 마우스 호버 | `#1D4ED8` (Blue 700) | 클릭 인터랙션 피드백 색상 |
| **Background** | 메인 콘텐츠 배경 | `#F8FAFC` (Slate 50) | 눈이 편안한 밝은 오프화이트 톤 |
| **Surface/Card** | 카드 및 헤더/입력창 배경 | `#FFFFFF` (White) | 순백색 패널 배경 |
| **Border** | 경계선 | `#E2E8F0` (Slate 200) | 헤더 하단, 입력창 테두리, 디바이더 |
| **Opponent Bubble** | 상대방 말풍선 | `#F1F5F9` (Slate 100) | 은은한 연회색 말풍선 |
| **Text Primary** | 메인 텍스트 | `#0F172A` (Slate 900) | 높은 가독성의 본문 색상 |
| **Text Muted** | 보조 텍스트/타임스탬프 | `#64748B` (Slate 500) | 시간, 부가 설명 텍스트 |

- **기본 폰트**: `-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, "Noto Sans KR", sans-serif`
- **UI 모서리(Border Radius)**:
  - 버튼 및 입력 필드: `8px`
  - 카드 패널: `12px`
  - 말풍선: `16px` (내 말풍선은 오른쪽 아래, 상대 말풍선은 왼쪽 아래 꼬리 처리)

---

## 3. 화면 구조 및 레이아웃 명세 (Layout & Wireframe)

화면은 크게 **1) 로그인 전 화면(Login View)** 과 **2) 로그인 후 메인 대시보드 화면(Main App View)** 2단계로 전환된다.

```
[전체 화면: 100vw x 100vh, overflow: hidden]
  ├── [화면 1] 로그인 전 뷰 (#view-login)
  │     └── 중앙 로그인 카드 (#login-card)
  │           ├── 로고/서비스명
  │           └── [Google 계정으로 로그인] 버튼
  │
  └── [화면 2] 로그인 후 메인 뷰 (#view-main)
        ├── 좌측 사이드바 (너비: 240px 고정) (#sidebar)
        │     ├── 서비스 타이틀 / 워크스페이스 명
        │     └── 네비게이션 메뉴 (💬 사내 채팅방 / 📚 게시판)
        │
        └── 우측 메인 작업 영역 (flex: 1) (#main-content)
              ├── 상단 글로벌 헤더 (높이: 60px) (#header)
              │     ├── 현재 메뉴 타이틀 (#current-menu-title)
              │     └── 사용자 정보 & [로그아웃] 버튼
              │
              └── 콘텐츠 뷰 영역 (#content-area)
                    ├── [채팅 탭 선택 시] (#tab-chat)
                    │     ├── 채팅 메시지 스크롤 영역 (#chat-messages)
                    │     └── 하단 메시지 입력창 바 (#chat-input-bar)
                    │
                    └── [게시판 탭 선택 시] (#tab-board)
                          └── "준비 중입니다" 안내 영역
```

---

## 4. 기능별 상세 요구사항 (Detailed Functional Specs)

### 4.1. 로그인 전 화면 (`#view-login`)
- **배치**: 화면 정중앙(`display: flex; justify-content: center; align-items: center; min-height: 100vh; background: #F1F5F9`)에 카드 형태 배치.
- **로그인 카드 (`#login-card`)**:
  - 흰색 배경, 테두리(`1px solid #E2E8F0`), 부드러운 그림자(`box-shadow: 0 10px 25px rgba(0,0,0,0.05)`), 패딩 `40px`, 너비 `360px`.
  - 메신저 타이틀: "🏢 사내 메신저" (볼드체, 22px).
  - 안내 문구: "사내 구성원 계정으로 간편하게 시작하세요." (14px, `#64748B`).
  - **"Google 계정으로 로그인" 버튼**:
    - 스타일: 테두리(`1px solid #CBD5E1`), 흰색 배경, 호버 시 연회색(`#F8FAFC`), 구글 심볼(G 로고 SVG 또는 이모지) + "Google 계정으로 로그인" 텍스트.
    - **클릭 인터랙션**:
      1. 클릭 시 실제 OAuth 창을 띄우지 않고, 임시 사용자 데이터(`{ id: 'user_guest', name: '게스트', email: 'guest@company.com', avatar: '👤' }`)를 JS 세션 변수에 저장.
      2. `#view-login` 요소를 숨기고(`display: none`), `#view-main` 요소를 화면에 노출(`display: flex`).
      3. 코드 위치에 `// TODO: Supabase 연동 - Google OAuth 로그인 (supabase.auth.signInWithOAuth)` 주석 기재.

---

### 4.2. 로그인 후 화면 - 좌측 사이드바 (`#sidebar`)
- **크기 및 배경**: 너비 `240px` 고정, 세로 `100vh`, 배경색 `#0F172A`, 텍스트 흰색 계열.
- **헤더부**:
  - 앱 이름: "🚀 WorkChat" 또는 "💬 사내 메신저" (볼드, 18px, 좌측 상단 로고).
  - 테두리선: 하단 `1px solid #334155`.
- **메뉴 목록 (`ul.nav-menu`)**:
  - 메뉴 1: `💬 사내 채팅방` (기본 선택 상태)
  - 메뉴 2: `📚 게시판`
  - **인터랙션**:
    - 기본 상태: 텍스트 색상 `#94A3B8`, 패딩 `12px 16px`, 모서리 `8px`, 커서 `pointer`.
    - 마우스 호버 시: 배경색 `#1E293B`, 텍스트 `#F8FAFC`.
    - 활성화(Active) 상태: 배경색 `#2563EB` (또는 `#1E293B`에 좌측 포인트 바), 텍스트 `#FFFFFF`, 볼드.
    - 메뉴 클릭 시:
      - 선택된 메뉴로 Active 클래스 이동.
      - 헤더의 메뉴 제목 변경.
      - "사내 채팅방" 클릭 시 채팅 뷰 표시, "게시판" 클릭 시 "📚 게시판 기능은 현재 준비 중입니다." 안내 화면 표시.

---

### 4.3. 로그인 후 화면 - 상단 헤더 (`#header`)
- **크기 및 배치**: 높이 `60px`, 너비 100%, 배경색 `#FFFFFF`, 하단 테두리 `1px solid #E2E8F0`, 양쪽 여백 `24px`, 플렉스 정렬(`space-between`, 세로 중앙).
- **좌측 영역**:
  - 현재 활성화된 메뉴명 표시 (예: "💬 사내 채팅방" / 18px, 굵은 폰트).
- **우측 영역**:
  - 사용자 프로필: 프로필 아바타 원형 아이콘 + "게스트 님" 텍스트.
  - **[로그아웃] 버튼**:
    - 작고 깔끔한 아웃라인 버튼 (배경 투명, 테두리 `1px solid #CBD5E1`, 글자색 `#64748B`, 호버 시 글자색 `#EF4444`, 테두리 `#FCA5A5`).
    - **클릭 인터랙션**:
      1. 클릭 시 현재 로그인 상태 초기화.
      2. `#view-main` 숨김, `#view-login` 노출.
      3. 코드 위치에 `// TODO: Supabase 연동 - 로그아웃 (supabase.auth.signOut)` 주석 기재.

---

### 4.4. 로그인 후 화면 - 채팅 영역 (`#tab-chat`)
채팅 탭은 세로 전체 flex 컨테이너(`flex-direction: column`)로 구성된다.

#### 1) 메시지 스크롤 영역 (`#chat-messages`)
- **레이아웃**: `flex: 1`, `overflow-y: auto`, 패딩 `20px 24px`, 배경색 `#F8FAFC`.
- **기본 더미 데이터 (페이지 로드 시 초기 2~3개 노출)**:
  1. 상대방 (김철수 팀장): "안녕하세요! 오늘 프로젝트 킥오프 회의는 2시입니다." (오전 10:15)
  2. 상대방 (이영희 디자이너): "네, 디자인 시안 준비해서 참석하겠습니다!" (오전 10:18)
  3. 본인 (게스트): "확인했습니다. 회의실에서 뵙겠습니다." (오전 10:20)
- **말풍선 UI 디자인 스펙**:
  - **상대방 메시지 (좌측 정렬)**:
    - 구조: `[아바타] + [이름 / 시간] + [말풍선]`
    - 말풍선 색상: 배경 `#FFFFFF`, 테두리 `1px solid #E2E8F0`, 텍스트 `#0F172A`.
    - 패딩: `10px 14px`, 둥근 모서리 `14px 14px 14px 2px`.
  - **내 메시지 (우측 정렬)**:
    - 구조: `[말풍선] + [시간]` (아바타 생략 또는 우측 표시)
    - 말풍선 색상: 배경 `#2563EB`, 텍스트 `#FFFFFF`.
    - 패딩: `10px 14px`, 둥근 모서리 `14px 14px 2px 14px`.
  - **시간 표시**: `font-size: 11px`, 색상 `#94A3B8`.

#### 2) 하단 메시지 입력창 바 (`#chat-input-bar`)
- **레이아웃**: 높이 약 `72px`, 배경색 `#FFFFFF`, 상단 테두리 `1px solid #E2E8F0`, 패딩 `16px 24px`, 플렉스 가로 배치(`gap: 12px`, 세로 중앙).
- **입력 필드 (`#input-message`)**:
  - `type="text"`, `placeholder="메시지를 입력하세요..."`.
  - `flex: 1`, 높이 `44px`, 패딩 `0 16px`, 모서리 `8px`, 테두리 `1px solid #CBD5E1`.
  - 포커스 시: 테두리 `#2563EB`, 아웃라인 없음, 은은한 블루 글로우(`box-shadow: 0 0 0 3px rgba(37,99,235,0.1)`).
- **전송 버튼 (`#btn-send`)**:
  - 배경색 `#2563EB`, 글자색 흰색, 높이 `44px`, 패딩 `0 20px`, 모서리 `8px`, 볼드체 `14px`, 커서 `pointer`.
  - 호버 시: 배경색 `#1D4ED8`.
  - 비활성화(disabled): 빈 텍스트일 때 버튼 투명도 `0.6` 또는 비활성화 스타일.
- **전송 인터랙션 동작**:
  1. 전송 버튼 클릭 또는 입력창에서 `Enter` 키 입력 시 동작 (단, 공백/빈 문자열은 전송 차단).
  2. 현재 시간(예: "오후 02:30") 포맷 계산.
  3. 새 메시지 객체(`{ id, sender: '게스트', isMe: true, text, time }`)를 JS 내부 배열에 추가.
  4. 채팅 메시지 리스트 DOM에 즉시 렌더링.
  5. 입력창 내용 비우기 및 포커스 유지.
  6. **스크롤 자동 이동**: 새 메시지가 추가되면 `#chat-messages` 영역이 항상 맨 아래로 부드럽게 스크롤됨(`scrollTop = scrollHeight`).
  7. 코드 위치에 `// TODO: Supabase 연동 - 실시간 DB insert 및 Broadcast (supabase.from('messages').insert)` 주석 기재.

---

### 4.5. 로그인 후 화면 - 게시판 영역 (`#tab-board`)
- 메뉴에서 "📚 게시판" 선택 시 화면 중앙에 아이콘과 함께 "📚 게시판 기능은 준비 중입니다." 문구 표시.
- 깔끔한 빈 화면(Empty State) 디자인 적용.

---

## 5. 자바스크립트 상태 관리 및 구조 설계 (State & Architecture)

단일 파일 내 스크립트 구조는 가독성과 향후 Supabase 교체가 용이하도록 모듈화된 객체 형태로 작성한다.

### 5.1. 인메모리 상태 (State)
```javascript
const state = {
  currentUser: null, // { id: 'guest', name: '게스트', email: 'guest@company.com' }
  currentTab: 'chat', // 'chat' | 'board'
  messages: [
    {
      id: 1,
      sender: '김철수 팀장',
      isMe: false,
      text: '안녕하세요! 오늘 프로젝트 킥오프 회의는 2시입니다.',
      time: '오전 10:15'
    },
    {
      id: 2,
      sender: '이영희 디자이너',
      isMe: false,
      text: '네, 디자인 시안 준비해서 참석하겠습니다!',
      time: '오전 10:18'
    },
    {
      id: 3,
      sender: '게스트',
      isMe: true,
      text: '확인했습니다. 회의실에서 뵙겠습니다.',
      time: '오전 10:20'
    }
  ]
};
```

### 5.2. Supabase 연동 대비 TODO 주석 위치 정의
개발자가 나중에 Supabase SDK 스크립트(`<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js"></script>`)를 추가하고 바로 치환할 수 있도록 아래 위치에 명확한 주석을 배치한다.

1. **초기화 부**:
   ```javascript
   // TODO: Supabase 연동 - 클라이언트 초기화
   // const supabase = supabase.createClient('SUPABASE_URL', 'SUPABASE_ANON_KEY');
   ```
2. **로그인 처리 함수 (`handleGoogleLogin`)**:
   ```javascript
   // TODO: Supabase 연동 - Google OAuth 로그인
   // await supabase.auth.signInWithOAuth({ provider: 'google' });
   ```
3. **로그아웃 처리 함수 (`handleLogout`)**:
   ```javascript
   // TODO: Supabase 연동 - 로그아웃
   // await supabase.auth.signOut();
   ```
4. **메시지 불러오기 함수 (`loadMessages`)**:
   ```javascript
   // TODO: Supabase 연동 - 이전 대화 내역 조회
   // const { data } = await supabase.from('messages').select('*').order('created_at', { ascending: true });
   ```
5. **실시간 메시지 구독 함수 (`subscribeRealtime`)**:
   ```javascript
   // TODO: Supabase 연동 - Realtime 채널 구독
   // supabase.channel('room-1').on('postgres_changes', { event: 'INSERT', schema: 'public', table: 'messages' }, payload => { ... }).subscribe();
   ```
6. **메시지 전송 함수 (`sendMessage`)**:
   ```javascript
   // TODO: Supabase 연동 - 신규 메시지 INSERT
   // await supabase.from('messages').insert([{ text: text, user_id: state.currentUser.id }]);
   ```

---

## 6. 품질 및 구현 체크리스트 (Acceptance Criteria)

- [ ] **단일 파일 완성도**: `index.html` 파일을 브라우저로 열었을 때 어떤 외부 런타임 오류 없이 즉시 렌더링되는가?
- [ ] **로그인 화면 및 전환**:
  - 첫 로딩 시 중앙에 정돈된 로그인 카드가 표시되는가?
  - "Google 계정으로 로그인" 클릭 시 부드럽게 메인 메신저 화면으로 전환되는가?
- [ ] **사이드바 레이아웃**:
  - 좌측 사이드바 너비가 `240px`이며 어두운 남색(`#0F172A`)으로 깔끔히 고정되어 있는가?
  - 메뉴(채팅방 / 게시판) 클릭 시 활성화 상태 표시와 우측 뷰 전환이 정상 동작하는가?
- [ ] **헤더 및 로그아웃**:
  - 로그인한 사용자 "게스트"의 이름이 헤더에 나타나는가?
  - [로그아웃] 클릭 시 즉시 로그인 전 카드로 돌아가는가?
- [ ] **채팅 기능**:
  - 기본 더미 메시지가 내 메시지(우측 파랑), 상대 메시지(좌측 연회색)로 정확히 구분되어 표시되는가?
  - 새 메시지를 입력하고 전송(클릭 또는 Enter)했을 때 즉시 우측 말풍선으로 추가되는가?
  - 새 메시지 등록 후 입력창이 비워지고 스크롤이 자동으로 하단으로 이동하는가?
- [ ] **TODO 주석 충실도**:
  - PRD에 지정된 6개 이상의 `// TODO: Supabase 연동` 주석이 정확한 위치에 빠짐없이 기재되어 있는가?

# shared-calendar
일정 공유 캘린더

---

# 1️⃣ MVP 1 — 달력 & 위젯 집중(단일 사용자)

### 요구기능

* 일정 **CRUD**: 제목, **날짜 및 기간**(`start_at`~`end_at`·`end_at` exclusive 권장), 종일 여부, 위치, 메모
* 태그 달기 + 태그별 조회
* 체크리스트(서브 일정) 완료 시 대표 일정 자동 완료(취소선)
* **홈 화면 위젯 v1(안드로이드)**: 월 보기 + “오늘 타임라인” 확인, **완료 토글**, “편집 열기” 딥링크
  *(위젯에서 직접 편집은 하지 않고, 앱으로 딥링크)*
* **공유 기능 없음**(오직 본인 캘린더)

### 개발 계획

* **DB(핵심 테이블)**
  `users`, `calendars(개인 1개)`, `events(start_at, end_at, is_all_day, completed_at)`,
  `sub_events`, `tags`, `event_tags`

  * 트리거: 모든 `sub_events` 완료 시 `events.completed_at` 자동 설정
* **동기화 흐름(간단)**

  * 앱 진입/사용자 새로고침 시 Supabase에서 Fetch → 로컬 영속 저장소에 반영
  * 위젯은 로컬 데이터만 읽고, 완료 토글은 인텐트 → 앱에서 저장 후 위젯 갱신
* **수용 기준(예)**

  * 위젯에서 완료 토글 시 UI 즉시 반영, 앱에서 1회 새로고침 버튼으로 서버 동기화
  * 월 보기에서 일정 점/뱃지, 오늘 타임라인에서 시간대별 목록 확인 가능

### 기술 스택

* **모바일(안드로이드)**: Kotlin, Jetpack Compose, **Glance(AppWidget)**, Room(로컬 영속), Hilt, Coroutines/Flow
* **백엔드**: **Supabase Free**(Postgres + Auth + PostgREST). RLS로 사용자 소유 데이터만 허용

---

# 2️⃣ MVP 2 — 공유 & 권한 + 반복/대시보드(기능 완성)

### 요구기능

* **공유/권한**: 초대 링크로 멤버 추가, **Owner/Editor/Viewer**

  * **태그 단위 권한 스코프**(읽기/수정/대리 생성·삭제를 태그별로 제한)
* **반복 일정(RRULE)** + 단일 인스턴스 예외 편집
* **대시보드 v1**: 반복 일정 **수행률(주/월)**·태그별 완료율
* (선택) **iOS 앱 + 위젯 v1**: 조회/완료 토글/편집 딥링크 동일 수준

### 개발 계획

* **DB 확장**

  * `memberships(role)`, `memberships.scope_json`(허용 태그·연산)
  * `events.rrule`, `event_overrides(override_date, fields_json)`로 예외 관리
  * RLS: `memberships` + `scope_json`을 이용해 **태그 교집합** 기준으로 읽기/쓰기 제한
* **초대/가입**

  * Edge Function으로 **서명된 초대 링크** 발급 → 수락 시 `memberships` 생성
* **대시보드**

  * Postgres **뷰/물질화 뷰**로 주·월 수행률 계산(크론으로 갱신)
* **클라이언트**

  * 공유 캘린더 전환, 태그 필터 유지
  * 충돌은 단순 **LWW(updated_at)**로 처리(실시간 동기화는 MVP3에서)

### 기술 스택

* **모바일**: (안드로이드) MVP1 동일 / (선택 iOS) SwiftUI + WidgetKit + App Intents
* **백엔드**: Supabase(Auth/RLS/PostgREST/Edge Functions/cron)

---

# 3️⃣ MVP 3 — 외부 캘린더 연동 + 최적화(캐싱·전력)

### 요구기능

* **외부 캘린더 연동**(선택적):

  * **Google / Outlook / Notion** 읽기 → 내부 태그/캘린더 매핑
  * (옵션) 양방향 동기화 + 충돌 규칙(외부 우선/내부 우선/필드별 머지)
* **최적화**

  * **위젯 노출용 프리컴퓨트/캐싱**: 월/오늘 스냅샷 테이블 생성, 위젯은 스냅샷만 조회
  * **변경 이벤트 기반 갱신**: FCM/APNs 데이터 메시지로 앱/위젯 업데이트 트리거
  * **배터리/네트워크 최적화**: 백그라운드 동기 주기 제어(WorkManager/BackgroundTasks), 저전력 모드 시 제한
  * 오프라인 강건성: 재시도 큐, 부분 실패 내구성, 인덱스/쿼리 최적화

### 개발 계획

* **커넥터/동기화**

  * Supabase **Edge Functions**로 각 외부 API 호출(증분 토큰/웹훅), 실패 시 재시도
  * 내부 ↔ 외부 매핑 테이블, 충돌 정책(필드 우선순위) 정의
* **캐싱/프리컴퓨트**

  * `calendar_snapshots_month`, `day_agenda_snapshot` 생성용 작업(크론/트리거)
  * 위젯/홈 화면은 스냅샷 조회 → 체감 성능·전력 절감
* **푸시**

  * DB 변경 시 Edge Function → **FCM/APNs** 발행 → 클라이언트 로컬 반영 후 위젯 갱신

### 기술 스택

* **모바일**: (안드로이드) WorkManager + FCM / (iOS) BackgroundTasks + APNs
* **백엔드**: Supabase(Edge Functions, Realtime 선택), 외부 API SDK, 스냅샷/인덱스 설계

---

## 한 줄 요약(릴리즈 순서)

* **MVP1**: 단일 사용자 달력+위젯 핵심 경험 완성
* **MVP2**: 공유/권한 + 반복/대시보드로 “제품형 캘린더” 완성
* **MVP3**: 외부 캘린더 연동 & 캐싱/전력 최적화로 완성도·확장성 확보


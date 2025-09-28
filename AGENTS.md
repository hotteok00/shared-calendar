# 개발완료

- 없음 (개발 미착수)

# 개발현황

- 상태: 0% (레포만 존재, 코드/설정 미작성)
- 현재 단계: 요구사항 정리 및 설계 준비
- 주의: 아래 구조/규칙을 기준으로 초기 스캐폴딩 후 착수 예정

# 개발예정

- MVP1(개인 기록·위젯 집중)
  - 기능: 일정 CRUD(제목, 날짜/기간, 종일, 위치, 메모), 태그 추가/조회, 체크리스트 기반 완료 처리
  - 위젯 v1(Android Glance): 오늘/내일 강조, 완료 토글(앱 반영 후 위젯 갱신)
  - 데이터: 로컬 우선(진입/사용 시 Fetch → 로컬 반영), 공유 기능 제외
  - 백엔드: Supabase(Postgres/Auth/PostgREST/RLS) 기본 스키마
- MVP2(공유·권한·반복·리포트)
  - 공유/권한: 초대 링크, 멤버 롤(Owner/Editor/Viewer), 태그/스코프 권한
  - 반복 일정: RRULE + 예외 편집, 리포트 v1(실행/완료율·태그별)
  - Edge Functions/cron 도입, (선택) iOS 앱+위젯 v1
- MVP3(외부 연동·캐싱 최적화)
  - Google/Outlook/Notion 연동, 스냅샷 캐싱(월/일 아젠다), 변동 이벤트 기반 갱신(FCM/APNs)
  - 배터리/인덱스/쿼리 최적화, 충돌 처리 LWW 고도화

# 디렉토리구조

모노레포

shared-calendar
- apps
  - android            # MVP1 시작점 (Compose + Glance 위젯)
  - ios                # MVP2부터 필요하면 생성 (선택적)
- backend
  - supabase
    - migrations       # 스키마/RLS/트리거
    - functions        # Edge Functions (MVP2: 초대링크, RRULE 확장)
    - seeds            # 개발용 더미데이터
    - config.toml
- .github/workflows    # CI (안드로이드 빌드 정도만 먼저)
- README.md

# 커밋규칙

- 형식: Conventional Commits
  - 타입: `feat`, `fix`, `refactor`, `perf`, `docs`, `test`, `build`, `ci`, `chore`, `style`, `revert`
  - 스코프 예시: `android`, `ios`, `supabase`, `migrations`, `functions`, `seeds`, `ci`, `docs`
  - 제목: 한글, 현재형·명령형, 50자 이내, 끝에 마침표 없음
  - 본문: 무엇/왜(배경, 설계 선택), 마이그레이션/호환성 영향, 관련 이슈 참조
  - 푸터: `BREAKING CHANGE: ...`, `Closes #123`
- 예시
  - `feat(android): Glance 위젯 v1 일정 목록 조회 추가`
  - `feat(supabase): sub_events 완료 시 events.completed_at 자동 반영 트리거 추가`
  - `fix(migrations): events.end_at not null 제약 복구`
  - `ci: Android 빌드 워크플로우 초안 추가`

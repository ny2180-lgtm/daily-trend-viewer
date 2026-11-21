# 데일리 트렌드 뷰어

유튜브 인기 동영상·쇼츠·인스타 릴스와 AI 영상 소식을 카테고리별 × 조회수순으로 매일 확인하는 로컬 웹사이트입니다.
API 키 없이 동작하며, 외부 패키지 설치도 필요 없습니다(파이썬 표준 라이브러리만 사용).

## 🌐 웹 버전 (설치 불필요)

**https://ny2180-lgtm.github.io/daily-trend-viewer/**

GitHub Actions가 매일 오전 7시(한국 시간, UTC 전날 22시)에 데이터를 수집해 GitHub Pages로 배포하는 스냅샷 버전입니다.
- 구성: `collect.py`(수집·정적 사이트 생성) + `static_shim.js`(API 호출을 JSON 파일로 변환) + `.github/workflows/pages.yml`(매일 실행)
- 실시간 기능(검색, 새로고침, 계정 추가/삭제, 좋아요순 정렬)은 웹 버전에서 비활성화됩니다
- 인스타그램 등 일부 소스는 GitHub 서버 IP를 차단해 웹 버전에서 비어 있을 수 있습니다 — 전체 데이터는 아래 로컬 실행 버전을 사용하세요

## 실행 방법

```bash
python3 ~/trend-viewer/server.py
```

브라우저에서 **http://localhost:8778** 접속

## 기능

- **유튜브 / 쇼츠 탭**: 카테고리(전체·뷰티·식품·K뷰티·뷰티인플루언서·UGC·콘텐츠 제작) × 기간(오늘/이번 주/이번 달) 필터, **접이식 정렬 메뉴로 조회수순/좋아요순 전환**(좋아요는 영상마다 서버가 조회해 보강), 카드 클릭 시 팝업 재생
- **🤖 AI 영상 탭**: 새로 나온 영상 생성 모델 + 트렌딩 모델(Hugging Face 실시간)과 국내·해외 AI 영상 뉴스(구글 뉴스 RSS)만 표시(글 중심)
- **🔍 검색창**(헤더): 유튜브를 임의 키워드로 검색해 조회수순으로 표시. 검색 시 영상 탭으로 전환되며, 카테고리 클릭으로 검색 해제
- **📸 릴스 탭**: 인스타그램 웹 내부 API(무인증)로 구독 계정들의 최신 릴스를 수집. **접이식 정렬 메뉴로 조회수순/좋아요순/댓글순 전환**(전부 실데이터, 즉시 재정렬). 계정은 화면에서 추가/삭제 (`reels_accounts.json`에 저장). 기본 계정: OpenAI·Runway·Pika·Luma·Midjourney·Kling·HeyGen·Higgsfield·DeepMind
- **𝕏 트위터 탭**: 트위터 임베드용 공개 API(무인증)로 구독 계정들의 최신 트윗을 참여수와 함께 수집. **접이식 정렬 메뉴**로 좋아요·댓글·리트윗·조회수순 전환. 계정 추가/삭제 가능(`x_accounts.json`). 기본 계정: OpenAI·Runway·Kling·DeepMind·Midjourney·Luma·Pika·HeyGen·ElevenLabs·Meta AI
- **🧵 스레드 탭**: 인스타그램(메타) 백엔드로 로그인 없이 실시간 조회를 시도. 메타가 조회를 막은 경우(현재 기본 상태) 각 계정 바로가기로 자동 폴백. 접이식 정렬 메뉴(좋아요·댓글·리포스트순) 제공, 계정 관리 가능(`threads_accounts.json`)
- **🎵 틱톡 탭**: tikwm 무료 공개 API로 **실시간 인기 피드(한국)+구독 계정 최신 영상**을 조회수·좋아요·댓글까지 수집. **접이식 정렬 메뉴로 조회수순/좋아요순/댓글순 전환**. 계정 추가/삭제(`tiktok_accounts.json`). 기본 계정: openai·runwayapp·krea.ai·elevenlabs·sora·zachking·khaby.lame·google
- **캐시**: 결과는 1시간 캐시되며, 우측 상단 새로고침 버튼으로 즉시 갱신 가능

## 참고 사항

- 유튜브 데이터는 내부 검색 API(InnerTube)를 조회수순 정렬로 호출해 가져옵니다. 유튜브의 공개 인기 급상승 피드는 2025년에 폐지되어 검색 기반으로 구성했습니다.
- 인스타 릴스: 인스타그램이 2026년부터 무인증 API를 차단(429)해 미러 사이트 **imginn.com**에서 목록·좋아요·댓글·캡션을 가져옵니다(조회수는 미러에 없어 좋아요순이 기본 정렬). 썸네일은 CDN 핫링크 차단을 피해 서버 프록시(`/api/img`)로 전달합니다.
- X(트위터): syndication 타임라인 API가 막혀 **nitter RSS**(트윗 목록, `NITTER_INSTANCES` 순서로 시도) + **트위터 임베드 CDN**(`cdn.syndication.twimg.com/tweet-result`, 트윗별 본문·좋아요·댓글·미디어) 조합으로 가져옵니다. 리트윗·조회수는 이 경로에 없어 0으로 둡니다.
- 스레드: GraphQL 무인증 조회(`THREADS_DOC_IDS`에 유효한 doc_id 필요)를 먼저 시도하고, 막히면 **RSSHub 공개 인스턴스**(`RSSHUB_INSTANCES`)로 최신 글(본문·이미지·링크)을 가져옵니다. RSS에는 통계가 없어 좋아요 등은 표시되지 않으며, 그마저 막히면 계정 바로가기로 폴백합니다.
- 웹 버전(collect.py)은 수집이 실패한 소스에 한해 **이전 배포 스냅샷을 재사용**하므로, 일시적 차단이 있어도 데이터가 사라지지 않습니다.
- 틱톡은 자체 웹/모바일 API가 서명(X-Bogus·msToken)과 TLS 지문 검사를 요구해 직접 무인증 호출이 막힙니다. 대신 **tikwm 무료 공개 API**(`www.tikwm.com/api`)가 서명을 대신 처리해 조회수·좋아요·댓글·썸네일·영상URL을 반환합니다. 무료 티어라 동시성이 높으면 일시 제한될 수 있어 서버는 낮은 동시성(3)으로 호출하고 1시간 캐시합니다. 썸네일(tiktokcdn)은 서버 이미지 프록시(`/api/img`, 메모리 캐시)로 전달합니다.

## 맞춤 검색 키워드

- **뷰티**: 메이크업 튜토리얼, 스킨케어 루틴, 화장품 리뷰, 올리브영 추천
- **식품**: 신상 식품 리뷰, 편의점 신상, 간편식 추천, 푸드 트렌드
- **K뷰티**: K beauty, Korean skincare, Korean makeup, K뷰티 트렌드
- **뷰티인플루언서**: 뷰티 유튜버 추천템, 인플루언서 메이크업, 겟레디윗미 메이크업, beauty influencer favorites
- **UGC**: 뷰티 UGC, UGC skincare, UGC product video, UGC creator tips
- **콘텐츠 제작**: 숏폼 제작, 제품 영상 촬영, 뷰티 콘텐츠 제작, 릴스 편집

## 등록한 릴스·틱톡 계정 표시

- `reels_accounts.json`과 `tiktok_accounts.json`은 배포에 포함합니다. Fork 저장소에도 두 파일을 커밋해야 같은 계정이 표시됩니다.
- 릴스 미러나 틱톡 API에서 게시물을 수집하지 못한 계정은 공식 프로필 임베드로 표시합니다. 인스타그램 프로필에는 릴스와 일반 게시물이 함께 보일 수 있습니다.
- 틱톡은 등록한 계정의 게시물만 수집합니다. 전체 인기 피드는 섞지 않습니다.
- 공식 임베드는 플랫폼이 제공하는 게시물 순서로 표시되며 사이트의 조회수·좋아요 정렬 대상이 아닙니다. 일부 영상은 원본 페이지에서 재생해야 합니다.
- 이전 배포 데이터를 재사용할 때도 현재 등록한 계정의 게시물만 남깁니다.

## 뷰티 광고 참고 계정 확장 (2026-11-21)

기존 계정을 유지하고 릴스 34개, 틱톡 27개 계정으로 확장했습니다. 광고 성과 순위가 아닌 콘텐츠 기획 참고용 선정입니다. 계정의 모든 게시물이 광고인 것은 아닙니다.

- K뷰티 브랜드: COSRX, Anua, SKIN1004, 조선미녀, 라운드랩, 토리든, 바이오던스, 메디큐브, 롬앤, 데이지크, 어뮤즈, 라네즈 등. 제품 사용법, 제형 표현, 패키지 연출과 크리에이터 협업을 참고합니다.
- 해외 브랜드: Rare Beauty, rhode, Fenty Beauty, e.l.f. 제품 시연, 메이크업 표현과 브랜드별 숏폼 연출을 참고합니다.
- 국내 크리에이터: 이사배, 포니, 레오제이, 민스코, 라뮤끄. 튜토리얼, 발색 비교와 제품 설명 방식을 참고합니다.
- 글로벌 크리에이터: Glowwithava, Ben Neiley, James Welsh, GlowbyRamon, NikkieTutorials, Meredith Duxbury, Sarah Cheung 등. 리뷰 구성, 도입부와 사용 과정을 참고합니다.
- UGC·협찬 콘텐츠 참고: Joy Burnett, Aisha Potter, Lavish Krish의 공개 브랜드 협업 사례를 참고합니다.
- 계정이 많아져 공식 미리보기는 한 번에 6개씩 표시합니다. 이전/다음 버튼 또는 계정 선택 메뉴로 이동할 수 있습니다.

아이디 확인 참고: [COSRX 공식 링크](https://linktr.ee/cosrx), [어뮤즈 공식 링크](https://linktr.ee/amusemakeup), [데이지크 공식 링크](https://linktr.ee/dasique), [롬앤 공식몰](https://romand.cafe24.com/), [Ben Neiley 본인 채널](https://www.youtube.com/channel/UCQdy_TEjOzpk_kAfSOjB0nw), [라네즈 공개 협업 사례](https://www.modash.io/content-library/brands/laneige-examples). 틱톡 추가 계정은 공식 공개 임베드의 아이디·이름·소개를 확인했고, 동명 빈 계정이나 뷰티와 무관한 계정은 제외했습니다. 인스타그램 계정은 공개 프로필과 공식 링크·협업 사례를 대조했습니다. 아이디와 임베드 제공 여부는 추후 변경될 수 있습니다.

# 레퍼런스

조사 범위 — 상용 서비스 7종(필메이커 · 데어무드 · 잇츠카드 · 보자기카드 · 달팽 · 메종드마리에 · 바른손),
오픈소스 자체 제작 사례 6종 + 개발 후기 2건, 가이드(청첩장 문구 · 송금 수수료 이슈 · 계좌 공유 보안).

**핵심 결론**

- 섹션 순서는 사실상 업계 표준으로 굳어져 있다. 새로 설계할 게 아니라 그대로 따르고, 순서만 config 배열로 바꿀 수 있게 한다.
- 백엔드가 필요한 건 방명록과 RSVP 둘뿐이다 → Google Apps Script + 스프레드시트. 나머지는 전부 정적.
- 지도 SDK는 P0에서 뺀다. 정적 이미지 + 딥링크 버튼이면 API 키도 번들도 없다. 하객은 지도를 조작하지 않는다.
- 갤러리 라이트박스는 직접 짜지 않는다. iOS 카카오톡 인앱브라우저 제스처 이슈를 통째로 떠안게 된다. photoswipe를 쓴다.
- 배경 흩날림 애니메이션은 canvas 하나 + 이미지 한 장, 약 140줄, 의존성 0으로 끝난다. `bgEffect` 옵션으로 on/off.

---

## 1. 표준 레이아웃 · 섹션 구성

상용 7종과 오픈소스 6종에서 공통으로 반복되는 세로 스크롤 한 장 구조.
순수 HTML/CSS/JS로 만든 오픈소스 사례의 섹션 구성이 상용 서비스와 거의 일치한다.

| # | 섹션 | 내용 | 비고 |
|---|---|---|---|
| — | BGM 플레이어 | 배경음악 재생/정지 | 우측 상단 고정. 파일 없으면 버튼 자동 숨김 |
| 1 | 인트로 / 커버 | 날짜 · 신랑신부 이름 · 메인 사진 | 오프닝 애니메이션 |
| 2 | 인사말 | 초대 문구 + 양가 가족 소개 | 연락하기 모달 진입점 |
| 3 | 러브레터 | 신랑신부가 쓴 편지 | 선택. 손글씨 폰트 / 스캔 이미지 |
| 4 | 갤러리 | 3열 그리드 + 더보기 | 라이트박스 · 스와이프 |
| 5 | 달력 | 예식일 하이라이트 | D-day 카운트다운 |
| 6 | 오시는 길 | 예식장 위치 | 지도앱 연동 버튼 · 주소 복사 · 교통편 |
| 7 | 마음 전하실 곳 | 축의금 계좌 | 신랑측/신부측 아코디언 + 복사 버튼 |
| 8 | RSVP | 참석 여부 | 팝업 + 중간 섹션 리마인드 |
| 9 | 방명록 | 축하 메시지 | 비밀번호 기반 작성/삭제 |
| 10 | 공유하기 | 카카오톡 공유 · 링크 복사 | |

**설계 원칙**

- 사진이 먼저, 인사말이 뒤. 종이 청첩장과 반대 순서다.
- 문구는 2~3문장이 잘 읽힌다. 화면이 작아 긴 글은 스크롤로 흘러간다.
- 필수 정보 4가지: 신랑·신부 이름 / 예식 날짜·시간 / 예식장 이름·위치 / 초대 인사말
- 터치 타겟 최소 48px. 사진 보기 화면에서는 조작 버튼이 사진과 겹치지 않도록 공간을 분리한다.
- 섹션 순서 변경·추가·삭제가 최근 프리미엄 서비스의 차별점이다 (데어무드 등).

**자체 제작 시 반영** — 섹션을 하드코딩하지 않고 config 배열로 관리한다. 배열 순서 = 렌더 순서, 배열에서 빼면 섹션이 사라진다.

```ts
sections: ["cover", "greeting", "gallery", "calendar",
           "location", "account", "rsvp", "guestbook", "share"]
```

전체 배경 흩날림도 같은 방식으로 `bgEffect: "petal" | "snow" | null` 옵션 하나로 켜고 끈다 (§5).

출처: [happy-wedding](https://github.com/ddalggakai-ops/happy-wedding) ·
[구성 가이드](https://scenekoong.com/blog/wedding-invitation-phrases-2026) ·
[데어무드](https://theirmood.com/)

---

## 2. 기능 우선순위 (P0 · P1 · P2)

> 여기의 P0/P1/P2는 **기능 우선순위**다. [PLAN.md](./PLAN.md)의 P0~P3는 **작업 단계** — 축이 다르다.

### P0 — 1차 구현

| 기능 | 상세 | 레퍼런스 |
|---|---|---|
| 메인 커버 | 사진 · 이름 · 일시 · 장소, 오프닝 애니메이션 | 데어무드, YOUNGEUN100 |
| 인사말 / 초대 문구 | 양가 혼주 표기 포함 | 씬쿵, 100yearshop |
| 갤러리 | 최대 30장 내외, 라이트박스 · 스와이프 | 필메이커, shimpark |
| 캘린더 / D-day | 예식일 하트 표시, 남은 시간 카운트다운 | YOUNGEUN100 |
| 오시는 길 | 내비게이션 연동, 대중교통 · 주차 안내 | 필메이커 |
| 마음 전하실 곳 | 청첩장 안에서 계좌 확인, 복사 버튼으로 은행 앱에 붙여넣기 | 필메이커 |
| 연락하기 | 전화 / 문자 모달 (신랑 · 신부 · 혼주) | happy-wedding, 더굿데이 |
| 공유하기 | 카카오톡 공유 + 링크 복사 | 잇츠카드, juhonamnam |
| 반응형 | 모바일 + 데스크톱 | juhonamnam |
| 배경 흩날림 | 꽃잎 / 눈 애니메이션. 옵션으로 on/off | programmerDH 포크 |

### P0에서 뺀 것 — 근거

| 뺀 것 | 대신 | 근거 |
|---|---|---|
| 지도 SDK (네이버 / 카카오) | 정적 지도 이미지 + 딥링크 버튼 | API 키 발급 · 도메인 등록 · 번들이 전부 불필요해진다. 하객은 지도 위에서 조작하지 않고 "네이버지도로 열기"만 누른다 |
| QR 공유 | 카카오톡 공유 + 링크 복사 | 둘이 있으면 QR을 쓸 일이 사실상 없다 |
| 간편 송금 | 계좌 복사 + 카카오페이 / 토스 딥링크 | 수수료 0 (§5 주의사항) |

### P1 — 2차

- **RSVP** — 참석 여부 · 인원 · 식사 여부 폼이 표준. 팝업으로 먼저 띄우고 닫아도 중간 섹션에서 리마인드하는 구조가 응답률에 효과적. 응답은 알림 + 엑셀 다운로드.
- **방명록** — 실시간 축하 메시지. 작성/삭제 시 비밀번호 검증이 최소 요건. 공개 URL이므로 스팸 대응 필요(rate limit, reCAPTCHA).
- 접근성 확대 — 초대글 · 오시는 길 · 혼주 정보의 폰트 크기 확대 옵션 (어르신 하객 대응). 잇츠카드.
- 카톡 공유 커스텀 — 공유 시 사진 · 제목 · 내용 · 버튼명 직접 설정. 잇츠카드.
- 식전영상 / 초대영상 — 1분 미만 모바일용 영상을 갤러리에 자동 추가. 보자기카드.
- 간편 송금 — 카카오페이 QR 업로드. 필메이커. (수수료 이슈 → §5)

### P2 — 차별화 / 보류

- 신랑신부 관련 퀴즈 (정답 · 오답 피드백)
- js-confetti 컨페티 효과 + 좋아요(likes)
- 러브레터 섹션 (손글씨 폰트 / 스캔 이미지)
- 하객 스냅 사진 업로드 (게스트스냅 연동)
- 결혼식 후 감사 인사 페이지로 전환

### 버전 분기

지인에게는 친근하게, 부모님·친척에게는 정중하게, 회사에는 간결하게 — 대상별로 나눠 보내는 흐름이 있다.
상용은 "무제한 다중 제작"으로 대응한다. 자체 제작이라면 `?v=friend` 같은 쿼리 파라미터로
`greeting` · `account`만 얕은 머지 오버라이드하면 한 줄로 끝난다. 처음부터 넣어도 비용이 없다.

---

## 3. 상용 서비스 레퍼런스

각 서비스에서 무엇을 가져올지만 적는다. 전체 기능 비교는 §2.

- **[필메이커](https://feelmaker.co.kr/list/invitation)** — 기능 목록이 가장 체계적. 스펙 기준으로 삼기 좋다.
  볼 것: 갤러리(30장 내외), 오시는 길(내비 + 대중교통 + 주차), 마음 전하실 곳(계좌 복사), 간편 송금(카카오페이 QR)
- **[데어무드](https://theirmood.com/)** — 섹션 커스터마이즈와 RSVP 팝업이 핵심. 자체 제작에서 `sections` 배열로 흉내 낼 부분.
  볼 것: 섹션 순서 변경 UX, RSVP 팝업 타이밍
- **[잇츠카드](https://www.itscard.co.kr/script/mcard/new/mcard_view.asp?CardCode=MCard39)** — 운영 편의 기능이 강하다.
  볼 것: 방명록 관리(비밀번호), 폰트 확대 옵션, 카톡 공유 커스텀
- **[보자기카드](https://bojagicard.com/card/doc/other/mobile/mobile_ecard.php)** — 모바일 웨딩영상 연계.
  볼 것: 영상 섹션이 갤러리와 섞이는 방식
- **[달팽 고객센터 FAQ](https://dalpeng.com/cs/index/81)** — 실제 운영에서 터지는 문제가 다 적혀 있다. 가장 실전적.
  볼 것: 카톡 썸네일 20일 후 회색 처리, 갤러리 비율 깨짐과 "자유형" 옵션
- **[메종드마리에](https://maisondemarie.co.kr/)** — 실시간 미리보기 에디터.
  볼 것: 에디터 ↔ 프리뷰 분할 레이아웃 (자체 제작에서는 config 파일이 그 역할)
- **[바른손M카드](https://mcard.barunsoncard.com/)** — 디자인 톤과 문구 레퍼런스

가이드: [청첩장 문구 2026](https://scenekoong.com/blog/wedding-invitation-phrases-2026) ·
[송금 수수료 이슈](https://gongysd.com/wedding-notion/?bmode=view&idx=169923474) ·
[계좌번호 공유 보안](https://lock.pub/ko/blog/share-bank-details-safely)

---

## 4. 오픈소스 자체 제작 사례

코드를 직접 읽을 수 있어서 상용보다 실용적이다.

### [programmerDH-github/wedding-invitation](https://github.com/programmerDH-github/wedding-invitation) (juhonamnam 포크)

가장 가까운 베이스. React + TypeScript + Vite + SASS, 데모 포함. 배경 흩날림(`bgEffect`)이 여기 있다.

- 스택: React, TypeScript, Vite(CRA에서 마이그레이션), SASS, GitHub Pages
- 기능: 반응형, 갤러리, 네이버 지도, 방명록, 카톡 공유, RSVP, 정적 전용 모드(예식 후 아카이브용)
- 커스터마이즈: `src/const.ts` 수정 + `src/images/` 교체

**bgEffect — 배경 흩날림 구현 (약 140줄, 의존성 0)**
`src/component/bgEffect/index.tsx` + `index.scss`, 이미지는 `src/icons/petal.png` 한 장.

- canvas 하나를 `position: fixed; z-index: 2`로 전체에 깔고 `drawImage`로 꽃잎 반복 렌더
- 파티클 수 = `innerWidth * innerHeight / 30000` — 화면 크기 비례. resize 시 debounce 후 증감
- 화면 밖으로 나가면 위 / 왼쪽 가장자리에서 재투입
- 회전은 `rotate` 변환 없이 flip 값에 cos/sin을 곱해 w/h만 찌그러뜨린다 — 싸게 팔랑이는 효과
- `@media print { display: none }`
- 상수: `X_SPEED 0.6`, `X_SPEED_VARIANCE 0.8`, `Y_SPEED 0.4`, `Y_SPEED_VARIANCE 0.4`, `FLIP_SPEED_VARIANCE 0.02`
- 옵션화: 이미지와 속도/크기/밀도 상수 3~4개만 타입별로 분기하면 `"petal" | "snow" | null` 처리가 끝난다. 파티클 라이브러리 불필요.

### [ddalggakai-ops/happy-wedding](https://github.com/ddalggakai-ops/happy-wedding)

Vanilla JS, 섹션 11개 + Google Apps Script 백엔드. 프레임워크 없이 만들어 섹션 구성이 가장 날것으로 드러난다.
볼 것: 섹션 순서(§1의 근거), Apps Script로 방명록/RSVP 처리, BGM 파일 없을 때 버튼 자동 숨김, 48px 터치 타겟.

### [shimpark/Weddings](https://github.com/shimpark/Weddings)

React + TypeScript + Vite. 라이브러리 선택 레퍼런스 — `react-photoswipe-gallery`, `react-naver-maps`.

### [YOUNGEUN100/react-wedding-card](https://github.com/YOUNGEUN100/react-wedding-card)

퀴즈 · D-day 카운트다운. P2 차별화 기능 참고 — 신랑신부 퀴즈(정답/오답 피드백), js-confetti, 좋아요, D-day 계산.

### [heejin-hwang/mobile-wedding-invitation](https://github.com/heejin-hwang/mobile-wedding-invitation)

가장 많이 포크되는 템플릿. 구조가 단순해서 입문 레퍼런스로 자주 쓰인다.

### [SNURFER/wedding-invi](https://github.com/SNURFER/wedding-invi)

트러블슈팅 레퍼런스. 실제 배포하며 겪은 문제가 코드와 이슈에 남아 있다 — 카카오톡 인앱브라우저 제스처 충돌, 클립보드 복사 실패 대응.
개발 후기: https://brunch.co.kr/@junha04/151

---

## 5. 기술 결정 · 구현 주의사항

### 기술 결정

| 영역 | 결정 | 근거 |
|---|---|---|
| 방명록 · RSVP 백엔드 | Google Apps Script + 스프레드시트 (doGet / doPost) | 의존성 0, 무료. 응답이 시트에 쌓여 엑셀 다운로드가 공짜. fetch 두 개면 끝. rate limit · 스팸 차단은 스크립트에서. Firebase는 프로젝트 설정 + SDK 번들 + 보안 룰이 붙는다 |
| 갤러리 라이트박스 | photoswipe (react-photoswipe-gallery) | 직접 짜면 iOS 카카오톡 인앱브라우저 제스처 이슈를 전부 떠안는다 |
| 지도 | 정적 지도 이미지 + 딥링크 버튼 (네이버 / 카카오 / T맵 / 구글) | API 키 · 도메인 등록 · 번들 불필요. 필요해지면 그때 SDK |
| 공유 | Kakao JS SDK, 키 없으면 링크 복사 폴백 | Kakao Developers 앱 생성 → 플랫폼에 배포 도메인 등록 → JavaScript 키 발급 |
| 배포 | 정적 호스팅 (업체 미정 — [PLAN.md](./PLAN.md)) | |
| 이미지 | 가로 1080px 이하, 장당 300KB 이하 압축 | |

백엔드가 필요한 건 방명록 · RSVP 둘뿐이다. 나머지 섹션은 전부 정적으로 끝난다.

**지도 라이브 SDK가 필요해졌을 때** — 네이버 Dynamic Map API(`react-naver-maps`) 또는 카카오맵(`react-kakao-maps-sdk`).
좌표 확인은 카카오맵에서 예식장 검색 → 우클릭 "여기가 어디인가요?".

### 실패 사례 기반 주의사항

- **iOS 카카오톡 인앱브라우저** — 좌우 스와이프 중 상하 스크롤이 함께 먹혀 갤러리 조작이 불편해진다. `touch-action` / 제스처 임계각 처리 필요. 대부분의 하객이 카톡 링크로 들어오므로 인앱브라우저가 사실상 주 환경이다.
- **클립보드 복사** — `navigator.clipboard` 기반 복사가 특정 기기에서만 동작하지 않는 문제가 보고됨. textarea 폴백 필수.
- **카톡 썸네일 수명** — 보낸 지 20일이 지나면 대화창 이미지가 회색 처리된다. 카카오 서버의 보관 정책이라 막을 수 없다. 발송 타이밍 설계에 반영.
- **갤러리 비율 깨짐** — 고정 비율 크롭 대신 원본 비율에 맞춰 세로/가로가 자동 노출되는 "자유형"을 옵션으로 둔다.
- **송금 수수료** — 일부 서비스의 간편 송금에서 회당 약 1,000~2,000원이 인지 없이 차감돼 논란이 된 사례가 있다. 계좌 복사 + 카카오페이/토스 딥링크 조합이 수수료 없이 안전하다.
- **개인정보** — 이름과 은행명이 결합되면 보이스피싱 · 스미싱의 재료가 된다. 계좌 · 연락처는 아코디언으로 기본 접어두고 예식 후 자동 비공개 처리. 방명록은 공개 URL이라 스팸 대응(reCAPTCHA, rate limit) 필요.
- **BGM** — 브라우저 자동재생 차단 대응 필요. 파일이 없으면 버튼 자체를 숨긴다.

### config 스키마 초안

`src/config.ts` 하나로 전부 제어. `sections` 배열 순서 = 렌더 순서.

```ts
export const config = {
  bgEffect: "petal",            // "petal" | "snow" | null
  bgm: "/bgm.mp3",              // null이면 버튼 자동 숨김
  sections: ["cover", "greeting", "gallery", "calendar",
             "location", "account", "rsvp", "guestbook", "share"],

  couple: { groom: {...}, bride: {...} },   // 이름, 연락처, 혼주, 계좌
  wedding: { date: "2026-...", venue: {...}, transport: [...] },
  greeting: { title, body },
  gallery: { images: [...], ratio: "free" },
  share: { kakaoKey, title, description, thumbnail },
}
```

버전 분기(`?v=friend`)는 `greeting` · `account`만 얕은 머지로 덮어쓴다.

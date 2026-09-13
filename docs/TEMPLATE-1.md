# 템플릿 1

템플릿 1의 스펙 · 시안 · 사용법. 기본 스펙은 [REFERENCES.md §1 표준 레이아웃](./REFERENCES.md#1-표준-레이아웃--섹션-구성)을 그대로 따른다.
TBD 표시된 항목은 이후 하나씩 채운다.

## 기본 정보

| 항목 | 값 |
|---|---|
| 템플릿 ID | `template-1` |
| 상태 | 스펙 작성 중 |
| 콘셉트 · 무드 | TBD |
| 기본 배경 효과 | `bgEffect: "petal"` (옵션으로 snow / null) |
| 기본 폰트 | TBD |
| 지원 환경 | 모바일 우선 + 데스크톱. 카카오톡 인앱브라우저 기준 |

---

## 스펙

### 섹션 구성

`sections` 배열 순서 = 렌더 순서. 배열에서 빼면 섹션이 사라진다.

| # | 섹션 | key | 기본 포함 | 내용 |
|---|---|---|---|---|
| — | BGM 플레이어 | `bgm` | ✅ | 우측 상단 고정. 파일 없으면 버튼 자동 숨김 |
| 1 | 인트로 / 커버 | `cover` | ✅ | 날짜 · 신랑신부 이름 · 메인 사진 + 오프닝 애니메이션 |
| 2 | 인사말 | `greeting` | ✅ | 초대 문구 + 양가 가족 + 연락하기 모달 |
| 3 | 러브레터 | `letter` | ❌ | 손글씨 폰트 / 스캔 이미지 |
| 4 | 갤러리 | `gallery` | ✅ | 3열 그리드 + 더보기 + 라이트박스(photoswipe) |
| 5 | 달력 | `calendar` | ✅ | 예식일 하이라이트 + D-day |
| 6 | 오시는 길 | `location` | ✅ | 정적 지도 + 딥링크 버튼 · 주소 복사 · 교통편 |
| 7 | 마음 전하실 곳 | `account` | ✅ | 신랑측/신부측 아코디언 + 계좌 복사 |
| 8 | RSVP | `rsvp` | ❌ | 참석 여부 · 인원 · 식사 |
| 9 | 방명록 | `guestbook` | ❌ | 비밀번호 기반 작성/삭제 |
| 10 | 공유하기 | `share` | ✅ | 카카오톡 공유 + 링크 복사 |

### 공통 규칙

- 세로 스크롤 한 장. 라우팅 없음, 단일 페이지.
- 사진이 먼저, 인사말이 뒤.
- 문구는 2~3문장. 화면이 작아 긴 글은 흘러간다.
- 터치 타겟 최소 48px. 사진 보기 화면에서는 조작 버튼이 사진과 겹치지 않게 공간 분리.
- 이미지는 가로 1080px 이하 / 장당 300KB 이하.
- 계좌 · 연락처는 아코디언 기본 접힘.
- 클립보드 복사는 `navigator.clipboard` + textarea 폴백.

### 섹션별 상세 스펙

각 섹션의 데이터 필드 · 상호작용 · 예외 처리. TBD는 이후 보완.

| 섹션 | 데이터 필드 | 상호작용 | 예외 처리 |
|---|---|---|---|
| cover | TBD | TBD | TBD |
| greeting | TBD | TBD | TBD |
| gallery | TBD | TBD | TBD |
| calendar | TBD | TBD | TBD |
| location | TBD | TBD | TBD |
| account | TBD | TBD | TBD |
| share | TBD | TBD | TBD |

### 디자인 토큰

| 토큰 | 값 |
|---|---|
| 배경색 | TBD |
| 본문색 | TBD |
| 포인트색 | TBD |
| 제목 폰트 | TBD |
| 본문 폰트 | TBD |
| 섹션 상하 여백 | TBD |
| 최대 너비 | TBD |

---

## 시안

**전체 흐름** — TBD (섹션 순서대로 이어붙인 전체 스크린샷)

| 섹션 | 시안 | 메모 |
|---|---|---|
| cover | TBD | |
| greeting | TBD | |
| gallery | TBD | |
| calendar | TBD | |
| location | TBD | |
| account | TBD | |
| share | TBD | |

**배경 효과 비교**

| 옵션 | 시안 | 메모 |
|---|---|---|
| petal | TBD | 기본값 |
| snow | TBD | |
| null | TBD | 효과 없음 |

---

## 사용법

### 1. 설치

TBD

### 2. `src/config.ts` 작성

수정할 파일은 이것 하나. 나머지는 건드리지 않는다.

```ts
export const config = {
  bgEffect: "petal",            // "petal" | "snow" | null
  bgm: "/bgm.mp3",              // null이면 버튼 자동 숨김
  sections: ["cover", "greeting", "gallery", "calendar",
             "location", "account", "share"],

  couple: { groom: {...}, bride: {...} },   // 이름, 연락처, 혼주, 계좌
  wedding: { date: "2026-...", venue: {...}, transport: [...] },
  greeting: { title, body },
  gallery: { images: [...], ratio: "free" },
  share: { kakaoKey, title, description, thumbnail },
}
```

필드별 설명 — TBD

### 3. 이미지 교체

- 경로: TBD
- 가로 1080px 이하, 장당 300KB 이하로 압축해서 넣는다.
- 갤러리 비율은 `ratio: "free"`가 기본 (원본 비율 유지).

### 4. 외부 연동

| 연동 | 필요한 것 | 없으면 |
|---|---|---|
| 카카오톡 공유 | Kakao Developers 앱 생성 → 플랫폼에 배포 도메인 등록 → JavaScript 키 | 링크 복사로 폴백 |
| 방명록 · RSVP | Google Apps Script 웹앱 배포 URL (doGet / doPost) | 해당 섹션 자동 비활성 |
| 지도 | 정적 지도 이미지 + 예식장 좌표 | — |

### 5. 배포

TBD — 업체 미정. [PLAN.md](./PLAN.md) 참고.

### 6. 버전 분기

`?v=friend` 같은 쿼리 파라미터로 `greeting` · `account`만 덮어쓴다. 지인 / 친척 / 회사용을 따로 만들지 않는다.

---

## 배포 전 체크리스트

- [ ] 예식 정보(날짜 · 시간 · 예식장) 오타 확인
- [ ] 양가 혼주 표기 확인
- [ ] 계좌번호 · 예금주 확인
- [ ] 이미지 용량 압축 확인
- [ ] iOS · Android 카카오톡 인앱브라우저에서 실제로 열어보기
- [ ] 갤러리 스와이프 / 라이트박스 동작 확인
- [ ] 계좌 복사 버튼 동작 확인
- [ ] 카톡 공유 썸네일 확인 (발송 20일 후 회색 처리되는 점 감안)
- [ ] BGM 자동재생 차단 시 동작 확인

## 남은 결정

- 콘셉트 · 무드 · 디자인 토큰 확정
- 섹션별 상세 스펙 채우기
- 시안 제작
- 설치 · 배포 절차 실제로 돌려보고 문서화

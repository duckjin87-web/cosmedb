# 배포와 데이터 연결

## 1. GitHub 저장소 만들기

> 지금은 **더미 데이터**라 Public으로 만들어도 괜찮습니다.
> 실제 협력업체 정보로 교체하는 순간 **Private으로 전환**하세요.
>
> 처음 올리는 경우 [GITHUB-UPLOAD.md](GITHUB-UPLOAD.md)에 화면 단위 절차가 있습니다.

```bash
mkdir cosmedb && cd cosmedb
git init
# 이 문서 묶음을 그대로 복사해 넣습니다
git add .
git commit -m "CosmeDB 초기 구성 — 마스터·조사·방문·등록평가"
git branch -M main
git remote add origin https://github.com/<계정>/cosmedb.git
git push -u origin main
```

### 파일 구조

```
cosmedb/
├── index.html                          앱 본체 (단일 파일)
├── README.md
├── docs/
│   ├── DATA-MODEL.md
│   ├── CAPABILITY-DICTIONARY.md
│   ├── WORKFLOW.md
│   ├── MATCHING.md
│   ├── MIGRATION.md
│   ├── DEPLOY.md
│   └── GITHUB-UPLOAD.md
├── data/
│   ├── dictionary.json                 역량 항목 사전
│   ├── master.demo.json                업체마스터 더미 30개사
│   ├── capability_values.demo.json     역량값 더미 161건
│   └── visits.demo.json                방문일정 더미 8건
└── .github/workflows/pages.yml         자동 배포
```

---

## 2. GitHub Pages 배포

Private 저장소의 Pages는 유료 플랜에서만 됩니다. 무료 플랜이면 아래 대안을 쓰세요.

### Pages 설정

1. 저장소 → **Settings** → **Pages**
2. Source를 **GitHub Actions**로 선택
3. `main`에 push하면 자동 배포

`.github/workflows/pages.yml`이 이미 들어 있습니다.

### 무료 플랜 대안

| 방법 | 설명 |
|---|---|
| **로컬 실행** | `python -m http.server 8000` — 개인 PC에서만 사용 |
| **사내 파일 서버** | `index.html`을 공유 폴더에 두고 브라우저로 열기 |
| **사내 웹서버** | IIS/Nginx에 정적 파일로 배치. 사내망 접근 제어 적용 |

사내 정보 보안 정책상 외부 호스팅이 어려울 수 있으니, **사내 서버 배치를 먼저 검토**하세요.

---

## 3. 데이터 영속화 — 가장 중요한 다음 단계

현재 `index.html`은 데이터를 **브라우저 메모리에만** 들고 있습니다.
새로고침하면 초기 상태로 돌아갑니다. 시안이라 그렇습니다.

팀원 공동 편집이 필요하므로 아래 중 하나를 붙여야 합니다.

### 방법 A — 구글 시트 백엔드 (권장)

가장 현실적입니다. 기존 Apps Script 경험을 그대로 씁니다.

```
구글 시트 (vendors / capability_values / surveys / visits / evaluations)
        ↕ Apps Script Web App (doGet / doPost)
   index.html (fetch로 읽고 쓰기)
```

| 장점 | 단점 |
|---|---|
| 팀원이 시트에서 직접 볼 수 있음 | 동시 편집 시 마지막 저장이 이김 |
| 권한 관리가 구글 계정 기반 | 행이 수천 건 넘으면 느려짐 |
| 백업·버전 기록 자동 | Apps Script 실행 시간 제한 |

63개사 규모에서는 충분합니다.

### 방법 B — GitHub 저장소를 DB로

`data/*.json`을 직접 커밋해 데이터로 씁니다.

| 장점 | 단점 |
|---|---|
| 변경 이력이 git에 남음 | 동시 편집 충돌 처리 필요 |
| 별도 서버 불필요 | 쓰기에 토큰 필요, 비개발자에게 어려움 |

읽기 전용 조회용으로는 좋고, 입력은 다른 경로를 두는 편이 낫습니다.

### 방법 C — SharePoint / Microsoft Lists

사내가 M365 환경이면 검토할 만합니다.

- 상태 변경 이력이 **자동** 기록 (`change_log` 불필요)
- "Capa회신대기중으로 바뀌면 7일 뒤 알림" 같은 규칙을 클릭으로 설정
- 다만 "한 행에 다 보이고 접었다 펴는" 화면은 구현되지 않습니다

---

## 4. 외부 API 연동

### 카카오맵 길찾기 (이동시간)

현재 이동 거리·시간은 지역별 추정치입니다. 실제 값을 쓰려면:

```javascript
// 기준점: 세종시 전의면 산단길 22-17 (한국콜마)
const ORIGIN = { x: 127.2076, y: 36.7186 };

// 1) 주소 → 좌표
//    GET https://dapi.kakao.com/v2/local/search/address.json?query={주소}
// 2) 좌표 → 경로
//    GET https://apis-navi.kakaomobility.com/v1/directions
//        ?origin={x,y}&destination={x,y}&priority=RECOMMEND
//    응답의 summary.distance(m), summary.duration(초) 사용
```

주소는 잘 안 바뀌므로 **좌표를 마스터에 캐시**해 두고, 경로만 필요할 때 조회하세요.
API 키는 소스에 넣지 말고 사내 프록시나 환경변수로 분리합니다.

### 식약처 공공데이터 (L0 자동 수집)

CGMP 인증 현황과 기능성화장품 신고 이력으로 `CE01` `CE06`을 자동으로 채웁니다.
CORS 문제는 Cloudflare Workers 프록시로 우회할 수 있습니다.

### 국민연금 가입사업장 (RK01)

종업원 수를 자동 수집합니다. 매월 갱신되므로 신선도 유지가 쉽습니다.

---

## 5. 운영 체크리스트

### 매주
- [ ] `Capa회신대기중` 7일 경과 건 독촉
- [ ] 방문 3일 이내 건 준비 확인
- [ ] `조사대기` 상태로 2주 이상 머문 조사 점검

### 매월
- [ ] 만료 임박 항목(TTL 3개월 이내) 목록 확인
- [ ] 신규 등록 업체의 필수 항목 채움률 점검
- [ ] 종결 건 아카이브 이관

### 매분기
- [ ] 재사용률·커버리지·신선도 지표 확인
- [ ] 자유 항목 중 3회 이상 반복된 것을 사전으로 승격
- [ ] 사전 항목 중 아무도 안 쓰는 것 폐기 검토

### 매년
- [ ] G5 재검증 — 등록 업체 전체에 문진표 재발송
- [ ] 인증 만료일 일괄 확인

---

## 6. 보안

- 현재 데이터는 전부 더미이므로 Public으로 시작해도 됩니다
- **실제 데이터로 교체하는 즉시 Private으로 전환**하세요
- 한 번 커밋된 데이터는 파일을 지워도 git 이력에 남습니다. 실수로 올렸다면 저장소를 삭제하고 새로 만드세요
- API 키를 소스에 하드코딩하지 말 것
- 사내 정보보안 정책상 외부 클라우드 반출이 제한될 수 있으니 **사전 확인** 필요

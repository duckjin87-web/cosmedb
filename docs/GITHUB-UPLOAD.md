# GitHub 업로드 방법

두 가지 방법이 있습니다. **처음이라면 방법 1(웹 브라우저)** 을 권합니다. 설치할 것이 없습니다.

---

## 방법 1 — 웹 브라우저로 업로드 (설치 불필요)

### 1단계 · 저장소 만들기

1. [github.com](https://github.com) 로그인 → 우측 상단 **`+`** → **New repository**
2. 아래처럼 입력합니다

| 항목 | 입력값 |
|---|---|
| Repository name | `cosmedb` |
| Description | 협력업체 조사·방문·등록평가 관리 |
| 공개 범위 | **Public** (지금은 더미 데이터라 괜찮습니다) |
| Add a README file | **체크 해제** (이미 있습니다) |

3. **Create repository**

> 나중에 실제 업체 데이터로 바꾸면 반드시 **Private으로 전환**하세요.
> Settings → 맨 아래 Danger Zone → Change repository visibility

### 2단계 · 파일 올리기

1. 방금 만든 저장소 화면에서 **uploading an existing file** 링크 클릭
   (이미 파일이 있으면 **Add file** → **Upload files**)
2. 압축을 푼 `cosmedb-repo` 폴더를 열어 **안에 있는 내용물을** 드래그해서 놓습니다

```
cosmedb-repo 폴더 안의 것들을 올립니다 (폴더째로 올리지 마세요)
├── index.html
├── README.md
├── docs/          ← 폴더째 드래그하면 구조가 유지됩니다
├── data/
└── .github/
```

3. 하단 **Commit changes** 클릭

> **`.github` 폴더가 안 올라갈 수 있습니다.** 점(`.`)으로 시작하는 폴더는
> 운영체제가 숨김 처리합니다. 자동 배포가 필요 없으면 건너뛰어도 됩니다.
> 필요하면 3단계의 웹 편집기로 직접 만듭니다.

### 3단계 · 파일 직접 만들기 (`.github`가 안 올라간 경우)

1. **Add file** → **Create new file**
2. 파일명 칸에 정확히 다음을 입력합니다 (슬래시를 치면 폴더가 자동 생성됩니다)

```
.github/workflows/pages.yml
```

3. 내용에 `cosmedb-repo/.github/workflows/pages.yml` 파일 내용을 붙여넣기
4. **Commit changes**

### 4단계 · GitHub Pages 켜기

1. 저장소 → **Settings** → 좌측 **Pages**
2. **Source**를 **GitHub Actions**로 선택
3. 1~2분 뒤 상단에 주소가 나옵니다

```
https://<계정명>.github.io/cosmedb/
```

기존 사이트가 `duckjin87-web.github.io/cosmeticdb`이므로,
같은 계정이라면 `duckjin87-web.github.io/cosmedb`가 됩니다.

---

## 방법 2 — 명령어로 업로드

Git이 설치돼 있고 명령줄이 익숙하다면 이쪽이 빠릅니다.

### 최초 1회 설정

```bash
git config --global user.name "이름"
git config --global user.email "메일주소"
```

### 업로드

```bash
cd cosmedb-repo

git init
git add .
git commit -m "CosmeDB 초기 구성 — 마스터·조사·방문·등록평가"
git branch -M main
git remote add origin https://github.com/<계정명>/cosmedb.git
git push -u origin main
```

비밀번호를 물으면 **GitHub 비밀번호가 아니라 토큰**을 넣어야 합니다.

### 토큰 발급

1. GitHub 우측 상단 프로필 → **Settings**
2. 맨 아래 **Developer settings** → **Personal access tokens** → **Tokens (classic)**
3. **Generate new token (classic)**
4. Note에 `cosmedb`, Expiration은 90일 정도
5. **`repo`** 항목에 체크
6. **Generate token** → 나온 문자열을 복사 (다시 볼 수 없으니 메모장에 임시 보관)
7. push할 때 비밀번호 자리에 붙여넣기

---

## 수정한 파일 다시 올리기

### 웹 브라우저

1. 저장소에서 수정할 파일 클릭
2. 우측 연필 아이콘 (**Edit this file**)
3. 고친 뒤 **Commit changes**

`index.html`처럼 통째로 바꿀 때는 **Add file** → **Upload files**로 같은 이름 파일을 올리면 덮어씁니다.

### 명령어

```bash
git add .
git commit -m "무엇을 바꿨는지 한 줄"
git push
```

---

## 자주 겪는 문제

| 증상 | 원인과 해결 |
|---|---|
| Pages 주소가 404 | 배포에 1~2분 걸립니다. **Actions** 탭에서 초록 체크를 확인하세요 |
| 화면이 안 뜨고 파일 목록만 보임 | 루트에 `index.html`이 있어야 합니다. 폴더 안에 들어가 있지 않은지 확인 |
| 한글 파일명이 깨짐 | 파일명은 영문으로 두세요. 문서 내용은 한글이어도 됩니다 |
| `.github` 폴더가 안 올라감 | 위 3단계 참고 — 웹 편집기로 직접 생성 |
| push할 때 인증 실패 | 비밀번호가 아니라 토큰이 필요합니다 |
| 실수로 실제 데이터를 올림 | 파일 삭제만으로는 이력에 남습니다. **저장소를 지우고 새로 만드세요** |

---

## 실제 데이터로 바꿀 때 체크리스트

- [ ] 저장소를 **Private으로 전환** (Settings → Danger Zone)
- [ ] `data/*.demo.json`을 실제 데이터로 교체하고 파일명에서 `demo` 제거
- [ ] `index.html` 안의 내장 데이터도 함께 교체
- [ ] 앱 상단의 `데모 데이터 · 실제 업체 정보 아님` 문구 삭제
- [ ] 사내 정보보안 정책상 외부 클라우드 반출이 가능한지 **사전 확인**
- [ ] 반출이 어려우면 사내 웹서버 배치로 전환 ([DEPLOY.md](DEPLOY.md) 참고)

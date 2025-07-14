# 🧱 Git의 기초 - Git 저장소 만들기

## 📁 Git Repository(저장소) 만들기

Git 저장소를 만드는 방법은 두 가지입니다:

1. **버전 관리하지 않던 로컬 디렉토리에 Git 저장소 적용**
2. **기존 원격 저장소(Repository)를 Clone 하기**

---

## 1️⃣ 기존 디렉토리를 Git 저장소로 만들기

기존 프로젝트를 Git으로 버전 관리하고자 할 때, 해당 디렉토리로 이동하여 초기화합니다.

### 📌 디렉토리 경로 복사 (macOS)
> `Command + Option + C`

### ✅ 디렉토리 이동 예시

**Windows (Git Bash 기준)**
```bash
cd "/c/Users/yangsuhyuck/Desktop/App Project/GitPractice/GitPractice"
```

**macOS**
```bash
cd "/Users/yangsuhyuck/Desktop/App Project/GitPractice/GitPractice"
```

### ✅ Git 저장소 초기화 및 파일 추가
디렉토리에 접근 후 git init 이라는 명령어를 통해 새로운 Git 저장소를 초기화합니다.
이 명령어는 .git 이라는 숨겨진 디렉토리를 생성하며 이 디렉토리에 Git이 프로젝트의 모든 버전을 추적하고 관리하는 데 필요한 파일들이 저장됩니다.
.git 디렉토리가 생성되면 이 디렉토리 안에 있는 파일들을 Git이 추적하도록 Staging Area에 추가합니다. 이는 git add 라는 명령어를 통해 가능합니다.
```bash
git init
# 성공 메시지 예시
Initialized empty Git repository in /.../.git/

git add .
# '.' : 현재 디렉토리 및 하위 디렉토리 전체를 Staging Area에 추가
# 특정 파일만 추가: git add [파일명] 또는 [디렉토리명]/
```

### ✅ 첫 커밋
이제 Staging에 올라간 파일들을 로컬 Git 저장소에 커밋합니다.
커밋할 때는 git commit 이라는 명령어를 사용하고 -m 옵션을 통해 커밋 메시지 작성이 가능합니다.
```bash
git commit -m "Initial commit for GitProject"
# 예시 결과
[main (root-commit) e6e8f74] Initial commit for GitProject
```
<img width="784" height="351" alt="스크린샷 2025-07-10 오후 9 58 23" src="https://github.com/user-attachments/assets/051b4c3c-1be4-41b2-b294-468c7e250876" />

> 💡 숨김 폴더 확인 (macOS): `Command + Shift + .`
<img width="471" height="430" alt="스크린샷 2025-07-10 오후 9 58 41" src="https://github.com/user-attachments/assets/43f52d90-f42c-453f-81d4-6d7c916c7627" />

---

## 2️⃣ 기존 저장소를 Clone 하기

다른 사람의 프로젝트에 참여하거나 원격 저장소 복제 시 사용합니다.

### ✅ 사용법

```bash
git clone <저장소 URL>
```

### 📁 디렉토리 이동 후 클론
<img width="356" height="142" alt="스크린샷 2025-07-10 오후 11 00 15" src="https://github.com/user-attachments/assets/c90e8c42-3d48-4700-b090-007635c77a75" />

```bash
cd "/Users/yangsuhyuck/Desktop/App Project/SeSAC Study"

git clone https://github.com/sesacAndroid5/SeSAC-Study.git
```

### ✅ 성공 메시지 예시
```
Cloning into 'SeSAC-Study'...
remote: Enumerating objects: 88, done.
remote: Counting objects: 100% (88/88), done.
remote: Compressing objects: 100% (73/73), done.
remote: Total 88 (delta 18), reused 0 (delta 0), pack-reused 0
Receiving objects: 100% (88/88), 35.29 KiB | 188.00 KiB/s, done.
Resolving deltas: 100% (18/18), done.
```
<img width="704" height="181" alt="스크린샷 2025-07-10 오후 11 02 54" src="https://github.com/user-attachments/assets/884382db-6747-4e9c-9810-20ddd64421f6" />

---

## ✅ 요약

| 작업 구분              | 명령어 또는 설명                              |
|------------------------|-----------------------------------------------|
| Git 저장소 초기화      | `git init`                                    |
| 전체 파일 추가         | `git add .`                                   |
| 특정 파일 추가         | `git add [파일명]`                            |
| 커밋                   | `git commit -m "메시지"`                     |
| 저장소 복제 (Clone)    | `git clone [저장소 URL]`                      |
| .git 숨김 폴더 확인    | `Command + Shift + .` (macOS)                 |
| Git 최적화             | `git gc`, Git LFS, `.gitignore` 사용         |

---
## 📦 .git 디렉토리 관리

`.git` 폴더는 Git의 핵심 디렉토리로 모든 버전 기록 및 스냅샷 정보가 저장됩니다.

- **파일, 커밋 히스토리, 브랜치, 태그 등 관리**
- **크기 권장: 100MB ~ 300MB**

### 💡 관리 팁

- `.gitignore`: Git으로 관리하지 않을 파일 지정
- **Git LFS**: 대용량 바이너리 파일(이미지, 영상 등) 관리용 Git 확장
- `git gc`: 저장소 정리 및 최적화

---
<br><br>
# 🧱 Git의 기초 - 수정하고 저장소에 저장하기

## ✍️ 수정하고 저장소에 저장하기

Git은 워킹 디렉토리의 모든 파일을 다음 두 가지로 구분합니다:

- **Tracked**: 이미 Git이 추적하고 있는 파일 (스냅샷에 포함된 파일)
- **Untracked**: Git이 아직 추적하지 않는 새로운 파일

### Tracked 상태의 세 가지 분류

| 상태       | 설명 |
|------------|------|
| Unmodified | 수정하지 않음 |
| Modified   | 수정된 상태 |
| Staged     | 커밋을 위해 준비된 상태 |

> 예시: 클론 받은 저장소의 파일은 모두 `Tracked`이며 초기 상태는 `Unmodified`입니다.
<img width="674" height="312" alt="스크린샷 2025-07-14 오후 6 11 55" src="https://github.com/user-attachments/assets/8ab11978-4697-40c5-9241-bdbc579924af" />

---

## 📄 새로운 파일 생성

- 새로운 파일을 생성하면 `Untracked` 상태입니다.
- `git add` 명령어를 통해 `Staged` 상태로 변경됩니다.

## ✏️ 기존 파일 수정

- 기존 Tracked 파일을 수정하면 `Modified` 상태가 됩니다.
- `git add`를 실행하면 수정 내용이 `Staged` 상태로 올라갑니다.
- 이후 파일을 다시 수정하면 해당 파일은 `Staged` + `Modified` 상태를 모두 가지게 됩니다.

---

## 🔙 되돌리기 (Undo)

### ✅ 커밋 메시지 또는 사소한 내용 수정: `--amend`

커밋 후 스펠링 오류, 누락 파일, 메시지 수정 등이 필요할 때는 `--amend`를 사용하여 커밋을 재작성할 수 있습니다.

```bash
git commit --amend
```

> 기존 커밋을 완전히 새로 고쳐 히스토리도 바뀝니다 (원래 커밋은 사라짐).

### 예시 시나리오

1. 커밋 후 스펠링 오류 발견
<img width="709" height="270" alt="스크린샷 2025-07-14 오후 7 23 48" src="https://github.com/user-attachments/assets/8dc3f5f4-6718-46b6-bb71-b4812ae39a75" />

2. SourceTree 등 GUI에서 `Amend` 옵션을 선택
<img width="1354" height="277" alt="스크린샷 2025-07-14 오후 7 26 39" src="https://github.com/user-attachments/assets/4df38506-f01e-4fd4-bb1b-289972e85533" />

3. 수정한 커밋으로 기존 커밋을 대체
<img width="826" height="315" alt="스크린샷 2025-07-14 오후 7 27 48" src="https://github.com/user-attachments/assets/5d93f186-7be6-46be-ac3f-206589769751" />

---

## ✅ 요약

| 작업 내용            | 상태 변화 설명 |
|---------------------|----------------|
| 새 파일 생성        | Untracked → Staged (`git add`) |
| 기존 파일 수정      | Unmodified → Modified → Staged (`git add`) |
| 다시 수정한 파일    | Staged + Modified 병존 가능 |
| 커밋 수정           | `git commit --amend` 사용하여 새 커밋으로 대체 |

---
<br><br>
# 🌐 Git의 기초 - 리모트 저장소 (Remote Repository)

## 📡 리모트 저장소란?

인터넷이나 네트워크 상의 다른 컴퓨터에 위치한 Git 저장소입니다. 협업을 위한 핵심 개념으로, **Push**와 **Pull**을 통해 데이터를 주고받습니다.

- `origin`: 기본 원격 저장소의 별칭

---

## 🔽 원격 저장소에서 데이터 가져오기

### 1️⃣ `git fetch`

- 원격 저장소의 변경 사항을 가져오지만 로컬 브랜치에는 자동 병합하지 않음
- 변경 내용 확인 후 병합을 수동으로 결정 가능

**사용 시점:**

- 로컬 작업을 유지한 채 변경사항 확인
- 자동 병합 충돌 방지

```bash
git fetch origin
```

### 2️⃣ `git pull`

- 원격 저장소의 변경 사항을 가져오고 로컬 브랜치에 자동 병합

**사용 시점:**

- 내 작업이 없거나 변경사항을 바로 병합하고 싶을 때

```bash
git pull origin main
```

---

## 🔼 원격 저장소에 Push 하기

작업한 결과를 원격 저장소에 업로드하여 다른 사용자들과 공유합니다.

```bash
git push origin main
```

> 💡 누군가가 먼저 Push 했다면 `pull` 후 병합 후 다시 `push`해야 충돌 없이 적용됩니다.

---

# 🏷️ Git의 기초 - 태그(Tag)

## 🔖 Tag란?

커밋에 의미 있는 이름을 붙여주는 기능입니다.

### ✅ 사용 목적

- 릴리즈 버전 명시: `v1.0`, `v2.1.1` 등
- 중요한 이벤트 기록: 기능 완료, 버그 수정 등
- 읽기 쉬운 참조 이름 부여 (SHA 대신)

---

## 🧾 태그의 종류

### 1️⃣ 경량 태그 (Lightweight Tag)

- 커밋에 대한 간단한 참조 (포인터)
- 메타데이터 없음

```bash
git tag v1.0
```
<img width="736" height="267" alt="스크린샷 2025-07-14 오후 9 01 23" src="https://github.com/user-attachments/assets/d1f844b3-7e49-4a53-b919-4411b7e83930" />

### 2️⃣ 주석 태그 (Annotated Tag) ✅ 권장

- 메타데이터 포함: 작성자, 날짜, 메시지 등

```bash
git tag -a v1.0 -m "Release version 1.0"
```
<img width="740" height="272" alt="image" src="https://github.com/user-attachments/assets/a186eeb1-114e-4bb5-b5b4-b7c1abc96c86" />

---

## 🚀 태그 푸시하기

기본적으로 `git push`는 브랜치만 푸시하고 **태그는 포함하지 않음**.
<img width="684" height="302" alt="스크린샷 2025-07-14 오후 8 45 41" src="https://github.com/user-attachments/assets/afaaefc7-2786-4a5f-831b-3fa4c6ff9c91" />


### ✅ 모든 태그 푸시
```bash
git push origin --tags
```

### ✅ 특정 태그만 푸시
```bash
git push origin v1.0
```

> 릴리즈나 배포 시 중요한 태그는 반드시 원격 저장소에 푸시해야 합니다.


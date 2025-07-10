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

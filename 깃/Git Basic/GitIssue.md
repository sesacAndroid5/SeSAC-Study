# 🚀 <ins>Git Issue</ins> 나도 써보자!
### 🤔Git Issue가 뭘까?
- Git Issue는 GitHub, GitLab 등에서 제공하는 __이슈추적__ 시스템 입니다.
- 버그 리포트, 새로운 기능요청, 개선사항 등을 체계적으로 관리할 수 있습니다.

### Issue의 중요 용도
- 🐞 __버그 추적__
- ✨ __기능 요청__
- 📋 __작업 관리__

## 1. 이슈 템플릿 생성하기
> 레포지토리의 Settings로 들어가 Features>Issues 에서 "Set up templates" 버튼 클릭

<img width="700" height="400" alt="Image" src="https://github.com/user-attachments/assets/dbac35a3-7267-412e-a4e5-12a751f3668c" /> <br><br>

> 원하는 유형의 템플릿을 선택 혹은 만들기

<img width="400" height="250" alt="Image" src="https://github.com/user-attachments/assets/1105708d-5e5d-4340-9502-be6a7ddc6f74" /> <br><br>

> 연필 아이콘을 눌러 자유롭게 템플릿을 편집한다.

<br>

### 템플릿 예시
- Template name: 🐞Bug
- About: Bug 발생 시 작성해 주세요.
```markdown
# 🐞 어떤 버그인가요?

<!--- 어떤 버그인지 간결하게 설명해주세요 -->

## 📝 어떤 상황에서 발생한 버그인가요?

<!--- (가능하면) Given-When-Then 형식으로 서술해주세요 -->

- **Given**: (상황 설명)
- **When**: (행동 설명)
- **Then**: (문제 발생 설명)

## 🎯 예상 결과

<!--- 예상했던 정상적인 결과가 어떤 것이었는지 설명해주세요 -->

## 📋 추가정보

<!--- 추가적인 로그, 스크린샷, 또는 관련 정보가 있다면 첨부해주세요. -->

## 📍레퍼런스

- [Title](https://...)
```

<br>

- Template name: ✅Feature request
- About: Feature 작업 상황을 입력해주세요.
```markdown
## 📄 이슈 내용

<!--- 기능에 대한 요약 설명을 작성해 주세요. -->

## 📝 상세 내용

<!--- 기능 추가와 관련된 상세 내용을 작성해 주세요. -->

## ✅ 체크리스트

- [ ] TODO A
- [ ] TODO B
- [ ] TODO C

## 📍 레퍼런스

- [Title](https://...)
```
> 템플릿을 다 작성 하였으면 Commit을 한다.

<br>

## 2. 이슈 생성하기
> 레포지토리의 Issues에서 만들어둔 템플릿을 사용하여 새로운 이슈를 생성한다.<br>
> 담당자, 라벨 등을 정한다.

<img width="300" height="300" alt="Image" src="https://github.com/user-attachments/assets/d9634d29-02ef-4b15-b2d1-1b9108006e8a" /> <br><br>

> 생성된 이슈의 번호를 확인한다. ( # N )이 이슈번호

<img width="400" height="300" alt="Image" src="https://github.com/user-attachments/assets/d2d039b5-4159-496b-be3c-3a0864537588" /> <br><br>

## 3. 이슈에 커밋 반영하기
> 회사 마다 알맞은 브랜치이름, 커밋방식에 맞춰 작성한다.<br>
> #N 을 커밋메세지에 반영하면 자동으로 들어간다.

<img width="600" height="130" alt="Image" src="https://github.com/user-attachments/assets/df2815c5-3a9f-42b6-8187-231f1c013837" /> <br><br>

<img width="600" height="200" alt="Image" src="https://github.com/user-attachments/assets/25157819-07f2-4b6a-9f95-9cc56287a6cd" /> <br><br>

## 4. 이슈 닫기
> 커밋 메세지에 다음과 같은 단어와 이슈번호가 같이 있으면 자동으로 닫힌다.
> - fix / fixes / fixed
> - close / closes / closed
> - resolve / resolves / resolved

### 조미료 뿌리기
> Git의 Projects와 같이 사용하여 협엽 능률을 더욱 올릴 수 있다.

### 📎 참조

- __[GitHub Issue 사용하여 협업하기](https://mynamesieun.github.io/git/GitHub-Issue-%EC%82%AC%EC%9A%A9%ED%95%98%EC%97%AC-%ED%98%91%EC%97%85%ED%95%98%EA%B8%B0/#-%EC%B0%B8%EC%A1%B0)__

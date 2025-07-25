## Unit Test

안드로이드의 유닛 테스트는 실행 환경에 따라 두 종류로 나뉩니다.

1. **로컬 단위 테스트 (test)**  
2. **계측 테스트 (androidTest)**
<img width="341" height="199" alt="스크린샷 2025-07-22 오후 7 34 37" src="https://github.com/user-attachments/assets/d49471e0-2987-4925-9eda-02e054d106de" />

### 1. 로컬 단위 테스트 (test)

- **실행 환경**: 로컬 JVM에서 실행됩니다. 
- **테스트 대상**: 순수 비즈니스 로직 (예: 데이터 유효성 검사, ViewModel, Repository 등)  
- **라이브러리**: JUnit, Mockito/MockK, Truth 등  

### 2. 계측 테스트 (androidTest)

- **실행 환경**: 에뮬레이터 또는 실제 디바이스  
- **테스트 대상**: UI, 통합 테스트 등 안드로이드 프레임워크에 의존하는 코드  
- **라이브러리**: JUnit, Espresso 등  

---

## 로컬 단위 테스트 예시

JUnit 기반으로 작성합니다.

### 일반 클래스 테스트
**[1] 타겟 클래스 생성**  
<img width="418" height="305" alt="스크린샷 2025-07-22 오후 7 54 45" src="https://github.com/user-attachments/assets/61215e20-07e9-4108-9e2c-63294d68d7c3" />
<br><br>
**< 테스트 코드 파일 생성 방법 >**
클래스명에서 option + enter (window : alt + enter)

<img width="415" height="300" alt="스크린샷 2025-07-22 오후 7 56 39" src="https://github.com/user-attachments/assets/0372efed-23d3-4f3b-be0f-7aba3117c433" />
<img width="470" height="451" alt="스크린샷 2025-07-22 오후 8 00 06" src="https://github.com/user-attachments/assets/e3f65eec-dd7d-4847-8be8-155597550ba2" />

- 테스트 코드 파일 생성
<img width="690" height="296" alt="스크린샷 2025-07-22 오후 8 00 38" src="https://github.com/user-attachments/assets/5264c8c2-9d0a-4b77-8c5c-8cc9f6467440" />
<br><br>

**[2] 테스트 코드 작성**
  
- `@Before`: 테스트 전에 실행되는 초기화 코드  
- `@After`: 테스트 후 실행되는 정리 코드  
- `@Test`: 실제 테스트 코드
  
<img width="663" height="662" alt="스크린샷 2025-07-22 오후 8 10 05" src="https://github.com/user-attachments/assets/a72807f4-c042-4934-b22a-c8714dfdf4c6" />  
<br><br>  
- 테스트 결과  
<img width="1388" height="200" alt="스크린샷 2025-07-22 오후 8 10 48" src="https://github.com/user-attachments/assets/04512cee-71ae-4361-a12c-e87bb82869b1" />

**< 잘못된 값이 나왔을 때 >**
0이 아닌 잘못된 값이 나왔을 때를 가정
<img width="639" height="140" alt="스크린샷 2025-07-22 오후 8 11 51" src="https://github.com/user-attachments/assets/19b868a0-37da-4e0a-8078-dccad68aff14" />
<img width="1108" height="105" alt="스크린샷 2025-07-22 오후 8 12 14" src="https://github.com/user-attachments/assets/630d8664-17c1-44ed-bb8a-fbe3d6421f75" />
<br><br>

### API 호출 테스트
<img width="676" height="565" alt="스크린샷 2025-07-22 오후 8 21 50" src="https://github.com/user-attachments/assets/80c14f86-7ecd-405a-b2b4-372fcea6b47d" />


---

## JUnit

자바 기반의 **단위 테스트 프레임워크**입니다.

- Android에서 JVM 기반 코드를 테스트할 때 사용  
- 코루틴, ViewModel 등 비즈니스 로직 테스트에 적합  
- 각 단위 함수/클래스의 동작을 검증  

---

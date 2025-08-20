# 안드로이드 실행 구조 정리 (JVM / Dalvik / ART)

## 1.1 VM(Virtual Machine) 기본 개념
- VM은 **CPU 동작을 소프트웨어로 흉내낸 가상 CPU**
- 자바는 `.java` → `.class(바이트코드)` → JVM 위에서 실행
- OS별 JVM 필요 (Windows JVM, macOS JVM 등)
- JVM은 **스택 기반(stack machine)**
---

## 1.2 Stack 기반 vs Register 기반 차이
### (1) 스택 기반 (JVM)
- 모든 연산을 스택 push/pop으로 수행
- 예: 4 + 5
```perl
iconst_4    ; 스택에 4 push
iconst_5    ; 스택에 5 push
iadd        ; 스택에서 두 값을 pop → 더한 값 push
istore_1    ; 결과를 지역 변수에 저장
```
스택을 통한 간접 접근 → 명령어 수 많음

### (2) 레지스터 기반 (Dalvik/ART)
- 연산을 가상 레지스터 v0, v1, v2 … 직접 사용
- 예: 4 + 5
```csharp
const/4 v0, #4   ; v0 = 4
const/4 v1, #5   ; v1 = 5
add-int v2, v0, v1 ; v2 = v0 + v1
```
스택 조작이 없어 효율적, 명령어 수 적음

---

## 2. Java 코드와 바이트코드 예시
```java
Java 코드
public static long sumArray(int[] arr) {
    long sum = 0;
    for (int i : arr) {
        sum += i;
    }
    return sum;
}
```
```yaml
JVM 바이트코드 (스택 기반)
000b: iload 05
000d: iload 04
000f: if_icmpge 0024
0012: aload_3
0013: iload 05
0015: iaload
0016: istore 06
0018: lload_1
0019: iload 06
001b: i2l
001c: ladd
001d: lstore_1
001e: iinc 05, +#01
0021: goto 000b
```
```yaml
Dalvik 바이트코드
0007: if-ge v0, v2, 0010
0009: aget v1, v8, v0
000b: int-to-long v5, v1
000c: add-long/2addr v3, v5
000d: add-int/lit8 v0, v0, #int 1
000f: goto 0007
```
<img width="342" height="154" alt="스크린샷 2025-08-20 오후 10 55 20" src="https://github.com/user-attachments/assets/c9346e3c-9705-4adb-b7c7-6fbd0b07ee6b" />


---

## 3. Class 파일과 DEX
- JVM: .class 여러 개, 상수풀(Constant Pool) 중복
- Dalvik: .dex로 합쳐서 Shared Constant Pool 사용
- 결과: 파일 크기↓, 메모리 사용↓, 실행 효율↑

```java
public interface Animal {
    void sound();
}

// 구현 클래스 1
public class Dog implements Animal {
    @Override
    public void sound() {
        System.out.println("멍멍");
    }
}

// 구현 클래스 2
public class Cat implements Animal {
    @Override
    public void sound() {
        System.out.println("야옹");
    }
}

// 실행 클래스
public class Main {
    public static void main(String[] args) {
        Animal dog = new Dog();
        Animal cat = new Cat();
        dog.sound();
        cat.sound();
    }
}
```
### 1) JVM에서 컴파일한 경우
위 코드를 javac로 컴파일하면 여러개의 .class 파일이 생성된다.
```ruby
Animal.class
Dog.class
Cat.class
Main.class
```
각 .class 파일 안에는 상수풀(Constant Pool) 이 존재하고, 예를 들어 "java/lang/Object", "sound()", "System.out" 같은 문자열/메서드 정보가 각 파일마다 중복 저장된다.
👉 문제: 클래스가 많아질수록 중복된 메타데이터가 계속 늘어남 → 메모리 낭비, 파일 크기 증가

### 2) Dalvik에서 DEX로 변환한 경우
- Animal.class, Dog.class, Cat.class, Main.class → 하나의 DEX 파일 내부에 포함
- Shared Constant Pool 적용 → 중복된 "sound()", "java/lang/Object" 같은 정보는 한 번만 기록
👉 결과: 파일 크기 감소 + 메모리 효율↑

### 3) 시각적 비교

JVM (.class 여러 개)
```cpp
[Animal.class]   → Constant Pool { "Object", "sound()", "System.out", ... }
[Dog.class]      → Constant Pool { "Object", "sound()", "System.out", ... } (중복)
[Cat.class]      → Constant Pool { "Object", "sound()", "System.out", ... } (중복)
[Main.class]     → Constant Pool { "Object", "sound()", "System.out", ... } (중복)
```

Dalvik (.dex 하나)
```csharp
[classes.dex] → Shared Constant Pool { "Object", "sound()", "System.out", ... } (1번만 저장)
                 ├─ Animal
                 ├─ Dog
                 ├─ Cat
                 └─ Main
```

---

## 4. 안드로이드 플랫폼 아키텍처
<img width="517" height="768" alt="스크린샷 2025-08-20 오후 10 41 58" src="https://github.com/user-attachments/assets/6044024a-b0d6-4880-91af-d318f154c3b8" />

- System Apps → 기본 앱
- Java API Framework → SDK 계층
- Native C/C++ Libraries → OpenGL, SQLite 등
- Android Runtime → Dalvik/ART + Core Libraries
- HAL + Linux Kernel → 하드웨어 제어
---

## 5. Dalvik VM
- 실행 단위: .dex
- 실행 방식: 인터프리트 + Trace JIT
- 설치 시 최적화 도구: dexopt
  - .dex → .odex(Optimized DEX) 변환
  - 중복 정보 제거, 실행 준비 테이블 생성
  - 설치는 빠르지만, 실행할 때 여전히 JIT 해석 부담이 큼
- 장점: 설치 빠름, 저장공간 적게 사용
- 단점: 실행 중 JIT 부담, 배터리 소모
---

## 6. ART
- 실행 단위: .dex
- Android 5.0: 설치 시 AOT(dex2oat) → .oat 생성
  - .dex → .oat(네이티브 코드, ELF 기반) + .vdex, .art 파일 생성
  - 실행 시 JIT 없이 바로 네이티브 실행 가능
- Android 7.0+: AOT + JIT 하이브리드
  - 앱 실행 중: 프로파일 기반 JIT 적용
  - 기기 유휴 시: dex2oat로 AOT 캐시 생성 → 다음 실행 시 빠름
- 장점: 실행 속도 빠름, GC 효율 개선
- 단점: (5.0 초기) 설치 시간 길고 저장 공간 많이 사용 → 이후 버전에서 개선

```cpp
Dalvik (Android 2.x ~ 4.x)
  .java → .class → .dex → [dexopt] → .odex → 실행(인터프리트 + JIT)

ART (Android 5.x ~)
  .java → .class → .dex → [dex2oat] → .oat/.vdex/.art(ELF) → 실행(AOT + JIT)
```

<img width="1180" height="619" alt="스크린샷 2025-08-20 오후 11 15 37" src="https://github.com/user-attachments/assets/69a98846-29e3-4344-9faf-df67223e6147" />

---

## 7. 실행 흐름 비교
<img width="655" height="628" alt="스크린샷 2025-08-20 오후 11 05 00" src="https://github.com/user-attachments/assets/56220ba2-70e7-405d-a70e-38f1b56a3575" />

---

## 8. Dalvik vs ART 비교
| 구분    | Dalvik VM         | ART                              |
| ----- | ----------------- | -------------------------------- |
| 아키텍처  | 레지스터 기반           | 레지스터 기반                          |
| 실행 방식 | 인터프리트 + Trace JIT | AOT + JIT 하이브리드                  |
| 실행 속도 | JIT 워밍업 필요 → 느림   | AOT 캐시로 빠름                       |
| 설치 속도 | 빠름                | (5.0) 느림 → (7.0+) 개선             |
| 파일    | `.dex`, `.odex`   | `.dex` → `.oat`, `.vdex`, `.art` |
---

## 9. Kotlin에서의 차이점
Kotlin도 최종적으로는 JVM 바이트코드 → DEX → ART 실행이므로 구조는 동일.
하지만 생성되는 바이트코드 패턴이 달라 몇 가지 차이가 있음.

- 프로퍼티 → getter/setter 메서드 생성
- 람다/고차 함수 → 익명 클래스 or invokedynamic으로 변환
- default parameter → synthetic 브릿지 메서드 증가
- data class → equals/hashCode/toString/copy 자동 생성 (메서드 수↑)
- suspend 함수 → 상태 머신(state machine) + Continuation 인자 추가
- inline 함수 → 호출부에 코드 삽입 (성능↑, 하지만 dex 크기↑ 가능)

👉 Kotlin은 편의 기능이 많아 바이트코드/DEX 크기와 메서드 수가 증가할 가능성이 있음.
하지만 ART의 JIT/AOT 최적화를 거치면 실행 성능은 Java와 큰 차이 없음.

---

## 10. 최종 요약
- JVM: PC/서버용, 스택 기반, .class 실행
- Dalvik: 안드로이드 초기 런타임, 레지스터 기반, .dex 실행, Trace JIT
- ART: 현재 런타임, .dex를 AOT + JIT로 최적화, 빠르고 효율적
- DEX: 여러 .class를 합쳐 상수풀 공유, 모바일 친화적 포맷
- Kotlin: 실행 경로는 같지만, 문법 특성상 바이트코드가 더 복잡 → 메서드 수↑, dex 크기↑ 가능
---


## 참고 링크 및 이미지 출처
[https://developer.android.com/guide/platform?hl=ko]

[https://www.youtube.com/watch?v=3mZukSTEE7A&ab_channel=SpringDrop]

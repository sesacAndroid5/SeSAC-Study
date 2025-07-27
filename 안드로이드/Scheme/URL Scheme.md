# URL Scheme, App Link, Deferred Deep Link 정리

## URL Scheme이란?

**앱을 특정 URL 패턴으로 직접 실행하거나 특정 화면으로 진입시키기 위해 정의하는 프로토콜**

웹 브라우저나 다른 애플리케이션이 특정 유형의 리소스에 접근하거나 특정 작업을 수행하기 위해 사용하는 식별자.

단순히 웹 주소(http:// 또는 https://)에 국한되지 않고 모바일 앱 간 통신, 특정 기능 실행 등 다양한 용도로 사용됩니다.

---

### URL Scheme의 기본 구조

`scheme://host:port/path?query#fragment`

- **`scheme` (스킴)** : 어떤 프로토콜이나 어떤 방식으로 리소스에 접근할지를 정의합니다.
- **`host` (호스트)** : 서버의 도메인 이름이나 IP 주소 (필수 아님)
- **`port` (포트)** : 서버의 특정 포트 번호 (필수 아님)
- **`path` (경로)** : 서버 내의 특정 리소스 경로 (필수 아님)
- **`query` (쿼리)** : 추가 매개변수 전달 (ex. `?id=1&name=Bob`)
- **`fragment` (프래그먼트)** : URL의 특정 부분 가리킴 (필수 아님)

---

### 일반적인 URL Scheme 예시

- **웹 접근:** `https://www.google.com`
- **이메일 작성:** `mailto:example@email.com`
- **전화 걸기:** `tel:01012345678`
- **파일 열기:** `file:///C:/Users/user/document.pdf`

---

### 모바일 앱에서의 URL Scheme 활용

Android 앱에서는 개발자가 자신의 앱을 특정 URL Scheme에 연결하여 다른 앱이나 웹에서 해당 Scheme을 호출했을 때 앱이 실행되도록 설정합니다.

**사용 목적**
- **앱 간 연동**: 인스타 게시물 → 카카오톡 공유
- **웹에서 앱 실행**: 웹 배너 클릭 시 앱의 상품 상세 화면 이동
- **푸시 알림 → 특정 화면 이동**

---

### Android에서 Scheme 등록 방법

`AndroidManifest.xml`에 Intent Filter 추가

```xml
<activity
    android:name=".MainActivity"
    android:exported="true">
  <intent-filter>
      <action android:name="android.intent.action.VIEW" />
      <category android:name="android.intent.category.DEFAULT" />
      <category android:name="android.intent.category.BROWSABLE" />
      <data android:scheme="myapp" android:host="home"/>
  </intent-filter>
</activity>
```

- `<intent-filter>` : 외부에서 특정 Intent가 들어왔을 때 반응할 조건
- `<action android:name="android.intent.action.VIEW"/>` : URL 요청을 받을 준비
- `<category android:name="android.intent.category.BROWSABLE"/>` : 브라우저에서 Scheme 클릭 가능
- `<data android:scheme="myapp" android:host="home"/>` : myapp://home 처리

---

### 동작 흐름

1. 사용자가 웹에서 `myapp://home` 클릭
2. 안드로이드가 처리 가능한 앱 검색
3. Manifest 내 Scheme 조건과 일치하는 액티비티 실행
4. `Intent.getData()`로 URL 정보 전달
5. 앱이 URL 파라미터 확인 후 해당 화면 이동

---

### URL Scheme 한계

- **중복 문제**: 여러 앱이 같은 Scheme 사용 가능
    - 예) `market://` → Google Play, 원스토어, 갤럭시 스토어 모두 사용
- 특정 앱만 열리지 않고 **Chooser** 창에서 선택해야 할 수 있음

---

## App Link (앱 링크)

- Android: App Links
- iOS: Universal Links

**웹 URL(http/https)** 을 사용하여 앱의 특정 화면으로 바로 연결하는 **고급 딥링크 기술**

### 특징

- **웹 URL 형태 유지** (ex. `https://shoppingmall.com/products/item1`)
- 앱 설치 → 바로 앱으로 이동
- 앱 미설치 → 웹 브라우저로 연결
- Scheme 중복 문제 없음 (실제 도메인 기반)

### 활용 사례

- 친구가 공유한 상품 링크 클릭 시 앱 설치 여부에 따라 앱 or 웹으로 이동
- 이메일 캠페인 링크 클릭 시 프로모션 화면으로 연결

---

## Deferred Deep Link (디퍼드 딥링크)

**앱이 설치되지 않은 상태**에서 딥링크 클릭 → 앱 설치 → **첫 실행 시 해당 콘텐츠로 이동**

### 동작 방식
1. 링크 클릭
2. 앱 설치 여부 확인
3. 미설치 → 스토어로 이동
4. 앱 설치 후 첫 실행 시 **원래 클릭했던 콘텐츠**로 이동

### 활용 사례
- 광고 캠페인: "특가 상품" 광고 클릭 → 앱 설치 → 앱 열면 바로 특가 상품 페이지
- 친구 초대: 초대 링크 클릭 → 앱 설치 → 실행 시 친구 프로필 화면 열림

---

## App Link vs Deferred Deep Link

| 특징 | App Link (Android App Links / iOS Universal Links) | Deferred Deep Link |
| --- | --- | --- |
| **URL 형태** | http/https | http/https |
| **핵심 목적** | 앱이 설치된 사용자에게 원활한 연결 제공 | 앱이 **설치 안 된 사용자**도 설치 후 연결 |
| **앱 미설치 시** | 웹 브라우저로 연결 | 앱 스토어로 연결 후 설치 유도 |
| **주요 사용처** | 웹-앱 연동, SEO, 기존 사용자 대상 | 신규 사용자 유치, 광고 캠페인 |

✅ **App Link** → 설치된 사용자 경험 강화  
✅ **Deferred Deep Link** → **설치 전 사용자**도 원하는 화면으로 연결

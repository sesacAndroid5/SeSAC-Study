# Gson vs Moshi

## JSON이란?

- **JSON(JavaScript Object Notation)**은 웹 서버와 클라이언트 간 데이터 교환을 목적으로 하는 경량의 데이터 교환 형식입니다.
- 자바스크립트 내에 데이터를 포함할 수 있으며, 다양한 언어에서 지원합니다.

### JSON의 기본 구조

- **name:value** 형태의 쌍으로 이루어진 컬렉션 타입 : `JSONObject`
- **값들의 순서화된 리스트** : `JSONArray`

---

## Gson vs Moshi

| 구분         | Gson                           | Moshi                                   |
|--------------|-------------------------------|------------------------------------------|
| 개발사       | Google                        | Square                                   |
| 언어 지원    | Java, Kotlin                  | Java, Kotlin                             |
| 성능         | 보통                          | 더 빠름, 메모리 사용 적음                |
| Kotlin 호환성| 일반적                        | 뛰어남                                   |
| 코드 생성    | 미지원                        | 지원 (codegen)                           |
| 유지관리     | 유지보수 모드(신규 기능 없음) | 활발하게 관리됨                          |
| 커스터마이징 | 제한적                        | 강력함                                   |

- **Moshi**는 Kotlin과의 호환성이 뛰어나고, 성능이 더 빠르며, 코드 생성 기능(codegen)으로 런타임 오버헤드가 적습니다.
- **Gson**은 오랫동안 표준처럼 쓰였으나, 최근에는 유지보수만 되고 있어 새로운 기능 추가는 기대하기 어렵습니다.

---

## 직렬화/역직렬화 라이브러리란?

- 객체를 JSON 문자열로 변환(직렬화)하거나, JSON 문자열을 객체로 변환(역직렬화)하는 기능을 제공하는 라이브러리입니다.

---

## REST에서의 데이터 교환 방식

- **REST API**는 서버와 클라이언트가 주로 JSON 포맷으로 데이터를 주고받는 구조입니다.
- 클라이언트는 객체(예: Kotlin/Java 클래스)를 JSON 문자열로 변환(직렬화)하여 서버에 요청을 보내고, 서버는 JSON 응답을 다시 객체로 변환(역직렬화)합니다.
- 이 과정에서 JSON ↔ 객체 변환이 필수적으로 반복됩니다.

### 실제 연동 흐름 예시

1. **클라이언트에서 REST API 호출**
    - 요청 객체를 Gson/Moshi로 JSON 문자열로 변환 → HTTP 바디에 포함해 전송
2. **서버에서 JSON 응답 수신**
    - 받은 JSON 문자열을 Gson/Moshi로 객체로 변환하여 앱에서 사용

---

# Gson과 Moshi

## Gson

### 개요

- GSON은 JSON ↔ Java(Kotlin) Object 변환(직렬화/역직렬화)을 도와주는 라이브러리입니다.
- [공식 문서](https://github.com/google/gson)
- [Gson Tutorial](https://www.tutorialspoint.com/gson/index.htm)

### 의존성 추가
```kotlin
dependencies {
implementation 'com.google.code.gson:gson:2.11.0'
}
```
---

### GSON read 기본 예제
```kotlin
data class Car(
  @SerializedName("kind")
  @Expose
  val mark: String = "",
  val model: String = "",
  val type: String = "",
  val maker: String = "",
  val cost: Int = -1,
  val colors: MutableList<String> = mutableListOf()
)

fun main() {
  val gson = Gson()
// val gson = GsonBuilder().create()
  val carObj = gson.fromJson(inputGSONType, Car::class.java)
  println(carObj.colors)
  println(carObj)
}

```



### GSON write 기본 예제

```kotlin
fun main() {
val colors = mutableListOf("GRAY", "BLACK", "WHITE")
val car = Car("AUDI", "2014", "PETROL", "AUDI Germany", 30000, colors)
val gson = GsonBuilder().create()
println(gson.toJson(car))
}
```

---


## Moshi

### Moshi 역직렬화 예제

```kotlin
const val inputJSONCarInfo = """
{
"mark": "소나타",
"model": "2017년도모델",
"type": "중형",
"maker": "Hyundai",
"cost": 45000,
"colors": ["GRAY", "BLACK", "WHITE"]
}
"""

fun main() {
val moshi = Moshi.Builder().addLast(KotlinJsonAdapterFactory()).build()
val moshiAdapter: JsonAdapter<Car> = moshi.adapter(Car::class.java)
val car: Car = moshiAdapter.nullSafe().fromJson(inputJSONCarInfo)!!
println(car.toString().replace(",", " ||"))
}
```


### Moshi 직렬화 예제

```kotlin
fun main() {
Car(
mark = "그랜저",
model = "2024년 모델",
type = "중형",
maker = "현대",
cost = 67000,
colors = mutableListOf("GRAY", "BLACK", "WHITE")
).apply {
val moshi: Moshi = Moshi.Builder().addLast(KotlinJsonAdapterFactory()).build()
val jsonAdapter: JsonAdapter<Car> = moshi.adapter(Car::class.java)
println(jsonAdapter.toJson(this))
}
} 
```


---

## 표준 API 와 Moshi 역직렬화 차이

### 표준 API 역직렬화 예제

```kotlin
fun main() {
val rootJSON = JSONObject(jsonString)
val personInfo = PersonInfo(
rootJSON.optString("name"),
rootJSON.optString("alias"),
rootJSON.optBoolean("isAlive"),
rootJSON.optInt("age"),
rootJSON.optFloat("height_cm"),
rootJSON.optJSONObject("address"),
rootJSON.optJSONArray("phoneNumbers"),
rootJSON.optJSONArray("hobby"),
rootJSON.optBoolean("spouse")
).apply {
val addressInfo = Address(
address.optString("streetAddress"),
address.optString("city"),
address.optString("state"),
address.optString("postalCode")
)
println("""addressInfo: ${addressInfo.toString()}""")
}.apply {
phoneNumbers.forEach {
val phoneJSON = it as JSONObject
val phoneInfo = PhoneNumberInfo(
phoneJSON.optString("type"),
phoneJSON.optString("number")
)
println(phoneInfo.toString())
}
}.apply {
hobby.forEach(::println)
}
println(personInfo.toString())
}
```

### Moshi PersonInfo 역직렬화 예제

```kotlin
fun main() {
val moshi = Moshi.Builder().addLast(KotlinJsonAdapterFactory()).build()
val moshiAdapter: JsonAdapter<PersonInfo> = moshi.adapter(PersonInfo::class.java)
val person: PersonInfo = moshiAdapter.nullSafe().fromJson(personJsonString)!!
println(person)
}
```

---

## Annotation

### Gson

- `@Expose`: object 중 해당 값이 null일 경우, json으로 만들 필드를 자동 생략
  @Expose(serialize = false)
  private int model;
  @Expose(serialize = false, deserialize = false)
  private int model;
  val gson = GsonBuilder().excludeFieldsWithoutExposeAnnotation().create()


- `@SerializedName`: JSON으로 serialize 될 때 매칭되는 이름을 명시하는 목적
  @SerializedName("index_name") // index_name json key에 indexName을 설정
  @Expose // null이면 생략
  private String indexName

### Moshi

- `@Json` : SON의 필드명과 클래스의 프로퍼티명이 다를 때 매핑을 지정
  @Json(name = "user_id") val userId: Long


- `@JsonClass(generateAdapter = true)` : 해당 데이터 클래스에 대한 JSON 어댑터를 컴파일 타임에 생성하도록 Moshi에 지시
  @JsonClass(generateAdapter = true)
  data class User(
  @Json(name = "user_id") val userId: Long,
  val name: String
  )

### 차이점
| 구분         | Gson                      | Moshi                                   |
|------------|-----------------------------|------------------------------------------|
| 어노테이션 이름   | @SerializedName        | @Json(name = ...)                                   |
| 어댑터 생성     | 런타임 리플렉션 기반	       | @JsonClass(generateAdapter = true)로 컴파일타임 생성                             |
| 코드 생성 지원   | 보통                          | 더 빠름, 메모리 사용 적음                |
---

## 참고

- Gson: [https://github.com/google/gson](https://github.com/google/gson)
- Moshi: [https://github.com/square/moshi](https://github.com/square/moshi)
- Gson Tutorial: [https://www.tutorialspoint.com/gson/index.htm](https://www.tutorialspoint.com/gson/index.htm)

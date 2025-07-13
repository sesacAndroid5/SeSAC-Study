# Compose에서 `AndroidView`와 `AndroidViewBinding`의 차이

## 핵심 차이 비교

| 항목 | `AndroidView` | `AndroidViewBinding` |
|------|----------------|------------------------|
| 주요 목적 | Compose에 없는 View(WebView, MapView 등)를 포함 | 기존 XML 레이아웃을 Compose에서 재사용 |
| View 생성 | `View` 객체를 직접 생성 | `ViewBinding` 객체를 `inflate` 하여 View 생성 |
| 요소 접근 | `findViewById()` 등 수동 방식 | 바인딩 객체로 type-safe하게 접근 |
| 적합한 상황 | 특수 View 또는 간단한 View 삽입 | 복잡하고 안정된 XML을 재활용할 때 |

---

## 동작 구조

### `factory` 블록
- 최초 컴포지션 시 1회 실행됨
- `AndroidView`: `View` 객체 직접 생성
- `AndroidViewBinding`: 바인딩 객체를 생성하고 내부적으로 XML 레이아웃을 `LayoutInflater`로 inflate

### `update` 블록
- 뷰가 이미 생성된 후 **리컴포지션이 필요한 경우** 조건적으로 실행됨
- View 상태를 최신 Compose 상태와 동기화하는 역할

#### 지나다 프로젝트에서의 AndroidViewBinding factory 블록 과 update 블록
```kotlin
AndroidViewBinding(
        modifier = Modifier,
        factory = { inflater, parent, attachToParent -> // map_container_layout.xml파일을 이용해 생성
            MapContainerLayoutBinding.inflate(inflater, parent, attachToParent)
        },
        update = {
            if (mapView == null) { // 팩토리 블록 실행 이후 최초 실행시에 상태변수인 mapView를 해당 맵뷰로 변경
                mapView = this.mapView
            } else { // 이후 리컴포지션시마다 사용자 위치 오버레이 및 마커, 인포윈도우 이벤트를 지도에 업데이트 
                mapController.updateMyLocationOverlay(myPosition)
                mapController.updateMemoMarkers(memoList,{ newSelectedMarkerId -> selectedMarkerId = newSelectedMarkerId })
                mapController.showInfoWindow(memoList,selectedMarkerId)
            }

        }
    )
```

### 생명주기 처리 시 주의점
Compose는 선언적 UI 프레임워크이므로 내부적으로 상태 변화에 따라 뷰를 재구성합니다. 그러나 AndroidView 및 AndroidViewBinding으로 생성된 뷰는 View System의 생명주기를 따르기 때문에 아래와 같은 관리가 필요합니다.

View 재생성 방지: remember를 사용해 객체의 재생성을 막아야 불필요한 초기화 방지

리소스 정리: DisposableEffect 또는 LifecycleOwner를 사용해 뷰 해제, 리스너 제거 등의 정리 작업 수행
#### 지나다 프로젝트에서의 네이버 지도뷰 생명주기 이벤트 처리
```kotlin
@Composable
fun MapLifecycleEffect(context: Context, mapView: MapView?, myPosition: LatLng, memoList: List<TodoItemData>, mapController: MapController, onMapLongClick: (String)-> Unit, onMarkerClick: (Int) -> Unit) {
    val lifecycleOwner = LocalLifecycleOwner.current
    DisposableEffect(lifecycleOwner, mapView) {
        val observer = object : DefaultLifecycleObserver {
            override fun onCreate(owner: LifecycleOwner) {
                mapView?.let { mapView ->
                    mapView.onCreate(null)
                    mapView.getMapAsync { naverMap -> // 맵뷰 최초 생성시에 사용자 위치 오버레이 및 지도 이벤트 등록 처리
                        mapController.initMapController(naverMap,context)
                        mapController.setCameraPosition(myPosition)
                        mapController.updateMyLocationOverlay(myPosition)
                        mapController.setMapLongClickListener(onMapLongClick)
                        mapController.updateMemoMarkers(memoList, onMarkerClick)
                    }
                }
            }

            override fun onStart(owner: LifecycleOwner) {
                mapView?.onStart()
            }

            override fun onResume(owner: LifecycleOwner) {
                mapView?.onResume()
            }

            override fun onPause(owner: LifecycleOwner) {
                mapView?.onPause()
            }

            override fun onStop(owner: LifecycleOwner) {
                mapView?.onStop()
            }
        }

        lifecycleOwner.lifecycle.addObserver(observer)

        onDispose {
            mapView?.onDestroy()
            lifecycleOwner.lifecycle.removeObserver(observer)
        }
    }
}
```

## 요약
AndroidView: 화면에 특수한 View(WebView, MapView 등)을 사용하고 나머지 UI는 컴포즈를 이용하고자 할 때. 복잡한 xml 사용시 내부 뷰 접근은 수동으로 처리해야 함.

AndroidViewBinding: 복잡한 XML 레이아웃 재활용에 적합. 타입 안정성과 코드 간결성이 장점.

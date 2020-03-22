# 2020-03-mvvm
## MVVM
* [Android Architecture starring Kotlin Coroutines, Jetpack (MVVM, Room, Paging), Retrofit and Dagger 2](https://proandroiddev.com/android-architecture-starring-kotlin-coroutines-jetpack-mvvm-room-paging-retrofit-and-dagger-7749b2bae5f7)
    * [LEGO-Catalog](https://github.com/Eli-Fox/LEGO-Catalog)

## Member
* [delpini](https://github.com/delpini)
* [na-in-lee](https://github.com/na-in-lee)

## Goals
1. MVVM, Retrofit을 사용하여 Google Books API 프로젝트 개발
2. Unit test
3. Jetpack (Navigation/Paging/Room)
4. Dagger (DI), Rx / Coroutine

## 3/25 (수)
### TODO
- 심플한 MVVM 프로젝트 찾아보기
    - https://github.com/android-ioi/architecture-samples
- MVVM without DataBinding가 있는지?
    - https://stackoverflow.com/questions/48292979/android-mvvm-without-databinding
- Dagger, Coroutine은 다음 스터디때 따로. MVVM 프로젝트는 간단하게.

## 3/11 (수)
### Lego Catalog 코드 분석
#### Navigation
* @navigation/nav_main xml에 navigation 액션들이 정의되어 있음
* fragment간 argument가 전달될 경우, xml에 정의하고, 자동으로 generated되는 소스 코드를 통해 값을 전달

``` kotlin
private fun createOnClickListener(id: Int, name: String): View.OnClickListener {
    return View.OnClickListener {
        val direction = LegoThemeFragmentDirections.actionThemeFragmentToSetsFragment(id, name)
        it.findNavController().navigate(direction)
    }
}
```
* 전달된 argument는 navArgs()로 args를 가져와서 사용

``` kotlin
private val args: LegoSetFragmentArgs by navArgs()
```

#### MVVM
![Overview MVVM](images/overview-mvvm.png)

기본적으로 Dagger와 MVVM으로 클래스간 의존성을 낮춰서 UnitTest가 용이한 구조로 되어 있음

#### Unit Test
Coroutine + MockWebServer로 테스트 구현

``` kotlin
@Test
fun getLegoSetsResponse() {
    runBlocking {
        enqueueResponse("legosets.json")
        val resultResponse = service.getSets().body()
        val legoSets = resultResponse!!.results

        assertThat(resultResponse.count, `is`(9))
        assertThat(legoSets.size, `is`(9))
    }
}

...

private fun enqueueResponse(fileName: String, headers: Map<String, String> = emptyMap()) {
    val inputStream = javaClass.classLoader
            .getResourceAsStream("api-response/$fileName")
    val source = Okio.buffer(Okio.source(inputStream))
    val mockResponse = MockResponse()
    for ((key, value) in headers) {
        mockResponse.addHeader(key, value)
    }
    mockWebServer.enqueue(mockResponse.setBody(
            source.readString(Charsets.UTF_8))
    )
}
```

#### Dagger, Coroutine
* Dagger를 사용하여 클래스간 의존성을 낮춤
* Coroutine으로 네트워크 통신 등을 구현
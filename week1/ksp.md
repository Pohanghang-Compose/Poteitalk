https://velog.io/@kwan_hee/kapt-%EC%97%90%EC%84%9C-ksp%EB%A1%9C-%EB%A7%88%EC%9D%B4%EA%B7%B8%EB%A0%88%EC%9D%B4%EC%85%98-%ED%95%B4%EB%B3%B4%EA%B8%B0

## 들어가기 앞서

kapt에서 ksp로 마이그레이션 하려는 이유가 뭘까요?

- **빌드 속도 향상** : Java 스텁을 만드는 kapt에서는 ksp 보다 빌드 속도가 늦어질 수 밖에 없습니다. 기본적으로 Android Studio 컴파일은 kotlinc 입니다. 그렇기 때문에 Kotlin 코드를 Java 스텁으로 변환하는 과정의 과정이 필요해집니다. 즉, ksp 는 Java 스텁으로 변환해야하는 과정이 필요없는 것입니다.
- **Kotlin 언어 사용** : ksp는 Kotlin 언어에 맞게 설계되어 더 잘 이해하고 활용할 수 있습니다.

이제 실제로 어떻게 ksp를 적용하고 시행착오들이 있었는지 아래에서 설명하도록 하겠습니다.

## Hilt

Hilt를 기준으로 kapt에서 ksp를 변경할 경우를 예시로 들어보려고 합니다.

- build.gradle(:project)

```kotlin
id("com.google.dagger.hilt.android") version "2.48" apply false
```

- build.gradle(:app)

```kotlin
plugins {
  id ("kotlin-kapt")
  id ("com.google.dagger.hilt.android")
}

dependencies {
    implementation ("com.google.dagger:hilt-android:2.48")
    kapt ("com.google.dagger:hilt-compiler:2.48")
}
```

kapt를 사용하는 경우에는 위처럼 진행됩니다.

하지만 ksp를 사용할 때는 어떤점을 고려하고 어떤 종속성을 추가해야할까요?

https://developer.android.com/build/migrate-to-ksp?hl=ko#kts 자세한 설명은 공식문서를 참고해주세요.

우선 대표적으로 Dagger, Room, Glide, Moshi 라이브러리는 ksp를 지원하고 있습니다. 하지만 더 많은 라이브러리들이 ksp를 지원하고 있으며, 어떤 라이브러리가 KSP를 지원하는 지 확인하려면 https://kotlinlang.org/docs/ksp-overview.html#supported-libraries 해당 링크를 들어가보면 됩니다.

- build.gradle(:project)

```groovy
pulgins {
    id("org.jetbrains.kotlin.android") version "1.9.21" apply false
  id("com.google.devtools.ksp") version "1.9.21-1.0.16" apply false
}
```

KSP 플러그인 등록 시 버전을 등록할 때는 https://github.com/google/ksp/releases?page=1 릴리즈 노트를 확인하여 버전을 맞춰줘야 합니다. 특히 코틀린 버전과 맞추어야 하기 때문에 관련된 코틀린 버전에 대응되는 버전을 사용해주면 됩니다.

추가로, Hilt ksp는 해당 요구사항을 따라야 합니다. 업데이트 될 수 있으니 https://dagger.dev/dev-guide/ksp 해당 문서를 참고해주세요!

- **Hilt 버전 2.48 이상**
- **Kotlin 버전 1.9.0 이상**
- **KSP 버전 1.9.0-1.0.12 이상**

- build.gradle(:app)

```groovy
plugins {
  id ("com.google.devtools.ksp")
  id ("com.google.dagger.hilt.android")
}

dependencies {
    implementation ("com.google.dagger:hilt-android:2.48")
    ksp ("com.google.dagger:hilt-compiler:2.48")
}
```

하지만 아직 Hilt와 Dagger는 Alpha & In progress 이기 때문에 Officially supported 될 때까지 상황을 지켜봐야할 것 같습니다.

![스크린샷 2024-12-12 오후 3.55.57.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/b9bb600f-e8b1-4fa1-b9b5-1e98783c5507/2acd699c-99b0-49d3-807f-94b0325c4744/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-12-12_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_3.55.57.png)

![스크린샷 2024-12-12 오후 3.56.03.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/b9bb600f-e8b1-4fa1-b9b5-1e98783c5507/bf52ce89-6ba4-447b-bd36-3db2f93d5766/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-12-12_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_3.56.03.png)

## Glide

Glide ksp를 추가하기 위해서는 조금 다른 라이브러리를 추가해야합니다.

기존에 kapt를 사용한 경우에는 glide:compiler를 추가하지만, ksp를 사용할 경우에는 glide:ksp를 사용해야합니다.

https://sjudd.github.io/glide/doc/download-setup.html#kotlin---ksp

```kotlin
// build.gradle(:app)

dependencies {
    implementation("com.github.bumptech.glide:glide:4.14.2")
  kapt("com.github.bumptech.glide:compiler:4.14.2")
    ksp("com.github.bumptech.glide:ksp:4.14.2")
}
```

Glide 버전을 해당 링크에서 살펴볼 수 있습니다. https://github.com/bumptech/glide/releases

### Glide 4.9.0 버전 사용 시, KSP 를 적용할 때 문제가 발생했습니다.

필자는 Glide `4.9.0` 버전을 사용하였고, KSP 는 `1.9.22-1.0.16` 버전을 사용했습니다.

4.9.0 ~ 4.13.0 버전까지 아래와 같은 에러가 발생했습니다. 

task는 4개이지만 모듈별로 겹치는 것이 있어서 총 3개의 task 에서 오류가 발생했습니다.

- javaPreCompileDebug : 자바 컴파일이 실패했습니다.
- kspDebugKotlin : ksp 를 사용하여 처리하는 과정이 실패했습니다.
- mergeExtDexDebug : 외부 라이브러리에서 DEX 파일을 병합하는 과정에서 실패했습니다.

![스크린샷 2024-12-13 오후 9.46.58.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/b9bb600f-e8b1-4fa1-b9b5-1e98783c5507/a8444733-ad80-4f58-9539-0e36b74323ee/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-12-13_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_9.46.58.png)

하지만, 4.14.0 버전부터는 빌드가 성공했습니다! 어떤 일이 벌어진 걸까요?

이유는 단순합니다. 해당 버전부터 KSP를 지원하기 시작했습니다. 해당 버전부터 KSP 를 지원하기 시작했습니다!!

![스크린샷 2024-12-13 오후 10.01.23.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/b9bb600f-e8b1-4fa1-b9b5-1e98783c5507/3bfb845a-fc5e-49cf-a0f2-49d0d88ffb11/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-12-13_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_10.01.23.png)

근데 저는 4.14.2 버전으로 사용해야 제대로 동작하더라구요.. 

아마도 이유는 여러 모듈에서 Glide를 사용하다보니 4.14.2 패치에서 버그 중 해결된 하나의 문장을 가져와봤습니다. 해당 이슈이지 않을까 싶습니다. Bugs 첫 번째 문장에 **Allow LibraryGlideModules to be processed in separate code modules when using KSP** 라고 말하고 있습니다. **KSP를 사용할 때, LibraryGlideModules 가 모듈 별 분리된 코드에 적용되도록 동작을 허용한다고 합니다.** 저는 core 모듈과 koin(app) 모듈 두 곳에서 라이브러리를 추가하고 사용하기 때문에 발생한 이슈같습니다.

![스크린샷 2024-12-13 오후 9.57.39.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/b9bb600f-e8b1-4fa1-b9b5-1e98783c5507/242ad02e-8385-4c44-81b4-a03378d2658b/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-12-13_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_9.57.39.png)

그런데 또 다른 이슈가 발생했습니다. 다른 이슈는 무엇일까요? 

### IllegalStateException: GeneratedAppGlideModuleImpl is implemented incorrectly. (에러 발생!)

위에서 KSP 를 사용하기 위해서 Glide 버전을 맞추어 줬는데, 아래와 같은 에러가 발생했습니다. 어떤 에러인지 확인해보니 generated API를 사용할 경우에 발생하는 에러인 것 같습니다.

```kotlin
java.lang.IllegalStateException: GeneratedAppGlideModuleImpl is implemented incorrectly. 
If you've manually implemented this class, remove your implementation. The Annotation processor 
will generate a correct implementation. (Ask Gemini)
    at com.bumptech.glide.Glide.throwIncorrectGlideModule(Glide.java:292)
    at com.bumptech.glide.Glide.getAnnotationGeneratedGlideModules(Glide.java:284)
    at com.bumptech.glide.Glide.get(Glide.java:128)
    at com.bumptech.glide.Glide.getRetriever(Glide.java:510)
    at com.bumptech.glide.Glide.with(Glide.java:623)
```

내부 구현을 살펴보면 GeneratedAppGlideModuleImpl 함수에서 NoSuchMethodException 예외를 catch 하여 에러를 던지고 있습니다. 저는 단순히 Glide만 사용했는데.. 어떤 이유일까요?

![스크린샷 2024-12-13 오후 10.05.16.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/b9bb600f-e8b1-4fa1-b9b5-1e98783c5507/1fbe689e-ecc4-4915-bca6-2f8ade2facf8/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-12-13_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_10.05.16.png)

작성된 Glide 코드는 apply 정적 메소드를 사용하고 있으며, generated API 이기 때문에 발생하는 에러 같습니다. 거의 모든 Glide 로직에서는 apply를 사용하고 있습니다. 즉, generated API를 사용하고 있습니다.

```kotlin
private val glideOptions: RequestOptions = RequestOptions()
  .fitCenter()
  .error(R.drawable.image_no_image)
  .placeholder(R.color.white)
        
        
Glide.with(...)
    .apply(glideOptions)
```

위에 문제를 해결할 수 있는 대답은 Glide 공식문서에서 살펴볼 수 있습니다.

아래의 Note를 살펴보면, KSP 프로세서는 Glide 의 deprecated 된 generated API를 사용하지 않는다고 하고 있습니다. 예를 들면, GlideApp, GlideRequests 등등 말하며, non-generated 된 것으로 대체가 필요하다고 말하고 있습니다.

![스크린샷 2024-12-13 오후 10.09.35.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/b9bb600f-e8b1-4fa1-b9b5-1e98783c5507/580368ca-91a9-45f8-b604-1d15cf22f0aa/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-12-13_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_10.09.35.png)

파란색 링크 들어가봐아겠죠? Generated API가 뭔지 확인해봐야겠습니다.

들어가보니 떡하니 이런 말이 쓰여있습니다. Glide 4.14.0 버전부터는 generated API는 deprecated 되었다고 합니다. 사용하지 않겠다는 것이죠. 추가적인 Glide의 어노테이션 프로세서를 확인하려면 configuration 문서를 확인해보라고 합니다.

![스크린샷 2024-12-13 오후 10.12.08.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/b9bb600f-e8b1-4fa1-b9b5-1e98783c5507/078795e7-0822-40db-b8dd-61cf2d080597/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-12-13_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_10.12.08.png)

그렇다면 generated class 는 어떤 것들이 있을까요?

GlideApp, GlideRequests, GlideRequest, GlideOptions 4가지가 존재합니다. 대신 사용되어야할 API도 설명해주고 있습니다.

![스크린샷 2024-12-13 오후 10.23.23.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/b9bb600f-e8b1-4fa1-b9b5-1e98783c5507/75d7f71b-e616-4c82-a544-b0e550360f3d/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-12-13_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_10.23.23.png)

근데 왜 generated API를 사용하지 않을까요? 이유에 대해서 공식문서에서는 4가지를 설명하고 있습니다.

- Glide 4.9.0 버전부터 상속을 사용하며 RequestOptions를 RequestBuilder 내부로 통합했기 때문입니다.
- generated API에 존재하는 Extensions 들은 드물게 사용되는 것이 확인되었기 때문입니다. (거의 사용되지 않는다는 것이죠)
- Extensions can be trivially replicated in Kotlin using extension functions with no additional support from Glide.
    - 세 번째 이유가 위의 문장인데, 뭔 뜻인지 모르겠어서…
    - 번역만 하자면, **Glide의 추가 지원없이 확장을 사용하여 코틀린에서 Extensions을 간단하게 복제할 수 있다**고 하는데, 기존 generated class 에서는 Glide의 지원으로 확장함수를 사용할 수 있었나 봅니다. 즉, RequestOptions가 RequestBuild 내부로 통합하면서 RequestOptions 을 호출하지 않고 RequestBuilder 만 사용해서 API를 호출할 수 있다는 건지.. 모르겠네요.
- Generate Class 인 GlideApp, GlideRequests 등등 클래스를 생성하면, Glide의 빌드 프로세서의 빌드 시간과 복잡도가 올라가기 때문입니다.

그래서 위와 같은 4가지 이유로 generated API를 사용하지 않는다고 합니다.

공식문서에서는 실제로 Glide의 자바 기반의 어노테이션 프로세서를 지울 계획은 없다고 합니다. 하지만 generated API를 KSP에 추가하거나 관련 기능은 추가할 계획이 없다고 합니다. KSP 에서 Glide 의 빌드 프로세스를 단축하기 위한 모습이 보입니다.

우선, 처음에 설명한 Glide 4.9.0 버전의 패치 노트를 살펴보면 아래와 같습니다.

apply 로 정적 메소드를 사용하여 RequestOptions 인 ceneterCropTransfrom 을 사용합니다. 추가적으로 GlideApp 어노테이션 프로세서를 사용하는 방법을 이전 버전에서는 사용했습니다. 

하지만 RequestOptions를 RequestBuilder 내부로 통합하면서 ceneterCrop 만 호출하여 generated API 또는 어노테이션 프로세서를 사용하지 않고 호출할 수 있습니다.

![스크린샷 2024-12-13 오후 9.35.41.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/b9bb600f-e8b1-4fa1-b9b5-1e98783c5507/e2dc0c77-c716-40a2-bd8d-900afbf8542e/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-12-13_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_9.35.41.png)

일단 Glide KSP를 적용하기 위해서 generated API를 지원하지 않아서 사용하면 안된다는 것은 알겠는데, 그렇다고 발생한 에러를 어떻게 해결하나요?

Glide v4의 어노테이션 프로세서와 생성된 API는 Glide의 API를 확장하는 데 유용한 도구였습니다. 

하지만 Kotlin 확장 함수가 등장하면서 생성된 API는 더 이상 권장되지 않습니다. Glide는 향후 생성된 API를 제거할 계획이지만,  configuration option은 계속 사용됩니다. 

그래서 KSP 를 사용할 경우에는 configuration option 이 사용되어야 해서 Configurtion 을 설정해주어야 해당 에러가 나타나지 않습니다.

https://bumptech.github.io/glide/doc/configuration.html#avoid-appglidemodule-in-libraries

우선 애플리케이션에는 단 하나의 **AppGlideModule** 을 상속하는 GlideModule을 만들어주어야 합니다.

만약에 애플리케이션이 아니라 라이브러리라면 **LibraryGlideModule** 을 상속하는 GlideModule을 만들어주면 됩니다.

코드는 아래와 같습니다.

```kotlin
@GlideModule
class GlideModule: AppGlideModule()
```

```kotlin
@GlideModule
class GlideLibraryModule: LibraryGlideModule()
```

실제로 위에 GlideModule을 추가해주고 에러가 나지 않으며, 빌드해보면 아래처러 ksp 폴더가 생기고 GlideModuleImpl 구현체가 만들어집니다.

![스크린샷 2024-12-13 오후 11.17.06.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/b9bb600f-e8b1-4fa1-b9b5-1e98783c5507/efb15805-a681-4882-9941-949e2369ba8b/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-12-13_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_11.17.06.png)

### 난독화(R8) 문제가 발생했습니다.

```kotlin
isMinifyEnabled = true
```

GeneratedAppGlideModuleImpl is defined multiple times 에러 문구가 뜨면서, 난독화를 할 때 GeneratedAppGlideModuleImpl 클래스가 여러 번 정의되어있기 때문에 발생하는 에러였습니다.

![스크린샷 2024-12-15 오후 8.51.30.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/b9bb600f-e8b1-4fa1-b9b5-1e98783c5507/c8241cf1-480d-4e57-a809-1a675b242eeb/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-12-15_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_8.51.30.png)

위에서 어노테이션 프로세서를 만들기 위해서 Empty GlideModule을 만들어주었습니다. GlideModule을 만들어주기 때문에 또 다른 GeneratedAppGlideModuleImpl 클래스가 생성되어서 생기는 문제였습니다.

문제는 기존에 자바 레거시를 가지고 있던 코드에서 사용된 라이브러리 문제였습니다.

![스크린샷 2024-12-15 오후 10.29.56.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/b9bb600f-e8b1-4fa1-b9b5-1e98783c5507/e5808c63-9b90-4e6c-9e30-9e6814ad3cdf/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-12-15_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_10.29.56.png)

```kotlin
implementation("com.github.irshulx:laser-native-editor:3.0.4")
```

라이브러리에서 또 다른 라이브러리를 사용한게 문제였는데, 이게 뭔소리냐면 irshulx 라이브러리에서 glide 라이브러리를 사용했기 때문에 예기치 못한 에러를 만날 수 있게 되었습니다.

현재 프로젝트에서 레거시 코드 중 theme, drawable 부분에서 irshulx 라이브러리를 사용하여 작성된 코드가 있었고, 그 부분을 수정한 뒤 irshulx 라이브러를 제거하니 multiple times 에러를 나오지 않았으며, R8 컴파일러도 정상적으로 동작했습니다.

### kapt 에서 ksp로 Migration 은 결과적으로 빌드 속도를 낮추는가?

제 프로젝트에서는 NO 입니다. kapt 플러그인이 빌드 속도를 늦추는 건 맞습니다. kapt 플러그인을 사용하지 않고 ksp 로 마이그레이션한다면, 성능적으로 이점이 있을 것 입니다. 하지만 kapt 와 ksp를 같이 사용하다면, 오히려 빌드 속도를 늦춥니다. 당연한 결과입니다.

두 개의 플러그인을 동시에 사용하니 늘어날 수 밖에 없습니다. hilt 를 사용하기 때문에 kapt 플러그인이 사용되어지고, glide를 사용하여 ksp 플러그인을 사용했는데, 오히려 이 부분이 빌드 속도를 늦추는 결과가 만들어졌습니다.

내부적으로 KAPT Task를 속도는 더 빠른가? 기대해봤지만, 대부분 KSP를 추가했을 때 더 느리게 동작했습니다.

Task도 256 → 258개로 KSP를 추가했을 때, KSP Task가 2개 추가되었습니다.

![스크린샷 2024-12-13 오후 11.57.25.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/b9bb600f-e8b1-4fa1-b9b5-1e98783c5507/5210133e-f20d-45de-a474-0bbaf7ddec50/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-12-13_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_11.57.25.png)

### kapt 플러그인을 모두 지우고, ksp로 대체한다면 어떨까?

그래서 hilt를 사용할 때, kapt 플러그인을 사용해야 하는데 그 부분을 삭제하고 ksp로 대체해봤습니다. 결과는 어떨까요?

결과는 아래와 같습니다. KAPT 보다는 KSP가 훨씬 빠르다는 것을 알 수 있습니다. 추가로 Task도 많이 줄었습니다. 각 모듈의 할당된 KAPT Task가 줄었기 때문이겠죠? 추가적으로 build clean 하지 않고 rebuild 한 경우에 속도차이는 크게 별 차이가 없었습니다.

![스크린샷 2024-12-14 오전 12.23.21.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/b9bb600f-e8b1-4fa1-b9b5-1e98783c5507/168a2ee2-d874-47ad-90af-c5e2eaffc9d8/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-12-14_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_12.23.21.png)

### 결론

hilt ksp는 현재 in progress 입니다. 진행중인 Dagger는 Alpha 단계입니다. 즉, 이 부분을 사용해도 되나? 라는 질문에는 명확한 답을 하지 못할 것 같습니다. 

몇 가지 부분을 고려해야할 것 같습니다.

- 빌드 속도가 느려서 빌드 속도를 개선하고 싶은가?
- Hilt KSP 는 In progress 단계로 안정적이지 못하니 충분한 테스트를 했는가?
- 향후 Hilt, Dagger 버전과의 호환성 보장이 지켜지지 않을 수 있다는 불안감이 드는가?
- 최신 자료이기 때문에 커뮤니티에서 관련 레퍼런스를 찾기 어려울 것 같은가?

등 여러 가지를 고려해야겠지만, 빌드 속도를 개선한다는 장점이 있어서 이 부분은 큰 장점이라고 생각합니다. 하지만 팀 내에서 논의 후 판단하여 프로젝트에 도입하는 것이 좋을 것 같습니다. 혹시 모를 사이드이펙트가 있을 수 있으니 그 부분도 고민해봐야할 것 같습니다.

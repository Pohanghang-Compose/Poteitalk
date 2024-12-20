[블로그 바로가기](https://naemamdaelo.tistory.com/entry/SDK-%EC%96%B4%EB%94%94%EA%B9%8C%EC%A7%80-%EC%95%84%EB%8B%88-compileSdk-targetSdk-minSdk)
</br>
</br>

![image](https://github.com/user-attachments/assets/972227c6-bf05-405c-a981-4f2e1fd8ad13)
</br>

Android 개발을 하다보면 build.gradle에서 이러한 코드를 많이 봤을 것이다

![image](https://github.com/user-attachments/assets/e5e67936-f327-46ac-9b25-ad8800f120d0)

처음 프로젝트를 만들면 기본적으로 값이 들어가있기 때문에 신경을 안썼을 수 도 있다. 

그렇다면 이제부터 같이 알아보자.

## SDK가 뭘까?

SDK(Software Development Kit)는 **애플리케이션 개발 도구**다.

다른 회사에서 구현한 기능을 바로 내 애플리케이션에 사용할 수 있게 해준다.
</br>

가장 대표적인 예시로 카카오 로그인을 할 때면 Kakao SDK를 연결하곤 한다.
</br>

여기서 SDK는 Android SDK를 의미한다.

즉, 우리가 Android를 개발하기 위해 필요한 툴 킷들을 의미한다.
</br>
</br>


## minSdk

**프로젝트의 apk 설치를 지원하는 기기의 최소 요구 SDK 이다.**

**앱을 실행하는데 필요한 최소 API 레벨**을 의미한다.

즉, minSdk보다 기기의 버전이 낮다면 설치되지 않고, 실행되지 않는다.
</br>

이를 막기 위해 기업에서는 낮은 SDK를 유지하고 있다고 들었다.

![image](https://github.com/user-attachments/assets/190e6b98-6900-47c6-b470-75cee152c7de)

현재 프로젝트를 생성한다면 기본적으로 minSdk는 24로 설정되어있다.

24버전이면 약 97.4%의 사용자가 사용 가능하다고 적혀있고, **Help me choose**를 누르면 더 자세한 설명을 들어가 볼 수 있다.

![image](https://github.com/user-attachments/assets/6401f2f6-6471-41c2-ab7e-89872552f061)

좌측의 그래프를 누른다면 각 버전별로 어떤 내용이 추가되었는지 설명이 나온다.
</br>
</br>

그런데 안드로이드 버전은 현재 가장 높은게 15이다. 그런데 sdk는 35까지 올라와있다.

그럼 어느 버전이 어느 sdk를 호환하는지 알수 있는가?
</br>

눈치가 빠른 사람은 위 그림을 보고 알 수 있었을 것이다. 좌측에 작게 14, 우측에 크게 34가 적혀있는 것을.

Android 14에서는 sdk/api level이 34이다.
</br>
</br>

하지만, Android 버전 하나에서 2개 이상의 sdk / api level을 지원할 수 있다.

대표적으로 Android 12의 경우 31, 32 sdk가 같은 버전이다.
</br>

그렇기에 이를 정리해둔 [사이트](https://apilevels.com/)가 있고, 아래처럼 Android 버전에 맞는 sdk를 확인할 수있다.


![image](https://github.com/user-attachments/assets/36d4c0fe-2641-418a-a4bc-d84206257e63)

이를 확인한다면 minSdk를 어느 level로 해야 어느 버전까지 지원이 가능한지 확인하고 개발할 수 있다.

예를 들어 내가 개발하는 앱이 Android 7(Nougat)까지지원하고자 한다면 minSdk를 24로 설정하면 된다.
</br>

이때, sdk가 23이하인 기기에서는 해당 앱을 설치할 수 없다.
</br>
그렇기에 다양한 기기들과 사용자를 고려한다면 minSdk는 낮게 설정하면 좋다. 물론 개발할때는 좀 더 귀찮겠지만...
</br>
</br>

## compileSdk

소스 코드를 컴파일할 때 사용할 수 있는 Android 및 Java API를 결정한다. 

특정 SDK로 버전이 올라가며 새로운 기능이 나왔다면 compileSdk를 해당 버전까지 올려야만 사용이 가능하다.
</br>

예를 들어 Android 12에서 SDK 31버전에서는 스플래시 화면을 쉽게 구현할 수 있는 API가 추가되었다.

그렇기에 이 API를 사용하려면 SDK 버전이 31 이상이어야 한다.
</br>

31버전 이상에서 쓸 수 있는 Splash 화면을 구현 완료했다고 치자.

그렇다면 **31 버전 이하인 최신기기가 아닌 경우**는 어떻게 되나?
</br>

minSdk가 31보다 낮다면, 31과 minSdk 사이의 sdk 사용자가 존재한다.

즉, 새 api에 엑세스 할 수 없는 구형 기기에 대해서도 같은 기능을 제공해줘야 한다.
</br>

또한 31까지는 있었지만, 32가 되며 deprecated된 api가 존재할 수 있다.

그럴때에는 이렇게 오류에 대한 알림이 온다.


![image](https://github.com/user-attachments/assets/13f493ff-7f7e-44ec-a8c1-0d79c06959f0)

</br>
</br>

## targetSdk

앱을 혹시나 출시한 후 운영한 적이 있다면(방치한적이 있다면) 이러한 연락을 구글로부터 받을 수도 있다.


![image](https://github.com/user-attachments/assets/9a70c63d-725d-4553-bce6-c2639f2c7f4c)

![image](https://github.com/user-attachments/assets/24cc8800-0f39-4360-a29f-321d0ea583f8)

targetSdk가 낮으니 올려달라는 이야기이다.

그게 뭐길래 이렇게 올려달라는 메일을 보낼까?
</br>

targetSdk는 2가지 의미를 가진다

1\. 애플리케이션의 런타임 동작을 설정

2\. 앱이 테스트된 Android 버전을 명시
</br>

앱이 targetSdk보다 높은 버전을 사용하는 기기에서 실행된다면 Android는 targetSdk로 지정된 낮은 버전과 유사하게 동작하는 호환모드에서 앱을 실행한다.
</br>

예를 들어 Android 12에서는 사용자 지정 알림의 모양이 변경되었다.

targetSdk가 31보다 낮은 경우(Android 11 이하인 경우) 시스템은 사용자가 해당 기능을 테스트하지 않은 것으로 가정하고 알림이 제대로 표시되지 않을 위험을 최소화하기 위해 이전 방식으로 알림을 표시한다.
</br>

대상 SDK 버전을 31로 업데이트한 후에만 새 알림 모양이 사용된다.


![image](https://github.com/user-attachments/assets/4f91dec7-42e9-40f4-9666-2f7175f4f1b3)


그렇기에 원하는 기능을 의도대로 보여주기 위해서는 테스트를 거친 후 targetSdk를 올려줘야만 한다.
</br>
</br>

## compileSdk와 targetSdk의 관계

그럼 compileSdk와 targeSdk의 관계는 어떻게 될까?
</br>

> targetSdk <= compileSdk

컴파일하는 동안 아무것도 모르는 것들을 대상으로 할 수 없기 때문에 반드시 targetSdk는 compileSdk보다 작거나 같아야 한다.
</br>

구글이 권장 하는 것은 이 둘을 같은 값으로 통일하는 것이다.
</br>
</br>

## 그럼 직접 한번 해보자.

사실 글을 작성하게 된 계기가 있다.
</br>

방에서 뒹굴뒹굴 누워있던 나에게 친한 동생의 연락이 왔다.

구글 콘솔에서 권장 조치가 생겼는데 이게 내 프로덕트에도 생겼는지 확인해 달라고 하더라.

그래서 후다닥 들어가보니 구글 콘솔에 권장 조치가 하나 생겼다.


![image](https://github.com/user-attachments/assets/7a56fbf9-f892-4da1-8702-d0a3d3c7889a)

</br>

또한 Android Studio를 실행시켜보니 내 이러한 권장알림이 올라왔다.

![image](https://github.com/user-attachments/assets/fd19f06b-ef8b-4fe9-bbdc-e38b381da290)


이 둘 모두 Android 15가 출시되어 이에 따른 대응에 대한 부분이었다.
</br>

그래서 올리고 대응해보고자 했다.
</br>

우선 Android 15에 해당하는 SDK35가 다운로드 되어있는지 확인해야 한다.
</br>

Android Studio에서 tools를 누르면 SDK Manager가 보인다.

![image](https://github.com/user-attachments/assets/5593b277-b366-43cb-988a-0d2725c84846)


SDK Manager를 누르면 아래와 같은 창이 뜬다


![image](https://github.com/user-attachments/assets/63cea901-3b5a-412a-beb7-1f059b62dadc)


나는 다행히 15가 다운로드 되어있어 그대로 진행하면 되지만, 다운되어있지 않다면 체크박스를 클릭한 후 다운받으면 된다.
</br>
</br>

그 후 내 프로젝트에 설정되어있던 compileSdk와 targetSdk를 34에서 35로 당당히 올린 후 실행해보면...


![image](https://github.com/user-attachments/assets/054cd6bf-fca2-4552-aa39-17b520107265)


빌드는 성공하지만 빠알간 recommend 메시지가 뜬다.

대충 요약하자면 agp가 8.3.2로 설정되어있는데 이건 sdk 34까지만 가능하단다.

그래서 agp를 올려달라는 메시지다.
</br>
</br>

그래서 agp를 최신 버전이 8.7.3으로 올리고 실행시키니..


![image](https://github.com/user-attachments/assets/4642bca9-1822-4415-98a1-343969fab9c2)


이번엔 gradle이 8.7인데 8.9로 올려달라고 한다.

</br>
</br>

좋다. 가보자.

gradle-wrapper.properties에서 8.7을 8.9로 바꿔주자.


![image](https://github.com/user-attachments/assets/a0d9d1e8-b110-46aa-8113-651fc773d473)


이렇게 하니 빌드에 성공했고, 실행을 시켜보자


![image](https://github.com/user-attachments/assets/d6510bb7-2ed3-4154-b86a-23195ca16677)


짜잔~

sdk 34에서 sdk 35로 올라가면 depreacted된 api에 대한 설명들이다. 즉, 바꿔줘야 한다. 하하…
</br>
</br>

이 오류의 경우 statusbar, navigationBarColor가 35가 되며 deprecated 된 것이다.

35부터는 edge to edge가 기본으로 적용되면서 해당 색상을 직접 컨트롤 할 필요가 없어진 그런 느낌인 것 같다
</br>

그래서 어차피 우리 앱은 enableEdgeToEdge 가 설정되어있었기 때문에 해당 코드들을 지워주었다.

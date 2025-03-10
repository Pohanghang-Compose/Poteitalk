## requiredWidth를 활용해 화면 크기를 초과하는 이미지 슬라이딩 애니메이션 구현

노션 링크 : https://marchbreeze.notion.site/requiredWidth-1afb6895dba9805d960adf0aa958349b?pvs=4

---

## 0. 구현 영상

[KakaoTalk_Video_2025-03-07-18-29-39.mp4](attachment:4a5fe7b5-8d39-4233-a49f-de825f70c055:KakaoTalk_Video_2025-03-07-18-29-39.mp4)

## 1. 부모 레이아웃을 무시하는 이미지 컴포넌트 설정

1. 이미지 Painter 생성 및 intrinsicSize를 통해 크기 측정

    ```kotlin
    @Composable
    fun MovingBackgroundImage(
        @DrawableRes imageRes: Int,
        modifier: Modifier = Modifier,
    ) {
        // 이미지 Painter 생성 및 크기 측정
        val painter = painterResource(id = imageRes)
        val imageSizePx = painter.intrinsicSize
      
    	  //...
    }
    ```


1. BoxWithConstraints를 통해 부모뷰 크기 측정

    ```kotlin
    BoxWithConstraints(modifier = modifier.fillMaxSize()) {
        val density = LocalDensity.current
    
        // 화면 크기를 픽셀 단위로 변환
        val parentHeightPx = constraints.maxHeight.toFloat()
        val parentWidthPx = constraints.maxWidth.toFloat()
    
        // 화면 높이에 맞춘 이미지의 가로 길이 측정 (비율 보존)
        val imageWidthPx = parentHeightPx * (imageSizePx.width / imageSizePx.height)
        val imageWidthDp = with(density) { imageWidthPx.toDp() }
        val imageHeightDp = with(density) { constraints.maxHeight.toDp() }
     
    		//...
     }
    ```


1. requiredHeight, requiredWidth를 통해 부모의 제약을 무시한 이미지 컴포넌트 크기 지정

    ```kotlin
    Image(
        painter = painter,
        contentDescription = null,
        modifier = Modifier
            .requiredHeight(imageHeightDp)
            .requiredWidth(imageWidthDp)
    )
    ```


## 2. 이미지 슬라이딩 애니메이션 구현

1. 수평 오프셋 이동값 측정

    ```kotlin
    val imageWidthGapPx = (imageWidthPx - parentWidthPx).coerceAtLeast(0f)
    ```

    - 화면 가로 크기와 크기 보정된 이미지 가로 크기의 차이만큼 수평 이동

1. 좌우로 -gap/2 ~ +gap/2 범위 내에서 움직이는 애니메이션 설정

    ```kotlin
    val infiniteTransition = rememberInfiniteTransition()
    val animatedOffset by infiniteTransition.animateFloat(
        initialValue = imageWidthGapPx / 2,
        targetValue = -imageWidthGapPx / 2,
        animationSpec = infiniteRepeatable(
            animation = tween(
                durationMillis = 12000,
                easing = LinearEasing,
            ),
            repeatMode = RepeatMode.Reverse
        )
    )
    ```


1. SlowInSlowOut Easing으로 교체

    ```kotlin
    easing = CubicBezierEasing(0.42f, 0f, 0.58f, 1f)
    ```

    - 참고 자료 :

      [Easing in to Easing Curves in Jetpack Compose 🎢](https://medium.com/androiddevelopers/easing-in-to-easing-curves-in-jetpack-compose-d72893eeeb4d)

      ![2025-03-07_19-02-58.jpg](attachment:bad055e5-2cc6-4f95-b5ad-52a889597f36:2025-03-07_19-02-58.jpg)


1. 이미지에 적용

    ```kotlin
    Image(
        painter = painter,
        contentDescription = null,
        modifier = Modifier
            .requiredHeight(imageHeightDp)
            .requiredWidth(imageWidthDp)
            .offset { IntOffset(x = animatedOffset.roundToInt(), y = 0) }
    )
    ```


## 3. 최종 코드

```kotlin
@Composable
fun MovingBackgroundImage(
    @DrawableRes imageRes: Int,
    modifier: Modifier = Modifier,
) {
    BoxWithConstraints(modifier = modifier.fillMaxSize()) {
        val density = LocalDensity.current

        // 이미지 Painter 생성 및 크기 측정
        val painter = painterResource(id = imageRes)
        val imageSizePx = painter.intrinsicSize

        // 화면 크기를 픽셀 단위로 변환
        val parentHeightPx = constraints.maxHeight.toFloat()
        val parentWidthPx = constraints.maxWidth.toFloat()

        // 화면 높이에 맞춘 이미지의 가로 길이 측정
        val imageWidthPx = parentHeightPx * (imageSizePx.width / imageSizePx.height)
        val imageWidthDp = with(density) { imageWidthPx.toDp() }
        val imageHeightDp = with(density) { constraints.maxHeight.toDp() }

        // 이미지의 확장된 너비와 화면 너비의 차이 (애니메이션 이동값)
        val imageWidthGapPx = (imageWidthPx - parentWidthPx).coerceAtLeast(0f)

        // 좌우로 -gap/2 ~ +gap/2 범위 내에서 움직이는 애니메이션
        val infiniteTransition = rememberInfiniteTransition()
        val animatedOffset by infiniteTransition.animateFloat(
            initialValue = imageWidthGapPx / 2,
            targetValue = -imageWidthGapPx / 2,
            animationSpec = infiniteRepeatable(
                animation = tween(
                    durationMillis = 12000,
                    easing = CubicBezierEasing(0.42f, 0f, 0.58f, 1f)
                ),
                repeatMode = RepeatMode.Reverse
            )
        )

        Image(
            painter = painter,
            contentDescription = null,
            modifier = Modifier
                .requiredHeight(imageHeightDp)
                .requiredWidth(imageWidthDp)
                .offset { IntOffset(x = animatedOffset.roundToInt(), y = 0) }
        )
    }
}

@Preview
@Composable
private fun MovingBackgroundImagePreview() {
    GentiTheme {
        MovingBackgroundImage(imageRes = R.drawable.img_login_bg)
    }
}
```
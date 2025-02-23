# 노션에서 봐주세요 ~
https://marchbreeze.notion.site/135b6895dba980f99a44c3bf169b73c2?pvs=4

---

- 참고 자료

  [비트맵 캐싱  |  App quality  |  Android Developers](https://developer.android.com/topic/performance/graphics/cache-bitmap?hl=ko)

  [LruCache  |  Android Developers](https://developer.android.com/reference/android/util/LruCache)

  [[Android] LRU Cache 살펴보기](https://medium.com/%EC%82%BC%EC%B9%98-%EC%8A%A4%ED%84%B0%EB%94%94/android-lru-cache-%EC%82%B4%ED%8E%B4%EB%B3%B4%EA%B8%B0-623ad454821b)

  [비트맵 메모리 관리  |  App quality  |  Android Developers](https://developer.android.com/topic/performance/graphics/manage-memory?hl=ko)

  [[Android][kotlin] Glide 이미지 캐시 및 preload 알아보기](https://minchanyoun.tistory.com/121)


---

## 안드로이드 캐시 종류

- 캐시 (Cache)
    - 앱 내부에서 임시 데이터를 저장해두고 재사용하여 속도와 성능을 최적화하는 역할
    - CPU에서 주기억장치, 보조기억장치까지 도달하는 비용은 매우 크지만, 캐시의 경우 CPU 바로 옆에 딱 달라붙어있기 때문에 물리적으로 거리가 매우 짧아 접근 비용이 매우 적음

1. 메모리 캐시
    - 앱 프로세스가 실행되는 동안 특정 데이터를 RAM에 저장해 두고 재사용하는 방식
    - 가장 빠른 접근 속도를 제공
    - 앱 프로세스가 종료되면 데이터가 사라짐(휘발성)
    - 메모리를 많이 사용하면 OOM(Out Of Memory) 에러 가능성
2. 디스크 캐시
    - 앱의 내부 저장소에 파일 형태로 저장해 두고 재사용하는 방식
    - 메모리 캐시보다 접근 속도가 느림 (디스크 I/O가 필요)
    - 앱 재시작 후에도 데이터가 남아 있음(비휘발성)
    - 디스크 용량이 가득 차면 더 이상 캐싱 불가
3. 네트워크 캐시
    - 네트워크 라이브러리가 HTTP 요청에 대한 응답을 로컬 디스크에 캐시하여 재사용하는 방식
    - 서버 응답을 캐싱하므로, 불필요한 네트워크 요청을 줄여 성능 개선과 데이터 절약
4. 시스템 레벨의 캐시
    - 안드로이드 OS(ART Runtime)에서 자체적으로 운영하는 캐시


## 캐시 관리 메커니즘

### 1. LRU 페이지 교체 알고리즘

- LRU(Least Recently Used) 알고리즘
    - 가장 오랫동안 사용되지 않은 페이지를 교체 대상으로 삼는 페이지 교체 알고리즘
    - 캐시에 공간이 부족할 때 가장 오랫동안 사용하지 않은 항목을 제거하고 새로운 녀석을 배치

- 동작 방식
    1. 데이터 조회 및 추가 : 데이터가 있으면 사용하고 가장 최근에 사용된 것으로 갱신, 없으면 추가
    2. 데이터 교체 : 캐시가 가득 찬 상태이면, 가장 오래 사용되지 않은 데이터를 찾아 삭제하고 데이터를 추가

       ![2025-02-03_15-30-26.jpg](attachment:32ef6c5a-c8de-402e-85cc-92081bef3197:2025-02-03_15-30-26.jpg)


- LruCache
    - 안드로이드에서 캐시를 관리하기 위해 사용하는 메모리 캐시 객체

        ```kotlin
        val cache = LruCache<String, Int>(5)  // maxSize = 5
        
        cache.put("A",0)  // [A]
        cache.put("B",0)  // [A, B]
        cache.put("C",0)  // [A, B, C]
        cache.put("D",0)  // [A, B, C, D]
        cache.put("E",0)  // [A, B, C, D, E] - A부터 E까지 캐싱 완료
        cache.put("F",0)  // [B, C, D, E, F] - F를 캐싱하면, A는 제거됨
        cache.put("D",0)  // [B, C, E, F, D] - D를 다시 캐싱하면 최근 참조된 상태로 변경
        cache.get("C")    // [B, E, F, D, C] - C를 통해 캐시된 데이터 접근시 최근 참조된 상태로 변경
        ```



### 2. LruCache 객체 내부구조

- LinkedHashMap
    - 내부 구현 : **해시 테이블 + 이중 연결 리스트**
        - HashMap과 같이 내부에 배열(bucket)을 사용해 키-값 쌍을 저장
        - O(1)의 시간 복잡도로 검색, 삽입, 삭제 작업을 수행 (hash 값을 키로 인덱스 활용)
    - 노드 구조
        - 각 엔트리는 LinkedHashMap.Entry<K,V> 객체로 관리
        - key, value, hash 값을 저장 + before와 after라는 두 개의 포인터 (엔트리 연결)
    - 순서 유지 메커니즘
        - 생성자에서 accessOrder 플래그를 지정 → get()이나 put() 호출 시 true로 설정해 해당 엔트리를 리스트의 끝으로 이동
        - 내부적으로 afterNodeAccess(Node<K,V> p) 호출 → 접근한 노드를 리스트의 끝으로 재배치

- HashMap : 동기화 처리가 없기 때문에 Multi Thread 환경에서 사용이 어려움

  → LruCache 자체에서 synchronized() 블록을 이용해 해결 (쓰레드가 동시 접근해도 하나만 접근이 가능)

    - get

        ```kotlin
        public final V get(K key) {
            V mapValue;
            synchronized (this) {
                mapValue = map.get(key);
                if (mapValue != null) {
                    return mapValue;
                }
            }
        }
        ```

    - set

        ```kotlin
        public final V put(K key, V value) {
        		V previous;
            synchronized (this) {
                size += safeSizeOf(key, value);
                previous = map.put(key, value);
                if (previous != null) {
                    size -= safeSizeOf(key, previous);
                }
            }
        }
        ```


### 3. DiskLruCache 객체 내부구조

- DiskLruCache
    - 메모리 캐시가 아닌, 디스크 캐시를 활용해서 캐시를 저장하는 Jake Wharton 라이브러리
    - 디스크 캐시 : 메모리(Heap)이 아닌 디스크(파일 시스템)에 데이터 저장
    - 내부적으로 코루틴을 활용하지 않고, 전통적인 스레드 기반의 I/O 및 백그라운드 작업 메커니즘을 사용

        ```kotlin
        private DiskLruCache(File directory, int appVersion, int valueCount, long maxSize) {
          this.directory = directory;
          this.appVersion = appVersion;
          this.journalFile = new File(directory, JOURNAL_FILE);
          this.journalFileTmp = new File(directory, JOURNAL_FILE_TEMP);
          this.journalFileBackup = new File(directory, JOURNAL_FILE_BACKUP);
          this.valueCount = valueCount;
          this.maxSize = maxSize;
        }
        ```


- Journal 파일
    - 캐시 항목의 생성, 수정, 삭제, 읽기 등의 모든 상태 변화를 순차적으로 기록하는 로그 파일
    - 캐시를 재시작하거나 복구할 때 일관된 상태로 복원

        ```kotlin
        public static DiskLruCache open(File directory, int appVersion, int valueCount, long maxSize)
        		// ...
            cache.readJournal();
            cache.processJournal();
            cache.journalWriter = new BufferedWriter(
                new OutputStreamWriter(new FileOutputStream(cache.journalFile, true), Util.US_ASCII));
            return cache;
        ```


- 캐시 쓰기
    - **Editor**를 사용하여 Dirty 파일에 데이터를 기록한 후, 완료 시 commit을 호출하여 변경 사항을 반영
    1. 편집 시작
        - 내부적으로 해당 항목의 Editor를 생성
        - 기존 항목이 있으면 새 Entry 객체 생성 & DIRTY 기록 추가, 편집 중인 항목이 있으면 null 반환
    2. 데이터 기록
        - Dirty 파일에서 파일 경로 생성 및 데이터 기록

            ```kotlin
            public File getDirtyFile(int i) {
              return new File(directory, key + "." + i + ".tmp");
            }
            ```

        - 데이터가 정상적으로 기록하여 commit() 호출 시, Dirty 파일이 Clean 파일로 이름 변경

            ```kotlin
            File clean = entry.getCleanFile(i);
            dirty.renameTo(clean);
            ```

        - Clean 파일 생성 후 읽기 가능한 캐시 데이터 저장

            ```kotlin
            public File getCleanFile(int i) {
              return new File(directory, key + "." + i);
            }
            ```

    3. 데이터 읽기
        - lruEntries에서 해당 키에 해당하는 readable한 Clean 파일들을 탐색
        - 각 값(파일)을 FileInputStream으로 열어 Snapshot 객체에 포함한 후 반환

            ```kotlin
            public synchronized Snapshot get(String key) throws IOException {
            	//...
            	InputStream[] ins = new InputStream[valueCount];
              for (int i = 0; i < valueCount; i++) {
                ins[i] = new FileInputStream(entry.getCleanFile(i));
              }
              journalWriter.append(READ + ' ' + key + '\n');
              return new Snapshot(key, entry.sequenceNumber, ins, entry.lengths);
            }
            ```


## 캐시 저장 구현

### 1. 메모리 캐시 - LruCache

- 싱클톤 객체 (object)로 선언해서 앱 내부에서 하나의 전역적인 인스턴스로 활용
- Coil 라이브러리를 참고해서 전체 앱 메모리 크기의 20%를 메모리 캐시로 할당

    ```kotlin
    object MemoryCacheManager {
        private lateinit var memoryCache: LruCache<String, ByteArray>
    
        fun initialize(context: Context) {
            val cacheSizeBytes = calculateMemoryCacheSize(context) / 1024
            memoryCache = object : LruCache<String, ByteArray>(cacheSizeBytes) {
                override fun sizeOf(key: String, value: ByteArray): Int {
                    return value.size / 1024
                }
            }
        }
    
        internal fun getFromMemoryCache(key: String): ByteArray? {
            return memoryCache[key]
        }
    
        internal fun putToMemoryCache(key: String, data: ByteArray) {
            memoryCache.put(key, data)
        }
    
        private fun calculateMemoryCacheSize(context: Context): Int {
            val activityManager =
                context.getSystemService(Context.ACTIVITY_SERVICE) as ActivityManager
            val memoryClassInMb = activityManager.memoryClass
            return (memoryClassInMb * 1024 * 1024 * 0.2f).toInt()
        }
    }
    ```


### 2. 디스크 캐시 - DiskLruCache

- 디스크 기반 캐시를 관리하기 위한 싱글턴 객체
- Coil 라이브러리를 참고해서 디바이스의 할당 가능한 저장 공간 중 2% (최소 10MB)를 캐시로 설정

    ```kotlin
    object DiskCacheManager {
        private const val DISK_CACHE = "image_cache"
        private const val MIN_DISK_CACHE_SIZE = 10L * 1024 * 1024
    
        private lateinit var diskCacheDir: File
        private lateinit var diskCache: DiskLruCache
    
        fun initialize(context: Context) {
            diskCacheDir = File(context.cacheDir, DISK_CACHE)
    
            val cacheSizeBytes = calculateDiskCacheSize(context)
            diskCache = DiskLruCache.open(diskCacheDir, 1, 1, cacheSizeBytes)
        }
    
        internal fun getFromDiskCache(key: String): ByteArray? {
            return runCatching {
                diskCache[key]?.getInputStream(0)?.use { it.readBytes() }
            }.getOrNull()
        }
    
        internal fun putToDiskCache(key: String, data: ByteArray) {
            diskCache.edit(key)?.apply {
                newOutputStream(0).use { it.write(data) }
                commit()
            }
        }
    
        private fun calculateDiskCacheSize(context: Context): Long {
            val storageManager = context.getSystemService(Context.STORAGE_SERVICE) as StorageManager
            val allocatableBytes = storageManager.getAllocatableBytes(StorageManager.UUID_DEFAULT)
            val cacheSize = (allocatableBytes * 0.02f).toLong().coerceAtLeast(MIN_DISK_CACHE_SIZE)
            return cacheSize
        }
    }
    ```


- DiskLruCache open의 파라미터
    1. directory : 파일이 저장될 위치 설정
    2. appVersion : 앱 버전 - 캐시 파일의 유효성 판단
    3. valueCount : 하나의 캐시 항목에 몇 개의 파일 저장할건지 지정
    4. maxSize : 디스크 캐시의 최대 크기

- appVersion을 build-logic에서 가져오는 방법
    - 예시 코드

        ```kotlin
        class VersionPlugin : Plugin<Project> {
            override fun apply(target: Project) = with(target) {
                with(extensions) {
                    extraProperties["versionName"] = Constants.versionName
                    extraProperties["versionCode"] = Constants.versionCode
                }
            }
        }
        ```

        ```kotlin
            defaultConfig {
                buildConfigField("String", "VERSION_NAME", "\"${extra["versionName"]}\"")
                buildConfigField("String", "VERSION_CODE", "\"${extra["versionCode"]}\"")
            }
        ```


- valueCount로 이미지와 썸네일을 같이 저장할 수도 있음

    ```kotlin
    // valueCount 가 2라고 가정 (0: 썸네일, 1: 원본 이미지)
    diskCache[key]?.apply {
        // 썸네일 저장
        newOutputStream(0).use { it.write(thumbnailData) }
        // 원본 이미지 저장
        newOutputStream(1).use { it.write(originalImageData) }
        commit()
    }
    ```

    ```kotlin
    diskCache.get(key)?.let { snapshot ->
        // 썸네일 읽기
        val thumbnailData = snapshot.getInputStream(0).use { it.readBytes() }
        // 원본 이미지 읽기
        val originalImageData = snapshot.getInputStream(1).use { it.readBytes() }
        snapshot.close()
    }
    ```


## 이미지 관리

### 1. Bitmap & ByteArray

- Bitmap
    - 디코딩된, 픽셀 단위의 이미지 데이터 (ex. ARGB_8888 형식)
    - 이미지의 픽셀 데이터는 네이티브 메모리 영역에 저장
    - 화면 렌더링 과정에서 GPU로 전송되어 텍스처로 사용되면, GPU의 메모리를 소모
    - 압축 해제된 상태의 픽셀 배열을 메모리에 유지하므로, 원본 파일보다 많은 메모리를 사용
    - 메모리 사용량이 크기 때문에, 사용 후 적절하게 recycle() 등으로 메모리 해제를 신경써야 함

- ByteArray
    - 이미지 파일의 압축된 원시 데이터 (ex. JPEG, PNG)
    - Java Heap 메모리에 저장
    - 이미지의 해상도나 디테일은 보존되지만, 직접 픽셀 조작을 하려면 Bitmap으로 변환 필요

⇒  캐시에 ByteArray 형태로 이미지 저장

- 압축 상태의 형태 → 메모리 사용 효율을 높일 수 있음
- 이미 직렬화가 되어 있는 형태 → 디스크에 그대로 쓰거나 읽어들이기 쉬움
- 읽기 전용 데이터 (immutable) → 여러 스레드에서 안전하게 공유할 수 있음

### 2. Coil, Glide의 내부 구현

- 이미지 라이브러리 (Glide, Coil)
    - 내부적으로 LRU 캐시 알고리즘이 이미 적용되어 있음
    - 메모리 캐시와 디스크 캐시를 모두 사용하여 이미지 로딩 성능을 최적화
    - 메모리 캐시, 디스크 캐시를 순차적으로 확인하여 이미지를 로드

        ```kotlin
        val imageLoader = ImageLoader.Builder(context)
            .diskCachePolicy(CachePolicy.ENABLED)  // 디스크 캐시 사용 여부
            .memoryCachePolicy(CachePolicy.ENABLED) // 메모리 캐시 사용 여부
            .build()
        ```


### 3. 이미지 압축

1. 손실 압축 (JPEG로 압축**)**
    - JPEG 포맷으로 이미지의 용량을 줄이기 위해 자주 사용

        ```kotlin
        // Bitmap 객체를 JPEG로 변환
        fun compressImageLossy(bitmap: Bitmap, quality: Int, outputStream: OutputStream) {
            bitmap.compress(Bitmap.CompressFormat.JPEG, quality, outputStream)
        }
        ```

    - 방법 : YCbCr + Quantization
        1. YCbCr 색 공간 변환
            - RGB 이미지를 YCbCr 색 공간으로 변환
            - 인간의 눈이 더 민감한 밝기(Y) 성분과 덜 민감한 색차(Cb, Cr) 성분을 분리
            - 컬러 서브샘플링 : Y 성분은 비교적 높은 해상도를 유지하고, Cb, Cr 성분의 해상도를 낮추는 방식으로 데이터를 줄임
        2. Quantization
            - 고주파 성분을 줄이는 과정, 특히 색상이나 텍스처와 같이 인간의 눈이 덜 민감한 부분을 줄임

1. 비손실 압축 (PNG로 압축)
    - 이미지의 모든 디테일을 유지하면서 용량을 줄일 때 사용

        ```kotlin
        // Bitmap 객체를 PNG로 변환
        fun compressImageLossless(bitmap: Bitmap, outputStream: OutputStream) {
            bitmap.compress(Bitmap.CompressFormat.PNG, 100, outputStream)
        }
        ```

    - 방법 : RLE (Run-Length Encoding, 반복 길이 부호화)
        - 반복되는 데이터를 간단한 방식으로 압축 → 하나의 값과 반복 횟수로 표현
        - 데이터가 완벽히 보존되며, 압축을 해제했을 때 원본과 동일한 품질을 유지

- 이미지의 크기
    - 일반적으로 픽셀 당 3바이트(RGB), 투명도 포함시 4바이트
    - 1바이트 = 8비트, 256가지의 값을 표현 가능, 음수와 양수를 포함하면 -128 ~ +127

## 자체 이미지 로더 구현

### 1. 전체 로직

- 구현 로직

  ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/edfd69d1-6c01-4d0c-9269-1bae8a4e3915/300ce3ce-c1c4-449b-9abf-039e24413129/image.png)

    1. 이미지 로딩 요청
    2. 메모리 캐시 확인 → 존재 시 반환
    3. 디스크 캐시 확인 → 존재 시 반환 및 메모리 캐시 저장
    4. 다운로드
    5. 이미지 변환
    6. 메모리, 디스크 캐시 저장 및 반환
    7. UI 업데이트

1. Manager 설정

    ```kotlin
    object ImageManager {
        private val memoryCacheManager = MemoryCacheManager
        private val diskCacheManager = DiskCacheManager
    
        private var screenWidth = 0
        private var screenHeight = 0
    
        fun initialize(context: Context) {
            val displayMetrics = context.resources.displayMetrics
            screenWidth = displayMetrics.widthPixels
            screenHeight = displayMetrics.heightPixels
        }
    ```

    - 앱 실행 시 initialize를 통해 context로 기기의 가로 세로 크기를 받아옴
    - 단일 인스턴스로 설정 - 여러 곳에서 접근해도 동일한 캐시를 공유되어 일관성을 유지

1. URL에서 ByteArray 값을 다운로드

    ```kotlin
    private suspend fun downloadPhotoFromUrl(fileUrl: String): Result<ByteArray> =
        withContext(Dispatchers.IO) {
            runCatching {
                val connection = URL(fileUrl).openConnection() as HttpURLConnection
                try {
                    connection.apply {
                        doInput = true
                        connectTimeout = 10000
                        readTimeout = 10000
                        requestMethod = "GET"
                    }
                    connection.connect()
    
                    if (connection.responseCode != HttpURLConnection.HTTP_OK) {
                        throw Exception("HTTP error code: ${connection.responseCode}")
                    }
    
                    connection.inputStream.use { inputStream ->
                        inputStream.readBytes()
                    }
                } finally {
                    connection.disconnect()
                }
            }
        }
    ```

    - HttpURLConnection 연결 → 내부적으로 소켓을 생성하여 대상 서버와 TCP 연결 수행
    - InputStream으로 바이트 데이터를 읽은 후 use로 종료

1. Flow를 기반으로 ImageBitmap을 로드

    ```kotlin
    private fun loadImageBitmapFlow(model: GalleryPhotoUiModel): Flow<ImageBitmap?> =
        flow {
            val cacheKey = model.id
    
            // 1. 메모리 캐시 조회 및 반영
            memoryCacheManager.getFromMemoryCache(cacheKey)?.let {
                emit(it.decodeByteArrayToImageBitmap())
                return@flow
            }
    
            // 2. 디스크 캐시 조회 및 반영
            diskCacheManager.getFromDiskCache(cacheKey)?.let {
                memoryCacheManager.putToMemoryCache(cacheKey, it)
                emit(it.decodeByteArrayToImageBitmap())
                return@flow
            }
    
            // 3. 네트워크에서 이미지 다운로드
            val downloadedByteArray = downloadPhotoFromUrl(model.photoUrl).getOrNull()
            if (downloadedByteArray == null) {
                emit(null)
                return@flow
            }
    
            // 4. 다운받은 원본 이미지를 디바이스 스크린 기준으로 축소/압축
            val compressedByteArray = downloadedByteArray.compressByteArray(
                model.width, model.height, screenWidth, screenHeight
            )
    
            // 5. 압축된 ByteArray를 캐시에 저장한 후 반환
            diskCacheManager.putToDiskCache(cacheKey, compressedByteArray)
            memoryCacheManager.putToMemoryCache(cacheKey, compressedByteArray)
            emit(compressedByteArray.decodeByteArrayToImageBitmap())
    
        }.flowOn(Dispatchers.IO)
    ```

    - Flow 빌더 (flow {})를 통해 코루틴 내에서 순차적으로 emit하도록 설정
    - Flow의 실행 컨텍스트를 I/O 작업에 최적화된 Dispatchers.IO로 설정
    - ByteArray → ImageBitmap

        ```kotlin
        private fun ByteArray?.decodeByteArrayToImageBitmap(): ImageBitmap? =
            this?.let {
                BitmapFactory.decodeByteArray(it, 0, it.size).asImageBitmap()
            }
        ```

    - ByteArray 이미지 크기 축소

        ```kotlin
        fun ByteArray.compressByteArray(
            originalWidth: Int,
            originalHeight: Int,
            targetWidth: Int,
            targetHeight: Int
        ): ByteArray {
            if (originalWidth <= targetWidth || originalHeight <= targetHeight) return this
        
            val ratioX = targetWidth.toFloat() / originalWidth
            val ratioY = targetHeight.toFloat() / originalHeight
            val scale = minOf(ratioX, ratioY)
        
            // 최대 inSampleSize 계산 후 option에 적용
            val options = BitmapFactory.Options().apply {
                this.inSampleSize = calculateInSampleSize(originalWidth, originalHeight, scale)
            }
        
            // ByteArray에 새로 측정된 inSampleSize 적용해 Bitmap 생성 (디코딩 실패 시 원본 반환)
            val scaledBitmap = BitmapFactory.decodeByteArray(this, 0, this.size, options)
                ?: return this
        
            // 압축된 Bitmap을 다시 ByteArray로 변환 후 recycle 적용
            val compressedByteArray = ByteArrayOutputStream().use { outputStream ->
                scaledBitmap.compress(Bitmap.CompressFormat.JPEG, 100, outputStream)
                outputStream.toByteArray()
            }
            scaledBitmap.recycle()
        
            return compressedByteArray
        }
        ```

    - inSampleSize 크기 계산

        ```kotlin
        fun calculateInSampleSize(
            originalWidth: Int,
            originalHeight: Int,
            scale: Float
        ): Int {
            val targetWidth = (originalWidth * scale).toInt()
            val targetHeight = (originalHeight * scale).toInt()
        
            var inSampleSize = 1
            if (originalWidth > targetWidth && originalHeight > targetHeight) {
                val halfWidth = originalWidth / 2
                val halfHeight = originalHeight / 2
                while ((halfWidth / inSampleSize) >= targetWidth && (halfHeight / inSampleSize) >= targetHeight) {
                    inSampleSize *= 2
                }
            }
            return inSampleSize
        }
        ```


1. Compose에서 이미지 로딩 상태 관리

    ```kotlin
    @Composable
    internal fun rememberImageLoader(model: GalleryPhotoUiModel): State<ImageBitmap?> =
        produceState<ImageBitmap?>(initialValue = null, model) {
            if (model.isPlaceHolder) {
                value = null
                return@produceState
            } else {
                loadImageBitmapFlow(model).collect { bitmap -> value = bitmap }
            }
        }
    ```

    - 이미지 로딩 결과를 ImageBitmap? 타입의 State로 반환
    - 초기값은 null이며, 이미지가 로드되면 상태 값이 업데이트되어 UI가 Recompose됨
    - produceState : 코루틴을 이용하여 비동기 작업의 결과를 State로 만들어주는 compose 헬퍼 함수

1. Screen에서 활용

    ```kotlin
    @Composable
    internal fun GalleryPhotoItem(
        model: GalleryPhotoUiModel,
        modifier: Modifier = Modifier,
        onPhotoClick: (Int) -> Unit = {}
    ) {
        val imageState = rememberImageLoader(model)
        //...
    }
    ```


### 2. 시간 & 메모리 소요 분석

- 방법
    - 소요 시간

        ```kotlin
        val startTime = System.nanoTime()
        // 로직
        val elapsedTime = (System.nanoTime() - startTime) / 1_000_000
        Timber.tag("breeze").d("$page 페이지 소요시간: $elapsedTime ms")
        ```

    - 메모리
        - Android Studio > Profiler > Track Memory Comsumption 활용

1. 캐시 적용 이후

   ![2025-02-04_01-24-45.jpg](attachment:04069ce1-1cde-401d-8385-b6084ce73062:2025-02-04_01-24-45.jpg)

   ![2025-02-17_21-06-58.jpg](attachment:df7ef6cd-254d-463b-82b0-0932790012a4:2025-02-17_21-06-58.jpg)

2. 압축 적용 이후

   ![2025-02-04_01-04-56.jpg](attachment:8e586cce-6273-4e30-ad99-d7a4f5e37d03:2025-02-04_01-04-56.jpg)

   ![2025-02-17_21-07-09.jpg](attachment:e83ea61f-ca0f-40d4-8cdd-79e133aea01b:2025-02-17_21-07-09.jpg)
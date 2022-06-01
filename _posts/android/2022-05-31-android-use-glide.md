# [Android] Glide를 사용하여 이미지 불러오기

![glide_image1.jpg](/assets/images/glide_image1.jpg?raw=true)

## Glide란?

Glide는 부드러운 스크롤에 중점을 둔 빠르고 효율적인 Android용 이미지 로딩 라이브러리이다.

Glide는 사용하기 쉬운 API, 성능이 뛰어나고 확장 가능한 리소스 디코딩 파이프라인 및 자동 리소스 풀링을 제공한다. 기본적으로 사진 로딩과 동영상, gif 파일 로딩까지 지원한다.

## 사용방법

### 사전 준비

먼저 랜덤 이미지를 표출해주는 [http://placehold.it/500x500를](http://placehold.it/500x500를) 통해 이미지를 로드하겠다.

인터넷을 사용하기에 매니페스트에서 인터넷 권한을 추가해준다.

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

그리고 Glide라이브러리와 ViewBinding을 추가해준다.

```groovy
android{
    viewBinding {
        enabled = true
    }
}

dependencies {
    implementation 'com.github.bumptech.glide:glide:4.11.0'
    annotationProcessor 'com.github.bumptech.glide:compiler:4.11.0'
}
```

### 이미지 표출

간단하다.

- activity_main.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <ImageView
        android:id="@+id/image"
        android:layout_width="match_parent"
        android:layout_height="500dp"
        app:layout_constraintBottom_toTopOf="@+id/button"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintVertical_chainStyle="packed"
        android:src="@drawable/ic_launcher_background"/>

    <com.google.android.material.button.MaterialButton
        android:id="@+id/button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="20dp"
        android:padding="10dp"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/image"
        android:text="이미지 생성"
        android:textSize="20sp"/>

</androidx.constraintlayout.widget.ConstraintLayout>
```

![glide_image2.png](/assets/images/glide_image2.png?raw=true)

- MainActivity.kt

```kotlin
class MainActivity : AppCompatActivity() {
    private val binding by lazy {
        ActivityMainBinding.inflate(layoutInflater)
    }
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(binding.root)

        binding.button.setOnClickListener {
            Glide.with(this)
                .load("https://picsum.photos/500/500")
                .into(binding.image)
        }

    }
}
```

특별한 옵션이 없이 단순 뷰에 이미지를 넣는것이라면 `with()`, `load()`, `into()` 으로도 표출할 수 있다.

각 메소드를 살펴보면

- with()

라이프사이클을 연결하고 여러 옵션들을 사용하기위헤 context, view, activity, fragment, fragmentActivity등을 받는다.

- load()

로드할 이미지리소스를 입력한다. 다양한 방법으로 이미지를 불러올 수 있다.

(Bitmap, Drawable, String, Uri, File, ResourId(Int), ByteArray)

- into()

이미지를 보여줄 View를 지정한다.

다음과같이 앱은 동작한다.

![glide_gif1.gif](/assets/images/glide_gif1.gif?raw=true)

## 여러 메소드들

위의 함수들은 뼈대가 되는 내용들이고 Glide는 여러가지 기능들이 있다.

### override

이미지 사이즈를 조절해줄 수 있다. 사이즈를 조절하며 이미지 로딩 속도를 최적화할 수 있다. 메모리를 절약하고 싶을 때 유용하게 사용된다.

```kotlin
Glide.with(this)
    .load("이미지 url")
    .override(100, 100)
    .into(imageView)
```

### **placeholder**

이미지가 로딩하는 동안 보일 이미지를 정해준다. resourceId나 drawable이 들어간다.

```kotlin
Glide.with(this)
    .load("이미지 url")
    .placeholder(R.drawable.loading)
    .into(imageView)
```

### **error**

이미지 로딩에 실패했을 경우 실패 이미지를 지정해줄 수 있다.

```kotlin
Glide.with(this)
    .load("이미지 url")
    .error(R.drawable.error)
    .into(imageView)
```

### **asGif**

gif 이미지도 로딩할 수 있다.

```kotlin
Glide.with(this)
    .asGif()
    .load("gif 이미지 url")
    .into(imageView)
```

### **thumbnail**

원본 이미지를 썸네일로 사용한다. 지정한 % 비율만큼 미리 이미지를 가져와서 보여준다. 0.1f로 지정했다면 실제 이미지 크기 중 10%만 먼저 가져와서 흐릿하게 보여준다.

```kotlin
Glide.with(this)
    .load("이미지 url")
    .thumbnail(0.1f)
    .into(imageView)
```

## 추가

Glide에서는 기본적으로 이미지의 URL을 이용해 메모리와 디스크에 캐싱을 한다.

Glide 는 이미지를 불러오기 위해 아래와 같은 절차로 여러 캐시를 확인한다.

![glide_image3.png](/assets/images/glide_image3.png?raw=true)

1. Active resources — 현재 이 이미지가 다른 뷰에 나타나는가?
2. Memory cache — 이 이미지가 최근에 메모리에 로드되었거나, 여전히 메모리에 남아있는가?
3. Resource — 이 이미지가 예전에 디코드되었거나, 변형되었거나, 디스크 캐시에 기록되었는가?
4. Data — 이 이미지가 이전에 디스크 캐시에 기록되었던 데이터였는가?

### skipMemoryCache

기본적으로 메모리 캐싱을 하기때문에, 메모리 캐싱을 위해 추가적으로 할 일은 없다.

단, URL 이미지 로딩 시 한번 로드한 이미지는 chache 에 저장되어 서버에서 해당 이미지를 변경해도 App 의 이미지는 갱신되지 않는다.

이런 경우, skipMemoryCache(true) 로 메모리 캐시를 사용하지 않을 수 있다.

### diskCacheStrategy

디스크에 캐싱하지 않으려면 DiskCacheStrategy.NONE로 준다.

다음과 같은 옵션이 있다.

- DiskCacheStrategy.ALL : 모든 이미지를 캐싱
- DiskCacheStrategy.RESOURCE : 해상도를 줄인 이미지만 캐싱
- DiskCacheStrategy.AUTOMATIC : RESOURCE 를 기반으로 전략적인 캐싱. (Default)
- DiskCacheStrategy.DATA : 원본 이미지만 캐싱
- DiskCacheStrategy.NONE : 디스크 캐싱 안 함

```kotlin
Glide.with(this)
    .load(R.drawable.img_file_name)
    .skipMemoryCache(true)
    .diskCacheStrategy(DiskCacheStrategy.NONE)
    .into(imageView)
```

이로써 메모리 사용과 서버 통신을 효율적으로 할 수 있게 해준다.

### Target

Glide 는 기본적으로 이미지를 비동기로 로드하여 이미지뷰에 보여준다.

Glide 에서 target 을 이용하면 bitmap, drawable, placeholder 를 받아 처리할 수 있다.

```kotlin
Glide.with(this)
    .load("이미지 url")
    .thumbnail(0.1f)
    .into(object : Target<Drawable>{
        override fun onStart() {

        }

        override fun onStop() {

        }

        override fun onDestroy() {

        }

        override fun onLoadStarted(placeholder: Drawable?) {

        }

        override fun onLoadFailed(errorDrawable: Drawable?) {

        }

        override fun onLoadCleared(placeholder: Drawable?) {

        }

        override fun getSize(cb: SizeReadyCallback) {

        }

        override fun removeCallback(cb: SizeReadyCallback) {

        }

        override fun setRequest(request: Request?) {

        }

        override fun getRequest(): Request? {

        }

        override fun onResourceReady(resource: Drawable, transition: Transition<in Drawable>?
        ) {

            }
        })
```

## 참조

> [https://bumptech.github.io/glide/](https://bumptech.github.io/glide/)
>
> [https://medium.com/android-news/best-strategy-to-load-images-using-glide-image-loading-library-for-android-e2b6ba9f75b2](https://medium.com/android-news/best-strategy-to-load-images-using-glide-image-loading-library-for-android-e2b6ba9f75b2)
>
> [https://leveloper.tistory.com/162?category=762053](https://leveloper.tistory.com/162?category=762053)
>
> [https://blog.yena.io/studynote/2020/06/10/Android-Glide.html](https://blog.yena.io/studynote/2020/06/10/Android-Glide.html)
>
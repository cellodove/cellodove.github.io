---
title : "[Android] Apollo를 이용해 Graph QL API 통신1"
excerpt: "[Android] Apollo를 이용해 Graph QL API 통신1"

categories :
  - Android

toc: true
toc_sticky: true
last_modified_at: 2022-04-03
---

![catch_ball2.jpg](/assets/images/catch_ball2.jpg?raw=true)

공식문서 기반으로 만드는데 새롭게 나오는 내용이 많아서 몇차례에 걸처 글을써야할것같다.

이번에는 아폴로 세팅과 쿼리 데이터를 가져오는것을 중점적으로 보겠다.

## **Apollo란?**

아폴로 [문서](https://www.apollographql.com/docs/)에가면 다음과 같이 설명되어있다.

“Apollo는 애플리케이션 클라이언트(웹 및 네이티브 애플리케이션 등)와 백엔드 서비스 간의 데이터 흐름을 관리하는 데 도움이 되는 통신 계층인 통합 그래프를 구축하기 위한 플랫폼입니다. 그래프의 중심에는 GraphQL이라는 쿼리 언어가 있습니다.”

레트로핏이 REST API를 간단하게 사용할수있도록 해준것처럼 아폴로는 GraphQL API를 간단하게 사용할수있도록 해주는 라이브러리다.

## 아폴로 구성요소

아폴로를 구성하는데 크게 3가지가 있으면 된다.

- 개체 유형으로 구성되어 어떤 종류의 개체를 요청할 수 있으며 어떠한 필드가 있는지 정의되어있는 Schema 파일
- 사용할 GraphQL 데이터 모델들을 정의해놓은 GraphQL 파일
- 아폴로를 생성해 서버와 통신하기위한 아폴로 빌더 클래스

## 아폴로 사용해보기

아폴로 공식문서에있는 튜토리얼을 기반으로 앱을 만들겠다. v3버전이다.

아폴로는 [예제](https://www.apollographql.com/docs/kotlin/tutorial/00-introduction)를 아주 친절하게 설명해주고있으니 한번 들어가서 봐보면 좋다.

필요한 파일은 다음과 같다.

- MainActivity.kt - 화면표출
- LaunchListAdapter.kt - 리사이클러뷰 어댑터
- LaunchListFragment.kt - 받은데이터를 리스트로 보여줌
- MainViewModel.kt - 서버로부터 데이터 받아 뷰로 전달
- Apollo.kt - 아폴로 빌더 클래스
- schema.graphql - 서버로부터 받은 graphql 스키마
- LaunchList.graphql - 사용할 graphql 데이터 모델

## 아폴로 세팅

먼저 서버와 통신하기때문에 인터넷 퍼미션을 추가해준다.

![apollo_example1_0](/assets/images/apollo_example1_0.png?raw=true)

```xml
<uses-permission android:name="android.permission.INTERNET"/>
```

### Dependency

이 프로젝트를 만들기 위한 라이브러리들을 먼저 추가한다.

- 프로젝트 그레들

```groovy
buildscript {
    dependencies {
        classpath("androidx.navigation:navigation-safe-args-gradle-plugin:2.4.1")
    }
}
```

- 모듈 그레들

```groovy
plugins {
    //apollo
    id("com.apollographql.apollo3").version("3.1.0")

    //navigation.safeargs
    id("androidx.navigation.safeargs.kotlin")
}

dependencies {
    //coroutines
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.5.2'
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-play-services:1.5.2'

    implementation 'androidx.activity:activity-ktx:1.4.0'
    implementation 'androidx.fragment:fragment-ktx:1.4.1'

    // LiveData
    implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:2.4.1"
    implementation "androidx.lifecycle:lifecycle-livedata-ktx:2.4.1"

    //apollo
    implementation("com.apollographql.apollo3:apollo-runtime:3.1.0")

    //glide
    implementation 'com.github.bumptech.glide:glide:4.11.0'
    annotationProcessor 'com.github.bumptech.glide:compiler:4.11.0'

    //Android Navigation component
    implementation "androidx.navigation:navigation-fragment-ktx:2.4.1"
    implementation "androidx.navigation:navigation-ui-ktx:2.4.1"
    implementation "androidx.navigation:navigation-dynamic-features-fragment:2.4.1"
    androidTestImplementation "androidx.navigation:navigation-testing:2.4.1"
    implementation "androidx.navigation:navigation-compose:2.4.1"
}

apollo {
    //자신이 만든 패키지 입력
    packageName.set("com.cellodove.apollo_example")
}
```

```groovy
android {
    viewBinding {
        enabled true
    }
}
```

아폴로 예제가 [네비게이션 컴포넌트](https://developer.android.com/guide/navigation/navigation-getting-started?hl=ko)를 사용하는데 똑같이 해보려고한다. 그래서 아폴로 관련 라이브러리뿐만아니라 네비게이션 컴포넌트들도 추가해준다.

### 스키마 추가

이제 그래프ql 스키마를 추가해준다.

먼저 우리가 사용할 GraphQL 서버에있는 스키마는 [다음 페이지](https://studio.apollographql.com/sandbox/explorer?_gl=1%2Afz5bc4%2A_ga%2ANDQxOTQ5NTAyLjE2NDc4NDE0MTE.%2A_ga_0BGG5V2W2K%2AMTY0Nzk5NjI4Ni4xMi4xLjE2NDc5OTY0NzcuMA..)에서 확인할 수 있다.

![apollo_example1_1](/assets/images/apollo_example1_1.png?raw=true)

저 페이지에들어가 빨간원에있는 입력창에 **https://apollo-fullstack-tutorial.herokuapp.com/graphql**

주소를 넣어주면 어떤 데이터 모델들이 있는지 알 수있다.

![apollo_example1_2](/assets/images/apollo_example1_2.png?raw=true)

이번에는 Launches 하나의 쿼리만 사용할 예정이다.

포스트맨같은 툴을 사용할필요없이 이런게 너무잘 되어있어서 잘 활용하면된다.

스키마는 터미널을 통해 추가한다. 아폴로 프로젝트 폴더로가서 해당폴더에서 터미널을 연다.

그후 아래내용을 입력하고 실행해준다.

```bash
./gradlew :app:downloadApolloSchema --endpoint='https://apollo-fullstack-tutorial.herokuapp.com/graphql' --schema=app/src/main/graphql/com/example/rocketreserver/schema.graphqls
```

정상적으로 추가가되었으면 다음과같이 나오고

![apollo_example1_3](/assets/images/apollo_example1_3.png?raw=true)

안드로이드 프로젝트 app/src/main/graphql/com/example/rocketreserver 루트에
schema.graphql가 생성되었으면 성공이다.

![apollo_example1_4](/assets/images/apollo_example1_4.png?raw=true)

### 쿼리 생성

GraphQL은 자신이 원하는 데이터만 가져올수있다. Launches쿼리를 예를들면

Launches쿼리의 모든 데이터를 가져올수도,

![apollo_example1_5](/assets/images/apollo_example1_5.png?raw=true)

내게 필요한 몇가지만 빼올 수 도있다.

![apollo_example1_6](/assets/images/apollo_example1_6.png?raw=true)

이게 GraphQL의 장점이다. 클라이언트가 유연하게 데이터를 가져올 수 있다.

그럼이제 쿼리를 작성하겠다.

필요한 데이터를 가지고올 수 있도록 스키마가 있는 위치에 graphql파일을 생성해 작성한다.

![apollo_example1_7](/assets/images/apollo_example1_7.png?raw=true)

파일의 이름은 원하는대로 만들면된다.

```graphql
query LaunchList {
    launches {
        cursor
        hasMore
        launches {
            id
            site
            mission {
                name
                missionPatch
            }
        }
    }
}
```

이렇게 쿼리를 작성한뒤에 리빌드를 하면

자동으로 다음과같이 모델을 생성해준다.

![apollo_example1_8](/assets/images/apollo_example1_8.png?raw=true)

해당 파일은 프로젝트 구조에서 보이진 않고

패키지로 바꾸면 보인다.

이제 아폴로 구성요소 3가지중에 2가지를 완료했다 이제 남은 한가지를 해보겠다.

### 아폴로 클라이언트 생성

아폴로 클라이언트 빌더는 레트로핏과 비슷하다고 생각하면된다.

```kotlin
object Apollo {
    fun apolloClient():ApolloClient{
        return ApolloClient.Builder()
            .serverUrl("https://apollo-fullstack-tutorial.herokuapp.com/graphql")
            .build()
    }
}
```

싱글톤으로 만들기위해 object로 만들어준다.

## UI연결

데이터 표출은 프래그먼트에서 리사이클러뷰를 통해 할예정이다.

### MainViewModel

```kotlin
class MainViewModel : ViewModel() {
    var launchListQueryData = MutableLiveData<ApolloResponse<LaunchListQuery.Data>>()

    fun getLaunchListQuery() = GlobalScope.launch{
        try {
            launchListQueryData.postValue(Apollo.apolloClient().query(LaunchListQuery()).execute())
        }catch (e: ApolloException){
            Log.e("LaunchList", "Failure", e)
            return@launch
        }
    }

}
```

`Apollo.apolloClient().query(LaunchListQuery()).execute()`

아폴로 클라이언트에서 아까 만들었던 `LaunchListQuery` 를 `execute()`하여 데이터를 가져온다.

그다음 라이브데이터를 사용해 View로 데이터를 보낸다.

### LaunchListAdapter

리사이클러 어댑터를 만들겠다.

```kotlin
class LaunchListAdapter(
    private val launches: List<LaunchListQuery.Launch>
) : RecyclerView.Adapter<LaunchListAdapter.ViewHolder>() {

    class ViewHolder(val binding: LaunchItemBinding) : RecyclerView.ViewHolder(binding.root)

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        val binding = LaunchItemBinding.inflate(LayoutInflater.from(parent.context),parent,false)
        return ViewHolder(binding)
    }

    override fun getItemCount(): Int {
        return launches.size
    }

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        val launch = launches[position]
        holder.binding.site.text = launch.site ?: ""
        holder.binding.missionName.text = launch.mission?.name
        Glide.with(holder.itemView.context).load(launch.mission?.missionPatch).into(holder.binding.missionPatch)
    }
}
```

들어오는 데이터중에 이미지 url이 있어 url를 이미지로 표출해주어야한다.

나는 글라이드를 쓰기로했다. 글라이드는 간단하게 url을 이미지로 표출해주는 라이브러리다.

`Glide.with(holder.itemView.context).load(launch.mission?.missionPatch).into(holder.binding.missionPatch)`

`.with`에는 context를 넣어주고 `.load`에 url을 넣어주고 `.into`에 어떤 뷰에 표출할것인지 원하는 뷰를 넣어준다.

### LaunchListFragment

어댑터를 만들었으니 이제 프래그먼트를 만들어준다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <androidx.recyclerview.widget.RecyclerView
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"
        android:id="@+id/launches"
        app:layoutManager="androidx.recyclerview.widget.LinearLayoutManager"/>

    <LinearLayout
        android:id="@+id/showProgress"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center"
        android:background="#80000000"
        android:clickable="false">
        <ProgressBar
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>
    </LinearLayout>

</androidx.constraintlayout.widget.ConstraintLayout>
```

```kotlin
class LaunchListFragment : Fragment() {
    private lateinit var binding: LaunchListFragmentBinding
    private val viewModel : MainViewModel by activityViewModels()

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        binding = LaunchListFragmentBinding.inflate(inflater)
        return binding.root
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        viewModel.getLaunchListQuery()
        viewModelObserver()
    }

    @SuppressLint("NotifyDataSetChanged")
    private fun viewModelObserver(){
        viewModel.launchListQueryData.observe(viewLifecycleOwner){ response ->
            binding.showProgress.visibility = View.GONE
            val launches = response.data?.launches?.launches?.filterNotNull()
            if (launches != null && !response.hasErrors()){
                val adapter = LaunchListAdapter(launches)
                binding.launches.adapter = adapter
                adapter.notifyDataSetChanged()
            }
        }
    }
}
```

이 프래그먼트가 앱의 처음화면이기때문에 프래그먼트가 생성되면 바로 데이터를 호출한다.

그때동안 프로그래스바를 띄어두고 결과가 들어오면 어댑터로 데이터를 보내 데이터를 표출하고 프로그래스바를 닫는다.

네비게이션 컴포넌트를 사용하고싶지 않으면 그냥 프래그먼트를 액티비티에 연결하는것으로 끝내면된다.

나는 네비게이션 컴포넌트를 연습할겸 써보려고한다.

먼저 경로를 나타내기 위한 xml이 필요하다. res에서 안드로이드 리소스 디텍토리를 선택한다.

![apollo_example1_9](/assets/images/apollo_example1_9.png?raw=true)

그다음

![apollo_example1_10](/assets/images/apollo_example1_10.png?raw=true)

리소스 타입을 네비게이션으로 선택한뒤 확인을 누른다.

![apollo_example1_11](/assets/images/apollo_example1_11.png?raw=true)

폴더가 생성되었으면 해당폴더안에 네비게이션 리소스 파일을 생성한다. 파일이름은 main으로했다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/nav"
    app:startDestination="@id/launchListFragment">
    <fragment
        android:id="@+id/launchListFragment"
        android:name="com.cellodove.apollo_example.view.LaunchListFragment"
        tools:layout="@layout/launch_list_fragment">
    </fragment>
</navigation>
```

우리는 아까만들었던 `LaunchListFragment` 처음화면에 나오게할 예정이므로

`startDestination="@id/launchListFragment"`로 설정한다.

### MainActivity

이제 메인 액티비티를 만들어 네비게이션을 연결하자

```kotlin
class MainActivity : AppCompatActivity() {
    private val binding by lazy {
        ActivityMainBinding.inflate(layoutInflater)
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(binding.root)
    }

}
```

코드상에는 딱히 할일이없다 뷰바인딩을 해주자

```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".view.MainActivity">

    <fragment
        android:id="@+id/nav_host_fragment"
        android:name="androidx.navigation.fragment.NavHostFragment"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:defaultNavHost="true"
        app:navGraph="@navigation/main" />

</FrameLayout>
```

프래그먼트를 만든뒤 네비게이션을 연결해준다.

주의깊게 봐야할것은 name, defaultNavHost, navGraph 이다.

- name 속성은 NavHost 구현의 클래스 이름을 포함해야한다.
- navGraph ****속성은 NavHostFragment를 탐색 그래프와 연결한다. 탐색 그래프는이 NavHostFragment에서 사용자가 탐색 할 수있는 모든 대상을 지정한다.
- defaultNavHost = "true"속성은 NavHostFragment가 시스템 뒤로 버튼을 가로 채도록한다. 하나의 NavHost 만 기본값이 될 수 있다. 동일한 레이아웃 (예 : 두 개의 창 레이아웃)에 여러 호스트가있는 경우 하나의 기본값 만 지정해야한다.

실행시키면 다음과 같이 나온다.

![apollo_example_gif.gif](/assets/images/apollo_example_gif.gif?raw=true)

전체 코드는 아래 링크에서 볼 수 있다.

[https://github.com/cellodove/Apollo_Example1](https://github.com/cellodove/Apollo_Example1)

**다음글 보기**
[[Android] Apollo를 이용해 Graph QL API 통신2](https://cellodove.github.io/android/android-use-apollo02/)

## 참조

> [https://www.apollographql.com/docs/](https://www.apollographql.com/docs/)
>
> [https://developer.android.com/guide/navigation/navigation-getting-started?hl=ko](https://developer.android.com/guide/navigation/navigation-getting-started?hl=ko)
>
> [https://myung6024.tistory.com/108](https://myung6024.tistory.com/108)
>
---
published: true
title : "[Android] Paging을 사용하여 리스트 표출하기"
excerpt: "페이징 간단하게 살펴보자"

categories :
  - Elements
  - 
toc: true
toc_sticky: true
last_modified_at: 2022-06-09
---

![paging1_image1.jpg](/assets/images/paging1_image1.jpg?raw=true)

## Paging이란?

Android Jetpack의 구성요소로 로컬 저장소에서나 네트워크를 통해 대규모 데이터 세트를 페이지 단위로 로드하고 표시할 수 있도록 해주는 라이브러리이다.

라이브러리를 사용하기전 옛날에는 리스트가 상단 또는 하단에 도달했는지 확인하는 코드와 다음 페이지를 로드하는 코드가 작성되었어야 했다. 그리고 페이징이 필요한 모든 화면에 동일한 코드를 작성해야하고 네트워크 오류와 같은 예외 처리 코드도 필요했다. 위와같은 문제를 해결하기위해 해당 라이브러리가 나왔다. 지금은 Paging3가 최신버전이다.

## Paging 장점

- 페이징된 데이터를 메모리 안에 캐싱한다. 이렇게 하면 앱이 패이징 데이터로 작업하는 동안 시스템 리소스를 효율적으로 사용할 수 있다.
- 요청 중복 제거 기능. 이 기능이 기본으로 제공되어 앱에서 네트워크 대역폭과 시스템 리소스를 효율적으로 시용할 수 있다.
- 사용자가 리스트의 끝까지 스크롤을 하면 리사이클러뷰 어댑터가 자동으로 데이터를 요청한다.
- Kotlin 코루틴 및 Flow뿐만 아니라 LiveData 및 RxJava를 최고 수준으로 지원한다.
- 새로고침 및 재시도 기능을 포함하여 오류 처리를 기본으로 지원한다.

## 라이브러리 구조

페이징 라이브러리는 [권장 Android 앱 아키텍처](https://developer.android.com/jetpack/docs/guide?hl=ko)를 염두해 두고 만들어졌다. 라이브러리의 구성요소는 앱의 세 가지 레이어에 걸쳐 동작한다.

![paging1_image2.svg](/assets/images/paging1_image2.svg?raw=true)

### Repository 레이어

저장소 레이어에서의 페이징 라이브러리의 구성요소는 `PagingSource` 이다. `PagingSource` 객체는 데이터 소스를 정의하고 데이터를 가져오는 방법을 정의한다. `PagingSource` 객체는 네트워크 소스 및 로컬 데이터베이스를 포함한 단일 소스에서 데이터를 로드할 수 있다.

다른 페이징 라이브러리 구성요소는 `RemoteMediator` 이다. `RemoteMediator` 객체는 네트워크에서 불러온 데이터를 로컬 데이터베이스에 캐시(Cache)하여 불러오는 것을 담당한다. 오프라인 상태에서도 캐시된 데이터를 불러옴으로 유저 경험을 향상시켜줄 수 있다.

이를 가능하게 하는 방법은 네트워크와 로컬 데이터베이스에서 동시에 페이징하는 것이다. 이렇게 하면 앱이 로컬 데이터베이스 캐시에서 직접 데이터를 로드하고 데이터베이스에 더 이상 데이터가 없을 때만 네트워크에 요청한다.

다음 그림은 RemoteMediator와 PagingSource가 이 유즈케이스를 충족하기 위해 함께 작동하는 방법을 보여주고 있다.

![paging1_image5.svg](/assets/images/paging1_image5.svg?raw=true)

RemoteMediator 구현은 네트워크에서 데이터베이스로 페이징 된 데이터를 로드하는 것을 관리하지만 데이터를 UI로 직접로드 하지는 않는다. 대신 앱은 데이터베이스를 데이터 소스로 사용한다. 즉, 앱은 데이터베이스에 캐시 된 데이터로만 나타낸다. PagingSource 구현 (예 : Room에서 생성 된 구현)은 데이터베이스에서 UI로 캐시 된 데이터로드를 처리한다.

### ViewModel 레이어

`Pager` 는 Repository Layer에서 구현된 PagingSource과 함께 PagingData 인스턴스를 구성하는
 반응형 스트림을 생성한다.

`PagingSource`에서 데이터를 로드하는 방법, 옵션을 정의한 `PagingConfig` 클래스와 함께 사용된다.

### UI 레이어

`PagingDataAdapter` 는 PagingData를 RecyclerView에 바인딩하기 위해 사용된다.

데이터를 어느 시점에 받아올 것인가 등 UI와 관련된 대부분의 일을 한다.

## Paging예제

페이징 라이브러리에 기능이 많아 기본기능부터 응용기능까지 나눠서 글을 쓸예정이다.

이번에는 기본으로 설정 및 데이터 가져와 표출하는 기능까지 만들겠다.

안드로이드 코드랩에있는 내용을 바탕으로 예제를 만들었다.

위에있는 구조처럼 크게 3개의 구조로 만들겠다.

![paging1_image3.png](/assets/images/paging1_image3.png?raw=true)

### 라이브러리 추가

```groovy
dependencies {

    implementation 'androidx.core:core-ktx:1.8.0'
    implementation 'androidx.appcompat:appcompat:1.4.2'
    implementation 'com.google.android.material:material:1.6.1'
    implementation 'androidx.constraintlayout:constraintlayout:2.1.4'
    testImplementation 'junit:junit:4.13.2'
    androidTestImplementation 'androidx.test.ext:junit:1.1.3'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.4.0'

    implementation 'androidx.activity:activity-ktx:1.4.0'
    implementation 'androidx.fragment:fragment-ktx:1.4.1'

    def paging_version = "3.1.1"
    implementation "androidx.paging:paging-runtime:$paging_version"
    testImplementation "androidx.paging:paging-common:$paging_version"

    def retrofitVersion = '2.9.0'
    implementation "com.squareup.retrofit2:retrofit:$retrofitVersion"
    implementation "com.squareup.retrofit2:converter-gson:$retrofitVersion"
    implementation "com.squareup.retrofit2:retrofit-mock:$retrofitVersion"
    implementation "com.squareup.okhttp3:logging-interceptor:4.9.0"

    def lifecycleVersion = '2.4.1'
    implementation "androidx.lifecycle:lifecycle-runtime-ktx:$lifecycleVersion"
    implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:$lifecycleVersion"
    implementation "androidx.lifecycle:lifecycle-livedata-ktx:$lifecycleVersion"
    implementation "androidx.lifecycle:lifecycle-viewmodel-savedstate:$lifecycleVersion"
}
```

뷰바인딩도 사용하기에 뷰바인딩을 설정해준다.

```groovy
android {
    viewBinding {
        enabled true
    }
}
```

인터넷을 사용하기에 메니페스트에 인터넷 권한을 추가해준다.

```xml
<uses-permission android:name="android.permission.INTERNET"/>
```

### 데이터 소스 정의

먼저 데이터가 정의되어야 다른코드들을 작성할 수 있기에 데이터 정의및 가져오는 코드를 만들겠다.

레트로핏을 사용해 서버에서 데이터를 가져온다. 이 예제에서는 깃허브 API를 사용한다.

- Repo

사용할 데이터들이다.

```kotlin
data class Repo(
    @field:SerializedName("id") val id: Long,
    @field:SerializedName("name") val name: String,
    @field:SerializedName("full_name") val fullName: String,
    @field:SerializedName("description") val description: String?,
    @field:SerializedName("html_url") val url: String,
    @field:SerializedName("stargazers_count") val stars: Int,
    @field:SerializedName("forks_count") val forks: Int,
    @field:SerializedName("language") val language: String?
)
```

- RepoSearchResponse

API문서에있는 리스폰스 데이터이다.

```kotlin
data class RepoSearchResponse(
    @SerializedName("total_count") val total: Int = 0,
    @SerializedName("items") val items: List<Repo> = emptyList(),
    val nextPage: Int? = null
)
```

- GithubService

레트로핏 빌더를 만들어 서버와 통신한다.

```kotlin
const val IN_QUALIFIER = "in:name,description"

interface GithubService {
    @GET("search/repositories?sort=stars")
    suspend fun searchRepos(
        @Query("q")query: String, @Query("page") page : Int, @Query("per_page") itemsPerPage: Int
    ) : RepoSearchResponse

    companion object{
        private const val BASE_URL = "https://api.github.com/"

        fun create(): GithubService{
            val logger = HttpLoggingInterceptor()
            logger.level = HttpLoggingInterceptor.Level.BASIC

            val client = OkHttpClient.Builder()
                .addInterceptor(logger)
                .build()

            return Retrofit.Builder()
                .baseUrl(BASE_URL)
                .client(client)
                .addConverterFactory(GsonConverterFactory.create())
                .build()
                .create(GithubService::class.java)
        }
    }
}
```

IN_QUALIFIER는 [상세 검색](https://docs.github.com/en/search-github/searching-on-github/searching-for-repositories)을 위해 사용한다. in:name,description뜻은 레퍼지토리에 원하는 이름이있거나 설명이있는 레퍼지토리를 찾는다.

- GithubPagingSource

```kotlin
private const val GITHUB_STARTING_PAGE_INDEX = 1

class GithubPagingSource(
    private val service: GithubService,
    private val query: String
) : PagingSource<Int,Repo>(){

    override suspend fun load(params: LoadParams<Int>): LoadResult<Int, Repo> {
        val position = params.key ?: GITHUB_STARTING_PAGE_INDEX
        val apiQuery = query + IN_QUALIFIER

        return try {
            val response = service.searchRepos(apiQuery,position,params.loadSize)
            val repos = response.items
            val nextKey = if (repos.isEmpty()){
                null
            }else{
                position + (params.loadSize / NETWORK_PAGE_SIZE)
            }
            LoadResult.Page(data = repos, prevKey = if (position == GITHUB_STARTING_PAGE_INDEX) null else position - 1, nextKey = nextKey)
        } catch (exception: IOException) {
            LoadResult.Error(exception)
        } catch (exception: HttpException) {
            LoadResult.Error(exception)
        }
    }

    override fun getRefreshKey(state: PagingState<Int, Repo>): Int? {
        return state.anchorPosition?.let { anchorPosition ->
            state.closestPageToPosition(anchorPosition)?.prevKey?.plus(1)
                ?: state.closestPageToPosition(anchorPosition)?.nextKey?.minus(1)
        }
    }
}
```

PagingSource를 만들기 위해서는 데이터 로드를 위한 식별자와 데이터를 정의해야 한다.

식별자는 페이지 번호등을 의미한다. 페이지번호(식별자)를 서버나 내부저장소에 전달하여 데이터를 가져오는것이다.

- **`load(params: LoadParams<Key>)`**  
load함수는 사용자가 스크롤 할 때마다 데이터를 비동기적으로 가져온다.
  - LoadParams 객체는 로드 작업과 관련된 정보를 가지고 있다.
    params.key에 현재 페이지 인덱스를 관리한다. 처음 데이터를 로드할 때에는 null이 반환된다.
    params.loadSize는 가져올 데이터의 갯수를 관리한다.

  - load 함수는 LoadResult를 반환한다.
    LoadResult.Page : 로드에 성공한 경우, 데이터와 이전 다음 페이지 Key가 포함된다.
    LoadResult.Error : 오류가 발생한 경우이다.
    지금예제에는 검색어, 페이지번호, 가져올 데이터 갯수를정의해 서버에요청한다. 그리고 그 결과값중
    `items` 객체만 가져와 키와함께 LoadResult로 반환한다.

- **getRefreshKey()**
  - 스와이프 Refresh나 데이터 업데이트 등으로 현재 목록을 대체할 새 데이터를 로드할 때 사용된다.PagingData는 Component에서 설명한 것처럼 새로고침 될 때마다 상응하는 PagingData를 생성해야한다.즉, 수정이 불가능하고 새로운 인스턴스를 만들어야한다.
  - 가장 최근에 접근한 인덱스인 anchorPosition으로 주변 데이터를 다시 로드한다.

### PagingData 빌드 및 구성

PagingData를 구성하기 위해서는 이를 다른 앱 레이어에 전달하기 위한 API를 결정해야한다. 여기서는 Flow타입을 사용하겠다.

- GithubRepository

```kotlin
class GithubRepository(private val service: GithubService) {

    fun getSearchResultStream(query : String): Flow<PagingData<Repo>> {
        return Pager(config = PagingConfig(pageSize = NETWORK_PAGE_SIZE,enablePlaceholders = false),
            pagingSourceFactory = {GithubPagingSource(service,query)}).flow
    }

    companion object {
        const val NETWORK_PAGE_SIZE = 30
    }
}
```

**`PagingConfig`** 이 클래스는 로드 대기 시간, 초기 로드의 크기 요청 등 `PagingSource`에서 컨텐츠를 로드하는 방법에 관한 옵션을 설정한다.

- pageSize - 각 페이지에 로드할 데이터 수를 가리킨다.
- enablePlaceholders - 플레이스 홀더 사용 여부
- maxSize - 기본적으로 Paging은 모든 페이지를 메모리에 유지한다. 스크롤할 때 메모리를 낭비하지 않기위해 설정할 수 있다.

`**pagingSourceFactory**`PagingSource 인스턴스를 생성한다.

### ViewModel에서 PagingData 요청 및 캐시

구글 예제에는 플로우를사용해 여러 기능들이있지만 여기서는 페이징의 고유 기능만 살펴볼것이기에 최대한 간단하게 만들었다.

뷰모델 생성을위해 팩토리와 인잭션 클래스를 만든다.

- ViewModelFactory

```kotlin
class ViewModelFactory(
    owner: SavedStateRegistryOwner,
    private val repository: GithubRepository
) : AbstractSavedStateViewModelFactory(owner, null) {

    override fun <T : ViewModel?> create(
        key: String,
        modelClass: Class<T>,
        handle: SavedStateHandle
    ): T {
        if (modelClass.isAssignableFrom(SearchRepositoriesViewModel::class.java)) {
            @Suppress("UNCHECKED_CAST")
            return SearchRepositoriesViewModel(repository, handle) as T
        }
        throw IllegalArgumentException("Unknown ViewModel class")
    }
}
```

- Injection

```kotlin
object Injection {

    private fun provideGithubRepository(): GithubRepository {
        return GithubRepository(GithubService.create())
    }

    fun provideViewModelFactory(owner: SavedStateRegistryOwner): ViewModelProvider.Factory {
        return ViewModelFactory(owner, provideGithubRepository())
    }
}
```

- SearchRepositoriesViewModel

```kotlin
class SearchRepositoriesViewModel(
    private val repository: GithubRepository,
    private val savedStateHandle: SavedStateHandle
) : ViewModel() {
    var pagingDataFlow: Flow<PagingData<Repo>>

    init {
        pagingDataFlow = searchRepo(queryString = "android")
    }

    private fun searchRepo(queryString: String): Flow<PagingData<Repo>> = repository.getSearchResultStream(queryString).cachedIn(viewModelScope)
}
```

`PagingData`는 `RecyclerView`에 표시되는 데이터의 변경 가능한 업데이트 스트림을 포함하는 독립된 유형이다. `PagingData`의 각 내보내기는 완전히 독립적이며 하나의 쿼리에 관해 여러 개의 `PagingData`를 내보낼 수 있다. 따라서 `PagingData`의 `Flows`는 다른 `Flows`와 독립적으로 노출되어야 한다.

`cachedIn(CoroutineScope)`는 CoroutineScope에서 데이터를 캐시할 수 있는 메소드이다.

### PagingData를 사용하도록 어댑터 설정

데이터 받아오는 파트가 완성됐으니 이제 UI를 만들자

- repo_view_item

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:paddingHorizontal="12dp"
    android:paddingTop="12dp"
    tools:ignore="UnusedAttribute">

    <TextView
        android:id="@+id/repo_name"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:textColor="#1F75FE"
        android:textSize="24sp"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        tools:text="android-architecture"/>

    <TextView
        android:id="@+id/repo_description"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:maxLines="10"
        android:paddingVertical="12dp"
        android:textColor="?android:textColorPrimary"
        android:textSize="16sp"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/repo_name"
        tools:ignore="UnusedAttribute"
        tools:text="A collection of samples to discuss and showcase different architectural tools and patterns for Android apps."/>

    <TextView
        android:id="@+id/repo_language"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginStart="0dp"
        android:paddingVertical="12dp"
        android:text="Language: %s"
        android:textSize="16sp"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toStartOf="@+id/rank"
        app:layout_constraintTop_toBottomOf="@+id/repo_description"
        tools:ignore="RtlCompat"/>

    <LinearLayout
        android:id="@+id/rank"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginStart="10dp"
        app:layout_constraintStart_toEndOf="@+id/repo_language"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/repo_description"
        android:orientation="horizontal">
        <ImageView
            android:id="@+id/star"
            android:layout_width="wrap_content"
            android:layout_marginVertical="12dp"
            android:layout_height="wrap_content"
            android:src="@drawable/ic_star"
            app:layout_constraintEnd_toStartOf="@+id/repo_stars"
            app:layout_constraintBottom_toBottomOf="@+id/repo_stars"
            app:layout_constraintTop_toTopOf="@+id/repo_stars" />

        <TextView
            android:id="@+id/repo_stars"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:paddingVertical="12dp"
            android:textSize="16sp"
            tools:text="30"/>

        <ImageView
            android:id="@+id/forks"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginVertical="12dp"
            android:src="@drawable/ic_git_branch"/>

        <TextView
            android:id="@+id/repo_forks"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:paddingVertical="12dp"
            android:textSize="16sp"
            tools:text="30"/>
    </LinearLayout>
</androidx.constraintlayout.widget.ConstraintLayout>
```

- RepoViewHolder

```kotlin
class RepoViewHolder(private val binding : RepoViewItemBinding) : RecyclerView.ViewHolder(binding.root) {
    private var repo: Repo? = null

    init {
        binding.root.setOnClickListener {
            repo?.url?.let { url ->
                val intent = Intent(Intent.ACTION_VIEW, Uri.parse(url))
                binding.root.context.startActivity(intent)
            }
        }
    }

    fun bind(repo: Repo?){
        if (repo == null){
            val resources = itemView.resources
            binding.apply {
                repoName.text = resources.getString(R.string.loading)
                repoDescription.visibility = View.GONE
                repoLanguage.visibility = View.GONE
                repoStars.text = resources.getString(R.string.unknown)
                repoForks.text = resources.getString(R.string.unknown)
            }
        }else{
            showRepoData(repo)
        }
    }

    private fun showRepoData(repo: Repo){
        this.repo = repo
        binding.repoName.text = repo.fullName

        var descriptionVisibility = View.GONE
        if (repo.description != null){
            binding.repoDescription.text = repo.description
            descriptionVisibility = View.VISIBLE
        }
        binding.repoDescription.visibility = descriptionVisibility

        binding.repoStars.text = repo.stars.toString()
        binding.repoForks.text = repo.forks.toString()

        var languageVisibility = View.GONE
        if (!repo.language.isNullOrEmpty()) {
            val resources = this.itemView.context.resources
            binding.repoLanguage.text = resources.getString(R.string.language, repo.language)
            languageVisibility = View.VISIBLE
        }
        binding.repoLanguage.visibility = languageVisibility
    }

    companion object{
        fun create(parent: ViewGroup): RepoViewHolder{
            val layoutInflater = LayoutInflater.from(parent.context)
            return RepoViewHolder(RepoViewItemBinding.inflate(layoutInflater))
        }
    }
}
```

- ReposAdapter

```kotlin
class ReposAdapter : PagingDataAdapter<Repo, RepoViewHolder>(REPO_COMPARATOR) {

    override fun onBindViewHolder(holder: RepoViewHolder, position: Int) {
        val repoItem = getItem(position)
        if (repoItem != null){
            holder.bind(repoItem)
        }
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): RepoViewHolder {
        return RepoViewHolder.create(parent)
    }

    companion object{
        private val REPO_COMPARATOR = object : DiffUtil.ItemCallback<Repo>(){
            override fun areItemsTheSame(oldItem: Repo, newItem: Repo): Boolean = oldItem.fullName == newItem.fullName

            override fun areContentsTheSame(oldItem: Repo, newItem: Repo): Boolean = oldItem == newItem
        }
    }
}
```

`PagingDataAdapter` 와 `DiffUtil`를 사용하여 어댑터를 만든다.

- activity_search_repositories

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".ui.SearchRepositoriesActivity">
    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/list"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:paddingVertical="12dp"
        android:scrollbars="vertical"
        app:layoutManager="LinearLayoutManager"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        tools:ignore="UnusedAttribute"/>

    <TextView android:id="@+id/emptyList"
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:gravity="center"
        android:text="No results 😓"
        android:textSize="24sp"
        android:visibility="gone"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"/>

    <ProgressBar
        android:id="@+id/progressBar"
        android:visibility="gone"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>

</androidx.constraintlayout.widget.ConstraintLayout>
```

- SearchRepositoriesActivity

```kotlin
class SearchRepositoriesActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val binding = ActivitySearchRepositoriesBinding.inflate(layoutInflater)
        setContentView(binding.root)

        val viewModel = ViewModelProvider(this,Injection.provideViewModelFactory(owner = this))[SearchRepositoriesViewModel::class.java]
        val decoration = DividerItemDecoration(this, DividerItemDecoration.VERTICAL)
        binding.list.addItemDecoration(decoration)

        val repoAdapter = ReposAdapter()
        binding.list.adapter = repoAdapter

        lifecycleScope.launch {
            viewModel.pagingDataFlow.collectLatest { pagingData ->
                repoAdapter.submitData(pagingData)
            }
        }

        repoAdapter.addLoadStateListener { combinedLoadStates ->
            binding.progressBar.isVisible = combinedLoadStates.source.refresh is LoadState.Loading
        }
    }
}
```

뷰모델의 flow를 가져와 데이터를 표출한다.

그리고 `addLoadStateListener` 를 통해 데이터 로딩 상태를 얻어 상태에따라 UI를 바꿔줄수있다.

- LoadState.NotLoading : 활성 로드 작업이 없고 오류가 없음
- LoadState.Loading : 활성 로드 작업이 있음
- LoadState.Error : 오류가 있음

실행시키면 다음과같이 나온다.

![paging1_image4.gif](/assets/images/paging1_image4.gif?raw=true)

전체코드는 아래링크에 있다.

[https://github.com/cellodove/Paging_Example](https://github.com/cellodove/Paging_Example)

## 참조

> [https://developer.android.com/topic/libraries/architecture/paging/v3-overview?hl=ko](https://developer.android.com/topic/libraries/architecture/paging/v3-overview?hl=ko)
>
> [https://developer.android.com/codelabs/android-paging?hl=ko#0](https://developer.android.com/codelabs/android-paging?hl=ko#0)
>
> [https://docs.github.com/en/search-github](https://docs.github.com/en/search-github)
>
> [https://genius-dev.tistory.com/entry/Android-Jetpack-Paging-3-라이브러리-사용하기](https://genius-dev.tistory.com/entry/Android-Jetpack-Paging-3-%EB%9D%BC%EC%9D%B4%EB%B8%8C%EB%9F%AC%EB%A6%AC-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0)
>
> [https://codechacha.com/ko/android-livedata-distinct-until-changed/](https://codechacha.com/ko/android-livedata-distinct-until-changed/)
>
> [https://tourspace.tistory.com/434](https://tourspace.tistory.com/434)
>
> [https://medium.com/hongbeomi-dev/정리-코틀린-flow-사용하기-android-dev-summit-2021-3606429f3c5f](https://medium.com/hongbeomi-dev/%EC%A0%95%EB%A6%AC-%EC%BD%94%ED%8B%80%EB%A6%B0-flow-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0-android-dev-summit-2021-3606429f3c5f)
>
> [https://www.charlezz.com/?p=44684](https://www.charlezz.com/?p=44684)
>
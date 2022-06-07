---
title : "클린 아키텍처(Clean Architecture)"
excerpt: "클린 아키텍처에 대하여 (with android)"

categories :
  - Cs

toc: true
toc_sticky: true
last_modified_at: 2022-06-06
---

![cs_architecture_image1.jpg](/assets/images/cs_architecture_image1.jpg?raw=true)

## 클린 아키텍처란?

우리가 만들고있는 애플리케이션은 수많은 기능들이 있기에 복잡도가 높다.

복잡도가 높은 애플리케이션을 개발할때 어떻게 해야 유지 보수하기 쉽고 고품질의 코드를 작성할 수 있을까?

애플리케이션은 새로운 기능이 추가된다거나 내부 로직이 변경되어야 하는 일이 생겼을때 유연하게 대처할 수 있도록 구조화 해야한다.

클린 아키텍처의 목표는 계층을 분리하여 관심사를 분리하는 것이다. 관심사를 분리하는 것이 어떤 의미가 있을까?

예를 들어 안드로이드 로컬 DB를 Realm을 사용하고 있었다. 이미 Realm으로 기능을 구현하고 제품까지 나와있는 상황이다. 이때 DB를 Room으로 교체해야한다면 어찌해야할까 이미 프로젝트 복잡도가 높아져 DB를 바꾸는것이 쉽지 않을것이다. 하지만 클린 아키텍처 구조로 만든다면 변화에 유연하게 코드를 작성할 수 있다.

![cs_architecture_image2.jpeg](/assets/images/cs_architecture_image2.jpeg?raw=true)

클린 아키텍처는 총 4개의 계층으로 이루어져 있다.

- Entities

엔티티는 비즈니스 규칙을 캡슐화한다. 메서드를 갖는 객체일 수도 있지만 데이터 구조와 함수의 집합일 수도 있다. 가장 일반적이고 높은 수준의 규칙을 캡슐화한다. 외부에서 무언가 변경될 때 변경될 가능성이 가장 적다.

- Use Cases

유스케이스는 애플리케이션의 고유 규칙을 캡슐화하며 엔티티로부터의 데이터 흐름을 조합한다. 유스케이스 계층의 변경이 엔티티에 영향을 줘서는 안 되며 데이터베이스, 공통 프레임워크 및 UI에 대한 변경으로부터 격리된다.

- Interface Adapters(Presenter)

인터페이스 어댑터는 Entity 및 UseCase의 편리한 형식에서 데이터베이스 및 웹에 적용할 수 있는 형식으로 변환한다. 이 계층에는 MVP 패턴의 Presenter, MVVM 패턴의 ViewModel가 포함된다. 즉, 순수한 비즈니스 로직만을 담당하는 역할을 한다.

- Frameworks & Drivers(Web, Db)

프레임워크와 드라이버는 상세한 정보들을 둔다. 웹 프레임워크, 데이터베이스, UI, HTTP client 등으로 구성된 가장 바깥쪽 계층이다.

클린 아키텍처가 동작하기 위해서는 의존성 규칙을 잘 지켜야 한다.

각각의 클래스는 한 가지 역할만 수행하고, 서로 의존 관계를 어떻게 할지 규칙이 정해져 있고 지켜야 한다.

의존성 규칙은 반드시 외부에서 내부로, 저수준 정책에서 고수준 정책으로 향해야 한다. 위 그림에서 내부로 갈수록 의존성이 낮아진다. 예를 들어 안드로이드에서 ViewModel은 로컬 DB나 Web과 같은 세부적인 사항에 의존하지 않아야 한다.

이렇게 관심사를 나누면 다음과 같은 이점을 얻을 수 있다.

- 새로운 기능을 빠르게 적용할 수 있다.
- 집중화된 클래스에 따른 프로젝트 유지 관리에 용이하다.
- 패키지 구조 탐색이 쉬워진다.
- 테스트 코드 작성에 용이하다.

## 안드로이드에서 클린 아키텍처

![cs_architecture_image3.png](/assets/images/cs_architecture_image3.png?raw=true)

클린 아키텍처를 안드로이드에 접목시킬 때는 일반적으로 Presentation, Domain, Data 총 3개의 계층으로 나눈다.

- Presentation (UI Layer)

화면과 입력에 대한 처리 등 UI와 관련된 부분을 담당한다. Activity, Fragment, View, Presenter 및 ViewModel을 포함한다. Presentation 계층은 Domain 계층에 대한 의존성을 가지고 있다.

- Domain

애플리케이션의 비즈니스 로직에서 필요한 UseCase와 Model을 포함하고 있다. UseCase는 각 개별 기능 또는 비즈니스 논리 단위이며, Presentation, Data 계층에 대한 의존성을 가지지 않고 독립적으로 분리되어 있다.

안드로이드의 의존성을 갖지 않고 java 및 kotlin 코드로만 구성하며 다른 애플리케이션에서도 사용할 수 있다. Repository 인터페이스도 포함되어 있다.

- Data

Domain 계층에 의존성을 가지고 있다. Domain 계층의 Repository 구현체를 포함하고 있으며, 데이터베이스, 서버와의 통신도 Data 계층에서 이루어진다. 또한 mapper 클래스를 통해 Data 계층의 모델을 Domain 계층의 모델로 변환해주는 역할도 하게 된다.

## Clean Architecture Android Sample

클린 아키텍처를 적용한 예제 앱을 만들어보자 깃헙 API를 사용해 사용자를 검색하여 목록을 불러오고 북마크 기능을 넣어 원하는 사용자를 내부데이터 저장소에 저장하고 삭제하는 기능을 구현하겠다.

의존성 주입을 위해 Hilt를 사용한다.

### 프로젝트 구조

Presentation, Domain, Data 계층으로 나누기 위해 모듈을 여러 개로 나누었다.

![cs_architecture_image4.png](/assets/images/cs_architecture_image4.png?raw=true)

app 모듈은 Application 객체를 가지고 있으며 Presentation, Domain, Data 계층에 대한 의존성을 모두 갖는다. Hilt를 통해 의존성 주입도 app 모듈에서 하게 된다.

모듈을 생성했으면 규칙에 맞게 의존성을 넣어준다.

```groovy
// build.gradle (app 모듈)
dependencies {
    implementation project(":data")
    implementation project(":domain")
    implementation project(":presentation")
}

// build.gradle (presentation 모듈)
dependencies {
    implementation project(":domain")
}

// build.gradle (data 모듈)
dependencies {
    implementation project(":domain")
}
```

### buildSrc

모듈이 많다보니 dependency를 관리가 까다롭다. 이때 buildSrc를 사용해 한번해 관리할 수가있다.

Gradle 공식 문서에 다음과 같이 나와있다.

Gradle이 수행되면 buildSrc 디렉토리가 존재하는지 체크한다. 이 경우에 Gradle은 자동적으로 코드를 컴파일하고 테스트한 뒤 빌드 스크립트의 classpath에 넣는다. 이 방법은 유지 보수, 리팩토링 및 코드 테스트가 더 쉬워진다고 한다.

buildSrc를 통해 dependency를 관리하는 방법은 다음과 같다.

1. 루트 폴더에 buildSrc폴더 생성
2. buildSrc 폴더 안에 build.gradle.kts 파일을 생성한 뒤, 아래의 코드와 같이 kotlin-dsl를 enable 한다. 그리고 gradle sync를 하여 이 plugin을 활성화 한다.

    ```groovy
    plugins {
        `kotlin-dsl`
    }
    repositories {
        mavenCentral()
    }
    ```

3. src > main > java 폴더를 생성한다.
4. java 폴더안에 Kotlin 파일을 생성하고, 원하는 이름을 지정한다.
5. 생성한 Kotlin 파일에 dependency를 지정한다.

```kotlin
object Versions {
    const val KOTLIN_VERSION = "1.5.0"
    const val KOTLINX_COROUTINES = "1.5.0"
    const val BUILD_GRADLE = "4.2.1"

    const val CORE_KTX = "1.5.0"
    const val APP_COMPAT = "1.3.0"
    const val ACTIVITY_KTX = "1.2.3"
    const val FRAGMENT_KTX = "1.3.4"
    const val LIFECYCLE_KTX = "2.3.1"
    const val ROOM = "2.3.0"

    const val HILT = "2.35.1"
    const val MATERIAL = "1.3.0"

    const val RETROFIT = "2.7.1"
    const val OKHTTP = "4.3.1"

    const val JUNIT = "4.13.2"
    const val ANDROID_JUNIT = "1.1.2"
    const val ESPRESSO_CORE = "3.3.0"

    const val GLIDE_VER = "4.11.0"
    const val PAGING_VERSION = "3.0.0"
}

object Kotlin {
    const val KOTLIN_STDLIB      = "org.jetbrains.kotlin:kotlin-stdlib:${Versions.KOTLIN_VERSION}"
    const val COROUTINES_CORE    = "org.jetbrains.kotlinx:kotlinx-coroutines-core:${Versions.KOTLINX_COROUTINES}"
    const val COROUTINES_ANDROID = "org.jetbrains.kotlinx:kotlinx-coroutines-android:${Versions.KOTLINX_COROUTINES}"
}

object AndroidX {
    const val CORE_KTX                = "androidx.core:core-ktx:${Versions.CORE_KTX}"
    const val APP_COMPAT              = "androidx.appcompat:appcompat:${Versions.APP_COMPAT}"

    const val ACTIVITY_KTX            = "androidx.activity:activity-ktx:${Versions.ACTIVITY_KTX}"
    const val FRAGMENT_KTX            = "androidx.fragment:fragment-ktx:${Versions.FRAGMENT_KTX}"

    const val LIFECYCLE_VIEWMODEL_KTX = "androidx.lifecycle:lifecycle-viewmodel-ktx:${Versions.LIFECYCLE_KTX}"
    const val LIFECYCLE_LIVEDATA_KTX  = "androidx.lifecycle:lifecycle-livedata-ktx:${Versions.LIFECYCLE_KTX}"

    const val ROOM_RUNTIME            = "androidx.room:room-runtime:${Versions.ROOM}"
    const val ROOM_KTX                = "androidx.room:room-ktx:${Versions.ROOM}"
    const val ROOM_COMPILER           = "androidx.room:room-compiler:${Versions.ROOM}"
}

object Google {
    const val HILT_ANDROID          = "com.google.dagger:hilt-android:${Versions.HILT}"
    const val HILT_ANDROID_COMPILER = "com.google.dagger:hilt-android-compiler:${Versions.HILT}"

    const val MATERIAL = "com.google.android.material:material:${Versions.MATERIAL}"
}

object Libraries {
    const val RETROFIT                   = "com.squareup.retrofit2:retrofit:${Versions.RETROFIT}"
    const val RETROFIT_CONVERTER_GSON    = "com.squareup.retrofit2:converter-gson:${Versions.RETROFIT}"
    const val OKHTTP                     = "com.squareup.okhttp3:okhttp:${Versions.OKHTTP}"
    const val OKHTTP_LOGGING_INTERCEPTOR = "com.squareup.okhttp3:logging-interceptor:${Versions.OKHTTP}"
    const val GLIDE = "com.github.bumptech.glide:glide:${Versions.GLIDE_VER}"
    const val GLIDE_ANNOTATION = "com.github.bumptech.glide:compiler:${Versions.GLIDE_VER}"
}

object Test {
    const val JUNIT         = "junit:junit:${Versions.JUNIT}"
    const val ANDROID_JUNIT = "androidx.test.ext:junit:${Versions.ANDROID_JUNIT}"
    const val ESPRESSO_CORE = "androidx.test.espresso:espresso-core:${Versions.ESPRESSO_CORE}"
}

object Paging{
    const val PAGING_RUNTIME = "androidx.paging:paging-runtime:$PAGING_VERSION"
    const val PAGING_COMMON  = "androidx.paging:paging-common:$PAGING_VERSION"
    const val PAGING_RXJAVA2 = "androidx.paging:paging-rxjava2:$PAGING_VERSION"
    const val PAGING_RXJAVA3 = "androidx.paging:paging-rxjava3:$PAGING_VERSION"
    const val PAGING_GUAVA   = "androidx.paging:paging-guava:$PAGING_VERSION"
    const val PAGING_COMPOSE = "androidx.paging:paging-compose:1.0.0-alpha09"
}
```

```groovy
// build.gradle
dependencies {
    implementation project(":data")
    implementation project(":domain")
    implementation project(":presentation")

    implementation(Kotlin.COROUTINES_CORE)
    implementation(Kotlin.COROUTINES_ANDROID)
    implementation(Kotlin.KOTLIN_STDLIB)

    implementation(AndroidX.CORE_KTX)
    implementation(AndroidX.APP_COMPAT)
    implementation(AndroidX.ROOM_RUNTIME)
    implementation(AndroidX.ROOM_KTX)
    kapt(AndroidX.ROOM_COMPILER)

    implementation(Libraries.RETROFIT)
    implementation(Libraries.RETROFIT_CONVERTER_GSON)
    implementation(Libraries.OKHTTP)
    implementation(Libraries.OKHTTP_LOGGING_INTERCEPTOR)

    implementation(Google.HILT_ANDROID)
    kapt (Google.HILT_ANDROID_COMPILER)

}
```

### Domain 계층

![cs_architecture_image5.png](/assets/images/cs_architecture_image5.png?raw=true)

- **Repository**

```kotlin
interface GithubRepository {
    suspend fun getRepos(owner : String) : GithubRepo

    suspend fun getAll() : List<LocalGithubItem>

    suspend fun insertItem(item: LocalGithubItem)

    suspend fun deleteItem(item: LocalGithubItem)
}
```

Github의 Repository 목록을 가져오기 위한 Repository의 인터페이스를 만들어준다. GithubRepository의 구현체는 Data 계층에 위치한다.

- **UseCase**

Github에서 Repository 목록을 가져오는 기능을 제공하는 유스케이스이다. GithubRepository를 생성자로 주입받아 데이터를 가져오는 역할을 한다.

1. GetGithubReposUseCase

    ```kotlin
    class GetGithubReposUseCase(private val githubRepository : GithubRepository) {
        operator fun invoke(
            owner : String,
            scope : CoroutineScope,
            onResult : (GithubRepo) -> Unit = {}
        ){
            scope.launch(Dispatchers.Main) {
                val deferred = async(Dispatchers.IO){
                    githubRepository.getRepos(owner)
                }
                onResult(deferred.await())
            }
        }
    }
    ```

2. SaveLocalGithubReposUseCase

    ```kotlin
    class SaveLocalGithubReposUseCase(private val githubRepository : GithubRepository) {
        operator fun invoke(
            item : LocalGithubItem,
            scope : CoroutineScope,
            onResult : (Unit) -> Unit = {}
        ){
            scope.launch(Dispatchers.Main) {
                val deferred = async(Dispatchers.IO){
                    githubRepository.insertItem(item)
                }
                onResult(deferred.await())
            }
        }
    }
    ```

3. DeleteLocalGithubReposUseCase

    ```kotlin
    class DeleteLocalGithubReposUseCase(private val githubRepository : GithubRepository) {
        operator fun invoke(
            item : LocalGithubItem,
            scope : CoroutineScope,
            onResult : (Unit) -> Unit = {}
        ){
            scope.launch(Dispatchers.Main) {
                val deferred = async(Dispatchers.IO){
                    githubRepository.deleteItem(item)
                }
                onResult(deferred.await())
            }
        }
    }
    ```

4. GetAllLocalGithubReposUseCase

    ```kotlin
    class GetAllLocalGithubReposUseCase(private val githubRepository : GithubRepository) {
        operator fun invoke(
            scope : CoroutineScope,
            onResult : (List<LocalGithubItem>) -> Unit = {}
        ){
            scope.launch(Dispatchers.Main) {
                val deferred = async(Dispatchers.IO){
                    githubRepository.getAll()
                }
                onResult(deferred.await())
            }
        }
    }
    ```

- Model

Domain 계층의 Model입니다. Github Repository의 정보를 가지고 있으며 안드로이드의 의존성을 갖지 않도록 작성해준다.

interface로 data 계층의 model이 domain model을 구현할 수 도있고 data class로 만들어 data계층의 model과 domain 계층의 model을 mapper로 변환해줄 수 도있다. 여기서는 두가지의 경우를 모두 사용하겠다.

1. GithubRepo

    ```kotlin
    interface GithubRepo {
        val item : List<Item>
    }
    ```

    1. Item

    ```kotlin
    data class Item(
        val login : String,
    
        val url : String,
    
        val avatar_url : String,
    
        val html_url : String
    )
    ```

1. LocalGithubItem

    ```kotlin
    data class LocalGithubItem (
    
        val login : String,
    
        val url : String,
    
        val avatarUrl : String,
    
        val htmlUrl : String
    )
    ```

### Data 계층

![cs_architecture_image6.png](/assets/images/cs_architecture_image6.png?raw=true)

- Model

1. GithubRepoRes

```kotlin
data class GithubRepoRes (

    @SerializedName("total_count")
    private val _total_count : Int,

    @SerializedName("incomplete_results")
    private val _incomplete_results : Boolean,

    @SerializedName("items")
    private val _items : List<Item>

) : GithubRepo{
    override val item: List<Item>
        get() = _items
}
```

Github API를 통해 데이터를 가져올 Model이다. Domain 계층의 GithubRepo를 구현해준다.

1. LocalGithubRepo

```kotlin
@Entity(tableName = "item")
data class LocalGithubRepo (
    @PrimaryKey()
    val login : String,

    val url : String,

    val avatar_url : String,

    val html_url : String
)
```

1. Mapper

```kotlin
fun mapperToLocalGithubRepo(localItem : List<LocalGithubItem>):List<LocalGithubRepo>{
    return localItem.toList().map {
        LocalGithubRepo(
            it.login,
            it.url,
            it.avatarUrl,
            it.htmlUrl
        )
    }
}

fun LocalGithubItem.map() = LocalGithubRepo(
    login,
    url,
    avatarUrl,
    htmlUrl
)

fun mapperToLocalGithubItem(localRepo : List<LocalGithubRepo>) : List<LocalGithubItem>{
    return localRepo.toList().map {
        LocalGithubItem(
            it.login,
            it.url,
            it.avatar_url,
            it.html_url
        )
    }
}

fun LocalGithubRepo.map() = LocalGithubItem(
    login,
    url,
    avatar_url,
    html_url
)
```

room을 사용해 데이터를 가져올 model이다. mapper를 사용해 domain데이터를 변환해준다.

- Service

레트로핏 api설정과 룸을 설정해준다.

1. GithubService

```kotlin
interface GithubService {

    @GET("search/users")
    suspend fun getRepos(@Query("q") q : String) : GithubRepoRes
}
```

1. LocalDao

```kotlin
@Dao
interface LocalDao {
    @Query("SELECT * FROM item ORDER BY login DESC")
    fun getAll() : List<LocalGithubRepo>

    @Insert
    fun insertItem(item: LocalGithubRepo)

    @Delete
    fun deleteItem(item: LocalGithubRepo)
}
```

1. AppDatabase

```kotlin
@Database(entities = [LocalGithubRepo::class], version = 1, exportSchema = false)
abstract class AppDatabase : RoomDatabase(){
    abstract fun localDao() : LocalDao

    companion object{
        @Volatile
        private lateinit var instance : AppDatabase

        fun getInstance(context: Context) : AppDatabase{
            synchronized(this){
                if (!::instance.isInitialized){
                    instance =
                        Room.databaseBuilder(
                            context,
                            AppDatabase::class.java,
                            "github-db"
                        ).build()
                }
            }
            return instance
        }
    }
}
```

- Source

1. GithubRemoteSource

```kotlin
interface GithubRemoteSource {
    suspend fun getRepos(owner : String) : GithubRepoRes
}

class GithubRemoteSourceImpl @Inject constructor(
    private val githubService: GithubService
) : GithubRemoteSource{
    override suspend fun getRepos(owner: String): GithubRepoRes {
        return githubService.getRepos(owner)
    }
}
```

1. LocalGithubRemoteSource

```kotlin
interface LocalGithubRemoteSource {
    suspend fun getAll() : List<LocalGithubRepo>

    suspend fun insertItem(item: LocalGithubRepo)

    suspend fun deleteItem(item: LocalGithubRepo)
}

class LocalGithubRemoteSourceImpl @Inject constructor(
    private val localDao: LocalDao
) : LocalGithubRemoteSource{

    override suspend fun getAll(): List<LocalGithubRepo> {
        return localDao.getAll()
    }

    override suspend fun insertItem(item: LocalGithubRepo) {
        localDao.insertItem(item)
    }

    override suspend fun deleteItem(item: LocalGithubRepo) {
        localDao.deleteItem(item)
    }
}
```

RemoteSource 또한 인터페이스로 구현한 뒤 구현체를 별도로 갖는 방식으로 구현했다.

- GithubRepositoryImpl

```kotlin
class GithubRepositoryImpl @Inject constructor(
    private val githubRemoteSource : GithubRemoteSource,
    private val localGithubRemoteSource : LocalGithubRemoteSource
) : GithubRepository {
    override suspend fun getRepos(owner: String): GithubRepo{
        return githubRemoteSource.getRepos(owner)
    }

    override suspend fun getAll(): List<LocalGithubItem> {
        return mapperToLocalGithubItem(localGithubRemoteSource.getAll())
    }

    override suspend fun insertItem(item: LocalGithubItem) {
        localGithubRemoteSource.insertItem(item.map())
    }

    override suspend fun deleteItem(item: LocalGithubItem) {
        localGithubRemoteSource.deleteItem(item.map())
    }
}
```

Domain 계층의 GithubRepository 인터페이스를 구현한다. RemoteSource를 생성자로 주입받아 데이터를 가져오게 된다.

GithubRemoteSource의 getRepos 함수는 `List<GithubRepoRes>`를 반환하지만 GithubRepoRes는 Domain 계층의 GithubRepo를 구현하고 있기 때문에 별도의 변환 과정 없이 반환이 가능하다.

room을 사용한 RemoteSource는 아까 만들어둔 mapper를 이용해 데이터를 변환해 준다.

### Presentation 계층

![cs_architecture_image7.png](/assets/images/cs_architecture_image7.png?raw=true)

- Base

중복 되는 코드들을 따로빼서 관리해준다. 여기서는 간단하게 컴포넌트별 기본으로 구현해야하는 내용만 작성하였다.

1. BaseActivity

```kotlin
abstract class BaseActivity<VB: ViewBinding>(private val bindingInflater:(inflater: LayoutInflater) -> VB) : AppCompatActivity() {
    lateinit var binding : VB
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = bindingInflater.invoke(layoutInflater)
        setContentView(binding.root)
    }
}
```

1. BaseFragment

```kotlin
abstract class BaseFragment<VB: ViewBinding>(private val bindingInflater:(inflater: LayoutInflater) -> VB) : Fragment() {
    lateinit var binding : VB

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        binding = bindingInflater.invoke(inflater)
        return binding.root
    }
}
```

1. BaseViewModel

```kotlin
abstract class BaseViewModel : ViewModel() {

    private val _isLoading by lazy { MutableLiveData(false) }
    val isLoading: LiveData<Boolean> by lazy { _isLoading }

    private val _toast by lazy { MutableLiveData<Event<String>>() }
    val toast: LiveData<Event<String>> by lazy { _toast }

    fun handleLoading(isLoading: Boolean) {
        _isLoading.postValue(isLoading)
    }

    fun showToast(message: String) {
        _toast.postValue(Event(message))
    }
}
```

1. Event

```kotlin
open class Event<out T>(private val content: T) {
    var hasBeenHandled = false
        private set

    fun getContentIfNotHandled(): T? {
        return if (hasBeenHandled) {
            null
        } else {
            hasBeenHandled = true
            content
        }
    }

    fun peekContent(): T = content
}
```

- Main

1. MainViewModel

```kotlin
@HiltViewModel
class MainViewModel @Inject constructor(
    private val getGithubReposUseCase: GetGithubReposUseCase,
    private val saveLocalGithubReposUseCase : SaveLocalGithubReposUseCase,
    private val deleteLocalGithubReposUseCase : DeleteLocalGithubReposUseCase,
    private val getAllLocalGithubReposUseCase : GetAllLocalGithubReposUseCase,

    ) : BaseViewModel() {

    private val scope = CoroutineScope(Dispatchers.IO)

    private val _githubRepositories = MutableLiveData<GithubRepo>()
    val githubRepositories: LiveData<GithubRepo> = _githubRepositories

    private val _localRepositories = MutableLiveData<List<LocalGithubItem>>()
    val localRepositories: LiveData<List<LocalGithubItem>> = _localRepositories

    fun getGithubRepositories(owner: String) {
        getGithubReposUseCase(owner, viewModelScope) {
            _githubRepositories.value = it
        }
    }

    fun bookmarkSave(item :Item){
        val localData = LocalGithubItem(login = item.login, url = item.url, avatarUrl = item.avatar_url, htmlUrl = item.html_url)
        scope.launch {
            saveLocalGithubReposUseCase(localData,viewModelScope)
        }
    }

    fun bookmarkDelete(item :Item){
        val localData = LocalGithubItem(login = item.login, url = item.url, avatarUrl = item.avatar_url, htmlUrl = item.html_url)
        scope.launch {
            deleteLocalGithubReposUseCase(localData,viewModelScope)
        }
    }

    fun bookmarkDeleteForLocal(item :LocalGithubItem){
        scope.launch {
            deleteLocalGithubReposUseCase(item,viewModelScope)
            bookmarkGet()
        }
    }

    fun bookmarkGet(){
        scope.launch {
            getAllLocalGithubReposUseCase(viewModelScope){
                _localRepositories.postValue(it)
            }
        }
    }
}
```

ViewModel에서는 Domain 계층의 유스케이스를 주입받아 데이터를 가져온다. Presentation 계층에서는 Data 계층의 의존성이 없기 때문에 Github API, 내부 저장소 데이터를 가져오는 구현체에 직접적으로 접근은 불가능하다.

1. MainActivity

```kotlin
@AndroidEntryPoint
class MainActivity : BaseActivity<ActivityMainBinding>(ActivityMainBinding::inflate){

    private val tabTitleArray = arrayOf("검색", "북마크")

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val viewPager = binding.viewPager
        val tabLayout = binding.tabLayout

        viewPager.adapter = ViewPagerAdapter(supportFragmentManager,lifecycle)
        TabLayoutMediator(tabLayout,viewPager){tab, position ->
            tab.text = tabTitleArray[position]
        }.attach()
    }
}
```

1. ViewPagerAdapter

```kotlin
private const val NUM_TABS = 2

class ViewPagerAdapter(fragmentManager: FragmentManager, lifecycle: Lifecycle) : FragmentStateAdapter(fragmentManager,lifecycle) {

    override fun getItemCount(): Int {
        return NUM_TABS
    }

    override fun createFragment(position: Int): Fragment {
        when(position){
            0 -> return SearchFragment()
            1 -> return BookMarkFragment()
        }
        return SearchFragment()
    }
}
```

뷰페이저를 사용해 화면을 이동한다.

- Search

api검색 화면 구현이다.

1. SearchFragment

```kotlin
@AndroidEntryPoint
class SearchFragment : BaseFragment<FragmentSearchBinding>(FragmentSearchBinding::inflate){
    private val viewModel: MainViewModel by viewModels()
    private val searchAdapter :SearchAdapter by lazy { SearchAdapter() }
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        viewModel.bookmarkGet()
        binding.recyclerView.adapter = searchAdapter
        binding.recyclerView.addItemDecoration(DividerItemDecoration(requireContext(), RecyclerView.VERTICAL))
        binding.searchButton.setOnClickListener {
            val owner = binding.searchEditText.text.toString()
            if (owner.isEmpty()){
                Toast.makeText(requireContext(),"내용을 입력하세요.",Toast.LENGTH_SHORT).show()
            }else{
                viewModel.getGithubRepositories(owner)
            }
        }
        subscribeToLiveData()

        searchAdapter.setOnItemClickListener(object : SearchAdapter.OnItemClickListener{
            override fun onItemClick(item: Item, isBookmark : Boolean) {
                if (isBookmark){
                    viewModel.bookmarkDelete(item)
                }else{
                    viewModel.bookmarkSave(item)
                }
                Toast.makeText(requireContext(),item.url,Toast.LENGTH_SHORT).show()
            }
        })
    }

    override fun onResume() {
        super.onResume()
        viewModel.bookmarkGet()
    }

    private fun subscribeToLiveData() {
        viewModel.localRepositories.observe(viewLifecycleOwner){ bookmarks ->
            viewModel.githubRepositories.observe(viewLifecycleOwner) {
                searchAdapter.setItems(it.item,bookmarks)
            }
        }
    }
}
```

1. SearchAdapter

```kotlin
class SearchAdapter : RecyclerView.Adapter<SearchAdapter.ViewHolder>(){
    private val items = mutableListOf<Item>()
    private val bookmarks = mutableListOf<LocalGithubItem>()

    interface OnItemClickListener{
        fun onItemClick(item : Item, isBookmark : Boolean)
    }
    private var listener : OnItemClickListener? = null
    fun setOnItemClickListener(listener : OnItemClickListener) {
        this.listener = listener
    }

    fun setItems(items: List<Item>, bookmark : List<LocalGithubItem>) {
        this.items.clear()
        this.items.addAll(items)
        this.bookmarks.clear()
        this.bookmarks.addAll(bookmark)

        notifyDataSetChanged()
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        val layoutInflater = LayoutInflater.from(parent.context)
        return ViewHolder(SearchItemBinding.inflate(layoutInflater))
    }

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        holder.bind(items[position])
    }

    override fun getItemCount(): Int {
        return items.size
    }

    inner class ViewHolder(private val binding: SearchItemBinding): RecyclerView.ViewHolder(binding.root){
        fun bind(repo : Item){
            binding.repoName.text = repo.login
            binding.repoUrl.text = repo.url
            Glide.with(binding.root)
                .load(repo.avatar_url)
                .override(100,100)
                .into(binding.profileImage)
            bookmarks.forEach {
                binding.bookMark.isChecked = repo.login == it.login
            }

            binding.bookMark.setOnClickListener {
                if (binding.bookMark.isChecked){
                    listener?.onItemClick(repo,false)
                }else{
                    listener?.onItemClick(repo,true)
                }
            }
        }
    }
}
```

api데이터와 내부저장소 데이터를 가져와 만약 내부저장소에 login데이터가 일치가는게있다면 북마크된 데이터라 생각하고 북마크 표시를 해주었다.

이미지는 glide를 통해 표출해주었다.

북마크 아이콘을 클릭하면 내부저장소에 저장되고 두번클릭하면 삭제된다.

- Bookmark

내부 저장소에 저장된 데이터를 표출한다.

1. BookMarkFragment

```kotlin
@AndroidEntryPoint
class BookMarkFragment : BaseFragment<FragmentBookmarkBinding>(FragmentBookmarkBinding::inflate){
    private val viewModel: MainViewModel by viewModels()
    private val bookmarkAdapter : BookmarkAdapter by lazy { BookmarkAdapter() }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        viewModel.bookmarkGet()
        subscribeToLiveData()

        binding.recyclerView.adapter = bookmarkAdapter
        binding.recyclerView.addItemDecoration(DividerItemDecoration(requireContext(), RecyclerView.VERTICAL))

        bookmarkAdapter.setOnItemClickListener(object : BookmarkAdapter.OnItemClickListener{
            override fun onItemClick(item: LocalGithubItem, isBookmark: Boolean) {
                viewModel.bookmarkDeleteForLocal(item)
            }
        })
    }

    override fun onResume() {
        super.onResume()
        viewModel.bookmarkGet()
    }

    private fun subscribeToLiveData() {
        viewModel.localRepositories.observe(viewLifecycleOwner){
            bookmarkAdapter.setItems(it)
        }
    }
}
```

1. BookmarkAdapter

```kotlin
class BookmarkAdapter : RecyclerView.Adapter<BookmarkAdapter.ViewHolder>(){
    private val items = mutableListOf<LocalGithubItem>()

    interface OnItemClickListener{
        fun onItemClick(item : LocalGithubItem, isBookmark : Boolean)
    }
    private var listener : OnItemClickListener? = null
    fun setOnItemClickListener(listener : OnItemClickListener) {
        this.listener = listener
    }

    fun setItems(items: List<LocalGithubItem>) {
        this.items.clear()
        this.items.addAll(items)

        notifyDataSetChanged()
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        val layoutInflater = LayoutInflater.from(parent.context)
        return ViewHolder(BookmarkItemBinding.inflate(layoutInflater))
    }

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        holder.bind(items[position])
    }

    override fun getItemCount(): Int {
        return items.size
    }

    inner class ViewHolder(private val binding: BookmarkItemBinding): RecyclerView.ViewHolder(binding.root){
        fun bind(repo : LocalGithubItem){
            binding.repoName.text = repo.login
            binding.repoUrl.text = repo.url
            Glide.with(binding.root)
                .load(repo.avatarUrl)
                .override(100,100)
                .into(binding.profileImage)

            binding.bookMark.setOnClickListener {
                listener?.onItemClick(repo,true)
            }
        }
    }
}
```

### App 모듈

![cs_architecture_image8.png](/assets/images/cs_architecture_image8.png?raw=true)

App 모듈에서는 의존성 주입을 위한 hilt 설정을 해준다.

1. App

```kotlin
@HiltAndroidApp
class App : Application()
```

1. ApiModule

```kotlin
@Module
@InstallIn(SingletonComponent::class)
internal object ApiModule {

    private const val CONNECT_TIMEOUT = 15L
    private const val WRITE_TIMEOUT = 15L
    private const val READ_TIMEOUT = 15L
    private const val BASE_URL = "https://api.github.com/"

    @Provides
    @Singleton
    fun provideCache(application: Application): Cache {
        return Cache(application.cacheDir, 10L * 1024 * 1024)
    }

    @Provides
    @Singleton
    fun provideOkHttpClient(cache: Cache): OkHttpClient {
        return OkHttpClient.Builder().apply {
            cache(cache)
            connectTimeout(CONNECT_TIMEOUT, TimeUnit.SECONDS)
            writeTimeout(WRITE_TIMEOUT, TimeUnit.SECONDS)
            readTimeout(READ_TIMEOUT, TimeUnit.SECONDS)
            retryOnConnectionFailure(true)
            addInterceptor(HttpLoggingInterceptor().apply {
                level = HttpLoggingInterceptor.Level.BODY
            })
        }.build()
    }

    @Provides
    @Singleton
    fun provideRetrofit(client: OkHttpClient): Retrofit {
        return Retrofit.Builder()
            .baseUrl(BASE_URL)
            .addConverterFactory(GsonConverterFactory.create())
            .client(client)
            .build()
    }

    @Provides
    @Singleton
    fun provideDeliveryService(retrofit: Retrofit): GithubService {
        return retrofit.create(GithubService::class.java)
    }
}
```

1. LocalModule

```kotlin
@Module
@InstallIn(SingletonComponent::class)
internal object LocalModule {

    @Singleton
    @Provides
    fun provideDatabase(@ApplicationContext context: Context): AppDatabase {
        return AppDatabase.getInstance(context)
    }

    @Provides
    fun provideDao(appDatabase : AppDatabase):LocalDao{
        return appDatabase.localDao()
    }

}
```

1. DataSourceModule

```kotlin
@Module
@InstallIn(SingletonComponent::class)
class DataSourceModule {

    @Singleton
    @Provides
    fun providesGithubRemoteSource(source: GithubRemoteSourceImpl): GithubRemoteSource {
        return source
    }

    @Singleton
    @Provides
    fun providesLocalGithubRemoteSource(source: LocalGithubRemoteSourceImpl): LocalGithubRemoteSource {
        return source
    }
}
```

1. RepositoryModule

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object RepositoryModule {

    @Singleton
    @Provides
    fun providesGithubRepository(repository: GithubRepositoryImpl): GithubRepository {
        return repository
    }
}
```

1. UseCaseModule

```kotlin
@Module
@InstallIn(ViewModelComponent::class)
object UseCaseModule {

    @Provides
    fun providesGetGithubReposUseCase(repository: GithubRepository): GetGithubReposUseCase {
        return GetGithubReposUseCase(repository)
    }

    @Provides
    fun providesSaveLocalGithubReposUseCase(repository: GithubRepository): SaveLocalGithubReposUseCase {
        return SaveLocalGithubReposUseCase(repository)
    }

    @Provides
    fun providesGetAllLocalGithubReposUseCase(repository: GithubRepository): GetAllLocalGithubReposUseCase {
        return GetAllLocalGithubReposUseCase(repository)
    }

    @Provides
    fun providesDeleteLocalGithubReposUseCase(repository: GithubRepository): DeleteLocalGithubReposUseCase {
        return DeleteLocalGithubReposUseCase(repository)
    }
}
```

앱은 다음과 같이 동작한다.

![cs_architecture_gif1.gif](/assets/images/cs_architecture_gif1.gif?raw=true)

전체 코드는 아래 링크에서 볼 수 있다.
finish1 브런치를 보면된다.

[https://github.com/cellodove/GithubSearch](https://github.com/cellodove/GithubSearch)

## 참조

> [http://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html](http://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
>
> [https://developer.android.com/jetpack/guide?hl=ko](https://developer.android.com/jetpack/guide?hl=ko)
>
> [https://leveloper.tistory.com/205?category=762053](https://leveloper.tistory.com/205?category=762053)
>
> [https://leveloper.tistory.com/206](https://leveloper.tistory.com/206)
>
> [https://docs.gradle.org/current/userguide/organizing_gradle_projects.html#sec:build_sources](https://docs.gradle.org/current/userguide/organizing_gradle_projects.html#sec:build_sources)
>
---
title : "[Android] RecyclerView - Recycler Adapter"
excerpt: "안드로이드 RecyclerView - Recycler Adapter에 대하여"

categories :
  - Android

toc: true
toc_sticky: true
last_modified_at: 2021-12-08 
---


![Ferris wheel.jpg](/assets/images/Ferris_wheel.jpg?raw=true)

## 리사이클러뷰

리사이클러뷰는 안드로이드를 개발하면서 빠질 수 없다. 기본적으로 알아야하고 중요한 뷰이다.

리사이클러뷰전에는 리스트뷰를 많이 사용했었다.

공통점

- 뷰를 통해 여러 목록을 표현해준다.

차이점

- 리스트뷰

화면에서 사라지는 아이템을 삭제하고 화면에서 나타나는 아이템을 생성한다.

생성과 삭제의 횟수가 많아질수록 새로운 뷰를 생성해야하기에 부하가 많이 걸린다.

- 리사이클러뷰

사라지는 아이템을 삭제하지 않고 나타나는 아이템쪽으로 뷰를 이동시켜 재사용한다.

뷰를 새로 생성하지 않아 리스트뷰보다 부하가 적게 걸린다.

![recyclerview.png](/assets/images/recyclerview.png?raw=true)

100개의 아이템이 있다면 리스트뷰는 100개의 아이템을 생성해야 하지만 리사이클러뷰는 10개정도의 아이템을 생성해 재활용할 수 있다.

## 리사이클러뷰 만들기

헤더가 포함된 리사이클러뷰를 RecyclerAdapter를 사용해 만들어보겠다.

헤더가 포함되지 않은 리사이클러뷰는 ListAdapter를 사용하면 만들면 손쉽게 만들수있으며 다음에 글을쓸 예정이다.

- Xml (activity,fragment, recyclerview item, recyclerview header)
- View Activity (여기서는 프레그먼트를 사용)
- ViewModel (데이터를 가져와 뷰에 뿌리기위해 사용)
- Model (표출하고싶은 아이템 데이터)
- Adapter, ViewHolder (뷰와 데이터를 연결해주기 위해 사용)

### Xml

- fragment_recycler_adapter.xml

다음과 같은 화면을 만든다.

`<androidx.recyclerview.widget.RecyclerView>`에서 `tools:listitem=""`를 사용하면 아이템에따라 화면이 어떤식으로 표출될지 미리 알수있다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">
    <LinearLayout
        android:id="@+id/settingBar"
        android:layout_width="match_parent"
        android:layout_height="90dp"
        app:layout_constraintTop_toTopOf="parent"
        android:orientation="horizontal"
        android:elevation="5dp">
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginStart="20dp"
            android:layout_gravity="start|center_vertical"
            android:text="비둘기 대학교 학생"
            android:textColor="#ff8200"
            android:textSize="26sp"/>
        <View
            android:layout_width="0dp"
            android:layout_height="0dp"
            android:layout_weight="1"/>
        <Button
            android:id="@+id/profileDetail"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginEnd="20dp"
            android:padding="5dp"
            android:layout_gravity="end|center_vertical"
            android:gravity="center"
            android:text="프로필 보기"
            android:textSize="15sp"
            android:textColor="#ffffff"/>
    </LinearLayout>
    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recyclerList"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_marginTop="10dp"
        app:layout_constraintTop_toBottomOf="@+id/settingBar"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layoutManager="androidx.recyclerview.widget.LinearLayoutManager"
        tools:listitem="@layout/recycler_adapter_item"/>
</androidx.constraintlayout.widget.ConstraintLayout>
```

![recyclerfragment.PNG](/assets/images/recyclerfragment.png?raw=true)

- recycler_adapter_header.xml

리사이클러뷰 최상단에 표시할 헤더를 만든다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <LinearLayout
        android:id="@+id/headerTitle"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:layout_constraintTop_toTopOf="parent"
        android:paddingTop="5dp"
        android:paddingBottom="5dp"
        android:gravity="center"
        android:orientation="horizontal"
        android:weightSum="4">
        <TextView
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:gravity="center"
            android:text="번호"
            android:textSize="15sp"
            android:textStyle="bold"
            android:textColor="@color/black"/>
        <TextView
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:gravity="center"
            android:text="학과"
            android:textSize="15sp"
            android:textStyle="bold"
            android:textColor="@color/black"/>
        <TextView
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:gravity="center"
            android:text="이름"
            android:textSize="15sp"
            android:textStyle="bold"
            android:textColor="@color/black"/>
        <TextView
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:gravity="center"
            android:text="학번"
            android:textSize="15sp"
            android:textStyle="bold"
            android:textColor="@color/black"/>
    </LinearLayout>
    <View
        android:layout_width="match_parent"
        android:layout_height="1dp"
        app:layout_constraintTop_toBottomOf="@+id/headerTitle"
        android:background="@color/black"/>
</androidx.constraintlayout.widget.ConstraintLayout>
```

![header.PNG](/assets/images/header.png?raw=true)

- recycler_adapter_item.xml

리사이클러뷰 아이템를 만든다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">
    <LinearLayout
        android:id="@+id/itemContainer"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:layout_constraintTop_toTopOf="parent"
        android:paddingTop="5dp"
        android:paddingBottom="5dp"
        android:orientation="horizontal"
        android:weightSum="4">
        <TextView
            android:id="@+id/userNumber"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:gravity="center"
            tools:text="1"
            android:textSize="17sp"
            android:textColor="@color/black"/>
        <TextView
            android:id="@+id/userDepartment"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:gravity="center"
            tools:text="기계공학과"
            android:textSize="17sp"
            android:textColor="@color/black"/>
        <TextView
            android:id="@+id/userName"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:gravity="center"
            tools:text="고길동"
            android:textSize="17sp"
            android:textColor="@color/black"/>
        <TextView
            android:id="@+id/userClassNumber"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:gravity="center"
            android:layout_weight="1"
            android:layout_gravity="center"
            tools:text="20211124"
            android:textSize="17sp"
            android:textColor="@color/black"/>
    </LinearLayout>
    <View
        android:layout_width="match_parent"
        android:layout_height="1dp"
        app:layout_constraintTop_toBottomOf="@+id/itemContainer"
        android:background="#0D000000"/>
</androidx.constraintlayout.widget.ConstraintLayout>
```

![item.PNG](/assets/images/item.png?raw=true)

### View (Fragment)

xml과의 연결은 ViewBinding을 사용했다.

Adapter에 넘길 데이터는 ViewModel에서 LiveData로 받는다.

- RecyclerAdapterFragment.kt

```kotlin
class RecyclerAdapterFragment : Fragment() {
    private lateinit var binding : FragmentRecylerAdapterBinding
    private val viewModel : MainViewModel by activityViewModels()

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View? {
        binding = FragmentRecylerAdapterBinding.inflate(inflater,container,false)
        return binding.root
    }
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        viewModel.profileDataInfo.observe(viewLifecycleOwner){
            var recyclerAdapter = RecyclerAdapter(it)
            binding.gatewayList.adapter = recyclerAdapter
            binding.profileDetail.setOnClickListener {
                viewModel.liveFragmentStep.value = MainViewModel.FragmentStep.PROFILE_DETAIL
            }
            recyclerAdapter.setOnItemClickListener(object : RecyclerAdapter.OnItemClickListener{
                override fun onItemClick(position: Int) {
                }
            })
        }

    }
}
```

RecyclerView는 기본적으로 아이템을 클릭할때 클릭 리스너가 따로없다. 그래서 필요하다면 Adapter에서 따로 추가적으로만들어 주어야한다. 아이템클릭은 많이 사용하브로 Adapter에서 추가 하는 방법을 설명하겠다.

### ViewModel

- MainViewModel.kt

```kotlin
class MainViewModel : ViewModel() {
    enum class FragmentStep { HOME, PROFILE_DETAIL }
    val liveFragmentStep = SingleLiveEvent<FragmentStep>()
    var profileDataInfo= MutableLiveData(arrayListOf<ProfileListInfo>())

    init {
        var profileData = arrayListOf<ProfileListInfo>()
        profileData.add(ProfileListInfo("1","기계공학부","홍길동","20217724"))
        profileData.add(ProfileListInfo("2","컴퓨터공학부","고길동","20217724"))
        profileData.add(ProfileListInfo("3","전기공학부","이철수","20217724"))
        profileData.add(ProfileListInfo("4","전자공학부","고길순","20217724"))
        profileData.add(ProfileListInfo("5","화학공학부","김짱아","20217724"))
        profileData.add(ProfileListInfo("6","컴퓨터공학부","신짱구","20217724"))
        profileData.add(ProfileListInfo("7","전자공학부","이유리","20217724"))
        profileData.add(ProfileListInfo("8","화학공학부","이세아","20217724"))
        profileData.add(ProfileListInfo("9","기계공학부","박진아","20217724"))
        profileDataInfo.value = profileData
    }
}
```

ViewModel이 생성되자마자 gatewayData 라이브데이터에 데이터가 입력되고 옵저버 걸린 RecyclerAdapterFragment에서 리사이클러뷰가 동작한다.

### Model

리사이클러뷰에 원하는 아이템을 표시할려면 아이템에 넣기위한 모델을 만들어야한다.

여기에서 필요한 내용은 다음과 같다.

- 순번
- 학과
- 이름
- 학번

- ProfileListInfo.kt

```kotlin
data class ProfileListInfo(
    var userNumber : String,
    var userDepartment :String,
    var userName : String,
    var userClassNumber : String
)
```

그래서 뷰모델에서 ProfileListInfo로 리스트를 만들어 뷰로 넘기는것을 볼수있다.

### Adapter, ViewHolder

- RecyclerAdapter.kt

```kotlin
class RecyclerAdapter(private var profileListInfo : ArrayList<ProfileListInfo>) : RecyclerView.Adapter<RecyclerView.ViewHolder>() {
    companion object{
        private const val TYPE_HEADER = 0
        private const val TYPE_ITEM = 1
    }
    interface OnItemClickListener{
        fun onItemClick(position: Int)
    }
    private lateinit var onItemClickListener : OnItemClickListener

    fun setOnItemClickListener(listener : OnItemClickListener){
        this.onItemClickListener = listener
    }

    inner class HeaderHolder(binding: RecyclerAdapterHeaderBinding) : RecyclerView.ViewHolder(binding.root)
    inner class ProfileListViewHolder(private val binding: RecyclerAdapterItemBinding) : RecyclerView.ViewHolder(binding.root), View.OnClickListener{
        var onBindPosition = 0

        fun onBind(profileListInfo: ProfileListInfo, position: Int){
            binding.userNumber.text = profileListInfo.userNumber
            binding.userDepartment.text = profileListInfo.userDepartment
            binding.userName.text = profileListInfo.userName
            binding.userClassNumber.text = profileListInfo.userClassNumber
            onBindPosition = position
            binding.userName.setOnClickListener(this)
        }

        override fun onClick(view: View?) {
            if (view?.id == binding.userName.id){
                onItemClickListener.onItemClick(onBindPosition-1)
            }
        }
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): RecyclerView.ViewHolder {
        var holder: RecyclerView.ViewHolder = if (viewType == TYPE_HEADER){
            HeaderHolder(RecyclerAdapterHeaderBinding.inflate(LayoutInflater.from(parent.context),parent,false))
        }else{
            ProfileListViewHolder(RecyclerAdapterItemBinding.inflate(LayoutInflater.from(parent.context),parent,false))
        }
        return holder
    }

    override fun onBindViewHolder(holder: RecyclerView.ViewHolder, position: Int) {
        if (holder is ProfileListViewHolder){
            var gatewayListViewHolder = holder
            gatewayListViewHolder.onBind(profileListInfo[position-1],position)
        }
    }

    override fun getItemViewType(position: Int): Int {
        return if (position == 0){
            TYPE_HEADER
        }else{
            TYPE_ITEM
        }
    }

    override fun getItemCount(): Int {
        return profileListInfo.size + 1
    }
}
```

어댑터를 만들기위해서는 뷰홀더에 대해 알아야한다.

- ViewHolder

RecyclerView 내 위치에 대한 아이템 뷰와 메타데이터를 설명한다.

위에서 설명했지만 리사이클러뷰는 아이템을 개속 생성하는것이아닌 이미 만들어진 아이템을 재활용한다. 아이템은 TextView, ImageView등의 뷰로 구성되어있을텐데 재사용되는 뷰의 구성 요소를 저장하여 바로바로 사용할수 있도록 한것이다.

HeaderHolder

헤더는 따로 데이터를 계속 바꾸는것이아닌 상단에 고정된 텍스트로 표현되기때문에 코드가 따로없다.

```kotlin
inner class HeaderHolder(binding: RecyclerAdapterHeaderBinding) : RecyclerView.ViewHolder(binding.root)
```

ProfileListViewHolder

```kotlin
inner class ProfileListViewHolder(private val binding: RecyclerAdapterItemBinding) : RecyclerView.ViewHolder(binding.root), View.OnClickListener{
        var onBindPosition = 0

        fun onBind(profileListInfo: ProfileListInfo, position: Int){
            binding.userNumber.text = profileListInfo.userNumber
            binding.userDepartment.text = profileListInfo.userDepartment
            binding.userName.text = profileListInfo.userName
            binding.userClassNumber.text = profileListInfo.userClassNumber
            onBindPosition = position
            binding.userName.setOnClickListener(this)
        }

        override fun onClick(view: View?) {
            if (view?.id == binding.userName.id){
                onItemClickListener.onItemClick(onBindPosition-1)
            }
        }
    }
```

`onBind` 에서 각 포지션의 데이터가 들어오는데 그 데이터들을 뷰와 연결시켜주는 일을한다.

그리고 뷰홀더 클래스에 `View.OnClickListener` 를 추가해주었는데 이를 통해

```kotlin
//RecyclerAdapter.kt
override fun onClick(view: View?) {
            if (view?.id == binding.userName.id){
                onItemClickListener.onItemClick(onBindPosition-1)
            }
        }
```

`onClick` 를 가져올수있다. 이 메소드를 사용해 클릭된 아이템의 포지션을 가지고와

```kotlin
//RecyclerAdapter.kt
interface OnItemClickListener{
    fun onItemClick(position: Int)
}
private lateinit var onItemClickListener : OnItemClickListener

fun setOnItemClickListener(listener : OnItemClickListener){
    this.onItemClickListener = listener
}
```

코드를 추가해 View단에서 클릭한 아이템의 포지션을 콜백 받을수있다.

```kotlin
//RecyclerAdapterFragment.kt
recyclerAdapter.setOnItemClickListener(object : RecyclerAdapter.OnItemClickListener{
    override fun onItemClick(position: Int) {
        //필요코드 입력
    }
})
```

자주 사용하니 그냥 외우는것이 좋다.

각각의 아이템들은 포지션을 가지고있는데 헤더는 가장 첫번제에 위치를한다 그렇기때문에 포지션이 0일때는 아이템타입을 Header로 설정하고 나머지는 item으로 설정한다.

```kotlin
override fun getItemViewType(position: Int): Int {
        return if (position == 0){
            TYPE_HEADER
        }else{
            TYPE_ITEM
        }
    }
```

```kotlin
override fun getItemCount(): Int {
        return profileListInfo.size + 1
    }
```

리사이클러뷰 포지션의 1번째는 헤더로 이미 사용을했다. 즉 전체아이템의 갯수는 헤더 + List.size이다. 그래서 카운팅을 할때 +1을 해주어야한다.

```kotlin
override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): RecyclerView.ViewHolder {
        var holder: RecyclerView.ViewHolder = if (viewType == TYPE_HEADER){
            HeaderHolder(RecyclerAdapterHeaderBinding.inflate(LayoutInflater.from(parent.context),parent,false))
        }else{
            ProfileListViewHolder(RecyclerAdapterItemBinding.inflate(LayoutInflater.from(parent.context),parent,false))
        }
        return holder
    }
```

위에서 포지션에따라 홀더 타입을 정했다. 이제 그타입에따라 만들어둔 홀더를 생성한다.

```kotlin
override fun onBindViewHolder(holder: RecyclerView.ViewHolder, position: Int) {
        if (holder is ProfileListViewHolder){
            var gatewayListViewHolder = holder
            gatewayListViewHolder.onBind(profileListInfo[position-1],position)
        }
    }
```

`onBindViewHolder`함수는 생성된 뷰홀더에 데이터를 바인딩 해주는 함수이다.

예를 들어 데이터가 스크롤 되어서 맨 위에있던 뷰 홀더(레이아웃) 객체가 맨 아래로 이동한다면, 그 레이아웃은 재사용 하되 데이터는 새롭게 바뀔 것이다.

아래에서 새롭게 올라오는 데이터가 리스트의 20번째 데이터라면 position으로 20이 들어오는 것이다.

`onCreateViewHolder`는 `ViewHolder`를 만들기 위해 13 ~ 15번 정도밖에 호출되지 않지만, `onBindViewHolder`는 스크롤을 해서 데이터 바인딩이 새롭게 필요할 때 마다 호출된다. 스크롤을 무한정 돌린다면, `onBindViewHolder`도 무한정 호출된다. 무한정 호출된다 하더라도 우리는 딱 13 ~ 15개의 뷰 객체만 사용하는 꼴이다.

여기서는 헤더를 사용하여 포지션이 +1 된 상태이기때문에 정상적인 데이터의 포지션을 가져올려면 -1를 해주어야한다.

실행을 시키면 다음과같은 화면이 나온다.

![result.png](/assets/images/result.png?raw=true)

전체코드는 아래 링크에서 볼수있다.

[https://github.com/cellodove/RecyclerView](https://github.com/cellodove/RecyclerView)

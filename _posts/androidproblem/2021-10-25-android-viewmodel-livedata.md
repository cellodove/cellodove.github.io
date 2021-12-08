---
title : "[안드로이드 Kotlin] MVVM의 ViewModel과 ACC ViewModel에 대하여"
excerpt: "MVVM의 ViewModel과 ACC ViewModel에 대하여"

categories :
  - AndroidProblem

toc: true
toc_sticky: true
last_modified_at: 2021-10-25 
---



나는 Activity에 있는 Fragment전환을 ViewModel에있는 LiveData를 통해 전환을 하려했다. ViewModel 클래스는 Android Architecture Components (AAC)을 상속받아 만들었다.

```kotlin
class MyActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        val viewModel: ViewModel by viewModels()
        viewModel.liveFragmentStep.observe(this@Activity) { step ->
            supportFragmentManager.beginTransaction().run {
                when (step) {
                    ViewModel.FragmentStep.INFO -> replace(R.id.fragment_container_view, GuideFragment(), "info_fragment")
                    ViewModel.FragmentStep.PROGRESS -> replace(R.id.fragment_container_view, ProgressFragment(), "progress_fragment")
                    ViewModel.FragmentStep.DONE -> replace(R.id.fragment_container_view, DoneFragment(), "done_fragment")
                    ViewModel.FragmentStep.GUIDE -> replace(R.id.fragment_container_view, GuideSelfFragment(), "guide_fragment")
                    else -> Unit
                }
                commit()
            }
        }

    }
}
```

```kotlin
class ViewModel : ViewModel() {
    enum class FragmentStep{
     INFO, PROGRESS, DONE, GUIDE
    }
    val liveFragmentStep = MutableLiveData(FragmentStep.INFO)
    
    fun goProgressFragment(){
     liveFragmentStep.value = FragmentStep.PROGRESS
    }
}
```

```kotlin
class GuideFragment: Fragment() {
    override fun onCreate(savedInstanceState: Bundle?) {
        val viewModel: ViewModel by viewModels()
        binding.okButton.setOnClickListener{
            viewModel.goProgressFragment()
            }
    }
}
```

Fragment에서 버튼을 누르면 ViewModel의 goProgressFragment() 함수를 호출해 LiveData값을 바꾼다.  Activity는 LiveData를 observe하고있다가 들어온 값에 맞춰 Fragment를 바꿔준다. 아주 간단한 코드이다.

## 문제

아주 간단한 코드이지만 동작하지 않았다... Fragment에서 ViewModel에 있는 함수를 정상적으로 호출하였으나 Activity에는 데이터가 전달되지 않았다. 확인해보니 ACC의 ViewModels()가 문제였다.

## MVVM의 ViewModel과 ACC의 ViewModel의 차이

정확히는 문제기보다는 내가 잘못사용했다. 우리가 알고있는 MVVM 디자인패턴의 ViewModel과 ACC의 ViewModel은 다르다.

**MVVM에서 ViewModel은 뷰와 모델 사이에서 데이터를 관리하고 바인딩해주는 역할을 한다.**

![viewmodel.png](/assets/images/viewmodel.png)

하지만 **ACC의 ViewModel**의 안드로이드 공식 문서를 보면 화면 회전같은 **환경 변화에서 뷰에 사용되는 데이터를 유지시키기 위한, 라이프사이클을 알고있는 클래스**라고 한다.

![viewmodel-lifecycle.png](/assets/images/viewmodel-lifecycle.png)

예를 들어 한 Activity가 사용자 정보를 가지고 있다고 하자 이때 화면을 회전 시키게되면 Activity는 종료되고 재생성되는데 사용자 정보가 날아가 다시 정보를 불러와야한다. 이때 ACC의 ViewModel을 사용하게 되면 데이터를 그대로 보존시켜준다. 딱 이역할만 해준다. **ViewModel을 생성하고 그안에 데이터를 넣어두었을뿐이다. 이걸로 MVVM패턴이 되지는 않는다.**

하지만 MVVM패턴의 ViewModel을 만들때 ACC의 ViewModel를 사용하면 더 좋다. 뷰와 모델 사이에서 데이터를 관리하고 바인딩해줄뿐 아니라 화면 회전시에 데이터까지 유지 시켜주기때문에.

AAC ViewModel은 Activity 내에서 1개만 생성가능하다. **AAC ViewModel은 Activity안에서의 싱글톤 개념인데 MVVM 패턴에서 뷰와 뷰모델은 1:n 관계를 가지기 때문에 Activity 내의 여러 Fragment를 가질시에 여러 Fragment에 ViewModel을 사용하기 어렵다.**(lifecycle이 달라지면 ViewModel 객체가 달라진다)

## 원인

![viewModelsingleton.png](/assets/images/viewModelsingleton.png)

위 그림과 같이 액티비티에서 생성된 ViewModel과 Fragment에서 생성된 ViewModel이 달랐기에 LiveData의 데이터가 전달이 되지 않았던것이다.

## 해결

![connectViewmodel.png](/assets/images/connectViewmodel.png)

위의 그림과 같이 각각 생성된 ViewModel을 하나의 ViewModel로 생성 되도록 하면 된다.

즉 Fragment의 ViewModel이 Activity의 lifecycle를 따라가게 생성하면 된다.

```kotlin
class MyActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        val viewModel: ViewModel by viewModels()
    }
}
```

by ViewModels()를 이용한 초기화는 해당 ViewModel이 초기화되는 Activity 혹은 Fragment의 Lifecycle에 종속된다.

```kotlin
class GuideFragment: Fragment() {
    override fun onCreate(savedInstanceState: Bundle?) {
        val viewModel: ViewModel by activityViewModels()
    }
}
```

by activityViewModels()는 Fragment에서만 사용 가능한 ViewModel 초기화 방식이다.

Fragment는 Activity에 종속되어 있기 때문에 Fragment가 생성된 Activity의 Lifecycle에 ViewModel을 종속시킨다. 이를 사용해 같은 Activity(lifecycle)를 공유하는 Fragment 간의 데이터 전달이 가능해진다.

## 참조

> [https://wooooooak.github.io/android/2019/05/07/aac_viewmodel/](https://wooooooak.github.io/android/2019/05/07/aac_viewmodel/)
>
> [https://medium.com/kenneth-android/android-mvvm-viewmodel과-aac-viewmodel의-차이-8c0d54922e07](https://medium.com/kenneth-android/android-mvvm-viewmodel%EA%B3%BC-aac-viewmodel%EC%9D%98-%EC%B0%A8%EC%9D%B4-8c0d54922e07)
>
> [https://developer.android.com/topic/libraries/architecture/viewmodel](https://developer.android.com/topic/libraries/architecture/viewmodel)
>
> [https://zion830.tistory.com/72](https://zion830.tistory.com/72)
>
> [https://kotlinworld.com/89](https://kotlinworld.com/89)
>
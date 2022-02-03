---
title : "[안드로이드 Kotlin]  권한요청 두번이상 거절시 요청팝업 안뜨는 문제"
excerpt: "권한요청 두번이상 거절시 요청팝업 안뜨는 문제"

categories :
  - AndroidProblem

toc: true
toc_sticky: true
last_modified_at: 2022-02-03
---

## 문제

대부분의 앱은 처음실행할때 필요한 권한들을 요청한다.

문제는 권한요청을 거절할때 발생한다.

두번 이상 권한 요청을 거절할시 더이상 권한 허용 시스템 팝업이 뜨지 않는 문제가 발생했다.

![permission_example1.gif](/assets/images/permission_example1.gif?raw=true)

## 원인

확인 결과 안드로이드 정책에 의해 발생한것을 알게되었다.

안드로이드 [developer](https://developer.android.com/about/versions/11/privacy/permissions?hl=ko)에 들어가 보면

“Android 11부터 사용자가 앱이 기기에 설치된 전체 기간 동안 특정 권한에 관해 **거부**를 두 번 이상 탭하면 앱에서 그 권한을 다시 요청하는 경우 사용자에게 시스템 권한 대화상자가 표시되지 않습니다. 이러한 사용자의 작업은 '다시 묻지 않음'을 의미합니다.”

“이전 버전에서는 사용자가 이전에 '다시 묻지 않음' 체크박스 또는 옵션을 선택하지 않는 한 앱에서 권한을 요청할 때마다 사용자에게 시스템 권한 대화상자가 표시되었습니다. Android 11에서 이 동작 변경은 사용자가 거부를 선택한 권한을 반복적으로 요청하지 못하게 합니다.”

이라고 나와있다.

## 해결

사용자가 2번 거절한 이후 다시 권한 허용을 하기를 원하는경우 사용자가 직접 앱 설정에 들어가 권한 허용을 눌러야 한다.

사용자에게 해줄수있는 최선은 직접들어가서 권한 허용을 해줘야한다는 팝업을 띄우고 팝업 버튼을 누르면 앱 설정화면으로 바로갈 수 있도록 해주는것이다.

직접 권한 허용을 하기위해서는 사용자가 해당 권한을 거절했는지를 확인해야 한다.

다행히도 사용자가 명시적으로 권한을 거부했는지를 알수있는 메소드가 있다.

바로 `shouldShowRequestPermissionRationale` 이다.

이를 사용해 권한 문제를 해결했다.

아래 코드는 해당 문제를 해결한 예제 코드이다.

모든 권한을 허용했는지 확인한후 허용되지 않았을경우 사용자가 명시적으로 거절하지 않았으면 권한허용 시스템 팝업을 띄우고 명시적으로 두번 거절했으면 앱 설정으로 갈수있도록 팝업창을 띄우 주도록 만들었다.

- MainActivity.kt

```kotlin
class MainActivity : AppCompatActivity() {
    private val binding by lazy { ActivityMainBinding.inflate(layoutInflater) }
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(binding.root)
        if (checkPermission()) {
            Toast.makeText(this,"권한 허용 완료",Toast.LENGTH_SHORT).show()
        }else{
            supportFragmentManager.beginTransaction()
                .setCustomAnimations(R.anim.slide_up, R.anim.slide_down)
                .add(R.id.container, IntroPermissionFragment())
                .commit()
        }
    }

    private fun checkPermission(): Boolean {
        return Constants.PERMISSIONS.none { checkSelfPermission(it) != PackageManager.PERMISSION_GRANTED }
    }
}
```

- IntroPermissionFragment.kt

```kotlin
class IntroPermissionFragment : Fragment() {
    private lateinit var binding : FragmentIntroPermissionBinding

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        binding = FragmentIntroPermissionBinding.inflate(inflater,container,false)
        return binding.root
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        binding.btnPermission.setOnClickListener {
            permissionLauncher.launch(Constants.PERMISSIONS)
        }
    }

    private val permissionLauncher = registerForActivityResult(ActivityResultContracts.RequestMultiplePermissions()) { result ->
        Log.e("permissionLauncher","permissionLauncher: $result")
        val deniedList: List<String> = result.filter {
            !it.value
        }.map {
            it.key
        }

        when {
            deniedList.isNotEmpty() -> {
                val map = deniedList.groupBy { permission ->
                    if (shouldShowRequestPermissionRationale(permission)) "DENIED" else "EXPLAINED"
                }
                map["DENIED"]?.let {
                    val intent = Intent(requireContext(), MainActivity::class.java)
                    intent.flags = Intent.FLAG_ACTIVITY_CLEAR_TOP or Intent.FLAG_ACTIVITY_SINGLE_TOP
                    startActivity(intent)
                    requireActivity().finishAffinity()
                }
                map["EXPLAINED"]?.let {
                    // 거부 두 번 했을경우 설정
                    val dialog = AlertDialog.Builder(requireContext())
                    dialog.apply {
                        setMessage("권한 설정을 하시겠습니까?")
                        setPositiveButton("확인"
                        ) { _, _ ->
                            val intent = Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS).setData(
                                Uri.parse("package:${requireContext().packageName}"))
                            startActivity(intent)
                        }
                        setNegativeButton("취소"){
                                _,_ ->
                            UInt
                        }
                    }
                    dialog.create().show()
                }
            }
            else -> {
                // All request are permitted
            }
        }
    }
}
```

버튼을 누루면 Intent를 사용해 앱의 설정화면으로 바로 갈 수 있다.

```kotlin
val intent = Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS).setData(Uri.parse("package:$packageName"))
startActivity(intent)
```

- Constants.kt

원하는 권한들을 정의해두고 한번에 호출한다.

```kotlin
object Constants {
    val PERMISSIONS = arrayOf(
        Manifest.permission.ACCESS_FINE_LOCATION,
        Manifest.permission.READ_EXTERNAL_STORAGE,
        Manifest.permission.WRITE_EXTERNAL_STORAGE,
        Manifest.permission.CAMERA,
        Manifest.permission.READ_PHONE_STATE
    )
}
```

이제 사용자가 권한 허용을 두번 거절하면 다음과같이 나온다.

![permission_example2.gif](/assets/images/permission_example2.gif?raw=true)

전체 코드는 아래 주소에서 확인할 수 있다.

> [https://github.com/cellodove/Permission-Example](https://github.com/cellodove/Permission-Example)
>

## 참조

> [https://developer.android.com/about/versions/11/privacy/permissions?hl=ko](https://developer.android.com/about/versions/11/privacy/permissions?hl=ko)
>
> [https://stackoverflow.com/questions/67825724/how-to-ask-again-for-permission-if-it-was-denied-in-android](https://stackoverflow.com/questions/67825724/how-to-ask-again-for-permission-if-it-was-denied-in-android)
>
> [https://debbi.tistory.com/31](https://debbi.tistory.com/31)
>
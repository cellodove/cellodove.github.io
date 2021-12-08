---
title : "[안드로이드 Kotlin] FragmentTransaction commit 오류"
excerpt: "FragmentTransaction commit 오류"

categories :
  - AndroidProblem

toc: true
toc_sticky: true
last_modified_at: 2021-11-09 
---


얼마전 이런 오류가 나왔다.

```jsx
E/UncaughtException: java.lang.IllegalStateException:
Can not perform this action after onSaveInstanceState
```

어디서 생기는지 확인해보니 Fragment가 전환할때 간헐적으로 저 오류가 생겼다.

나는 하나의 Activity에 여러 Fragment를 두었고 ViewModel쪽에 Fragment Step LiveData를 사용해 FragmentTransaction이 동작할수 있도록 했다.

```kotlin
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
```

위 코드에는 크게 문제가 없어 보인다. 그러나 오류를 보면 fragmentTransaction.commit에서 문제가 발생한다.

지금부터 왜 문제가 발생하는지, 그리고 해결방법을 알아보자.

## Fragment 전환 IllegalStateException 원인

Activity의 onSaveInstanceState()가 호출된 후에 FragmentTransaction의 commit()이 동작하여 생긴다.

안드로이드 시스템은 메모리를 비우기 위해 어느 시점에서든 프로세스(액티비티, 서비스 등)를 종료할 권한을 가지고 있다. 이때 onSaveInstanceState() 콜백 메소드는 Activity에게 Activity의 상태를 저장할 수 있도록 해준다.

![activity_lifecycle.png](/assets/images/activity_lifecycle.png?raw=true)

onSaveInstanceState() 메소드는 Activity running 상태에서 onPause(), onStop(), onDestroy()메소드 들이 호출되는 상황에서 호출된다.

그렇다면 onSaveInstanceState() 메소드와 FragmentTransaction이 IllegalStateException을 발생시키는 이유는 무엇을까?

onSaveInstanceState() 메소드는 View 등 Activity에 대한 정보를 Bundle 형태로 시스템 프로세스에 저장하고있다가 Activity를 다시 생성하기로 결정했을 때에 복구하는 용으로 사용할 수 있도록 해준다.

 다시말해 Bundle은 onSaveInstanceState()가 호출된 시점의 사진과 같다. 그래서 onSaveInstanceState()이 호출된 이후에 Fragment 전환이 발생하면 onSaveInstanceState()을 통해 복구된 시점과 다르게 해당 Fragment 전환은 복구 할 수 없게된다. 이런 상황이 되면 UX를 해치게 되고. 안드로이드는 이를 방지하고자  IllegalStateException을 던집니다.

## 해결

Activity Lifecycle 메소드 내에서 Transaction을 커밋할 때 항상 주의해야한다.

IllegalStateException을 예방하기 위한 방법은 두가지가 있다.

1. 비동기 콜백 메소드(Asynchronous Callback Method) 안에서 Transaction commit을 하지 않아야 한다. 되도록이면 동기(Synchronous)로, 거기에다가 액티비티의 상태를 저장하는 onSaveInstanceState()가 호출되기 전이 보장되는 곳에서 commit을 수행한다.
2. Transaction의 commitAllowingStateLoss() 메소드를 사용한다. Activity State Loss 현상에 대해 무시하겠다는 것으로, 복구 시점에서 상태 손실이 발생하더라도 괜찮을 때 사용하면 된다.

현재 코드는 상황에따라 Fragment가 변경되다가 마지막에 서버와 통신을 하게되는데 이때 실패를 하게되면 Dialog가 나오고 Dialog를 클릭하면 처음 화면으로 돌아가게된다.

이때 Dialog가 생성되면 Activity는 onPause()상태가 되고 onSaveInstanceState()가 호출된다. 그뒤 Dialog를 클릭하면 ViewModel에있는 LiveData값을 변경하고 그 값이 전달되어 Transaction이 일어난다.

```kotlin
manager.eventOutput.observe(this@Activity){ output ->
            if (output != null) {
                viewModel.requestResult(output) { result ->
                    if (result != null) {
                        viewModel.liveResult.value = result
                        viewModel.liveFragmentStep.value = ViewModel.FragmentStep.DONE
                    } else {
                        CommonDialog.getInstance(
                            isImgVisible = true,
                            title = getString(R.string.failed_send_data),
                            msg = getString(R.string.fail_upload_data),
                            rightBtnText = getString(R.string.confirm)
                        ).show(supportFragmentManager, "dialog_common_dialog")
                        viewModel.liveFragmentStep.value = ViewModel.FragmentStep.INFO
                    }
                }
            }

        }
```

onSaveInstanceState() 이후에 Transaction이 일어났기에 IllegalStateException이 생긴것이였다.

이문제를 해결하는 방법은 Dialog가 생성되고 버튼을 누르면 Activity Finish()를 하여 액티비티를 종료후 다시 호출하던지 Transaction에서 commitAllowingStateLoss() 메소드를 사용하면된다.

```kotlin
manager.eventOutput.observe(this@Activity){ output ->
            if (output != null) {
                viewModel.requestResult(output) { result ->
                    if (result != null) {
                        viewModel.liveResult.value = result
                        viewModel.liveFragmentStep.value = ViewModel.FragmentStep.DONE
                    } else {
                        CommonDialog.getInstance(
                            isImgVisible = true,
                            title = getString(R.string.failed_send_data),
                            msg = getString(R.string.fail_upload_data),
                            rightBtnText = getString(R.string.confirm)
                        ){
                            finish()
                        }.show(supportFragmentManager, "dialog_common_dialog")
                    }
                }
            }

        }

viewModel.liveFragmentStep.observe(this@Activity) { step ->
            supportFragmentManager.beginTransaction().run {
                when (step) {
                    ViewModel.FragmentStep.INFO -> replace(R.id.fragment_container_view, GuideFragment(), "info_fragment")
                    ViewModel.FragmentStep.PROGRESS -> replace(R.id.fragment_container_view, ProgressFragment(), "progress_fragment")
                    ViewModel.FragmentStep.DONE -> replace(R.id.fragment_container_view, DoneFragment(), "done_fragment")
                    ViewModel.FragmentStep.GUIDE -> replace(R.id.fragment_container_view, GuideSelfFragment(), "guide_fragment")
                    else -> Unit
                }
                commitAllowingStateLoss()
            }
        }
```

나는 간단하게 수정할 수 있고 어차피 처음 상태로 돌아가야 했기 때문에 commitAllowingStateLoss() 메소드를 사용해 문제를 해결했다.

## 참조

> [https://developer.android.com/guide/components/activities/activity-lifecycle?hl=ko](https://developer.android.com/guide/components/activities/activity-lifecycle?hl=ko)
>
> [https://developer.android.com/reference/android/app/FragmentTransaction](https://developer.android.com/reference/android/app/FragmentTransaction)
>
> [https://readystory.tistory.com/113?category=861094](https://readystory.tistory.com/113?category=861094)
>
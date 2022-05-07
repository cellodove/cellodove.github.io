---
title : "[안드로이드 Kotlin] BLE가 계속 연결되어있는 문제2"
excerpt: "BLE가 계속 연결되어있는 문제"

categories :
  - AndroidProblem

toc: true
toc_sticky: true
last_modified_at: 2022-05-06
---

저번에 비해 많이 좋아졌지만 아직 간헐적으로 블루투스가 연결되어있다고 표시되는 경우가 있었다. 이번에 재밌는 내용을 몇개 찾아내어 이 내용과 함께 글을쓴다.

## 블루투스가 꺼져있음에도 연결

블루투스가 계속 연결되어있는 문제는 간헐적으로나타나 이런저런 테스트를 하는중, 연결을 끊기위해 안드로이드 시스템 설정에서 블루투스를 껏다 그런데도 블루투스가 연결되어 기기로부터 데이터를 받고있는것을 로그로 확인했다. 내 눈을 의심했다 블루투스 버튼은 비활성화 되어있는데 데이터를 받고있으니.

처음에는 운영 체제 버그인줄 알았으나 확인을 해보니 특정 설정을 활성화화면 시스템에서 블루투스를 끄더라도 꺼지지 않을 수 있다는 [내용](https://github.com/AltBeacon/android-beacon-library/issues/754)을 찾아냈다.

위치 서비스에서 정확도 향상을 위해 블루투스 찾기라는 기능이 있었다. 지도 앱이나 네비게이션을 사용할때 선택해서 동작중이였던것같다.

![ble_problem2_image1.jpeg](/assets/images/ble_problem2_image1.jpeg?raw=true)

이 기능이 활성화 되어있으면 블루투스가 꺼져있어도 주변 기기를 검색하고 연결이 가능하다.

## BluetoothGattCallback의 불확실성

블루투스를 연결할때 콜백도 같이 연결해야한다.

```kotlin
fun connectDevice(context: Context, device: BluetoothDevice) {
    bluetoothGatt = device.connectGatt(context, false, bluetoothGattCallback, BluetoothDevice.TRANSPORT_LE)
}
```

```kotlin
private val bluetoothGattCallback = object : BluetoothGattCallback() {
    override fun onConnectionStateChange(gatt: BluetoothGatt?, status: Int, newState: Int) {
        super.onConnectionStateChange(gatt, status, newState)
        when (newState) {
            BluetoothProfile.STATE_CONNECTED -> {

            }
            BluetoothProfile.STATE_DISCONNECTED -> {

            }
            else -> Unit
        }

    }
}
```

문제는`bluetoothGatt.disconnect()` 할때 `onConnectionStateChange` 콜백이 올 수도있도 안올 수 도있다고 한다... [이슈트레커](https://issuetracker.google.com/issues/37057260)

해제 콜백을 보장하기않기에 콜백이 오지않을경우를 대비해 타이머가 필요할 수도있다고한다.

안드로이드 Bluetooth LE API쪽이 여러 방면으로 재미있는것같다...

## BluetoothGatt의 비동기 메소드

[전에](https://cellodove.github.io/androidproblem/android-ble-re-connect-error/) 이야기했듯이 블루투스와 연결을 끊기 위해서는 disconnect(), close() 두개의 메소드를 사용해야한다.

```kotlin
fun close(){
    bluetoothGatt?.disconnect()
    bluetoothGatt?.close()
    bluetoothGatt = null
    writeCharacteristic = null
}
```

저 두개 메소드 이외에 `bluetoothGatt = null` 를 해주었는데 이 부분이 문제가 되었다.

저 코드가 순차적으로 실행될것이라 생각했지만 아니였다.`disconnect()`, `close()`가 비동기적으로 동작했고 그로인해 순서를 보장하지 않았다. 그래서 `disconnect()`가 동작중에 `bluetoothGatt = null` 로 객체를 초기화시켜버리면 `bluetoothGatt` 의 데이터가 사라져 버리고 해제할 기기 정보를 알 수 없어 연결을 해제할 수 없게 되어버린다.

콜백의 여부와 실제 리소스 해제는 다르게 보아야할것같다. 콜벡으로만 보았을때는 동기적이다.

그래서 앱 코드상으론 `bluetoothGatt`객체가 널이되어 연결이 끊어진것으로 인식하나 시스템상으로는 끊기지 않은것이였다. 그리고 `close()` 에서 알아서 데이터를 초기화 해주기에 따로 널을 사용해 초기화를 시킬 필요가 없는것으로 보인다.

위의 두가지 이유로 `bluetoothGatt = null` 를 사용하지 않기로했다.

추가적으로 `synchronized` 를 사용해 연결을 관리하는 [방법](https://stackoverflow.com/questions/34871364/android-bluetoothgatt-becomes-null-on-activity-change)도 있는것으로 보이나 아직 테스트를 해보지 않았다.

## 참조

> [https://stackoverflow.com/questions/34871364/android-bluetoothgatt-becomes-null-on-activity-change](https://stackoverflow.com/questions/34871364/android-bluetoothgatt-becomes-null-on-activity-change)
>
> [https://issuetracker.google.com/issues/37057260](https://issuetracker.google.com/issues/37057260)
>
> [https://github.com/AltBeacon/android-beacon-library/issues/754](https://github.com/AltBeacon/android-beacon-library/issues/754)
>
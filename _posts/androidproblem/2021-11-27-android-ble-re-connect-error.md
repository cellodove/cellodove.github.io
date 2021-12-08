---
title : "[안드로이드 Kotlin]  BLE 재연결 간헐적 오류"
excerpt: "BLE 재연결 간헐적 오류"

categories :
  - AndroidProblem

toc: true
toc_sticky: true
last_modified_at: 2021-11-27 
---

## 문제

BLE 기기와 통신을하다 가끔식 재연결할때 검색이안되 연결을 못하는 경우가 있었다.

이유를 잘몰랐으나 몇가지 특징들이 있었다.

1. 처음 연결이 아닌 항상 똑같은 기기를 재연결할때 발생
2. 연결을 끊고 재연결을 했으나 가끔 Connect이 아닌Disconnect오는것

이 두가지를 통해 기기연결뒤 해제할때 문제가 있지 않을까 생각이 들었다.

블루투스 통신과정은 다음과 같다.

![bluetoothDiagram.png](/assets/images/bluetoothDiagram.png?raw=true)

블루투스가 종료될때의 상황은 두가지가있다.

1. 화면이 종료될때 or 앱이 종료할때
2. 기기와의 거리가 멀어저 연결이 끊어질때

첫번째의 경우 화면또는 앱이 종료되면서 BluetoothService를 종료하고 BluetoothService가 종료되면서 DeviceController의 `bluetoothGatt.close()`를 호출하여 연결을 끝낸다.

```kotlin
//in Fragment()
override fun onDestroy() {
    super.onDestroy()
    bluetoothService.stopScan()
    bluetoothService.disConnect()
}

//in BluetoothService
fun disConnect(){
    deviceControl.close()
    deviceControl = null
}

//in DeviceControl
fun close(){
    bluetoothGatt?.close()
    bluetoothGatt = nullwriteCharacteristic = null
}
```

두번째의 경우 디바이스와 연결이 끊겨지면 DeviceController에서 BluetoothGattCallback()가 호출되고 상태를 확인한후 연결을 끝낸다.

```kotlin
//in DeviceControl BluetoothGattCallback()
private val gattCallback = object: BluetoothGattCallback() {
    override fun onConnectionStateChange(gatt: BluetoothGatt?, status: Int, newState: Int) {
        super.onConnectionStateChange(gatt, status, newState)
        bluetoothGatt = gatt
        when (newState) {
            BluetoothProfile.STATE_CONNECTED -> {
                Log.e("BluetoothConnect","STATE_CONNECTED")
                connectionState = ConnectionMode.STATE_CONNECTED
                ioScope.launch {
                    delay(5000)
                    bluetoothGatt?.discoverServices()
                    }
                }
            
            BluetoothProfile.STATE_DISCONNECTED -> {
                Log.e("BluetoothConnect","STATE_DISCONNECTED")
                connectionState = ConnectionMode.STATE_DISCONNECTED
                disconnected()
            }
        }
    }
}

//in DeviceControl disconnected()
fun disconnected(){
    bluetoothGatt?.close()
    bluetoothGatt = null
    writeCharacteristic = null
}

```

## 원인

위의 코드를보면 알겠지만 close()와 disconnected()둘다 연결을 끊을때 bluetoothGatt.close()만 호출하는것을 알수있다. 이 하나만 호출하여 문제가 생겼다.

`BluetoothGatt`의 `close()` 메소드 문서를 보면 ***"Bluetooth GATT Client를 닫습니다. 응용 프로그램은 이 GATT Client를 사용한 후 가능한 빨리 이 방법을 호출해야 합니다."*** 라고 써져있다.

**close()메소드는 Bluetooth GATT Client를 닫을뿐 연결을 해제하지는 않았던 것이다..!**

그래서 앱이 종료할때는 bluetoothGatt.close()하여 Bluetooth GATT Client를 닫았지만 블루투스 기기는 그대로 연결이 되어있는 상태였고 나중에 다시 연결을 하기위해 기기 검색을해도 해당기기가 검색이되지않아 다시 연결을 할수 없었던 것이다.

하지만 기기가 멀어져서 연결이 끊어지는 경우는 앱에서*`STATE_DISCONNECTED`* 가 호출되어 bluetoothGatt.close()메소드가 Bluetooth GATT Client를 닫았을뿐만아니라 실제로 거리가 멀어져 연결이 끊어졌기때문에 블루투스 기기도 연결이끊어진 상태로 돌아가 검색이 가능해져 재연결이 되었던것으로 생각이된다.

## 해결

`BluetoothGatt`의 close()만으로는 완전히 해제가 잘안되었기때문에 `disconnect()`메소드를 추가했다. `BluetoothGatt`의 `disconnect()` 메소드 문서를 보면 ***"설정된 연결을 끊거나 현재 진행 중인 연결 시도를 취소합니다."*** 라고 명확하게 연결을 끊는다고 나와있다.

```kotlin
//in DeviceControl
fun close(){
    bluetoothGatt?.disconnect()
    bluetoothGatt?.close()
    bluetoothGatt = null
    writeCharacteristic = null
}

fun disconnected(){
    bluetoothGatt?.disconnect()
    bluetoothGatt?.close()
    bluetoothGatt = null
    writeCharacteristic = null
}
```

그래서 위의 코드와 같이 먼저 블루투스 연결을 끊어주고 Bluetooth GATT Client를 닫기로 하였다. 이렇게하니 연결이 끊긴뒤 재연결을 해도 잘 연결이 되는것을 확인하였다.

또한 `bluetoothGatt.disconnect()` 만 호출하고 `bluetoothGatt.close()` 를 안하는 경우도 있을수있는데 이또한 연결에 실패한다. `BluetoothGattCallback()` 에서 status = 133이 리턴되는데 이는 GATT_ERROR로 에러 메시지다. 즉 지금 블루투스 device의 연결이 제대로 종료되지 못했다는 의미로 `disconnect()` 를 통해 블루투스 연결은 해지했지만 Bluetooth GATT Client는 아직 연결되어 있기때문에 재연결을 허용하지 않는것이다.

연결과 달리 해제할때는 좀더 많은 과정이 필요하다는것을 알 수 있다. 안전하게 기기와 해제하기위해 위의 함수같이 블루투스 연결을 해제하고, Bluetooth GATT Client를 닫아주고, null로 초기화 해주면 되겠다.

## 참조

> [https://stackoverflow.com/questions/23110295/difference-between-close-and-disconnect](https://stackoverflow.com/questions/23110295/difference-between-close-and-disconnect)
>
> [https://qmffor09.tistory.com/11](https://qmffor09.tistory.com/11)
>
> [https://www.it-gundan.dev/ko/android/close-와-disconnect-의-차이점은-무엇입니까/1046241175/amp/](https://www.it-gundan.dev/ko/android/close-%EC%99%80-disconnect-%EC%9D%98-%EC%B0%A8%EC%9D%B4%EC%A0%90%EC%9D%80-%EB%AC%B4%EC%97%87%EC%9E%85%EB%8B%88%EA%B9%8C/1046241175/amp/)
>
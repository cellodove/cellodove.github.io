---
title : "[안드로이드 Kotlin] BLE가 계속 연결되어있는 문제"
excerpt: "BLE가 계속 연결되어있는 문제"

categories :
  - AndroidProblem

toc: true
toc_sticky: true
last_modified_at: 2022-01-07
---

- **SoftDevice stack architecture**

![SoftDevice stack architecture.svg](/assets/images/SoftDevice_stack_architecture.svg?raw=true)

## 문제

App에서는 BLE기기가 끊긴상태인데 BLE기기는 APP과 연결되어있는 상태가 간헐적으로 발생했다.

App은 BLE기기에서 알림을 보내도 기기와 끊어진상태로 알고있어 데이터 처리를 할수가없다. 그리고 앱에서 기기와 연결을하기위해 스캔을해도 기기는 앱과 연결된 상태이기 때문에 스캔할때 검색이 되지않았다.

![ble_error.png](/assets/images/ble_error.png?raw=true)

도저히 어찌할 수 없는 상태였다. 한가지 응급처치는 앱의 블루투스를 꺼 강제로 연결을 끊거나 앱을 강제종료한뒤 재실행하면 다시 정상적으로 연결이 되었다.

이러한 증상과 해결방법을 통해 블루투스 객체가 여러개 생성되는게 아닌가 의심이 들었다. 그런데 이미 기기와 연결하고 해지할때 반드시 블루투스 연결을 종료하고 객체를 널로 만들어 해당 문제를 방지했다고 생각했다.

```kotlin
fun close() {
    bluetoothGatt?.disconnect()
    bluetoothGatt?.close()
    bluetoothGatt = null
    writeCharacteristic = null
}
```

하지만 증상은 계속되었다.

## 원인

계속해서 원인을 찾는중 몇개의 스텍오버플로우에서 실마리를 찾았다. **동일한 장치에 여러 번 연결을 할 수 있다는것이다.**

```kotlin
fun connectDevice(context: Context, device: BluetoothDevice) {
    bluetoothGatt = device.connectGatt(context, false, bluetoothGattCallback)
}
```

연결하는 부분도 블루투스 객체가 비어있지 않으면 비운다음에 연결하도록했지만 그렇게해도 기기 연결을 여러번하면 여러번 연결이 되어,결과적으로 `BluetoothGatt` 인스턴스가 여러개 생성되었던것이다.

![ble_error2.png](/assets/images/ble_error2.png?raw=true)

안드로이드 블루투스 라이브러리를 확인해보니 연결할때 객체가 생성되는것을 확인할 수 있다.

```java
//BluetoothDevice.java
package android.bluetooth;
@UnsupportedAppUsage
public BluetoothGatt connectGatt(Context context, boolean autoConnect,
    BluetoothGattCallback callback, int transport,boolean opportunistic, int phy, Handler handler) {
        if (callback == null) {
            throw new NullPointerException("callback is null");
        }
        // TODO(Bluetooth) check whether platform support BLE
        // Do the check here or in GattServer?
        BluetoothAdapter adapter = BluetoothAdapter.getDefaultAdapter();
        IBluetoothManager managerService = adapter.getBluetoothManager();
        try {
            IBluetoothGatt iGatt = managerService.getBluetoothGatt();
            if (iGatt == null) {
                // BLE is not supported
                return null;
            }
            BluetoothGatt gatt = new BluetoothGatt(iGatt, this, transport, opportunistic, phy);
            gatt.connect(autoConnect, callback, handler);
            return gatt;
            } catch (RemoteException e) {
                Log.e(TAG, "", e);
            }
    return null;
}
```

그래서 하나를 종료 하여도 연결된 나머지 인스턴스로인해 블루투스 기기는 연결된 상태로 인식된게 아닐까 생각이된다.

BLE 장치는 한번에 하나의 중앙장치에만 연결할 수 있다고 알고있는데 같은 중앙장치에는 여러번 연결이 가능한건지, 블루투스 기기에서 동시에 연결을 될수있도록 만들어져서 그런건지는 잘모르겠다.

## 해결

쉽게 생각해 블루투스가 중복으로 연결되는게 문제이다. 그럼 해결방법은 중복으로 연결되지 않게 하면된다. 3개정도를 생각해 보았다.

1. **장치에서 하나의 BluetoothGatt만 연결**
2. **APP에서 하나의 BluetoothGatt만 가짐**
3. **APP에서 여러개의 BluetoothGatt 를 가지고있다면 연결 해제할때 모든 BluetoothGatt를 제거**

1번은 나의 영역이 아니기에 2번과 3번을 적용해보기로했다.

- APP에서 하나의 BluetoothGatt만 가짐

하나의 인스터스를 가져야하기때문에 `BluetoothGatt`가 널이 아니면 연결하지 않도록 하였다.

```kotlin
fun connectDevice(context: Context, device: BluetoothDevice) {
    if (bluetoothGatt==null){
        device.connectGatt(context, false, bluetoothGattCallback)
    }else{
        Log.e("connectDevice","already connectDevice")
    }
}
```

- APP에서 여러개의 BluetoothGatt 를 가지고있다면 연결 해제할때 모든 BluetoothGatt를 제거

`BluetoothGatt` 를 담을수있는 리스트를 만든뒤에 연결이 되어 새로운 인스턴스가 생성되면 리스트에 추가한다. 그리고 해제할때 리스트에 담긴 인스턴스를 하나씩꺼내 전부 해제한다.

```kotlin
private val gattInstances = LinkedList<BluetoothGatt>()
private var bluetoothGatt: BluetoothGatt? = null

private val bluetoothGattCallback = object : BluetoothGattCallback() {
    override fun onConnectionStateChange(gatt: BluetoothGatt?, status: Int, newState: Int) {
        super.onConnectionStateChange(gatt, status, newState)
        //연결이되면 리스트에 담는다.
        bluetoothGatt = gatt
        bluetoothGatt?.let { gattInstances.add(it)}
        when (newState) {
        BluetoothProfile.STATE_CONNECTED -> gatt?.requestMtu(243)
        BluetoothProfile.STATE_DISCONNECTED -> disconnected()
        else -> Unit
        }
    }
    ...
    ...
}
```

```kotlin
// 연결 해제할때
fun close() {
    bluetoothGatt?.disconnect()
    bluetoothGatt?.close()
    bluetoothGatt = null
    writeCharacteristic = null
    while (gattInstances.isNotEmpty()){
        gattInstances.pop().apply {
        disconnect()
        close()
        }
    }
}
```

## 추가

디버그 모드로 코드를 돌렸을때 기기검색때 무지성으로 `connectDevice` 메소드가 호출되는것을 확인했다.(디버그 모드로인해 `bluetoothAdapter?.*bluetoothLeScanner*?.stopScan(this)` 가 늦게 동작해서 그런듯하다.)

확인을 해보니 APP내 코드는 여러부분에서 블루투스 기기가 연결되어있는지 확인한후 연결이 되어있지 않으면 연결한다. 문제는 `connectDevice` 를 호출하고난뒤 기기로 부터 응답을 받는데 시간이 걸리는데 그사이에 연결이되어있는지 체크를하게되면 `connectDevice` 를 한번더 호출할수도 있다는것을 확인했다. 이러한 문제로인해 위에서 만든 널체크도 넘어가게된다...

비동기적으로 동작하면 위의 증상이 더 잘 발현될것같다는 생각이들었다. 블루투스 연결부분이 코루틴으로 비동기적으로 연결하도록 되어있는데 코루틴으로 동작하던 부분을 모두 걷어냈다. 그래서 총 3가지를 수정했다. 이 후 재연결이 안되는 증상이 나타나지는 않았지만 좀더 테스트를 해보아야 할것같다.

## 추가+

블루투스 기기에서 `BluetoothGatt` 가 여러개 생성되면 알아서 `close` 하는것을 확인했다.  그렇다면 왜...?

## 추가++

### 문제

좀더 확인을 해보니 블루투스가 연결이 안되는 현상에서 원인이 하나더 있었다. 이경우는 위의 경우와 상황이 다른데 앱에서는 연결이 안되고 다른 폰으로 블루투스 검색을하면 검색이 되는 상황이다.

![ble_error3.png](/assets/images/ble_error3.png?raw=true)

### 원인

블루투스 서비스가 종료되면서 블루투스 모듈이 종료되어버리는 상황인데 몇몇상태에서 서비스를 종료시키는 `stopSelf()` 를 호출했다. 원래는 `stopSelf()` 를 호출하여 서비스를 종료시키고 새로 서비스를 시작해야하지만 그러지 않는경우가 있었고 그때 오류가 발생했다.

### 해결

`stopSelf()` 를 호출해야하는 부분을 보니 꼭 필요가 없다는것을 확인했다. (서비스는 항상 하나만 존재하고 포그라운드 서비스로 지속적으로 존재한다.) 그래서 `stopSelf()` 호출을 모두 제거했다. 그 이후 몇일간 테스트를 하고있는데 아직까지 동일한 증상이 발생하지 않았다.

## 참조

> [https://developer.android.com/guide/topics/connectivity/bluetooth-le?hl=ko](https://developer.android.com/guide/topics/connectivity/bluetooth-le?hl=ko)
>
> [https://infocenter.nordicsemi.com/index.jsp?topic=%2Fsds_s140%2FSDS%2Fs1xx%2Fble_protocol_stack%2Fble_protocol_stack.html](https://infocenter.nordicsemi.com/index.jsp?topic=%2Fsds_s140%2FSDS%2Fs1xx%2Fble_protocol_stack%2Fble_protocol_stack.html)
>
> [https://learn.adafruit.com/introduction-to-bluetooth-low-energy/gatt](https://learn.adafruit.com/introduction-to-bluetooth-low-energy/gatt)
>
> [https://enidanny.github.io/ble/ble-att-gatt/](https://enidanny.github.io/ble/ble-att-gatt/)
>
> [https://stackoverflow.com/questions/35986549/totally-disconnect-a-bluetooth-low-energy-device](https://stackoverflow.com/questions/35986549/totally-disconnect-a-bluetooth-low-energy-device)
>
> [https://stackoverflow.com/questions/37653487/android-ble-device-is-not-disconnecting-sometime](https://stackoverflow.com/questions/37653487/android-ble-device-is-not-disconnecting-sometime)
>
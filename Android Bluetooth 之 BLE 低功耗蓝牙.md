# Android Bluetooth 之 BLE 低功耗蓝牙
- Bluetooth Low Energy 低功耗蓝牙，旨在显著降低功耗（优化数据传输效率，通过最小化字节使用实现低功耗），适用于可穿戴设备、智能家居、传感器设备、心率监测器和车载系统等场景，支持短距离（通常在 100 米以内）间歇性数据传输，BLE 搜索、连接的速度更快，不过传输的速度慢，传输的数据量也很小
- 基于 GATT 协议，所有数据通过 Attributes 属性（属性就代表一小段数据）进行传输，属性结构包含 Service 服务、Characteristic 特征值和 Descriptor 描述符三层，每个属性均由 128 位 UUID 通用唯一标识符进行标识，规定每次最多有效传输 20 个字节，超过了就需要分批（分包）多次进行传输了
- 基于客户端-服务端架构，分为 Central 中央设备与 Peripheral 外围设备，中央设备作为 GATT 客户端（主设备，比如安卓手机设备），外围设备作为 GATT 服务端（从设备，比如心率监测器），如果两个设备都仅支持中央角色，则无法相互通信，如果两个设备都仅支持外围角色，也无法相互通信，另外通信都是由客户端主动发起并接收服务端的响应的
    - 中央设备：主动进行扫描、查找、接收广播的设备（比如安卓手机设备）
    - 外围设备：发送广播的设备（比如智能手环）
- 一个外围设备同时只能被一个中央设备连接，一旦这个外围设备被连接就会马上停止广播，对其他设备就不可见了，而当这个外围设备断开后又开始广播，另外如果想要让两个外围设备能通信，只能通过中心设备进行中转
- 可以同时使用传统蓝牙和低功耗蓝牙，但是不能同时扫描（搜索、发现、查询）低功耗蓝牙设备和传统蓝牙设备，相对于传统蓝牙设备需要先配对，而 BLE 无需配对

## GATT
- GATT：Generic Attribute Profile 通用属性配置文件，基于 ATT 属性协议实现应用级数据结构（比如服务、特征值等）和数据交互规范，实现设备间复杂数据传输
- ATT：Attribute Protocol 属性协议，定义了 Attribute 属性的读写、通知等操作规则，直接负责数据的传输
- GAP：Generic Access Profile 通用访问配置文件，定义设备发现、连接、断开、设备角色和安全配置等基础规则，控制设备广播与扫描，使得设备之间能够通过 ATT 和 GATT 进行通信

## Attribute
- Service：是一组相关的功能或数据，包含一个或者多个 Characteristic 特征值
- Characteristic：是数据传输的最小基本单位，是 Service 服务包含的具体数据项
- Descriptor：特征值补充描述属性（对特征值的额外说明，比如通知配置）

## MTU
- Maximum Transmission Unit 最大传输单元
- BLE 传输速率较低，所以推荐单次传输数据建议不超过 20 字节（MTU 默认是 23 个字节，其中只有 20 个有效字节）
```kotlin
//建立连接之后调用
bluetoothGatt.requestMtu(247) //MTU 协商，实际生效不会超过设备的最大支持数
```

## 基本流程
- 1 权限声明、权限申请、硬件检测与蓝牙启用
- 2 BluetoothLeScanner#startScan 扫描 BLE 设备
- 3 BluetoothDevice#connectGatt 连接 BLE 设备
- 4 BluetoothGatt#discoverServices 发现服务
- 5 特征值的读写与通知订阅
- 6 断开连接与释放清理资源

## 1 权限声明、权限申请、硬件检测与蓝牙启用
权限相关
```xml
<uses-permission android:name="android.permission.BLUETOOTH"/>
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>
<!-- 高版本额外需要位置权限 -->
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
<!-- 可选硬件声明，声明设备支持 BLE，防止不支持的设备安装 -->
<uses-feature android:name="android.hardware.bluetooth_le" android:required="true"/>
```

检查硬件支持情况
```kotlin
//使用 PackageManager 来检查设备是否支持 BLE 功能
packageManager.hasSystemFeature(PackageManager.FEATURE_BLUETOOTH_LE)
```

蓝牙启用
```kotlin
val bluetoothManager = getSystemService(Context.BLUETOOTH_SERVICE) as BluetoothManager
val bluetoothAdapter = bluetoothManager.adapter //获取蓝牙适配器
if (bluetoothAdapter?.isEnabled != true) {
    //询问用户开启蓝牙
    Log.e(TAG, "请先开启蓝牙")
    val enableBtIntent = Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE)
    startActivityForResult(enableBtIntent, REQUEST_CODE_ENABLE_BT)
    return
}
```

## 2 扫描 BLE 设备
```kotlin
private val REQUEST_CODE_ENABLE_BT = 1
private val SCAN_TIMEOUT = 10_000L //扫描超时时间
private var bluetoothLeScanner: BluetoothLeScanner? = null
private var bluetoothGatt: BluetoothGatt? = null
//
private val handler = Handler(Looper.getMainLooper())
//BLE 扫描回调 android.bluetooth.le.ScanCallback
private val leScanCallback = object : ScanCallback() {
    override fun onScanResult(callbackType: Int, result: ScanResult) {
        val bluetoothDevice = result.device
        //处理扫描到的设备（比如展示在列表中供选择，存入列表之前可能需要先判断去重）
        Log.e(TAG, "onScanResult: name：${bluetoothDevice.name} address：${bluetoothDevice.address}")
    }
    override fun onScanFailed(errorCode: Int) {
        Log.e(TAG, "onScanFailed 扫描失败，错误码：$errorCode")
    }
}
private fun startBleScan() {
    val bluetoothManager = getSystemService(Context.BLUETOOTH_SERVICE) as BluetoothManager
    val bluetoothAdapter = bluetoothManager.adapter //获取蓝牙适配器
    if (bluetoothAdapter?.isEnabled != true) {
        //开启蓝牙（需用户确认）
        Log.e(TAG, "startBleScan: 请先开启蓝牙")
        val enableBtIntent = Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE)
        startActivityForResult(enableBtIntent, REQUEST_CODE_ENABLE_BT)
        return
    }
    //配置扫描过滤
    val scanFilter = ScanFilter.Builder()
        //.setDeviceName("deviceName")
        .build()
    //配置扫描参数
    val scanSettings = ScanSettings.Builder()
        //扫描模式：SCAN_MODE_LOW_POWER 省电模式（默认）、SCAN_MODE_BALANCED 平衡模式
        .setScanMode(ScanSettings.SCAN_MODE_LOW_LATENCY) //低延迟模式
        //回调类型：CALLBACK_TYPE_FIRST_MATCH 仅接收第一个匹配的
        .setCallbackType(ScanSettings.CALLBACK_TYPE_ALL_MATCHES)
        .build()
    //获取 BluetoothLeScanner 扫描器
    bluetoothLeScanner = bluetoothAdapter.bluetoothLeScanner
    //启动扫描
    bluetoothLeScanner?.startScan(listOf(scanFilter), scanSettings, leScanCallback)
    //定时停止扫描，推荐设置合适的扫描时间间隔，避免长时间持续扫描，在不需要的时候要及时调用停止扫描，以节省功耗
    handler.postDelayed({ stopBleScan() }, SCAN_TIMEOUT)
}
//停止扫描
private fun stopBleScan() {
    bluetoothLeScanner?.stopScan(leScanCallback)
}
```

## 3 连接 BLE 设备、4 发现服务、5 特征值的读写与通知订阅
```kotlin
private fun connectBleDevice(macAddress: String) {
    val gattCallback = object : BluetoothGattCallback() {
        override fun onConnectionStateChange(gatt: BluetoothGatt, status: Int, newState: Int) {
            if (newState == BluetoothProfile.STATE_CONNECTED) {
                //与设备 GATT 服务端建立连接成功
                gatt.discoverServices() //连接成功后进行发现服务
            } else if (newState == BluetoothGatt.STATE_DISCONNECTED) {
                gatt.close()
            }
        }
        override fun onServicesDiscovered(gatt: BluetoothGatt?, status: Int) {
            if (status == BluetoothGatt.GATT_SUCCESS) {
                //服务发现成功
                val gattServiceList = gatt.services
                for (gattService in gattServiceList) {
                    for (characteristic in gattService.characteristics) {
                        Log.e(TAG, "onServicesDiscovered: ${characteristic.value}")
                    }
                }
                //
                val gattService = gatt.getService(UUID.fromString("UUID"))
                val characteristic = gattService.getCharacteristic(UUID.fromString("UUID"))
                //读取特征值，触发 BluetoothGattCallback#onCharacteristicRead 回调
                gatt.readCharacteristic(characteristic)
                //写入特征值，触发 BluetoothGattCallback#onCharacteristicWrite 回调
                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
                    gatt.writeCharacteristic(characteristic, "data", 0)
                } else {
                    characteristic.setValue("data")
                    gatt.writeCharacteristic(characteristic)
                }
                //为某个特征值启用通知（监听数据变化），触发 BluetoothGattCallback#onCharacteristicChanged 回调
                gatt.setCharacteristicNotification(characteristic, true)
                //
                val descriptor = characteristic.getDescriptor(UUID.fromString("UUID"))
                descriptor.setValue(BluetoothGattDescriptor.ENABLE_NOTIFICATION_VALUE)
                gatt.writeDescriptor(descriptor) //触发 BluetoothGattCallback.onDescriptorWrite 回调
            }
        }
        override fun onCharacteristicRead(
            gatt: BluetoothGatt,
            characteristic: BluetoothGattCharacteristic,
            value: ByteArray,
            status: Int
        ) {
            if (status === BluetoothGatt.GATT_SUCCESS) {
                Log.d(TAG, "读取特征值 value: ${String(value)}")
            }
        }
        override fun onCharacteristicWrite(
            gatt: BluetoothGatt?,
            characteristic: BluetoothGattCharacteristic?,
            status: Int
        ) {
            if (status === BluetoothGatt.GATT_SUCCESS) {
                Log.d(TAG, "写入特征值成功!")
            }
        }
        override fun onCharacteristicChanged(
            gatt: BluetoothGatt,
            characteristic: BluetoothGattCharacteristic,
            value: ByteArray
        ) {
            //接收特征值变化
        }
    }
    val bluetoothManager = getSystemService(Context.BLUETOOTH_SERVICE) as BluetoothManager
    val bluetoothAdapter = bluetoothManager.adapter //获取蓝牙适配器
    val bluetoothDevice = bluetoothAdapter?.getRemoteDevice(macAddress)
    //通过 connectGatt 方法与设备 GATT 服务端建立连接
    //bluetoothGatt = bluetoothDevice?.connectGatt(this, false, gattCallback)
    bluetoothGatt = bluetoothDevice?.connectGatt(this, false, gattCallback, BluetoothDevice.TRANSPORT_LE)
}
```

## 6 断开连接与释放清理资源
```kotlin
fun releaseGatt() {
    bluetoothGatt?.disconnect()
    bluetoothGatt?.close()
    bluetoothGatt = null
}
```

## 总结
- BLE 基于 GATT 协议，设备之间通过 Service 和 Characteristic 等 Attribute 属性进行数据通信（设备暴露服务和特征值供读写和通知），每个 Service 可以包含一个或多个 Characteristic
- 可以从 Characteristic 读取数据，也可以往 Characteristic 写入数据，从而实现了双向的通信
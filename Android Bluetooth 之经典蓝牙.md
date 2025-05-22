# Android Bluetooth 之经典蓝牙
- 蓝牙开发分为经典蓝牙（Bluetooth Classic，标准蓝牙）和 BLE 低功耗蓝牙（Bluetooth Low Energy，低能耗蓝牙）两种，经典蓝牙速度快，功耗高，适合大数据传输，低功耗蓝牙能耗低，但数据速率较低，适用于传感器数据采集
- 基于 RFCOMM 协议（Radio Frequency Communication），通过 BluetoothSocket 进行连接和数据传输（一个 UUID 唯一标识对应一个服务，比如 SPP_UUID 串口通信、A2DP_UUID 音频传输和 HEADFREE_UUID 免提等），SPP 串行端口配置文件（Serial Port Profile），允许两个设备建立 P2P 点对点连接，模拟 RS232 串口，使用 RFCOMM 协议进行数据传输
- 可以同时使用传统蓝牙和低功耗蓝牙，但是不能同时扫描（搜索、发现、查询）低功耗蓝牙设备和传统蓝牙设备

## 基本流程
- 1 权限声明、权限申请、硬件检测与蓝牙启用
- 2 BluetoothAdapter#startDiscovery 蓝牙设备发现（配合 BroadcastReceiver 接收发现新设备的结果）
- 3 BluetoothSocket#connect 蓝牙设备连接
- 4 数据读写
- 5 释放清理资源

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

## 2 蓝牙设备发现
```kotlin
private val REQUEST_CODE_ENABLE_BT = 1
private val SPP_UUID = UUID.fromString("00001101-0000-1000-8000-00805F9B34FB") //固定的 SPP 协议 UUID，连接串口设备（标准串口协议）
private val DISCOVERY_TIMEOUT = 10_000L //设备发现超时时间
private var bluetoothAdapter: BluetoothAdapter? = null
//
private val handler = Handler(Looper.getMainLooper())
//接收发现新设备的结果的广播接收器
private val discoveryBroadcastReceiver = object : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        if (BluetoothDevice.ACTION_FOUND == intent.action) {
            //发现新设备
            val bluetoothDevice = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
                intent.getParcelableExtra(
                    BluetoothDevice.EXTRA_DEVICE,
                    BluetoothDevice::class.java
                )
            } else {
                intent.getParcelableExtra<BluetoothDevice>(BluetoothDevice.EXTRA_DEVICE)
            }
            //处理发现到的设备（比如展示在列表中供选择）
            if (bluetoothDevice?.bondState != BluetoothDevice.BOND_BONDED) {
                Log.e(
                    TAG,
                    "onReceive: name：${bluetoothDevice?.name} address：${bluetoothDevice?.address}"
                )
                //发起配对（需用户确认）
                val pairIntent = Intent(BluetoothDevice.ACTION_PAIRING_REQUEST)
                pairIntent.putExtra(BluetoothDevice.EXTRA_DEVICE, bluetoothDevice)
                startActivity(pairIntent)
            }
        } else if (BluetoothAdapter.ACTION_DISCOVERY_FINISHED == intent.action) {
            //设备发现完成回调
            stopBtDiscovery()
        }
    }
}
private fun startBtDiscovery() {
    val bluetoothManager = getSystemService(Context.BLUETOOTH_SERVICE) as BluetoothManager
    bluetoothAdapter = bluetoothManager.adapter
    if (bluetoothAdapter?.isEnabled != true) {
        //开启蓝牙（需用户确认）
        Log.e(TAG, "startBtDiscovery: 请先开启蓝牙")
        val enableBtIntent = Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE)
        startActivityForResult(enableBtIntent, REQUEST_CODE_ENABLE_BT)
        return
    }
    //查询已绑定（配对）设备
    val bondedDeviceSet = bluetoothAdapter?.bondedDevices
    bondedDeviceSet?.let {
        if (bondedDeviceSet.size > 0) {
            for (bondedDevice in bondedDeviceSet) {
                //处理已绑定设备（可以用列表展示出来）
            }
        }
    }
    //注册广播接收器
    val intentFilter = IntentFilter()
    intentFilter.addAction(BluetoothDevice.ACTION_FOUND)
    intentFilter.addAction(BluetoothAdapter.ACTION_DISCOVERY_FINISHED)
    registerReceiver(discoveryBroadcastReceiver, intentFilter)
    //启动发现
    stopBtDiscovery()
    bluetoothAdapter?.startDiscovery()
    //定时停止设备发现
    handler.postDelayed({ stopBtDiscovery() }, DISCOVERY_TIMEOUT)
}
//停止设备发现
private fun stopBtDiscovery() {
    if (bluetoothAdapter?.isDiscovering() == true) {
        bluetoothAdapter?.cancelDiscovery()
    }
}
```

## 3 蓝牙设备连接、4 数据读写
```kotlin
private fun connectBtDevice(macAddress: String) {
    stopBtDiscovery()
    //
    Thread {
        var inputStream: InputStream? = null
        var outputStream: OutputStream? = null
        var bluetoothSocket: BluetoothSocket? = null
        try {
            val bluetoothDevice = bluetoothAdapter?.getRemoteDevice(macAddress)
            bluetoothDevice.createBond() //手动触发配对
            //创建 Rfcomm 套接字（传统蓝牙的核心通信方式）
            bluetoothSocket = bluetoothDevice?.createRfcommSocketToServiceRecord(SPP_UUID)
            bluetoothSocket?.connect() //阻塞式连接，需在子线程执行
            Log.e(TAG, "connectBtDevice: 连接成功")
            inputStream = bluetoothSocket?.inputStream
            outputStream = bluetoothSocket?.outputStream
            //读取数据（实际情况通常需要死循环不断读取数据  while (true) { }）
            inputStream?.let {
                val bufferBytes = ByteArray(1024)
                var length: Int
                while (inputStream.read(bufferBytes).also { length = it } != -1) {
                    val receivedData = String(bufferBytes, 0, length)
                    Log.e(TAG, "收到数据: $receivedData")
                }
            }
            //写入数据
            val message = "Hello Bluetooth!"
            outputStream?.write(message.toByteArray())
        } catch (e: IOException) {
            e.printStackTrace()
            Log.e(TAG, "connectBtDevice: 连接失败：${e.message}")
        } finally {
            inputStream?.close()
            outputStream?.close()
            bluetoothSocket?.close()
        }
    }.start()
}
//服务端启动监听
fun startServer() {
    Thread {
        var inputStream: InputStream? = null
        var outputStream: OutputStream? = null
        var bluetoothServerSocket: BluetoothServerSocket? = null
        var bluetoothSocket: BluetoothSocket? = null
        try {
            //服务端 Socket（UUID 需与客户端一致）
            bluetoothServerSocket = bluetoothAdapter?.listenUsingRfcommWithServiceRecord("BtServer",SPP_UUID)
            //接受客户端连接
            bluetoothSocket = bluetoothServerSocket?.accept()
            Log.d("BluetoothServer", "客户端已连接")
            inputStream = bluetoothSocket?.inputStream
            outputStream = bluetoothSocket?.outputStream
            //读取数据（实际情况通常需要死循环不断读取数据  while (true) { }）
            inputStream?.let {
                val bufferBytes = ByteArray(1024)
                var length: Int
                while (inputStream.read(bufferBytes).also { length = it } != -1) {
                    val receivedData = String(bufferBytes, 0, length)
                    Log.e(TAG, "收到数据: $receivedData")
                }
            }
            //写入数据
            val message = "Hello Bluetooth!"
            outputStream?.write(message.toByteArray())
        } catch (e: IOException) {
            e.printStackTrace()
        } finally {
            inputStream?.close()
            outputStream?.close()
            bluetoothSocket?.close()
            bluetoothServerSocket?.close()
        }
    }.start()
}
//服务端设置为可发现模式（让设备可被检测）
fun discoverable() {
    val discoverableIntent = Intent(BluetoothAdapter.ACTION_REQUEST_DISCOVERABLE)
    //默认不传是 120 秒，传 0 代表一直可被检测到（官方不建议使用），最长支持 3600 秒
    discoverableIntent.putExtra(BluetoothAdapter.EXTRA_DISCOVERABLE_DURATION, 300)
    startActivity(discoverableIntent)
}
```
 
## 5 释放清理资源
```koltin
fun releaseDiscovery() {
    unregisterReceiver(discoveryBroadcastReceiver)
    bluetoothAdapter?.cancelDiscovery()
    bluetoothAdapter = null
}
```

## 总结
- 经典蓝牙基于客户端-服务端架构，使用 RFCOMM 通信开启模拟串口，通过 SPP_UUID 进行交互
- 通过 BluetoothSocket 的 InputStream 和 OutputStream 输入输出流读写数据，从而实现双向通信
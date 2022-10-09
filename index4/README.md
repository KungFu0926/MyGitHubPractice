# Bob

## 封包協定
封包使用串列埠進行傳輸，手機使用藍芽與機器人建立連線，並開啟串列端口。

### 編碼流程
資料編碼成封包，需要經過以下步驟。
1. 將資料編碼成 **UTF-8位元組陣列**。
2. 將**1步驟之位元組陣列**透過URL Safe Base64編碼成**Base64字串**。
3. 在**2步驟之Base64字串**結尾加上換行符，實現封包分隔之用途。

### 解碼流程
封包解碼成資料，需要經過以下步驟。
1. 將收到之封包換行符切割。
2. 將**1步驟之字串解碼成位元組陣列**，並透過URL Safe Base64解碼成**位元組陣列**。
3. 將**2步驟之位元組陣列**透過UTF-8解碼成字串資料。

## Base64
### 簡介
Base64主要的功能是將無法印出的位元組，可以透過64個可印出字元表示。
Base64的編碼/解碼以6個位元為一個單元。
Base64是一種基於64個可列印字元來表示二進位資料的表示方法。

由於$log_2(64)$，所以每6個位元為一個單元，對應某個可列印字元。3個位元組相當於24個位元，對應於4個Base64單元，即3個位元組可由4個可列印字元來表示。在Base64中的可列印字元包括字母A-Z、a-z、數字0-9，這樣共有62個字元，此外兩個可列印符號在不同的系統中而不同。一些如uuencode的其他編碼方法，和之後BinHex的版本使用不同的64字元集來代表6個二進位數字，但是不被稱為Base64。

Base64常用於在通常處理文字資料的場合，表示、傳輸、儲存一些二進位資料，包括MIME的電子郵件及XML的一些複雜資料。

### 編碼
| 字元 | ASCII(1 byte) |
| -------- | -------- |
| B   |01000010|
| o   |01101111|
| b   |01100010|

每6個位元為一個單元
| 單元 | 索引 |對應字元|
| -------- | -------- |-------- |
|010000    |16|Q|
|100110    |38|m|
|111101    |61|9|
|100010    |34|i|

## Bob
### HC-05 藍芽連接
使用CP2102接上HC-05進行UART傳輸，鮑率為38400。


## objects.json
為了避免造成後續更改的困難，增加/更新/刪除物件定義只要更改objects.json即可，不須更改程式碼，一方面增加程式之擴充性也增加開發時的方便性。
objects.json裡包含物件的json資料。


## Android
### 參考
- https://github.com/koral--/android-gif-drawable
- https://developer.android.com/guide/topics/connectivity/bluetooth.html
- https://xnfood.com.tw/activity-life-cycle/
- https://developer.android.com/guide/components/activities/activity-lifecycle
- https://developer.android.com/guide/components/fragments
- https://developer.android.com/guide/components/services
- https://www.tutorialspoint.com/android/android_text_to_speech.htm
- https://medium.com/verybuy-dev/android-%E8%A3%A1%E7%9A%84%E7%B4%84%E6%9D%9F%E6%80%96%E5%B1%80-constraintlayout-6225227945ab
- https://materialdesignicons.com/

## Python
### 資訊
- 版本:Python 3.6.9


### 安裝 python 3.6
```
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt-get update
sudo apt-get install python3.6
```
- https://askubuntu.com/questions/865554/how-do-i-install-python-3-6-using-apt-get
- https://linuxhint.com/update_alternatives_ubuntu/
- https://myoceane.fr/index.php/python-alternatives/

### 序列裝置分配器
因為XXX接上USB Hub，造成序列埠編號在每次重開機時改變例如原本設定COM0為HC-05之序列埠編號，有可能於下次開機時改變為COM1。造成開發及執行上的困難。因此，使用了以下方式進行序列埠編號判別。

```pyhton=
import re
global robotics, app

coms=comports()
for com in coms:
    print(com.description)
    if re.search(".*CP2102.*",com.description):
           print(com.description)
       app = HC05Serial(com.device)
    elif re.search(".*FT232R.*",com.description):
       print(com.description)
       robotics = RoboticsSerial(com.device)

```

re為正規表達式之函式庫。
re.search(x,y) 則是從y輸入字串中比對是否符合x正規表達示。

因為HC-05會接上CP2102，所以可以認定含有CP2102描述的即為HC-05序列埠。藉此在判斷式中進行初始化。

### 安裝 PyBluez
```shell
sudo apt install python-dev libpython3.6-dev libbluetooth-dev
```

```
sudo nano /etc/systemd/system/dbus-org.bluez.service
ExecStart=/usr/lib/bluetooth/bluetoothd -C
sudo systemctl daemon-reload
sudo systemctl restart bluetooth
```
#### 手機無法連線Serial Port Server

```shell
sudo hciconfig hci0 piscan
```

#### 藍芽sp權限遭拒

```
During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "/home/vincent/git_projects/Bob/Bob_python/bt_main.py", line 19, in <module>
    server = BluetoothServer(ConnectListener())
  File "/home/vincent/git_projects/Bob/Bob_python/Bob/bluetooth_utils/utils.py", line 28, in __init__
    profiles=[bluetooth.SERIAL_PORT_PROFILE],
  File "/home/vincent/git_projects/Bob/Bob_python/venv/lib/python3.6/site-packages/bluetooth/bluez.py", line 275, in advertise_service
    raise BluetoothError (*e.args)
bluetooth.btcommon.BluetoothError: [Errno 13] Permission denied
```

```shell
sudo chmod 666 /var/run/sdp
```



- https://stackoverflow.com/questions/37913796/bluetooth-error-no-advertisable-device
- https://pybluez.readthedocs.io/en/latest/install.html
- https://github.com/MeetMe/newrelic-plugin-agent/issues/151
- https://github.com/pybluez/pybluez/blob/master/examples/simple/rfcomm-server.py
- https://raspberrypi.stackexchange.com/questions/41776/failed-to-connect-to-sdp-server-on-ffffff000000-no-such-file-or-directory
- https://forums.developer.nvidia.com/t/nvidia-jetson-xavier-nx-bluetooth-connection-issue/156351/18
- https://blog.csdn.net/qq_33475105/article/details/111995309

## Linux
### Bluetooth
- https://scribles.net/setting-up-bluetooth-serial-port-profile-on-raspberry-pi/
- https://blog.csdn.net/Adrian503/article/details/110947477
- https://blog.csdn.net/XiaoXiaoPengBo/article/details/108125755
- https://connectivity-staging.s3.us-east-2.amazonaws.com/s3fs-public/2018-10/HCI%20Bluetooth%20Module%20SPP%20Connection%20on%20Linux%20v1_0.pdf
- https://raspberrypi.stackexchange.com/questions/41776/failed-to-connect-to-sdp-server-on-ffffff000000-no-such-file-or-directory
- https://forums.developer.nvidia.com/t/jetson-nano-bluetooth-issue-rfcomm-tty-support-not-available/81432


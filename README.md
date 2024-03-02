# 2018년 정보통신산업진흥원 주관 삼성 타이젠 미니해커톤 
- Tizen Midea Vision Surveillance API Object Tracking 및 센서 활용  
- 차량 내 어린이 질식사고를 예방하기 위한 스마트 블랙박스  

![](https://images.velog.io/images/agugu95/post/9b0c8b86-00fc-4777-a87c-adb8964aa56f/2018%20%E1%84%86%E1%85%B5%E1%84%82%E1%85%B5%E1%84%92%E1%85%A2%E1%84%8F%E1%85%A5%E1%84%90%E1%85%A9%E1%86%AB%20%E1%84%91%E1%85%A9%E1%84%89%E1%85%B3%E1%84%90%E1%85%A5.jpg)  

## Object Tracking을 통한 차량 내 질식예방 카메라  
- Samsung Smart Things를 연동하여 사용자에게 알림 
- 차량 문이 잠겨 있을 때와 시동이 걸려있지 않을 때, 카메라를 통해 움직임 감지 시 차량비상등 점멸 및 창문을 개방

### 객체 검출 및 추적 API 사용
- Tizen Midea Vision Surveillnce API 사용  
Tizen Dev API Refrence
[링크](https://developer.tizen.org/development/api-references/native-application?redirect=https://developer.tizen.org/dev-guide/5.0.0/org.tizen.native.mobile.apireference/group__CAPI__MEDIA__VISION__SURVEILLANCE__MODULE.html)
``` 
mv_surveillance 헤더는
http://tizen.org/feature/vision.image_recognition
http://tizen.org/feature/vision.face_recognition 
를 참조하였음.
```

- mv_surveillance API에 따른 App 구동 Loop  
![image](https://user-images.githubusercontent.com/38939634/63633555-31cd5100-c685-11e9-9787-9b3e2113b710.png)


## Tizen Craft Room API Refrence to USB Camera  
- Application's Camera Life Cycle  
![image](https://user-images.githubusercontent.com/38939634/63688820-5c650880-c843-11e9-8980-c2a0073c0c21.png)  

1. 카메라 객체 생성  
2. Preview 객체를 통한 미리보기 상태  
3. 캡쳐 후 자동반복  
각 함수들은 프리뷰/캡쳐 중에도 이벤트를 받아야함으로 콜백으로 작성됨  
[함수 참조](https://craftroom.tizen.org/simple-iot-camera/)  

## Tizen Craft Room API Refrence to mv_surveillance  
- 실제로 핵심적으로 사용 되었던 Object Tracking API  
- Image Priview를 받아와 Vision API를 통해 처리하는 것이라 어느정도의 지연이 있었음  
Image Privew -> Media Vision Callback Function -> Object Detecking -> Call callback Function  
[참조](https://craftroom.tizen.org/%ec%95%84%ed%8b%b1%ec%9c%bc%eb%a1%9c-smart-surveillance-camera-%eb%a7%8c%eb%93%a4%ea%b8%b0/)  

## Profiling Image Data

### 카메라의 물리적 이동시간
슈퍼슬로우 카메라로 촬영결과 500ms 정도 소요

### Rpi Board

#### Image Encoding (buffer -> jpg file)
소요시간 20 ~ 135ms (가끔씩 오래걸림, 장담 못함)

#### Vision Survailance (input -> cb event)
소요시간 38 ~ 50ms


### Artik Board

#### Image Encoding (buffer -> jpg file)
소요시간 10 ~ 20ms, 대부분 10ms 초반 안정적

#### Vision Survailance (input -> cb event)
소요시간 24 ~ 42ms

## Vision 움직임 정보 형식 (exif)
최대 244Byte 크기의 스트링으로 모두 숫자로 이루어져있다.

첫 2자리 숫자는 분석결과의 타입으로 TT 의 값을 갖는다.

그 다음 2자리 숫자는 포함된 움직임의 갯수 NN 의 값을 갖는다.

하나의 움직임은 8개 숫자로 구성되며 xxyywwhh 의 값을 갖는다.

xx: x 상대 좌표
yy: y 상대 좌표
ww: 상대 넓이
hh: 상대 높이

각각 0~99까지의 범위를 갖는 4개의 숫자를 이어놓은 것이다.
1자리 숫자의 경우 0을 넣어서 전체 길이를 고정한다.

TTNNxxyywwhhxxyywwhh....xxyywwhh 의 형태의 스트링이 된다.  

# 
# How To Set up Tizen and Artik Board 
## HOW TO RUN - First run

### 1. Flash binary
Tizen 5.0 M2 IoT 부트 이미지 다운로드 [[링크](http://download.tizen.org/releases/milestone/tizen/unified/tizen-unified_20181024.1/images/standard/iot-boot-armv7l-artik533s/)]

Tizen 5.0 M2 Iot Headed 이미지 다운로드 [[링크](http://download.tizen.org/releases/milestone/tizen/unified/tizen-unified_20181024.1/images/standard/iot-headed-3parts-armv7l-artik530_710/)]
```
sudo minicom
    thordown
lthor tizen-unified_20181024.1_iot-boot-armv7l-artik533s.tar.gz tizen-unified_20181024.1_iot-headed-3parts-armv7l-artik530_710.tar.gz
```

### 2. Install plug-in

ARTIK 530(5.0) Plugin download from https://developer.samsung.com/tizendevice/firmware


### 3. Install Wifi util (*optional)
Tizen 5.0 M2 wifi-manager-tool 다운로드 [[링크](http://download.tizen.org/releases/milestone/tizen/unified/tizen-unified_20181024.1/repos/standard/packages/armv7l/capi-network-wifi-manager-tool-1.0.39-81.5.armv7l.rpm)]
```
sdb root on; sdb shell 'mount -o remount,rw /'

sdb push capi-network-wifi-manager-tool-1.0.39-81.5.armv7l.rpm /tmp

sdb shell 'rpm -ivh /tmp/capi-network-wifi-manager-tool-1.0.39-81.5.armv7l.rpm'
```

### 4. Build and install custom version of IoT.js to include WebSocket feature
```
git clone https://github.com/Samsung/iotjs.git

cd iotjs

sed -i 's/Release: 0/Release: 99/' config/tizen/packaging/iotjs.spec

sed -i '/--no-parallel-build/a \ \ --cmake-param=-DENABLE_MODULE_WEBSOCKET=ON \\' config/tizen/packaging/iotjs.spec

git diff config/tizen/packaging/iotjs.spec

./config/tizen/gbsbuild.sh --clean

sdb push ~/GBS-ROOT/local/repos/tizen_unified_m1/armv7l/RPMS/iotjs-1.0.0-9.0.armv7l.rpm /tmp

sdb shell 'cd /tmp; rpm -ivh iotjs-1.0.0-9.0.armv7l.rpm'
```
- https://github.com/Samsung/iotjs/wiki/Build-for-RPi3-Tizen
- https://github.com/Samsung/iotjs/blob/master/docs/api/IoT.js-API-WebSocket.md


### 5. Change camera setting
```
sdb shell "cat /etc/multimedia/mmfw_camcorder_camera0.ini | grep 640"

sdb shell "sed -i 's/640,480/320,240/g' /etc/multimedia/mmfw_camcorder_camera0.ini"

sdb shell "cat /etc/multimedia/mmfw_camcorder_camera0.ini | grep 320"
```

### 6. Build and install vision app
```
git clone git://git.tizen.org/apps/native/smart-surveillance-camera # This git

cd smart-surveillance-camera

gbs build -A armv7l --include-all

sdb push ~/GBS-ROOT/local/BUILD-ROOTS/scratch.armv7l.0/home/abuild/rpmbuild/RPMS/armv7l/iot-vision-camera-0.0.1-1.armv7l.rpm /tmp

sdb shell 'rpm -ivh --force /tmp/iot-vision-camera-0.0.1-1.armv7l.rpm'
```
### 7. Some more settings
```
sdb shell 'pkg_initdb'
```
### 8. Launch vision app
```
sdb shell 'app_launcher -s iot-vision-camera'
```
### 9. Check vision app activity
```
sdb shell 'ls -l /tmp/latest.jpg'
```
### 10. Install and run monitor server
* If you install latest version of "iot-vision-camera" package, the monitor server is automatically launched in booting time

* To check status of the monitor server
	```
	sdb shell 'systemctl status iot-dashboard'
	```

* To restart the monitor server
	```
	sdb shell 'systemctl stop iot-dashboard'
	sdb shell 'systemctl start iot-dashboard'
	sdb shell 'systemctl status iot-dashboard'  # To check status
	```

* To update some files of the monitor server, push the files to dashboard path `/opt/home/dashboard` and restart the monitor server as above
	```
	sdb shell push {path/yourfile} /opt/home/dashboard/{path}
	```
OR! YOU CAN SIMPLY USE THE SCRIPT:
```
./update-dashboard.sh
```


## HOW TO RUN - Subsequent Runs
```
sdb root on; sdb shell 'mount -o remount,rw /'

sdb shell 'app_launcher -s iot-vision-camera'

 ./update-dashboard.sh
```


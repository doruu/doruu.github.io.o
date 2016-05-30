---
layout: page
title: rpi-camera
permalink: /projects/rpi-camera/
---

# rpi-camera
http://github.com/doruu/rpi-camera

![](http://doruu.github.io/projects/rpi-camera/img/rpi-camera.gif)

카메라를 제어하고자 시작한 프로젝트 입니다. 
추후 안드로이드앱 을만들어 VR 까지 적용해볼 예정입니다. 

## sg90 servo 모터 제어
무난하고 저렴한 가격으로 선택하게되었습니다.
웹서핑 결과 pwm 제어가능 핀은 18, 13번, 좌우 제어용으로 18번, 상하 제어용으로 13번을 사용할 것입니다. 

### 핀연결

#### 1번 servo
- 빨간색(5v 전원), 노란색(18번), 검정색(gnd) 

![](http://doruu.github.io/projects/rpi-camera/img/servo_connect.jpg)

![](http://doruu.github.io/projects/rpi-camera/img/servo_connect2.jpg)

#### 2번 servo까지 연결한 모습

![](http://doruu.github.io/projects/rpi-camera/img/servo_connect3.jpg)

### wiringPi 

```
pinMode(18, PWM_OUTPUT);
pwmSetMode(PWM_MODE_MS);
pwmSetClock(384);
pwmSetRange(1000);
pwmWrite(18, 75);
```
위는 pwm 관련 함수들 입니다. 처음 봤을때 clock 값과 range 값은 어떻게 나온숫자인지 알수가 없었습니다.  
검색을 하다보니 아래 식이 보이네요. 
```
pwmFrequency in Hz = 19.2e6 Hz / pwmClock / pwmRange.
```

참고한 싸이트 링크 입니다 :  http://stackoverflow.com/questions/20081286/controlling-a-servo-with-raspberry-pi-using-the-hardware-pwm-with-wiringpi

일단 위내용을 토대로 분석해보면 sg90 은 20ms 에 1ms(-90도) ~ 2ms(90도) 입력 사양을 가집니다.

```
50HZ = 19200000 / pwmClock / pwmRange
```
위식대로 계산하면 pwmclock : 384 , pwmrange : 1000  이 맞는것을 확인할수 있습니다. 
그럼 위 공식만 맞으면 다른 값도 가능한가 싶어 몇가지 더 테스트를 해보았습니다.

```
1, 50HZ = 19200000 / 19200 / 20     -->  1ms ~ 2ms
2, 50HZ = 19200000 / 3840 / 100     --> 5 ~ 10
3, 50HZ = 19200000 / 960 / 400      --> 20 ~ 40
4, 50HZ = 19200000 / 384 / 1000     --> 50 ~ 100
5, 50HZ = 19200000 / 192 / 2000     --- 100 ~ 200
```

range 가 20ms(?) 이면 위 모터 사양대로 1~2 까지 입력을 주면 될까 싶어 테스트를 해 보았으나 모터가 정상동작을 하지 않더군요. 가운데 3개는 별문제 없었으나  마지막 2000 역시 동작하지 않았습니다. 1번테스트는 잠깐 놔두니 모터에서 발열이 발생합니다.. 정확한 이유는 나중에 찾아봐야할듯합니다. 

#### servo.cpp
c 로 간단히 2개 인자를 넘겨받는 코드를 작성합니다. 1번 인자는 좌우 제어 각, 2번인자는 상하 제어각, 각각 ~90도 ~ 90 도 값을 입력값으로 합니다. 그리고 pwm 입력값인  50 ~ 100 으로 변환 해주는 간단한 코드입니다.

##### 빌드
``` 
$ gcc servo.cpp -lwiringpi -lstdc++ -o servo
```
##### 실행
```
$ sudo ./servo 0 0
```


### servo.js

서버로 돌릴 node 코드 입니다. http://192.168.0.20:3030/servo?lr=0&tb=0 lr과 tb 를 입력값으로 servo.cpp 로 나온 실행파일을 실행합니다. 소스에서 라즈베리파이가 연결된 ip 로 변경해주어야합니다. 
```
server.listen(3030, "라즈베리파이 ip");
```

# pan, tilt 킷 
카메라를 좌우 상하로 움직이기 위한 킷이 몇개 팔지만 집에 남는 포맥스가 있어서 간단하게 만들어 봤습니다. 

- 50mm X 50mm 정도 포맥스에 카메라를 고정시키고 아래뒤쪽을 제외한 3면을 막아주었습니다.

![](http://doruu.github.io/projects/rpi-camera/img/pantilt.jpg)

- 안쪽에 모터 고정을 위한 날개를 순간접착제로 붙여주었습니다. 

![](http://doruu.github.io/projects/rpi-camera/img/pantilt5.jpg)

- 그리고 좌우로 회전하는 모터와 연결해줄 별도의 포맥스 조각에 모터 연결용 날개를 접착제로 고정.

![](http://doruu.github.io/projects/rpi-camera/img/pantilt2.jpg)

![](http://doruu.github.io/projects/rpi-camera/img/pantilt8.jpg)

- 2개 모터를 연결해줍니다.

![](http://doruu.github.io/projects/rpi-camera/img/pantilt6.jpg)

- 마무리... 는 일단 클램프로 고정하고 추후 케이스 제작예정 입니다. 

![](http://doruu.github.io/projects/rpi-camera/img/pantilt7.jpg)



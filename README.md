#USB power on/off

##목차
1. 시스템 개요
2. 시스템 설치
3. 시스템 실행
4. 시스템 입력
5. 시스템 출력
6. 시스템 오류 시 확인 사항


##1. 시스템 개요
* Master Host가 이더넷을 통해 Slave Host IP로 curl 명령을 수행하면 Slave Host가 자신의 USB Port 전원을 On/Off 하는 시스템
	- Master Host : 명령을 주는 개체로 Mac이나 Linux가 탑재된 컴퓨터
	- Slave Host : 명령을 받는 개체로 Linux가 설치된 라즈베리파이


##2. 시스템 설치

###a. 시스템 설치 환경
* H/W : Raspberry Pi 2 Model B
* OS : RASPBIAN


###b. Pre-install

>####1) libusb-dev
* USB 전원 제어를 위한 라이브러리 패키지
	<pre>
	$ sudo apt install libusb-dev
	</pre>

>####2) hub-ctrl
* USB 전원을 제어를 하는 프로그램
* git에서 코드를 받은 후 컴파일
	<pre>
	$ git clone https://github.com/codazoda/hub-ctrl.c.git 
	$ cd hub-ctrl.c
	$ gcc -o hub-ctrl hub-ctrl.c -lusb
	</pre>

###c. install
* usb\_on\_off\_server.py 파일을 다운로드


###d. 설치 후 코드 수정 사항
* 다운 받은 파이썬 코드 상단의 변수를 수정
	* path : 미리 설치한 hub-ctrl의 절대 경로 명시
	* ip : 사용할 IP 명시 (기본 : 로컬 IP 사용)
	* port : 사용할 Port 명시 (기본 : 8000번 Port)
	<pre>
	#-------------user modification-----------------

	path = '/home/pi/hub-ctrl.c/hub-ctrl'
	ip=''
	port=8000

	#-----------------------------------------------
	</pre>

##3. 시스템 실행
* usb\_on\_off\_server.py 파일에서 hub-ctrl 실행 파일의 위치와 서버 포트 번호를 수정하고 실행
	- 코드 수정은 __2-d) 설치 후 중요 수정 사항__ 참조
	<pre>
	$ python3 usb_on_off_server.py
	</pre>

##4. 시스템 입력
* curl 명령어 사용
	<pre>
$ curl -X GET http://<strong>IP</strong>:<strong>Port</strong>/<strong>수행할 행동</strong>
	</pre>
	* HTTP의 GET 요청을 사용하여 통신
	* Slave Host의 IP와 PORT를 명시한 URL 사용
	* URL의 자원 Path부분에 수행할 행동을 명시
		* off : 전원을 끔
		* on : 전원을 켬
		* restart : 전원을 끄고 다시 켬
		* 그 외 입력 시 수행 실패

	<pre>
	ex) $ curl –X GET http://172.30.1.39:8000/off
	</pre>
	* -X GET : GET 요청 사용
	* http://172.30.1.39:8000/ : IP와 Port를 명시한 URL
	* off : 수행할 행동 명시 - 전원을 끔
		


##5. 시스템 출력
* curl 기능 중에 HTTP 프로토콜을 사용하고 그 중에서도 GET 요청을 사용하기 때문에 서버로 부터 응답을 받음
	* off 명령 정상 수행 시 : USB Off Success
	* on 명령 정상 수행 시 : USB On Success
	* restart 명령 정상 수행 시 : USB Restart Success
	* 명령 수행 실패 시 : USB Restart Failure
	
>

* GET 요청을 사용하지 않을 경우 HTTP 프로토콜에 따른 메시지를 받음
	* HTTP Error code : 501 
	* Unsupported method

##6. 시스템 오류 시 확인 사항

###a. off 명령을 실행하였지만  USB 전원이 꺼진 후 자동으로 다시 켜지는 경우
* 라즈베리파이의 경우 USB에 연결된 장치의 전압이 전원을 공급해줄 수 있을 만큼 큰 경우 일반 전원을 대체하도록 하드웨어적으로 설계됨
* USB 전원을 끈 후 연결된 장치에 잔류 전압이 큰 경우 USB가 다시 동작하는 오류 발생
* off메소드에 있는 time.sleep()함수의 시간을 수정하여 USB가 완전히 동작하기 전에 잔류 전압을 소진시키고 USB 전원을 끔
<pre>
…
def off(self):
        result = subprocess.Popen(on_cmd).wait()
        self.check_result(result)
        result = subprocess.Popen(off_cmd).wait()
        self.check_result(result)
        time.sleep(1)
        result = subprocess.Popen(off_cmd).wait()
        self.check_result(result)
        print("USB power off")
…
</pre>
* 위 방법으로 안될 경우
* 
<pre>
time.sleep(1)
result = subprocess.Popen(off_cmd).wait()
self.check_result(result)
</pre>
	* 위 코드를 반복하여 연결된 장치에 있는 잔류 전압을 소진 시킴
# 네트워크 케이블로 커널디버깅 환경 구성하기
---
+ host
	- debugger 가 실행중인 PC
	- XP 이상 

+ target 
	- debugging 당할 pc
	- Windows 8 이상 

+ kenel mode debugging over network 이 좋은 점
	- host 와 target 이 local network 에 있기만 하면 됨
	- 하나의 host 에서 여러 target 을 디버깅하는데 용이
	- network cable 은 보통 준비되어있고, 저렴함
	- 두 컴퓨터에는 1394 나 serial 보다는 ethernet adapter 가 달려있는 경우가 많을거다.


## 네트워크 디버깅에 	사용할 포트 선택하기 
- 49152 ~ 65532 범위에서 맘대로
- 선택한 port 는 host 컴퓨터에서 debugger 가 사용함
- 다른 프로그램이 쓰지 않는 포트


## target 설정
1. target 의 network adapter 가 네트워크 디버깅을 지원하는지 확인
1. CAT5 나 그보다 좋은 네트워크 케이블로 hub 나 switch 에 연결
	- crossover cable 은 안됨
1. 관리자 권한으로 command prompt 를 띄우고, 아래 명령을 실행 
	- `w.x.y.z` 는 host 의 ip
	- `n` 은 선택한 port
	
			bcdedit /debug on
			bcdedit /dbgsettings net hostip:w.x.y.z port:n
			
1. bcdedit 이 자동으로 생성한 key 를 출력하면, 해당 키를 usb 같은 곳에 잘 저장해둠. 이 키는 나중에 host 에서 debugging session 을 시작할때 필요함 
	- key 를 직접 만들어서 쓸수도 있지만 자동으로 생성된 키를 사용하기를 강추함

1. 만일 target 에 network adapter 가 여러개인 경우 
	- device manager 를 이용, 디버깅에 이용할 어댑터의 PCI bus, device, function number 를 확인하고
	+ 권한 상승된 cmd 를 통해서 아래 명령을 실행한다 (`b`, `d`, `f` 는 bus number, device number, function number)

			bcdedit /set "{dbgsettings}" busparams b.d.f

1. target 리붓


## host 설정
crossover cable 말고, CAT5 나 그거보다 좋은 케이블로 hub 또는 switch 에 연결한다. 

## debugging 세션 시작하기 
### WinDbg 이용
1. host 에서 `WinDbg` 를 실행하고, `File` -> `Kernel Debug` -> `Net` 탭 선택, port 번호와 key 입력 후 OK
1. 아래 명령을 이용해서 실행해도 됨
		windbg -k net:port=n,key=Key 
1. WinDbg 관련 방화벽 알람이 있으면 허용해줌

### KD 이용
1. host 에서 아래 명령 실행 
		kd -k net:port=n,key=Key
1. KD 관련 방화벽 알람이 있으면 허용해줌

## REF
+ [Setting Up Kernel-Mode Debugging over a Network Cable Manually](https://goo.gl/6mepWb)



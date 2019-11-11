# 네트워크 디버깅 환경 설정하기

1. bcdedit 환경 보기

		C:\>bcdedit

		Windows Boot Manager
		--------------------
		identifier              {bootmgr}
		device                  partition=\Device\HarddiskVolume1
		description             Windows Boot Manager
		locale                  en-US
		inherit                 {globalsettings}
		default                 {current}
		resumeobject            {4f573204-f5ab-11e7-b195-c37af9c40bc0}
		displayorder            {current}
		toolsdisplayorder       {memdiag}
		timeout                 30

		Windows Boot Loader
		-------------------
		identifier              {current}
		device                  partition=C:
		path                    \Windows\system32\winload.exe
		description             Windows 10
		locale                  en-US
		inherit                 {bootloadersettings}
		recoverysequence        {4f573206-f5ab-11e7-b195-c37af9c40bc0}
		displaymessageoverride  Recovery
		recoveryenabled         Yes
		allowedinmemorysettings 0x15000075
		osdevice                partition=C:
		systemroot              \Windows
		resumeobject            {4f573204-f5ab-11e7-b195-c37af9c40bc0}
		nx                      OptIn
		bootmenupolicy          Standard

1. `debug` 엔트리 생성

		C:\>bcdedit /copy {current} /d "debug"
		The entry was successfully copied to {4f573208-f5ab-11e7-b195-c37af9c40bc0}.

1. `testsigning` mode 를 활성화 (참고: [bcdedit /set](https://docs.microsoft.com/en-us/windows-hardware/drivers/devtest/bcdedit--set))

		--bcdedit /set nointegritychecks on
		C:\>bcdedit /set {4f573208-f5ab-11e7-b195-c37af9c40bc0} testsigning on
		The operation completed successfully.

1. `debug` 엔트리를 default 부트 엔트리로 변경

		bcdedit /default {4f573208-f5ab-11e7-b195-c37af9c40bc0}

1. `debug` 엔트리의 디버깅 모드 활성화 

		C:\>bcdedit /debug {4f573208-f5ab-11e7-b195-c37af9c40bc0} on
		The operation completed successfully.

1. 네트워크 디버깅 환경 설정 (참고: [Setting Up Kernel-Mode Debugging over a Network Cable Manually](https://goo.gl/6mepWb))

		C:\>bcdedit /dbgsettings net hostip:192.168.0.3 port:50000
		Key=3w0dshz7rba4.315r7m1sq5a10.3kxzq2so7zx6o.2v9anh20hgj9c

	- hostip: debugger 의 ip
	- port: debugger 의 listen port
		- 49152 ~ 65532 범위에서 맘대로
		- 선택한 port 는 host 컴퓨터에서 debugger 가 사용함
		- 다른 프로그램이 쓰지 않는 포트

	- key: debugger 에서 인증에 사용하는 key 값

1. bcdedit 환경 보기

	부트 엔트리 정보 

		C:\>bcdedit /v

		Windows Boot Manager
		--------------------
		identifier              {9dea862c-5cdd-4e70-acc1-f32b344d4795}
		device                  partition=\Device\HarddiskVolume1
		description             Windows Boot Manager
		locale                  en-US
		inherit                 {7ea2e1ac-2e61-4728-aaa3-896d9d0a9f0e}
		default                 {4f573208-f5ab-11e7-b195-c37af9c40bc0}
		resumeobject            {4f573204-f5ab-11e7-b195-c37af9c40bc0}
		displayorder            {4f573205-f5ab-11e7-b195-c37af9c40bc0}
								{4f573208-f5ab-11e7-b195-c37af9c40bc0}
		toolsdisplayorder       {b2721d73-1db4-4c62-bf78-c548a880142d}
		timeout                 30

		Windows Boot Loader
		-------------------
		identifier              {4f573205-f5ab-11e7-b195-c37af9c40bc0}
		device                  partition=C:
		path                    \Windows\system32\winload.exe
		description             Windows 10
		locale                  en-US
		inherit                 {6efb52bf-1766-41db-a6b3-0ee5eff72bd7}
		recoverysequence        {4f573206-f5ab-11e7-b195-c37af9c40bc0}
		displaymessageoverride  Recovery
		recoveryenabled         Yes
		allowedinmemorysettings 0x15000075
		osdevice                partition=C:
		systemroot              \Windows
		resumeobject            {4f573204-f5ab-11e7-b195-c37af9c40bc0}
		nx                      OptIn
		bootmenupolicy          Standard

		Windows Boot Loader
		-------------------
		identifier              {4f573208-f5ab-11e7-b195-c37af9c40bc0}
		device                  partition=C:
		path                    \Windows\system32\winload.exe
		description             debug									//<!
		locale                  en-US
		inherit                 {6efb52bf-1766-41db-a6b3-0ee5eff72bd7}
		recoverysequence        {4f573206-f5ab-11e7-b195-c37af9c40bc0}
		displaymessageoverride  Recovery
		recoveryenabled         Yes
		testsigning             Yes										//<!
		allowedinmemorysettings 0x15000075
		osdevice                partition=C:
		systemroot              \Windows
		resumeobject            {4f573204-f5ab-11e7-b195-c37af9c40bc0}
		nx                      OptIn
		bootmenupolicy          Standard
		debug                   Yes										//<!


	디버깅 설정 보기 

		C:\>bcdedit /dbgsettings
		key                     3w0dshz7rba4.315r7m1sq5a10.3kxzq2so7zx6o.2v9anh20hgj9c
		debugtype               NET
		hostip                  192.168.0.3		//<! debugger 의 IP 주소
		port                    50000
		dhcp                    Yes
		The operation completed successfully.
		
1. `debugger` 에서 windbg 실행 후 `debuggee` 의 접속 대기

	- host 에서 `WinDbg` 를 실행하고, `File` -> `Kernel Debug` -> `Net` 탭 선택, port 번호와 key 입력 후 OK
	- 아래 명령을 이용해서 실행해도 됨
			windbg -k net:port=n,key=Key 
	- WinDbg 관련 방화벽 알람이 있으면 허용해줌

1. `debuggee` 리부트

	> shutdown -r -t 0
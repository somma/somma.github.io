# VirtualKD on Windows 10

가상머신(Debuggee)에서 vminstall.exe 를 설치했고 디버그 모드로 부팅이 되었음에도 불구하고, WinDbg 연결이 안되는 경우가 있다. 

1. `VirtualKD\target\x64` 아래 `kdbazis.dll`, `kdpatch.sys` 파일을 `c:\windows\system32\drivers` 디렉토리에 복사한다. 
1. 가상머신에서 `bcdedit /dbgsettings` 명령으로 디버그 설정이 제대로 되었는지 확인한다. 

        PS C:\Windows\system32> bcdedit /dbgsettings
        debugtype               Serial
        debugport               1
        baudrate                115200
        The operation completed successfully.

    가끔 `debugtype` 이 `local` 로 되어있는 경우가 있는데, 아래 명령을 통해서 `serial` 타입으로 변경해 주어야 한다.
    
        PS C:\Windows\system32> bcdedit /dbgsettings serial debugport:1 baudrate:115200

1. 호스트에서 `vmmon64.exe` 을 실행한 상태에서 가상머신을 재 부팅하고, 디버그 모드로 부팅하면 WinDbg 가 연결이 된다. 

    ![VirtualKD](img\virtualkd_1.png "VirtualKD")
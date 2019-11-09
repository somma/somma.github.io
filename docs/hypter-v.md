# Hyper-v Note

- Windows7 가상머신 생성시 반드시 generation 1 으로 해야 한다. 
+ 가상머신 내에서 네트워크 연결이 안되는 경우 (default switch 사용 시)
    - ICS 서비스에 문제가 있는경우일 수 있다고 함
    - 호스트를 리부팅 하니 잘 됨
    - 가상머신 운영체제 최초 설치시 자주 발생하는 듯함

## 커널디버깅 환경 설정하기

WinDbg help 의 `Setting Up Kernel-Mode debugging of a Virtual Machine Manually` 항목에 가장 정확한 설명이 있다. 
Generation 1 타입의 경우 COM 포트 정보가 HyperV manager 에서 보이지만 Generation 2 타입의 가상머신인 경우 COM 포트가 보이지 않는데 PowerShell 을 통해 설정을 추가해야 한다. 가상머신이 실행중인 경우 설정값을 변경할 수 없으니 가상머신을 종료한 상태에서 아래 과정을 수행한다. 

1. `Secure Boot` 비활성화하기 

`BCDEdit` 을 통해서 부트 설정을 변경하기 전에 `Secure Boot`, `BitLocker` 등의 보안 기능들을 비 활성화해야 한다.

        PS C:\WINDOWS\system32> Set-VMFirmware -VMName win10_x64 -EnableSecureBoot Off
        PS C:\WINDOWS\system32> Get-VMFirmware -VMName win10_x64

        VMName    SecureBoot SecureBootTemplate PreferredNetworkBootProtocol BootOrder
        ------    ---------- ------------------ ---------------------------- ---------
        win10_x64 Off        MicrosoftWindows   IPv4                         {File, Drive, Network, Drive}

1. `COM` 포트 추가하기 

    PowerShell 에서는 COM 포트가 보이지만 가상머신 내부의 장치관리자에서는 COM 포트가 보이지 않는데, 아래 명령으로 파이프를 COM 포트와 연결시켜주면 된다 (가상머신 내부의 장치관리자에서 보이지 않는 경우 가상머신을 리부팅하면 된다). 

        PS C:\WINDOWS\system32> Set-VMComPort -VMName win10_x64 1 \\.\pipe\pipe_win10
        PS C:\WINDOWS\system32> Get-VMComPort -VMName win10_x64

        VMName    Name  Path
        ------    ----  ----
        win10_x64 COM 1 \\.\pipe\win10_x64
        win10_x64 COM 2


    이 명령은 호스트의 `win10_x64` 파이프에 가상머신의 `COM 1` 을 연결한다. 


1. 가상머신 디버그 부트 설정 (Debuggee)

가상머신을 실행하고, 가상머신의 디버그 부트 옵션을 활성화하고, `bootmenupolicy` 를 `Legacy` 로 변경(Advanced boot option 메뉴가 보이도록)한 후  리부팅한다.

    PS C:\Windows\system32> bcdedit /debug on    
    PS C:\Windows\system32> bcdedit /dbgsettings serial debugport:1 baudrate:115200
    PS C:\Windows\system32> bcdedit /set bootmenupolicy Legacy
    PS C:\Windows\system32> shutdown -r -t 0

1. `WinDbg` 연결하기 

Windows 10 의 경우 서명안된 드라이버 로딩을 위해 `Driver sign enforcement` 를 부팅옵션에서 꺼주어야 하는데 Hyper-v 는 가상머신 부팅시 대기시간이 0 으로 설정되어 바로 부팅하도록 설정되어있기때문에 부팅 대기 시간을 5초로 변경해 준다. 

![Management 옵션 설정](img\hvn_2.png "Automatic Start Action")

`WinDbg` 를 `elevated mode` 로 실행하고(관리자 권한 실행), `VM over serial pipe` 로 연결한다.

![WinDbg](img\hvn_1.png "WinDbg config")


1. 실패
WinDbg 연결은 잘 되는데, Advanced Boot Option (F8) 이 안먹어서 짜증나서 포기. 그냥 vmware 쓰기로




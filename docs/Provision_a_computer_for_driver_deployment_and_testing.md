## Provision a computer for driver deployment and testing (WDK 10)
원본 URL : https://msdn.microsoft.com/en-us/library/windows/hardware/dn745909


### Prepare the target computer for provisioning
1. 테스트할 os 설치하고

1. x86, x64 컴퓨터이고, Secure Boot 이 활성화되어있다면 끈다. UEFI 와 Secure Boot 에 관해서는 [UEFI Firmware](https://technet.microsoft.com/ko-kr/library/hh824898.aspx)를 참고. ARM 프로세서인 경우 **Windows Debug Policy** 를 설치한다. 요건 MS 나 제조사에 의해서만 가능하다. 넌 Secure Boot 를 disable 할 필요가 없다. 

1. 타겟 컴퓨터에 타겟 호스트의 플랫폼에 맞는 **WDK Test Target Setup MSI** 를 설치한다. 요놈은 **Windows Driver Kit (WDK)** 설치 경로의 **Remote** 폴더 아래에 있다. 예를 들면 `C:\Program Files (x86)\Windows Kits\10\Remote\x64\WDK Test Target Setup x64-x64_en-us.msi`

1. 타겟 컴퓨터가 N 또는 KN 버전이면 **Media Feature Pack for N and KN versions of Windows** 를 설치한다. 
	- [Media Feature Pack for N and KN versions of Windows 8.1](https://www.microsoft.com/ko-kr/download/details.aspx?id=40744)
	- [Media Feature Pack for N and KN versions of Windows 8](https://www.microsoft.com/ko-kr/download/details.aspx?id=30685)
	- [Media Feature Pack for N and KN versions of Windows 7](https://www.microsoft.com/ko-kr/download/details.aspx?id=16546)
1. 타겟컴퓨터에 Windows Server 가 실행중이라면, **WDK Test Target Setup MSI** 가 생성한 DriverTest 폴더(**C:\DriverTest**)를 오른쪽 클릭 > 속성 > Security 탭에서 **Authenticated Users** 그룹에 **Modify** 권한을 줘라.

호스트와 타겟간에 ping 이 되는지 확인한다. (host 이름으로든 IP 로드 하나만 되면 된다.)


만일 호스트와 타겟이 workgroup 에 속해있고, 다른 서브넷에 존재하는 경우 방화벽설정을 변경해서 호스트와 타겟이 서로 통신할 수 있게 해야 한다. 

1. 타겟 컴퓨터의 제어판 > Network and Internet > Network Sharing Center 로가서 활성화된 네트워크를 확인한다. **Public network, Private Network** 또는 **Domain** 일거다. 
1. 타겟 컴퓨터의 제어판 > System and Security > Windows Firewall > Advanced Settings > Inbound Rules 로 간다. 
1. Inbound Rules 에서 활성화된 네트워크의 모든 Network Discovery 규칙들을 찾는다 (예를들어, Private 의 Profile 을 가진 모든 Network Discovery 규칙들을 찾는다). 각각의 룰을 더블클릭하고, **Scope** 탭을 열고, **Remote IP address** 의 **Any IP address** 를 선택한다. 
1. Inbound 규칙 리스트에서 활성화된 네트워크의 **File and Printer Sharing** 규칙들을 모두 찾는다.  각각의 룰을 더블클릭 > **Scope** 탭 > **Remote IP address** > **Any IP address** 를 선택한다.


### Provision the target computer
이제 호스트의 Visual Studio 에서 타겟 Provision 할 준비가 끝났다.

1. 호스트 Visual Studio > Driver > Test > Configure Devices > Add new device 를 선택
1. **Nework host name** 에 타겟의 이름을 입력하고, **Provision device and choose debugger settings** 를 선택한다.
![config](img/IC798034.png)
**Next** 를 선택한다.
1. debugging 타입을 선택하고, 필요한 파라미터들을 입력한다. 다양한 연결 타입에 대한 추가정보는 CHM 파일내 [Setting Up Kernel-Mode Debugging Visual Studio](https://msdn.microsoft.com/ko-kr/library/windows/hardware/hh439376(v=vs.85).aspx) 또는 온라인 문서 [Debugging Tools for Windows](https://msdn.microsoft.com/ko-kr/library/windows/hardware/ff551063(v=VS.85).aspx)를 참고해라.
1. provisioning 과정은 몇분정도 걸리고, 자동으로 한번 또는 두번 타겟을 리부팅한다. provisioning 이 완료되면 **Finish** 버튼을 클릭~!




## Deploying a Driver to a Test Computer 
원본 [Deploying a Driver to a Test Computer](https://msdn.microsoft.com/en-us/windows/hardware/drivers/develop/deploying-a-driver-to-a-test-computer)
Last Updated: 7/30/2016

Taking advantage of the Visual Studio development environment, the WDK provides a test feature that enables you to build, deploy, and debug a driver on a test computer. To successfully deploy a driver to a test system using the WDK, you must first set up and configure a test computer. You can set up and configure multiple computers if you want to test your driver under different testing scenarios.

### Setting up the test computer

Follow the instructions for Provision a computer for driver deployment and testing (WDK 10).
Note If you run into difficulties setting up the test computer, see Troubleshooting Configuration of Driver Deployment, Testing and Debugging.

### Setting deployment properties for your driver solution

From the property pages for your driver package, you have additional control over how you want your driver deployed for testing. You can choose to deploy the driver automatically whenever you build the driver solution in each configuration.

Open the property pages for your driver package. Right-click the driver package project in Solution Explorer and select Properties.
In the property pages for the driver package, click Configuration Properties, click Driver Install, and then click Deployment.
Select the Enable deployment option. When this option is selected, you must select a test computer that you have configured, or select the name of a computer that you want to configure for testing. See Provision a computer for driver deployment and testing (WDK 10).

When you enable deployment for your driver package project, the driver is automatically deployed to the test computer you have selected when you build your solution. You can use the Deployment property page to configure options for driver installation and deployment. See Deployment Properties for Driver Package Projects.

When you enable deployment on a test computer, you can also automatically enable and configure Driver Verifier, KMDF Verifier, or UMDF Verifier on the test computer to enhance the effectiveness of testing. To set these options for the driver package project, click Configuration Properties, click Driver Install, and then click the following property pages.

Driver Verifier Properties for Driver Package Projects
KMDF Verifier Properties for Driver Package Projects
UMDF Verifier Properties for Driver Package Projects
Building a driver and deploying the driver to test computer

Before you deploy your driver, make sure that you can build your driver solution. A driver solution must include the driver and driver package so that the driver can be installed on the test computer. For more information, see Creating a Driver Package and Building a Driver.
Before you deploy the driver to the test computer, you also need to sign the driver package. See Signing a Driver During Development and Testing.
Select the test computer that you have configured.
To deploy the driver, click Build Solution or Deploy Solution from the Build menu, or press F5 to build, deploy, and start debugging.
When you deploy a driver, the driver files are copied to the %Systemdrive%\drivertest\drivers folder on the test computer. If something goes wrong during deployment, you can check to see if the files are copied to the test computer. Verify that the .inf, .cat, test cert, and .sys files, and any other necessary files, are present %systemdrive%\drivertest\drivers folder.

Troubleshooting driver deployment

Here are some tips for troubleshooting driver deployment to a test computer when you use Visual Studio and the WDK.

Deployment fails due to Error code: 2

Add the following registry key:

HKLM\Software\Microsoft\DriverTest\Service

Under this key, create a DWORD value DebugSession, and set it to 0.

You only need to set this value once, and it persists for future deployments.

Can't find the deployment properties for the driver project
The deployment properties are only available if you have a driver package. If your driver solution does not have a driver package project, you need to add one. The driver package contains components, such as the INF file that are needed for installation. For more information, see Driver Packages and Creating a Driver Package.

Afer you have added the driver package, you can right-click the driver package project in Solution Explorer and select Properties. In the property pages for the driver package, click Configuration Properties, click Driver Install, and then click Deployment.

Problems selecting, configuring or locating the target computer
For instruction on how to set up the target computer, using Windows Driver Kit (WDK) 8.1 and Windows Driver Kit (WDK) 8, see Provision a computer for driver deployment and testing (WDK 10). If you have problems with provisioning the target computer, see Troubleshooting Configuration of Driver Deployment, Testing and Debugging.

If the target computer is running an N or KN version of Windows, you must install the Media Feature Pack for N and KN versions of Windows. See Provision a computer for driver deployment and testing (WDK 10) for more information.

Problems installing the driver on 64-bit version of Windows
Starting with Windows Vista, all 64-bit versions of Windows require driver code to have a digital signature for the driver to load. See Signing a Driver and Signing a Driver During Development and Testing.

Problems installing the driver (general)
The WDK can deploy and install a driver package on a test computer, but only if the driver has all the necessary components for installation, such as an INF file. See Driver Packages more information. Make sure you can install the driver outside of Visual Studio and the WDK. For example, use the Device Console utility, Devcon to test whether you can install the driver. Make sure the device (if you have one) is connected to the target computer. For more information, see Device and Driver Installation and Creating a Driver Package.
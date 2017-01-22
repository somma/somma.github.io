# vcpkg

https://github.com/Microsoft/vcpkg

## Quick start
- `c:\work.vcpkg` 에 clone 한다. 
- `C:\work.vcpkg> powershell -exec bypass scripts\bootstrap.ps1` 실행하면 `vcpkg.exe` 가 생성됨
- `vcpkg.exe install curl` 명령으로 패키지 인스톨
- `vcpkg integrate install` 명령으로 사용가능하게 함 (최초 실행시 관리자 권한 필요)

## 라이브러리 설치

+ `>vcpkg install package:triplet` 명령으로 설치 가능
    - 사용가능한 triplet 은 `vcpkg help triplet` 명령으로 확인할 수 있다.
    - arm-uwp / x64-uwp / x64-windows-static / x64-windows / x86-uwp / x86-windows-static / x86-windows

+ boost 라이브러리 설치 
    - x86 windows dynamic/static, x64 windows dynamic/static
    - `> vcpkg install boost:x86-windows boost:x86-windows-static boost:x64-windows boost:x64-windows-static`

## static link 설정
+ default 가 md (using dll) 로 링크되는데, 관련 설명은 있는데, 어떻게 하라는것인지 잘 모르겠음
    - [vcpkg-updates-static-linking-is-now-available](https://blogs.msdn.microsoft.com/vcblog/2016/11/01/vcpkg-updates-static-linking-is-now-available/)
    - [Disable automatic integration for a particuar VS project](https://github.com/Microsoft/vcpkg/issues/281)

+ `c:\work.vcpkg\scripts\buildsystems\msbuild\vcpkg.targets` 파일을 수정하는 방법
    - 프로젝트 빌드 과정에서 참조되는 파일로 `VcpkgEnabled`, `VcpkgTriplet` 변수를 적당히 수정해주면 됨
    - 전역 파일을 수정하는것이 맘에 들지 않음

+ 개별 프로젝트의 설정 `.vcxproj` 파일을 편집하는 방법
    `Global` 아래에 내용을 추가한다. 

    ```
    <PropertyGroup Label="Globals">
        <ProjectGuid>{0CB511FD-3E50-4548-A4F2-91D55B983656}</ProjectGuid>
        <Keyword>Win32Proj</Keyword>
        <RootNamespace>_MyLib_test</RootNamespace>
        <WindowsTargetPlatformVersion>8.1</WindowsTargetPlatformVersion>
    </PropertyGroup>

    <!-- vcpkg, static link 용 설정 -->
    <PropertyGroup Condition="'$(Platform)'=='Win32'">
        <VcpkgTriplet>x86-windows-static</VcpkgTriplet>    
        <VcpkgRoot>c:\work.vcpkg\installed\$(VcpkgTriplet)\</VcpkgRoot>
        <VcpkgEnabled>true</VcpkgEnabled>
    </PropertyGroup>
    <PropertyGroup Condition="'$(Platform)'=='x64'">
        <VcpkgTriplet>x64-windows-static</VcpkgTriplet>
        <VcpkgRoot>c:\work.vcpkg\installed\$(VcpkgTriplet)\</VcpkgRoot>
        <VcpkgEnabled>true</VcpkgEnabled>
    </PropertyGroup>

    ```



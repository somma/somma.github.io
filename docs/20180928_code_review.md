BoB 에서 프로젝트 멘토링을 하면서 코드 리뷰를 하다가, 기록을 남겨두면 좋을것 같아서 적어봅니다. 

아래 함수는 특정 레지스트리 키 경로를 입력받고, 키 내부의 값을 읽어서 출력하는 함수로 보이네요.

```
int Reg_Read(const char* subkey, TCHAR* value) {
	LONG ret;
	DWORD data_size = 1024;
	DWORD data_type;
	TCHAR* data_buffer = (TCHAR*)malloc(data_size);
	HKEY hKey;

	RtlZeroMemory(data_buffer, sizeof(data_buffer));

	ret = RegOpenKeyEx(HKEY_LOCAL_MACHINE,
		subkey,
		0, KEY_ALL_ACCESS, &hKey);

	if (ret != ERROR_SUCCESS) {
		printf("RegOpenKey Failed! \n ");
		return 0;
	}

	//memset(data_buffer, 0, sizeof(data_buffer));
	//data_size = sizeof(data_buffer);
	RegQueryValueEx(hKey, "UninstallString", 0, &data_type, (LPBYTE)data_buffer, (DWORD*)&data_size);
	RegCloseKey(hKey);
	value = data_buffer;
	printf("Value : %s\n", value);
	free(data_buffer);
	data_buffer = NULL;
	return 1;
}

int main() {
	TCHAR* value = nullptr;
	const char* subkey = "SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Uninstall\\{19DD1D8D-927F-45DF-ADF4-75D38267848D}";
	Reg_Read(subkey, value);
	
    printf("Value : %s\n", value);
	getchar();
}
```

몇 가지 고쳤으면 하는 점들이 보이네요.

+ 리턴 값의 의미가 모호하다.
+ 입력값 검증이 없다. 
+ 변수의 사용
    - 사용하는 시점에 선언하자.
    - 변수는 선언시 초기화하자.
+ 리소스 관리 
    - 할당한 메모리는 반드시 소멸하자. 
    - 시스템 리소스 (레지스트리 키 핸들 등)는 반드시 반환해야 한다.
+ 유니코드 함수와 안시코드 함수, 변수를 구분하자. 
    - char* 변수를 입력으로 받는 함수는 RegOpenKeyExA()
    - wchar* 변수를 입력으로 받는 함수는 RegOpenKeyExW()
    - 귀찮으면 TCHAR* 타입의 변수를 사용하고, RegOpenKeyEx()
    - 당연히 printf 도 _tprintf() 를 사용해야 합니다.
    - char*, wchar* 를 정확히 인지하지 않은상태에서 혼용하면 안됨
+ 에러로그를 남길때는 최소한 어떤 이유로 에러가 났는지 추적에 필요한 단서를 남기자.
    - 리턴값 또는 GetLastError() 코드 정도는 기록하자.
    - 에러로그는 에러 추적을 위해 남기는 것이다.
+ 리턴값이 있는 외부 함수를 호출했으면 반드시 리턴 값을 확인하자.
+ 함수 설계 상의 문제
    - value 값이 output 파라미터로 사용되고 있으나
    - value 값의 타입을 어디에서도 정확히 명시 하지 않고 있다(string 인지, dword 인지).
    - 전체적인 느낌으로는 string 타입이라고 가정하고 작성한 것 같네요.
+ 아주 심각한 문제인데, 소멸된 메모리에 접근하는 문제가 있습니다.
    - out-parameter 로 사용된 value 가 가리키는 포인터는 이미 소멸되었습니다. 

정도가 될 것 같습니다. 휴~

```
int Reg_Read(const char* subkey, TCHAR* value) {
	LONG ret;
	DWORD data_size = 1024;
	DWORD data_type;
	TCHAR* data_buffer = (TCHAR*)malloc(data_size);
	HKEY hKey;

	RtlZeroMemory(data_buffer, sizeof(data_buffer));

	ret = RegOpenKeyEx(HKEY_LOCAL_MACHINE,
		subkey,
		0, KEY_ALL_ACCESS, &hKey);

	if (ret != ERROR_SUCCESS) {
		printf("RegOpenKey Failed! \n ");

        //
        //  이렇게 리턴해버리면 data_buffer 에 할당한 메모리는 해제하지 못하고
        //  프로그램이 종료될 때 까지 자원만 차지하고 있게됩니다.
        //  이건 아주 심각한 문제입니다. 
        //
		return 0;
	}

	//memset(data_buffer, 0, sizeof(data_buffer));
	//data_size = sizeof(data_buffer);
	RegQueryValueEx(hKey, "UninstallString", 0, &data_type, (LPBYTE)data_buffer, (DWORD*)&data_size);
	RegCloseKey(hKey);

    //
    // value 에 포인터를 할당해 두고, free 해버리면 value 는 
    // 더이상 유효한 메모리를 가리키고 있지 않게됩니다. 
    //
    value = data_buffer;
	printf("Value : %s\n", value);
	free(data_buffer);
	data_buffer = NULL;

	return 1;
}

int main() {

	TCHAR* value = nullptr;
	const char* subkey = "SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Uninstall\\{19DD1D8D-927F-45DF-ADF4-75D38267848D}";

    //
    //  Reg_Read() 가 실패했다면 value 는 여전히 nullptr 입니다. 
    //
	Reg_Read(subkey, value);
	
    //
    // value 는 
    //      - Reg_Read() 가 성공했다면 이미 유효한 포인터가 아니므로 펑펑펑~~
    //      - Reg_Read() 가 실패했다면 nullptr 이니까 당연히 펑펑펑~~
    // 어쨌거나 펑펑펑~ ㅅㅂ
    //
    printf("Value : %s\n", value);
	getchar();

    //
    //  리턴값도 없어요. 아마 컴파일러에서 에러를 뿜었을텐데...(컴파일도 안해본건가요...ㅠㅠ)?
    //
}
```

고쳐볼까요?

```
#include "stdafx.h"
#define WIN32_LEAN_AND_MEAN
#include <windows.h>
#include <memory>
#include <crtdbg.h>
#include <sal.h>


/// @brief	subkey 내부 value 값을 읽어 리턴한다. 
/// @param	subkey	레지스트리 키
/// @param	value	읽을 대상 키 이름
/// @return	성공시 true 를 리턴하고, value 포인터에 읽은 문자열 포인터를 할당
///			호출자는 반드시 value 포인터를 소멸시켜주어야 함
bool 
Reg_Read(
	_In_z_ const TCHAR* subkey,
	_Outptr_ TCHAR* value
)
{
	_ASSERTE(nullptr != subkey);
	_ASSERTE(nullptr == value);
	if (nullptr == subkey || nullptr != value) return false;

	HKEY hKey = NULL;
	DWORD ret = RegOpenKeyEx(HKEY_LOCAL_MACHINE,
							 subkey,
							 0, 
							 KEY_ALL_ACCESS, 
							 &hKey);

	if (ret != ERROR_SUCCESS) 
	{
		_tprintf(_T("RegOpenKey Failed!, ret=0x%08x\n"), ret);
		return false;
	}

    //
    //  여기서는 절대로 hKey 가 NULL 이 아니어야 합니다.
    //
    _ASSERTE(NULL != hKey);

	//
	//	이제 앞으로 어떤 일이 발생하든지간에 hKey 는 반드시 
	//	RegCloseHandle() 로 닫아주어야 합니다.
	//

	DWORD data_size = 1024;
	TCHAR* data_buffer = (TCHAR*)malloc(data_size);
    if (nullptr == data_buffer)
	{
		_tprintf(_T("Not enough memory \n"));

		RegCloseKey(hKey);
		return false;
	}
    _ASSERTE(nullptr != data_buffer);
	RtlZeroMemory(data_buffer, sizeof(data_buffer));

	DWORD data_type = REG_NONE;
	ret = RegQueryValueEx(hKey, 
						  _T("UninstallString"), 
						  0, 
						  &data_type, 
						  (LPBYTE)data_buffer, 
						  (DWORD*)&data_size);
	if (ret != ERROR_SUCCESS)
	{
		_tprintf(_T("RegQueryValueEx Failed!, ret=0x%08x\n"), ret);

		//
		//	잊지 말고 핸들과 메모리를 소멸시켜 주어야 합니다.
		//
		free(data_buffer);
		RegCloseKey(hKey);
		return false;
	}

	//
	// value 가 string 타입이 아니라면 문제가 있을 수 있으니
	// 확인해주어야 합니다.
	//
	if (data_type != REG_SZ || data_type != REG_MULTI_SZ || data_type != REG_EXPAND_SZ)
	{
		_tprintf(_T("Type of value is not string. actual type=%u"), data_type);

		//
		//	역시 잊지 말고 핸들과 메모리를 소멸시켜 주어야 합니다.
		//
		free(data_buffer);
		RegCloseKey(hKey);
		return false;
	}

	//
	//	역시 잊지 말고 핸들을 닫아주어야지요.
	//
	RegCloseKey(hKey);

	//
	// data_buffer 는 free 하면 안됍니다.
	// 
	value = data_buffer;
	//free(data_buffer);	
	_tprintf(_T("Value : %s\n"), value);
	data_buffer = NULL;

	return true;
}

int main()
{
	TCHAR* value = nullptr;
	const TCHAR* subkey = _T("SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Uninstall\\{19DD1D8D-927F-45DF-ADF4-75D38267848D}");
	if (true != Reg_Read(subkey, value))
	{
		_tprintf(_T("Reg_Read() failed. \n"));
		return -1;
	}

	//
	// 여기까지 왔다면 value 는 절대 nullptr 이 아니어야 합니다 .
	// 그리고, value 는 잊지 말고 소멸시켜주어야 합니다. 
	//
	_ASSERTE(nullptr != value);
	_tprintf(_T("Value : %s\n"), value);
	free(value);
	getchar();

    return 0;
}
```

수정된 내용을 보면 

+ [sal notation](https://docs.microsoft.com/en-us/visualstudio/code-quality/understanding-sal?view=vs-2017) 적용
    - 코드를 읽는데 도움이 되는것 뿐만 아니라 컴파일러가 오류도 잡아줍니다. 
    - 강추! 강추! Visual Studio 를 사용한다면 무조건 쓰세요.
+ _ASSERT 코드 (아무리 강조해도 지나치지 않습니다.)
+ 리턴값 체크
+ 변수 선언은 최대한 늦게, 생성즉시 초기화하기
+ char, wchar_t 를 TCHAR 로 통일
+ 리소스 릭 제거 (레지스트리 핸들, 메모리)
+ 레지스트리 value 타입 확인
+ value 포인터 잘 다루기 (ㅠ,.ㅠ)
+ 등등...

정도 입니다. 

하지만 아직 많은 문제가 있습니다. 

+ 실수하기 너무 쉬운 코드
    - memory 소멸을 여러군데에서 하고있고, 성공하는 경우 호출자가 value 메모리를 해제해야 합니다. 
    - 코드가 길어지거나, 로직이 더 복잡해 지거나, 다른 사람이 코드를 수정한다면 실수하기 딱 좋은 코드입니다. 

+ RegQueryValueEx() 함수의 사용
    - 현재는 1024 바이트로 고정된 버퍼를 사용합니다. 
    - 더 긴 문자열을 읽어야 한다면? 

그래서 한번 더 고쳤습니다. 


```
#include "stdafx.h"
#define WIN32_LEAN_AND_MEAN
#include <windows.h>
#include <memory>
#include <crtdbg.h>
#include <sal.h>


class SafeRegHandle
{
public:
	SafeRegHandle(_In_ HKEY key) : _key(key)
	{
		_ASSERTE(NULL != key);
	}

	~SafeRegHandle()
	{
		_ASSERTE(NULL != _key);
		RegCloseKey(_key);
	}
private:
	HKEY _key;
};


/// @brief	subkey 내부 value 값을 읽어 리턴한다. 
/// @param	subkey	레지스트리 키
/// @param	value	읽을 대상 키 이름
/// @return	성공시 true 를 리턴하고, value 포인터에 읽은 문자열 포인터를 할당
///			호출자는 반드시 value 포인터를 소멸시켜주어야 함
bool 
Reg_Read(
	_In_z_ const TCHAR* subkey,
	_Outptr_ TCHAR* value
)
{
	_ASSERTE(nullptr != subkey);
	_ASSERTE(nullptr == value);
	if (nullptr == subkey || nullptr != value) return false;

	HKEY hKey = NULL;
	DWORD ret = RegOpenKeyEx(HKEY_LOCAL_MACHINE,
							 subkey,
							 0,
							 KEY_ALL_ACCESS,
							 &hKey);

	if (ret != ERROR_SUCCESS)
	{
		_tprintf(_T("RegOpenKey Failed!, ret=0x%08x\n"), ret);
		return false;
	}
	_ASSERTE(NULL != hKey);

	//
	// SafeRegHandle 객체는 함수가 리턴되는 시점에 알아서 소멸자가 호출되고
	// hKey 는 SafeRegHandle 클래스의 소멸자에서 닫아줍니다. 
	// 따라서 이제 hKey는 잊고 살면 됩니다. 
	//
	SafeRegHandle safe_key(hKey);

	//
	// RegQueryValueEx() 함수는 입력받은 버퍼가 부족한 경우 ERROR_MORE_DATA 를 리턴합니다. 
	// 만일 입력받은 버퍼가 NULL 을 입력한 경우 필요한 사이즈를 data_size 변수에 바이트 단위로
	// 설정하고 ERROR_SUCCESS 를 리턴합니다. 
	// 
	// 참고로 가변길이의 데이터를 다루는 대부분의 Windows API 가 이런 패턴을 보입니다. 
	//
	// 따라서 
	//	1) RegQueryValueEx( NULL, size_need ) 호출로 필요한 메모리 크기를 구하고
	//	2) 메모리를 할당하고, 
	//  3) RegQueryValueEx() 를 호출
	// 데이터를 읽어옵니다. 
	//
	DWORD data_type = REG_NONE;
	DWORD data_size = 0;
	char* data_buffer = nullptr;

	// 필요한 사이즈 구하기 
	ret = RegQueryValueEx(hKey,
						  _T("UninstallString"),
						  0,
						  &data_type,
						  (LPBYTE)data_buffer,
						  (DWORD*)&data_size);
	if (ret != ERROR_SUCCESS)
	{
		_tprintf(_T("RegQueryValueEx Failed!, ret=0x%08x\n"), ret);
		return false;
	}

	// null terminate 를 위한 여분의 메모리를 추가 할당
	data_buffer = (char*)malloc(data_size + sizeof(TCHAR));
	if (nullptr == data_buffer)
	{
		_tprintf(_T("Not enough memory \n"));
		return false;
	}
	_ASSERTE(nullptr != data_buffer);

	// 데이터를 읽습니다. 
	ret = RegQueryValueEx(hKey,
						  _T("UninstallString"),
						  0,
						  &data_type,
						  (LPBYTE)data_buffer,
						  (DWORD*)&data_size);
	if (ret != ERROR_SUCCESS)
	{
		_tprintf(_T("RegQueryValueEx Failed!, ret=0x%08x\n"), ret);

		//
		// 잊지 말고, 메모리를 소멸시켜 주어야 합니다.
		//
		free(data_buffer);
		return false;
	}

	//
	// value 가 string 타입이 아니라면 문제가 있을 수 있으니
	// 확인해주어야 합니다.
	//
	if (data_type != REG_SZ || data_type != REG_MULTI_SZ || data_type != REG_EXPAND_SZ)
	{
		_tprintf(_T("Type of value is not string. actual type=%u"), data_type);

		//
		// 잊지 말고, 메모리를 소멸시켜 주어야 합니다.
		//
		free(data_buffer);
		return false;
	}

	//
	// data_buffer 는 free 하면 안됍니다.
	// 
	value = (TCHAR*)data_buffer;
	_tprintf(_T("Value : %s\n"), value);
	data_buffer = NULL;

	return true;
}

int main()
{
	TCHAR* value = nullptr;
	const TCHAR* subkey = _T("SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Uninstall\\{19DD1D8D-927F-45DF-ADF4-75D38267848D}");
	if (true != Reg_Read(subkey, value))
	{
		_tprintf(_T("Reg_Read() failed. \n"));
		return -1;
	}

	//
	// 여기까지 왔다면 value 는 절대 nullptr 이 아니어야 합니다 .
	// 그리고, value 는 잊지 말고 소멸시켜주어야 합니다. 
	//
	_ASSERTE(nullptr != value);
	_tprintf(_T("Value : %s\n"), value);
	free(value);
	getchar();

    return 0;
}
```

다시 한번 수정 된 내용을 보면 

+ SafeRegHandle 클래스 추가
    - 이건 아주 자주 사용되는 패턴인데, 아래 내용들을 참고하시면...
    - [Resource acquisition is initialization](https://en.wikipedia.org/wiki/Resource_acquisition_is_initialization)
    - [Smart pointer](https://en.wikipedia.org/wiki/Smart_pointer)

+ RegQueryValueEx() 함수 사용 패턴 변화
    - Windows API 사용시 아주 기본적인 사용 패턴이니까 익숙해지도록...

더 고치고 싶은것은 있지만, 이쯤에서 마무리 해야겠습니다. 

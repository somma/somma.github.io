

이번 포스트에서는 [CreateFile()](https://msdn.microsoft.com/en-us/library/windows/desktop/aa363858(v=vs.85).aspx) API 두번째, 세번째 파라미터인 `dwDesiredAccess`, `dwShareMode` 에 대해서 정확히 파악해보고자 합니다.

먼저 퀴즈로 시작해 보겠습니다. 

A 라는 프로세스는 `abc.log` 라른 파일에 데이터를 쓰고, B 라는 프로세스는 해당 파일을 읽어서 화면에 출력하는 코드를 작성한다고 가정합니다.
A 프로세스는 `abc.log` 파일에 로그를 쓰기 위해 `FILE_GENERIC_WRITE` 접근 권한을 요청하고, B 프로세스가 해당 로그파일을 읽을 수 있도록 `FILE_SHARE_READ` 공유권한을 넘겨야 한다고 생각하시겠죠?
( A 프로세스가 먼저 실행된다고 가정합니다 )


```c
// A 프로세스의 코드
HANDLE fileHandle = CreateFile('abc.log',
                               FILE_WRITE_DATA,     // 로그를 쓰려면 쓰기 권한으로 파일을 열어야죠.
                               FILE_SHARE_READ,     // B 프로세스가 해당 파일을 읽어야 하니까 `공유 읽기` 를.
                               ...
                               ...);


// B 프로세스의 코드
HANDLE fileHandle = CreateFile('abc.log',
                               FILE_READ_DATA,      // 로그 파일을 읽어야 하니까
                               FILE_SHARE_READ,     // A 프로세스의 코드랑 그냥 같은걸로???
                               ...
                               ...);
```

..

..

..

..

..

..

..

이 코드는 생각대로, 잘 동작하지 않습니다. 바로 무엇이 문제인지 대답하실 수 있다면 더이상 읽지 않으셔도 됩니다.

..

..

..

..

..

..

..



[CreateFile()](https://msdn.microsoft.com/en-us/library/windows/desktop/aa363858(v=vs.85).aspx) 함수는 Windows API 에서 가장 많이 사용되는 API 중에 하나이고, 가장 어렵고, 가장 많은것을 알아야 하는 API 중 하나입니다.

```c
HANDLE WINAPI CreateFile(
  _In_     LPCTSTR               lpFileName,
  _In_     DWORD                 dwDesiredAccess,
  _In_     DWORD                 dwShareMode,
  _In_opt_ LPSECURITY_ATTRIBUTES lpSecurityAttributes,
  _In_     DWORD                 dwCreationDisposition,
  _In_     DWORD                 dwFlagsAndAttributes,
  _In_opt_ HANDLE                hTemplateFile
);
```


두번째 `dwDesiredAccess` 는 해당 파일에 어떤 권한으로 접근을 하려고 하는지를 명시합니다 (읽기를 할것인지, 쓰기를 할것인지, 삭제를 할것인지). 세번째 `dwShareMode` 는 해당 파일에 접근하고있는 동안 다른 프로세스가 해당 파일에 접근하고자 하는 경우 어떤 권한을 허용할지를 명시합니다 (읽기를 허용할지, 쓰기를 허용할지, 삭제를 용할지). 

Windows Kernel 은 CreateFile 요청이 발생하면 다양하고, 복잡한 함수호출을 통해서 특정 파일에 대한 오브젝트(FILE_OBJECT)를 생성하거나 이미 생성된 FILE_OBJECT 에 접근합니다. 
FILE_OBJECT 는 Windows 커널이 파일을 표현하기 위해서 커널내부적으로 사용하는 자료구조이며, 특정프로세스에 종속되지 않는 커널전체에서 공유되는 자원입니다. 매번 파일을 열고자 할때마다 새롭게 FILE_OBJECT 를 생성하면 효율적이지 못할테니까요.
응용프로그램이 사용하는 HANDLE 은 Windows 커널이 관리하는 오브젝트 배열의 인덱스 값 같은 것입니다. 이에 대한 자세한 내용은 [Windows Internals](https://technet.microsoft.com/en-us/sysinternals/bb963901.aspx) 또는 제 [예전 블로그 포스트](http://egloos.zum.com/somma/v/2947301)를 참고하시면 도움이 될것 같습니다.

커널은 특정 오브젝트로의 접근 요청을 받으면 해당 오브젝트에 접근이 가능한 권한이 있는 지 확인하는 과정을 거치는데, 파일의 경우 추가적으로 요청권한 및 공유권한을 확인하는 과정을 거칩니다. 
이 공유위반이 발생하면 `ERROR_SHARING_VIOLATION` 에러코드를 리턴하게 됩니다.

Windows Kernel 이 FILE_OBJECT 에 요청된 접근권한(`dwDesiredAccess`)과 공유권한(`dwShareMode`)가 유효한지 판단하는 과정을 자세히 확인해보겠습니다.

FILE_OBJECT 는 매우 복잡하고, 알아야 할것도 많고, 모르는것도 굉장히 많지만, 우리에게 필요한 필드는 아래와 같습니다.

```text
kd> dt nt!_file_object
   ...
   +0x04a ReadAccess       : UChar
   +0x04b WriteAccess      : UChar
   +0x04c DeleteAccess     : UChar
   +0x04d SharedRead       : UChar
   +0x04e SharedWrite      : UChar
   +0x04f SharedDelete     : UChar
   ...
```

별다른 설명이 없더라도 `dwDesiredAccess` 와 `dwShareMode` 인걸 알 수 있을 것입니다. FILE_OBJECT 의 값들과 우리가 입력한 파라미터가 매칭되는 방식은 아래 코드와 같습니다. 

```c
FILE_OBJECT.ReadAccess = (BOOLEAN) ((DesiredAccess & (FILE_EXECUTE| FILE_READ_DATA)) != 0);
FILE_OBJECT.WriteAccess = (BOOLEAN) ((DesiredAccess & (FILE_WRITE_DATA | FILE_APPEND_DATA)) != 0);
FILE_OBJECT.DeleteAccess = (BOOLEAN) ((DesiredAccess & DELETE) != 0);
...
FILE_OBJECT.SharedRead = (BOOLEAN) ((DesiredShareAccess & FILE_SHARE_READ) != 0);
FILE_OBJECT.SharedWrite = (BOOLEAN) ((DesiredShareAccess & FILE_SHARE_WRITE) != 0);
FILE_OBJECT.SharedDelete = (BOOLEAN) ((DesiredShareAccess & FILE_SHARE_DELETE) != 0);
```

Windows kernel 은 유효성 여부를 검사와 해당 FILE_OBJECT 에 대한 요청들을 추적하기 위해서 `SHARE_ACCESS`라는 자료구조를 사용하는데, windbg 를 통해서 확인할 수 있습니다. FILE_OBJECT 접근에 성공하면 각 필드를 업데이트 합니다. 

```text
kd> dt nt!share_access
   +0x000 OpenCount        : Uint4B
   +0x004 Readers          : Uint4B
   +0x008 Writers          : Uint4B
   +0x00c Deleters         : Uint4B
   +0x010 SharedRead       : Uint4B
   +0x014 SharedWrite      : Uint4B
   +0x018 SharedDelete     : Uint4B
```

C 코드로 변경해보면 아래와 같고, 각각의 의미는 코드의 주석으로 표현해 두었습니다.

```c
typedef struct _SHARE_ACCESS
{
    uint32_t OpenCount;     // 성공한 파일 열기 횟수

    uint32_t Readers;       // 성공한 읽기 권한 횟수
    uint32_t Writers;       // 성공한 쓰기 권한 횟수
    uint32_t Deleters;      // 성공한 삭제 권한 횟수

    uint32_t SharedRead;    // 성공한 공유 읽기 권한 횟수
    uint32_t SharedWrite;   // 성공한 공유 쓰기 권한 횟수
    uint32_t SharedDelete;  // 성공한 공유 삭제 권한 횟수
} SHARE_ACCESS;
```

`SHARE_ACCESS` 와 `dwDesiredAccess`, `dwShareMode` 간의 관계는 아래와 같이 정리 할 수 있습니다. 

```text
[규칙 #1]
`SHARE_ACCESS.Shared[Read|Write|Delete]` 가 OpenCount 보다 작고,
`[Read|Write|Delete]` 권한을 요청하는 경우 SHARE VIOLATION.

기존에 생성된 공유권한이 없다면 공유권한과 매칭되는 접근권한을 요청 할 수 없습니다.
예를 들어 이전 호출 중 `FILE_SHARE_READ` 공유권한 요청이 없었다면, `FILE_READ_DATA` 접근 권한을 요청할 수 없다.

[규칙 #2]
`SHARE_ACCESS.[Readers|Writers|Deleters]` 가 0 이 아닌데,
요청 된 공유권한이 `FILE_SHARE_[READ|WRITE|DELETE]` 가 0 인 경우 SHARE VIOLATION.

기존에 요청된 접근권한과 매칭되는 공유권한이 현재 요청에 없다면 요청에 실패한다.
예를 들어 이전 호출 중 `FILE_READ_DATA` 접근권한 요청이 있었다면, 현재 요청에는 반드시 `FILE_SHARE_READ` 공유권한이 포함되어 있어야 한다.
```

앞에서 보여드렸던 아래의 코드는 `ERROR_SHARING_VIOLATION` 를 리턴하게 됩니다.

```c
// A 프로세스의 코드
HANDLE fileHandle = CreateFile('abc.log',
                               FILE_WRITE_DATA,     // 로그를 쓰려면 쓰기 권한으로 파일을 열어야죠.
                               FILE_SHARE_READ,     // B 프로세스가 해당 파일을 읽어야 하니까 `공유 읽기` 를.
                               ...
                               ...);

// B 프로세스의 코드
HANDLE fileHandle = CreateFile('abc.log',
                               FILE_READ_DATA,      // 로그 파일을 읽어야 하니까
                               FILE_SHARE_READ,     // A 프로세스의 코드랑 그냥 같은걸로???
                               ...
                               ...);
```

왜 그럴까요?

A 프로세스의 코드가 실행되면 해당 FILE_OBJECT 의 상태는 아래와 같습니다. 

```text
SHARE_ACCESS.OpenCount = 1;

SHARE_ACCESS.Readers = 0;
SHARE_ACCESS.Writers = 1;       // FILE_GENERIC_WRITE 를 지정했으니까.
SHARE_ACCESS.Deleters = 0;

SHARE_ACCESS.SharedRead = 1;    // FILE_SHARE_READ 를 정했으니까.
SHARE_ACCESS.SharedWrite = 0;
SHARE_ACCESS.SharedDelete = 0;
```

B 프로세스의 코드가 실행되는 시점에 공유위반 검사를 해보면

+ 규칙 #1, `SHARE_ACCESS.SharedRead < SHARE_ACCESS.OpenCount` 조건이 거짓이므로 공유위반 아님
+ 규칙 #2, `SHARE_ACCESS.Writers != 0` 이고, 요청된 공유모드가 FILE_SHARE_WRITE 가 0 입니다. (FILE_SHARE_READ 만 지정했으니까요)

즉 [규칙 #2] 에 의해서 공유위반입니다. 

따라서 위의 코드는 아래처럼 작성해야 합니다. 

```c
// A 프로세스의 코드
HANDLE fileHandle = CreateFile('abc.log',
                               FILE_WRITE_DATA,     // 로그를 쓰려면 쓰기 권한으로 파일을 열어야죠.
                               FILE_SHARE_READ,     // B 프로세스가 해당 파일을 읽어야 하니까 `공유 읽기` 를.
                               ...
                               ...);

// B 프로세스의 코드
HANDLE fileHandle = CreateFile('abc.log',
                               FILE_READ_DATA,      // 로그 파일을 읽어야 하니까
                               FILE_SHARE_READ | FILE_SHARE_WRITE,
                               ...
                               ...);
```

사실 상식적으로 생각해보면 굉장히 당연한 규칙입니다만, 코드를 말로 풀어서 설명하다보니 쓸데없이 길어진 것 같습니다. 의외로 예제로 보여드린 코드와 유사한 실수를 저지르는 경우가 많고, 정확히 규칙을 모르는 분들이 많은것 같습니다 (저도 정확히 모르고 대충 썼습니다).

매번 정확히 한번 따져봐야지 생각만 하고 미루다가 드디어 정확히 알게된것 같아 기쁩니다 ^__^

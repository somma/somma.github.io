

이번 포스트에서는 [CreateFile()](https://msdn.microsoft.com/en-us/library/windows/desktop/aa363858(v=vs.85).aspx) API 두번째, 세번째 파라미터에 대해서 설명하고자 합니다.

먼저 퀴즈 나갑니다.
A 라는 프로세스는 `abc.log` 라른 파일에 데이터를 쓰고, B 라는 프로세스는 해당 파일을 읽어서 화면에 출력하는 코드를 작성한다고 가정합니다.
A 프로세스는 `abc.log` 파일에 로그를 쓰기 위해 `FILE_GENERIC_WRITE` 접근 권한을 요청하고, B 프로세스가 해당 로그파일을 읽을 수 있도록 `FILE_SHARE_READ` 공유권한을 허용해야 하는건 다들 알고계시겠죠?
( A 프로세스가 먼저 실행된다고 가정합니다 )


A 프로세스의 코드입니다. 
```c
HANDLE fileHandle = CreateFile('abc.log', 
                               FILE_GENERIC_WRITE,  // 로그를 쓰려면 쓰기 권한으로 파일을 열어야죠.
                               FILE_SHARE_READ,     // B 프로세스가 해당 파일을 읽어야 하니까 `공유 읽기` 를.
                               ...
                               ...);
```


B 프로세스의 코드입니다. 
```c
HANDLE fileHandle = CreateFile('abc.log', 
                               FILE_GENERIC_READ,   // 로그 파일을 읽어야 하니까
                               FILE_SHARE_READ,     // A 프로세스의 코드랑 그냥 같은걸로???
                               ...
                               ...);
```

자, 이 코드가 잘 동작할까요? 바로 무엇이 문제인지 대답하실 수 있다면 더이상 읽지 않으셔도 됩니다. 

..

..

..

..

..

..

[CreateFile()](https://msdn.microsoft.com/en-us/library/windows/desktop/aa363858(v=vs.85).aspx) 함수는 Windows API 에서 가장 많이 사용되는 API 중에 하나이고, 가장 어렵고, 가장 많은것을 알아야 하는 API 중 하나입니다. 이 함수는 여러개의 파라미터를 입력으로 받는데, 그 중 두번째와 세번째 파라미터에 대해서 설명하려고 합니다.

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

이게 별것 아닌것 처럼 보이지만, 자칫하면 실수하기 쉽기때문에 오늘 확실하게 정리하죠.

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

Windows kernel 은 유효성 여부를 검사와 해당 FILE_OBJECT 에 대한 요청들을 추적하기 위해서 `SHARE_ACCESS`라는 자료구조를 사용하는데, windbg 를 통해서 확인할 수 있습니다. 

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
규칙 #1 

`SHARE_ACCESS.Shared[Read|Write|Delete]` 가 OpenCount 보다 작고, `[Read|Write|Delete]` 권한을 요청하는 경우 SHARE VIOLATION.
```

    기존에 생성된 ShareAccess 가 없다면 ShareAccess 와 일치하는 Access 요청을 할 수 없다.     
    예를 들어 이전 호출 중 성공한 `FILE_SHARE_READ` 가 없었다면 `FILE_READ_DATA` 요청을 할 수 없다.

```text
규칙 #2 

`SHARE_ACCESS.[Readers|Writers|Deleters]` 가 0 이 아니고, `[ShareRead|ShareWrite|ShareDelte]` 가 0 이 아닌경우 SHARE VIOLATION.
```
    기존에 생성된 ShareAccess 가 있는데, 해당 ShareAccess 가 현재 요청에 없다면 요청에 실패한다.    
    예를 들어 이전 호출 중 성공한 `FILE_SHARE_READ` 가 있는데, 현재 요청에 `FILE_SHARE_READ` 가 없다면 공유 위반이다. 

앞의 퀴즈를 다시 보면 이해가 되실 겁니다. 

아래의 코드는 `ERROR_SHARING_VIOLATION` 를 리턴하게 될 것입니다.

```c
// A 프로세스의 코드
HANDLE fileHandle = CreateFile('abc.log',
                               FILE_GENERIC_WRITE,  // 로그를 쓰려면 쓰기 권한으로 파일을 열어야죠.
                               FILE_SHARE_READ,     // B 프로세스가 해당 파일을 읽어야 하니까 `공유 읽기` 를.
                               ...
                               ...);

// B 프로세스의 코드입니다. 
HANDLE fileHandle = CreateFile('abc.log',
                               FILE_GENERIC_READ,   // 로그 파일을 읽어야 하니까
                               FILE_SHARE_READ,     // A 프로세스의 코드랑 그냥 같은걸로???
                               ...
                               ...);
```

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

B 프로세스가 실행되는 시점에 공유위반 검사를 해보면 

1. `SHARE_ACCESS.SharedRead < 0` 조건이 거짓이고, `FILE_GENERIC_READ` 권한을 요청했으므로 공유위반이 아닙니다. 
2. 



shadow handle 을 생성하고자 하는 경우 `ShareRead|ShareWrite|ShareDelete` 공유권한으로 파일을 열어두면 이후 모든 공유권한으로 요청이 와도 성공할 거라고 착각 할 수 있는데, 실제는 그렇지 않다. 
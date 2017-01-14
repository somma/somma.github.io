
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

```c
FILE_OBJECT.ReadAccess = (BOOLEAN) ((DesiredAccess & (FILE_EXECUTE| FILE_READ_DATA)) != 0);
FILE_OBJECT.WriteAccess = (BOOLEAN) ((DesiredAccess & (FILE_WRITE_DATA | FILE_APPEND_DATA)) != 0);
FILE_OBJECT.DeleteAccess = (BOOLEAN) ((DesiredAccess & DELETE) != 0);
...
FILE_OBJECT.SharedRead = (BOOLEAN) ((DesiredShareAccess & FILE_SHARE_READ) != 0);
FILE_OBJECT.SharedWrite = (BOOLEAN) ((DesiredShareAccess & FILE_SHARE_WRITE) != 0);
FILE_OBJECT.SharedDelete = (BOOLEAN) ((DesiredShareAccess & FILE_SHARE_DELETE) != 0);
```

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

1. `[ShareRead|ShareWrite|ShareDelete]` 가 OpenCount 보다 작고, `[Read|Write|Delete]` 권한을 요청하는 경우.
기존에 생성된 ShareAccess 가 없다면 ShareAccess 와 일치하는 Access 요청을 할 수 없다. 예를 들어 이전 호출 중 성공한 `FILE_SHARE_READ` 가 없었다면 `FILE_READ_DATA` 요청을 할 수 없다.

1. `[Readers|Writers|Deleters]` 가 0 이 아니고, `[ShareRead|ShareWrite|ShareDelte]` 가 0 이 아닌경우.
기존에 생성된 ShareAccess 가 있는데, 해당 ShareAccess 가 현재 요청에 없다면 요청에 실패한다.예를 들어 이전 호출 중 성공한 `FILE_SHARE_READ` 가 있는데, 현재 요청에 `FILE_SHARE_READ` 가 없다면 공유 위반이다. 

shadow handle 을 생성하고자 하는 경우 `ShareRead|ShareWrite|ShareDelete` 공유권한으로 파일을 열어두면 이후 모든 공유권한으로 요청이 와도 성공할 거라고 착각 할 수 있는데, 실제는 그렇지 않다. 
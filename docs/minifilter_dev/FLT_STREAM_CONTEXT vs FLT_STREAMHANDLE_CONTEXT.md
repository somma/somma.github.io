# FLT_STREAM_CONTEXT 와 FLT_STREAMHANDLE_CONTEXT

+ STREAM_CONTEXT
    - file stream 마다 붙일 수 있는 context. 
    - FILE_OBJECT.FsContext 를 추적하는데 사용. 

+ STREAMHANDLE_CONTEXT
    - 개개의 file open 시 생성되는 file object (I/O 서브시스템이 생성하는)마다 붙일 수 있는 context. 
    - FILE_OBJECT 를 추적하는데 사용.

## 참고1
+ [osronline](http://www.osronline.com/showThread.cfm?link=66033)

## 참고2
+ [MSDN 원문, File Streams, Stream Contexts, and Per-Stream Contexts](https://msdn.microsoft.com/windows/hardware/drivers/ifs/file-streams--stream-contexts--and-per-stream-contexts)

+ file stream

    파일 데이터를 저장하는데 사용되는 바이트 시퀀스이다. 
    보통 파일은 하나의 file stream 을 가지는데, 요걸 파일의 default data stream 이라고 한다. 
    그러나 multiple data stream 을 지원하는 파일시스템에서는 각각의 파일은 여러개의 file stream 을 가질 수 있다. 
    그 중 하나는 default data stream 이고, 얘는 unnamed 이다. 
    다른 놈들은 alternate data stream 이다. 파일을 열면, 실제로는 해당 파일의 스트림을 여는 것이다.

    file system 이 최초로 파일을 열때, ``file control block(FCB)`` 이나 ``stream control block (SCB)`` 같은 
    file-system-specific stream context 구조체를 생성하고, 이 구조체의 주소를 file object 의 `FsContext` 멤버에 저장한다. 

    로컬 파일시스템에서, 이미 열려있는 file streeam 이 다시 열리면 (shared read access 같은 경우로 인해), 
    I/O 서브시스템은 새로운 file object 를 생성하지만, file system 은 새로운 stream context 를 생성하지는 않는다. 
    따라서 로컬 파일시스템에서는 stream context pointer 는 file stream 을 식별하는 유니크한 값으로 사용될 수 있다. 

    per-stream context 를 지원하는 네트워크 파일시스템에서는 이미 열려있는 file stream 이 동일한 network share name 이나 IP 주소 등을 
    이용해서 다시 열린다면 로컬 파일시스템과 동일하게 동작한다. 
    I/O 서브시스템은 새로운 file object 를 생성하지만 file system 은 새로운 stream context 를 생성하지 않고, 
    두 file object 에 동일한 `FsContext` 포인터를 할당한다. 

    그러나 file stream 이 서로 다른 경로로 열리는 경우 (다른 share name 또는 IP 가 다른 경우), file system 은 
    새로운 file stream context 를 생성한다. 
    그러므로 per-stream context 를 지원하는 네트워크 파일시스템에서는 `FsContext` 는 file stream 을 구분하는 
    유일한 값으로 사용될 수 없다.            

    ``per-stream context`` 는 [FSRTL_PER_STREAM_CONTEXT](https://msdn.microsoft.com/library/windows/hardware/ff547357) 구조체를 멤버로 포함하는 
    filter-defined 구조체이다. 필터드라이버는 이 구조체를 file system 이 열어놓은 각각의 file stream 에 대한 정보를 추적하는데 사용한다.   

+ File Sytem Support for Per-Stream Contexts

    Microsoft Windows XP 이후, per-stream context 를 지원하는 파일시스템은 [FSRTL_ADVANCED_FCB_HEADER]()구조체를 포함하는 
    stream context 구조체를 사용해야 한다. 

    특정 file stream 에 연관된 per-stream context 의 global list 는 file system 이 관리한다. 
    file system 이 file stream 을 위한 새 stream context (FSRTL_ADVANCED_FCB_HEADER object)를 생성할 때, 이 list 를 초기화하기 위해 
    [FsRtlSetupAdvancedHeader]() 를 호출한다. 
    file system filter 드라이버가 [FsRtlInsertPerStreamContext]() 함수를 호출하면, filter 가 생성한 per-steram context 는 
    그 global list 에 추가된다. 

    file system 이 file stream 에 대한 stream context 를 삭제하때, filter 가 가진 file stream 에 연관된 모든 per-stream context 를 
    제거하기 위해 [FsRtlTeardownPerStreamContexts]() 를 호출한다. 
    이 루틴은 global list 에 있는 각각의 per-stream context 에 대해서 [FreeCallback]()루틴을 호출한다. 
    `FreeCallback` 루틴은 file stream 과 연관된 file object 가 이미 소멸되었음을 인지하고 있어야 한다. 

    file system 이 file object 로 대표되는 file stream 에 대해서 per-stream context 를 지원하는지 확인하려면, 해당 file object 를 통해서 
    [FsRtlSupportsPerStreamContexts]()를 호출할 수 있다. file system 은 어떤 파일 타입에 대해서는 per-stream context 를 지원할 수 도 있고, 
    그렇지 않을 수도 있다. 예를들어 NTFS 와 FAT 는 paging file 에 대해서 per-stream context 를 지원하지 않는다. 
    따라서 `FsRtlSupportsPerStreamContexts` 함수는 하나의 file stream 에 대해서는 TRUE 를 리턴하지만, 모든 file stream 에 대해서 
    TRUE 를 리턴하지는 않을것이다.      
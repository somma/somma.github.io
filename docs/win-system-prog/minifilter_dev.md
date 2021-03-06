

## Installing a Minifilter Driver
- [Installing a Minifilter Driver](https://msdn.microsoft.com/en-us/windows/hardware/drivers/ifs/installing-a-minifilter-driver)
- windows 2000 까지는 Service Control Manager 를 통해서 필터드라이버 인스톨을 했으나
- xp 이후부터는 inf 파일과 설치 프로그램을 통해 설치하는 것을 권장
- 나중에는 inf 기반 설치가 Windows Hardware Certification Kit 요구사항이 될것임

## FilterUnloadCallback Routine
- [When the FilterUnloadCallback Routine Is Called](https://msdn.microsoft.com/en-us/windows/hardware/drivers/ifs/when-the-filterunloadcallback-routine-is-called)

두가지 타입의 unload 가 있다. 
- Non-mandatory uynload
    - usermode app 에서 FilterUnload 함수를 호출하거나 kernel mode 에서 FltUnloadFilter 함수를 호출하는 경우.
    - fltmc unload 명령을 실행하는 경우
    - minifilter 의 ``FilterUnloadCallback`` 루틴이 에러 또는 경고에 해당하는 NTSTATUS (STATUS_FLT_DO_NOT_DETACH 같은)를 리턴하는 경우 filter manager 는 minifilter 를 unload 하지 않는다. 
- Mandatory unload  
    - sc stop / net stop 명령을 실행하는 경우
    - user mode app 에서 ControlService 함수를 이용, SERVICE_CONTROL_STOP 코드를 전송한 경우
    - minifilter 의 ``FilterUnloadCallback`` 이 어떤 NTSTATUS 를 리턴하든지 상관없이 minifilter 를 unload 한다. 
    - minifilter 의 unload 를 막으려면  `FLT_REGISTRATION` 구조체의 flag 에 `FLTL_REGISTRATION_DO_NOT_SUPPORT_SERVICE_STOP` 플래그를 설정해야 한다.   
    - minifilter 의 DriverEntry 루틴이 warning 또는 error NTSTAUS 값을 리턴하는 경우 FilterUnlaodCallback 루틴은 호출되지 않는다. 
    - system shutdown 시에는 FilterUnlaodCallback 루틴이 호출되지 않음
    - system shutdown 처리를 하려면 IRP_MJ_SHUTDOWN 오퍼레이션을 처리해야 한다. 

## Contexts in a Minifilter Driver
- [Managing Contexts in a Minifilter Driver](https://msdn.microsoft.com/en-us/windows/hardware/drivers/ifs/managing-contexts-in-a-minifilter-driver)
- FLT_CONTEXT_REGISTRATION 의 FLTFL_CONTEXT_REGISTRATION_NO_EXACT_SIZE_MATCH 플래그를
    - fixed size context 를 사용하고 이 플래그가 설정된 경우, 요청된 사이즈보다 context size 가 크거나 같은 경우 lookaside list 로부터 context 를 생성한다.
    - 그렇지 않은 경우, context 의 size 는 요청된 사이즈와 동일해야 함

## Writing Preoperation and Postoperation Callback Routines
- [Writing Preoperation and Postoperation Callback Routines](https://msdn.microsoft.com/en-us/windows/hardware/drivers/ifs/writing-preoperation-and-postoperation-callback-routines)
+ Filtering I/O operation in minifilter
    - IRP_MJ_CREATE 의 preoperation 에서는 context 쿼리 또는 세팅을 할 수 없다. 
    - IRP_MJ_CLOSE  의 postoperation 에서는  context 쿼리 또는 세팅을 할 수 없다. 
    - IRP_MJ_CLEANUP / IRP_MJ_CLOSE 의 preoperation 은 절대 실패를 리턴하지 말아야 한다.
    - IRP_MJ_SHUTDOWN 의 postoperation 은 등록할 수 없다.  

### Passing an I/O Operation Down the Minifilter Driver Instance Stack
- [Passing an I/O Operation Down the Minifilter Driver Instance Stack](https://msdn.microsoft.com/en-us/windows/hardware/drivers/ifs/passing-an-i-o-operation-down-the-minifilter-driver-instance-stack)
+ FLT_PREOP_SUCCESS_WITH_CALLBACK
    - preoperation 에서 이걸 리턴하면 filter manager 는 minifilter 의 postoperation callback routine 을 호출해준다.
    - 만일 postoperation callback routine 을 등록하지 않았다면 assert 를 발생하게 됨(checked build 에서)
+ FLT_PREOP_SUCCESS_NO_CALLBACK
    - preoperation 에서 이걸 리턴하면 minifilter 의 postoperation callback routine 을 호출하지 않는다. 
+ FLT_PREOP_SYNCHRONIZE
    - preoperation 에서 이걸 리턴하면 minifilter 의 postoperation callback routine 을 호출한다. 
    - 이때 filter manager 는 preoperation 과 동일한 thread context, IRQL <= APC_LEVEL 에서 호출해준다.
    - 만일 postoperation callback routine 을 등록하지 않았다면 assert 를 발생하게 됨(checked build 에서)
    - IRP based I/O operation 에서 FLT_PREOP_SYNCHRONIZE 를 리턴한다. 이외의 operation type 에서 리턴하면 filter manager 는 FLT_PREOP_SUCCESS_WITH_CALLBACK 과 동일하게 처리한다. I/O operation 이 IRP based I/O 인지 확인하기 위해서는 `FLT_IS_IRP_OPERATION` 매크로를 이용하면 된다. 
    - create operation 에서는 filter manager 가 이미 동기화를 하기 때문에 FLT_PREOP_SYNCHRONIZE 를 리턴해서는 안된다. 만일 minifilter 가 IRP_MJ_CREATE pre/post operation 을 등록해둔 경우 post-create 콜백은 pre-create 콜백루틴과 동일한 스레드 컨텍스트, IRQL==PASSIVE_LEVEL 에서 호출된다.
    - asynchronous read/write 에 대해서 절대 FLT_PREOP_SYNCHRONIZE 를 리턴해서는 안된다.  

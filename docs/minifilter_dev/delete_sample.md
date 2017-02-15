# *FLTFL_OPERATION_REGISTRATION_SKIP_PAGING_IO* 플래그를 사용하는 이유

[delete 미니필터 샘플](https://github.com/Microsoft/Windows-driver-samples/blob/master/filesys/miniFilter/delete/delete.c) 을 보면 아래와 같이 `IRP_MJ_SET_INFORMATION` 콜백 핸들러를 등록할 때 `FLTFL_OPERATION_REGISTRATION_SKIP_PAGING_IO` 를 사용하는 것을 확인할 수 있다. 

        CONST FLT_OPERATION_REGISTRATION Callbacks[] = {
            ...
            { IRP_MJ_SET_INFORMATION,
            FLTFL_OPERATION_REGISTRATION_SKIP_PAGING_IO,
            DfPreSetInfoCallback,
            DfPostSetInfoCallback },
            ...
            { IRP_MJ_OPERATION_END }
        };

이유가 궁금해서 검색을 해보니 관련된 [질문과 답](https://www.osronline.com/showthread.cfm?link=238116)이 있었다.

내용을 정리해두자면,
On demand paging 을 구현하기 위해 VMM(Virtual Memory Manager)이 생성한 IRP 기반의 I/O 요청이 Paging I/O 이다.
IRP_MJ_CREATE, IRP_MJ_CLEAN_UP 에는 Paging I/O 가 없고, Paging I/O 를 지원하는 I/O 는 아래와 같다.

- IRP_MJ_READ
- IRP_MJ_WRITE
- IRP_MJ_SET_INFORMATION
- IRP_MJ_QUERY_INFORMATION

IRP_MJ_READ 나 IRP_MJ_WRITE 의 는 Paging I/O 를 지원하는 것이 당연하고, IRP_MJ_SET_INFORMATION 이나 IRP_MJ_QUERY_INFORMATION 은 메모리 매니저가 섹션 객체를 생성하거나 접근할 때 필요할것이다.

해당 샘플(delete)의 경우 파일 삭제를 식별하기 위해 IRP_MJ_SET_INFORMATION 콜백을 등록하고, FileDispositionInformation, FileDispositionInformationEx 를 모니터링 한다. 
이 I/O 는 Paging I/O 와는 상관없기때문에 `FLTFL_OPERATION_REGISTRATION_SKIP_PAGING_IO` 를 사용한 것이다. 

# 참고

- [IFS FAQ](http://www.osronline.com/article.cfm?article=17#Q25)
- [[FLTFL_OPERATION_REGISTRATION_SKIP_PAGING_IO] Why is this flag used here? (minifilter)](https://www.osronline.com/showthread.cfm?link=238116)
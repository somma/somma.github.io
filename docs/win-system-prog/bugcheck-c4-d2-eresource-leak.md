
아주 간단한 verifier bugcheck 입니다. 

    2: kd> !analyze -v
    *******************************************************************************
    *                                                                             *
    *                        Bugcheck Analysis                                    *
    *                                                                             *
    *******************************************************************************

    DRIVER_VERIFIER_DETECTED_VIOLATION (c4)
    A device driver attempting to corrupt the system has been caught.  This is
    because the driver was specified in the registry as being suspect (by the
    administrator) and the kernel has enabled substantial checking of this driver.
    If the driver attempts to corrupt the system, bugchecks 0xC4, 0xC1 and 0xA will
    be among the most commonly seen crashes.
    Arguments:
    Arg1: 00000000000000d2, Freeing pool allocation that contains active ERESOURCE.
    Arg2: ffffbb0b0c004070, ERESOURCE address.  
    Arg3: ffffbb0b0c004000, Pool allocation start address.
    Arg4: 0000000000000110, Pool allocation size.         

동적으로 할당한 `ERESOURCE` 를 포함하고 있는 메모리를 해제하지 않은것을 Driver verifier 가 잡아서 Bugcheck 을 발생시켰습니다.

    Arg1: 00000000000000d2, Freeing pool allocation that contains active ERESOURCE.
    Arg2: ffffbb0b0c004070, ERESOURCE address.                  << ERESOURCE 구조체 주소
    Arg3: ffffbb0b0c004000, Pool allocation start address.      << ERESOURCE 가 포함된 pool 의 시작 주소
    Arg4: 0000000000000110, Pool allocation size.               << ERESOURCE 가 포함된 pool 의 할당 크기

따라서 `ERESOURCE` 가 포함된 동적할당 영역을 어디서/언제/누가 할당했는지 알아내면 되는데요. `ERESOURCE` 가 포함된 pool 의 시작주소를 verifier 에서 알려주고 있기때문에 아주 간단히 찾아낼 수 있습니다.

    2: kd> !pool ffffbb0b0c004000
    Pool page ffffbb0b0c004000 region is Nonpaged pool
    *ffffbb0b0c004000 size:  110 previous size:    0  (Allocated) *ICX 
        ...

Leak 이 발생한 pool 의 TAG 가 ` XCI` 임을 쉽게 확인 할 수 있습니다. 이제 그냥 코드에서 해당 Tag 를 사용하는 할당을 찾아서 확인하면 됩니다. 
말할 필요도 없지만 `verifier` 를 항상 활성화하고 메모리 할당시에는 Tag 를 잘 사용해야 겠지요.

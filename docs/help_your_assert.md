# ASSERT, ASSERT, ASSERT

아래의 코드를 봅시다. busy_write 라는 전역변수를 통해서 아주 간단한 배타적 접근을 구현하고 있습니다.

    int busy_write = 0;
    int value_write = 0;

    int funcA()
    {
        //
        //  #1 lock 을 걸어주고
        //
        if (1 == InterlockedCompareExchange(&busy_write, 1, 0))
        {
            goto Cleanup;
        }

        //
        //  #2 뭔가 열심히 복잡한 작업을 하고, value_write 값을 갱신합니다.
        //
        ...
        value_write++;
        ...

        //
        //  #3 이 ASSERT 가 필요한가?
        //
        ASSERT(1 == StreamContext->busy_write);

    Cleanup:
        //
        //  #4 busy_write 를 다시 0 으로.
        //
        InterlockedAnd(&busy_write, 0);
        return 0;
    }

코드 #3 에 있는 `ASSERT` 는 별로 필요없어 보입니다. 위에서 busy_write 를 1로 변경한 상태이기 때문에 언제나 `true` 입니다. 따라서 당연히 ASSERT 를 넣는것이 이해가 되지 않을 수도 있습니다. 하지만 시간이 흘러, 어찌 어찌하다 보니 이 함수가 #2 에서 재귀호출을 하도록 수정되어버렸다고 가정해 봅시다. 어떻게 될까요?

\#2 에서 재귀호출이 발생하면 #4 로 jump 하게 될것이고, busy_write 를 0 으로 변경한 후 리턴될겁니다. 그럼 원래 호출했던 코드에서는 #3 ASSERT 에 걸리게 되고, 개발자는 바로 이 코드에 문제가 있음을 알아챌 수 있을겁니다. 

만일 #3 에 ASSERT 가 없었다면 어떻게 되었을까요? 아마 코드의 문제를 인지하고, 디버깅 하는데 ASSERT 가 있었던 상황보다 분명히 더 많은 시간이 소요될 겁니다.

ASSERT 를 사용하는 규칙은, 당연히 확실한 상태라고 믿어 의심치 않아서, ASSERT 가 전혀 필요없어 보이는, 그런코드에 ASSERT 를 사용하는것입니다.

가끔 강의를 하거나 코드 리뷰를 할때 ASSERT 사용을 가볍게 여기는 개발자분들을 많이 봐서, 오늘 겪은 좋은 예가 있길래 글로 남겼습니다.

\#초보_개발자_참고용 \#손가락이_아플때까지
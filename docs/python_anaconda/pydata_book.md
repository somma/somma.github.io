# Python for Data Analysis

## setup

가상 환경을 생성하고, 해당 가상환경을 활성화한다.

        conda create python=3.5 --prefix d:\work.pydata\env
        activate d:\work.pydata\env

필요한 패키지를 설치한다.

        (d:\work.pydata\env) c:\work.pydata>conda install ipython jupyter numpy pandas matplotlib

텐서플로우 패키지도 설치한다.

        (d:\work.pydata\env) c:\work.pydata>conda install -c conda-forge tensorflow

pycharm 사용시 project interpreter 를 `c:\work.pydata\env\python.exe` 로 지정하면 알아서 env 정보를 읽어온다.

## ipython 소개

* qt 기반 콘솔 실행하기

        ipython qtconsole --pylab=inline

* ipython 을 matplotlib 과 함께 사용하기

        ipython --pylab

    *ValueError: _getfullpathname: embedded null character* 에러가 발생하는데, 파이썬 자체의 버그라고 함( [참고](http://stackoverflow.com/questions/34004063/error-on-import-matplotlib-pyplot-on-anaconda3-for-windows-10-home-64-bit-pc) )

    `\env\Lib\site-packages\matplotlib\font_manager.py` 내 `win32InstalledFonts()` 함수를 아래처럼 수정해 주면 잘 됨

            key, direc, any = winreg.EnumValue( local, j)
            if not is_string_like(direc):
                continue
            if not os.path.dirname(direc):
                direc = os.path.join(directory, direc)
            direc = direc.split('\0', 1)[0]


### 명령어 및 단축키

* %run

* Ctrl + [P | UP], Ctrl + [N | DOWN]
    이전에 실행했던 명령어 탐색

* Ctrl + R
    이전에 실행했던 명령어 매칭

* !쉘명령어
    쉘명령어를 실행한다.

        In [13]: share = !net share | findstr dbg
        In [14]: print(share)
        ['dbg          C:\\dbg                          ']
        In [15]:

* %bookmark

    디렉토리 북마크 설정 (ipython 이 종료되어도 계속 유지 됨)

        In [18]: %bookmark dn c:/Users/somma/Downloads/
        In [20]: cd dn
        (bookmark:dn) -> c:/Users/somma/Downloads/
        c:\Users\somma\Downloads

    북마크 리스팅

        In [25]: %bookmark -l
        Current bookmarks:
        dn -> c:/Users/somma/Downloads/

* %debug

    디버거 실행, u/d 명령으로 콜스택을 이동할 수 있음





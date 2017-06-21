# conda 사용하기

+ [document](http://conda.pydata.org/docs/index.html)

## managing conda

        conda --version
        conda update conda

## managing environments

+ 기본경로에 환경생성 / 삭제 (e.g. c:\users\somma\.anaconda\env)

        conda create --name bp
        conda remove --name bp --all
        conda create --name bp python=3.4 

+ 지정한 경로에 환경생성 / 삭제

        conda create --prefix c:\bp\env
        conda remove --prefix c:\bp\env --all
        conda create --prefix c:\bp\env python=3.4 
        conda create --prefix .\env python=3.4 

+ 환경 활성화/비활성화

        activate bp
        deactivate bp


## managing python

+ 사용가능한 파이썬 버전 보기

        conda search --full-name python

+ 환경정보 보기
        conda info
        conda info --envs

## managing packages

+ 현재 환경에 설치된 패키지 목록 / 버전 정보 보기

        conda list


+ 패키지 찾기

        conda search beautifulsoup4

+ 패키지 설치하기

        conda install --name bp beautifulsoup4
        pip install beautifulsoup4


## tensorflow

+ 지정 경로에 tensorflow 환경을 생성한다. (c:\work.tf\env)

        c:\>conda create  python=3.5 --prefix c:\work.tf\env
        Fetching package metadata ...........
        Solving package specifications: .

        Package plan for installation in environment c:\work.tf\env:

        The following NEW packages will be INSTALLED:

        pip:            9.0.1-py35_1
        python:         3.5.2-0
        setuptools:     27.2.0-py35_1
        vs2015_runtime: 14.0.25123-0
        wheel:          0.29.0-py35_0

        Proceed ([y]/n)?

        #
        # To activate this environment, use:
        # > activate c:\work.tf\env
        #
        # To deactivate this environment, use:
        # > deactivate c:\work.tf\env
        #
        # * for power-users using bash, you must source
        #

+ 생성한 environment 를 활성화하고, 필요한 패키지를 해당 환경에 설치한다.

        (c:\work.tf\env) c:\work.tf>conda install ipython jupyter numpy pandas matplotlib

+ conda-forge 레퍼지토리에서 tensorflow 패키지를 설치한다.(-c 옵션 이용)

        (c:\work.tf\env) c:\work.tf>conda install -c conda-forge tensorflow

+ pycharm 사용시 project interpreter 를 `c:\work.tf\env\python.exe` 로 지정하면 알아서 env 정보를 읽어온다.
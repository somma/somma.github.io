# conda 사용하기
+ [document](http://conda.pydata.org/docs/index.html)

## managing conda
    
        conda --version
        conda update conda

## managing environments
+ 기본경로에 환경생성 / 삭제 
    
        conda create --name bp
        conda remove --name bp --all
        conda create --name bp python=3.4 

+ 지정한 경로에 환경생성 / 삭제
    
        conda create --prefix c:\bp
        conda remove --prefix c:\bp --all

+ 환경 활성화/비활성화

        activate bp
        deactivate bp


## managing python
+ 사용가능한 파이썬 버전 보기

        conda search --full-name python

+ 환경정보 보기 
        
        conda info --envs
        
## managing packages
+ 현재 환경에 설치된 패키지 목록 / 버전 정보 보기
        
        conda list


+ 패키지 찾기 
        
        conda search beautifulsoup4

+ 패키지 설치하기
        
        conda install --name bp beautifulsoup4
        pip install beautifulsoup4
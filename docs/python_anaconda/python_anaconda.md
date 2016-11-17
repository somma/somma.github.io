# conda 사용하기
+ [document](http://conda.pydata.org/docs/index.html)

## managing conda
    
        conda --version
        conda update conda

## managing environments
+ 기본경로에 환경생성 / 삭제 
    
        conda create --name bp
        conda remove --name bp --all

+ 지정한 경로에 환경생성 / 삭제
    
        conda create --prefix c:\bp
        conda remove --prefix c:\bp --all

+ 환경 활성화/비활성화

        conda info --envs
        activate bp
        deactivate bp 

        
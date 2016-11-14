# Boost library build

> bootstrap.bat 

> b2.exe -j8 toolset=msvc-14.0 address-model=64 \
		--build-type=complete \
		--build-dir=.\build_dir_vc14_x64 \
		--prefix=c:\boost1.62_x64_msvc14 \
		install


> b2.exe -j8 toolset=msvc-14.0 \
		--build-type=complete \
		--build-dir=.\build_dir_vc14_x86 \
		--prefix=c:\boost1.62_x86_msvc14 \
		install
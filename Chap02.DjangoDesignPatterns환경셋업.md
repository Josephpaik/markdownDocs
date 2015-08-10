#장고 디자인 패턴 윈도우 개발환경#
1. 파이썬3.4를 다운로드 한다. http://www.python.org/download/
2. 설치할 때 Python3.4의 실행 경로를 추가한다.
3. PowerShell을 관리자 권한으로 실행하고 다음 명령을 수행한다.

	>$ Set-ExecutionPolicy Unrestricted

	>$ mkdir e:\envs

	>$ cd e:\envs

	>$ (new-object System.Net.WebClient).DownloadFile('https://bootstrap.pypa.io/ez_setup.py', 'e:\envs\distribute_setup.py')

	>$ (new-object System.Net.WebClient).DownloadFile('https://raw.github.com/pypa/pip/master/contrib/get-pip.py', 'e:\envs\get-pip.py')
    
	>$ python e:\envs\distribute_setup.py

	>$ python e:\envs\get-pip.py
	
	-> 위의 과정 진행중 파워셀 실행오류가 난다면 링크주소를 우클릭하여 다른 이름으로 저장하여 지정된 경로에서 실행해도 된다.

4. 파이썬 설치 버전 확인
	> $ python --version
	
	> $ Python 3.4.3

5. Virtual Environment 생성 및 활성화
	> $ python -m venv sbenv
	
	> $ .\sbenv\Scripts\Activate.bat

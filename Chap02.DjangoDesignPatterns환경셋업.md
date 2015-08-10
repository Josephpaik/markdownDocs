#장고 디자인 패턴 윈도우 개발환경#
1. 파이썬3.4를 다운로드 한다. http://www.python.org/download/
2. 설치할 때 Python3.4의 실행 경로를 추가한다.
3. PowerShell을 관리자 권한으로 실행하고 다음 명령을 수행한다.

	> Set-ExecutionPolicy Unrestricted

	> mkdir e:\envs

	> cd e:\envs

	> (new-object System.Net.WebClient).DownloadFile('https://bootstrap.pypa.io/ez_setup.py', 'c:\envs\distribute_setup.py')

	> (new-object System.Net.WebClient).DownloadFile('https://raw.github.com/pypa/pip/master/contrib/get-pip.py', 'c:\envs\get-pip.py')

	> python e:\envs\distribute_setup.py

	> python e:\envs\get-pip.py

4. 파이썬 설치 버전 확인
	> $ python --version
	> Python 3.4.3

5. Virtual Environment 생성 및 활성화
	> python -m venv sbenv
	> .\sbenv\Scripts\Activate.bat

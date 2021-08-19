# Linux Command
## 리눅스 명령어 정리


man 명령어 : 명령어 매뉴얼  
pwd : 현재 경로  
ls : 현재 폴더/파일  
ls -l : 수정자 날짜 포함  
ls -a : 숨김파일 포함  
ls -la : 수정자 날짜 + 숨김파일  
open 파일명 : 파일 열때  
cd : 경로 이동  
cd ~ : 최상위 경로  
cd - : 이전 경로  
find . -type file -name "*.json" : 파일 찾기  
which : 설치된 경로  
touch 파일명 : 새로운 파일 생성  
cat 파일명 : 파일 열기  
echo 문자열 > 파일명 : 파일에 문자열 생성(덮어쓰기)  
echo 문자열 >> 파일명 : 파일에 문자열 생성(append 이어붙이기)  
mkdir 디렉토리 : 디렉토리 만들기  
mkdir -p dir1/test/test.... : 디렉토리 경로대로 만들기  
cp 파일명 디렉토리경로 : 파일 해당 디렉토리로 복사  
mv 파일명 디렉토리경로 : 파일 해당 디렉토리로 이동  
rm 파일명 : 파일 삭제  
rm -r 경로명 : 경로삭제시  
grep 키워드 파일명 : 키워드 찾기  
grep -n 키워드 파일명 : 몇번째 줄인가 찾기  
tail -n 숫자 파일명 : 끝에서 n번째 줄까지 출력  


vim 파일명 : 파일 생성 vim editor  
i : insert  
ESC -> :wq   저장후 종료   
:w -> 저장  
:q -> 종료  
:qa -> 강제 종료  

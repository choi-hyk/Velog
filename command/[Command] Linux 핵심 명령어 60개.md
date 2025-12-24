[Velog로 가기](https://velog.io/@choi-hyk/Command-Linux-핵심-명령어-60개)

released at 2025-08-12 09:31:08 KST

updated at 2025-12-25 00:47:04 KST

|[command](https://velog.io/tags/command)|[linux](https://velog.io/tags/linux)|
|----|----|

## 1. **ls** — 디렉터리 목록 표시 List

**형식**

```bash
ls [옵션] [경로]
```

**주요 옵션**

- -l 상세 목록

- -a 숨김 파일 포함

- -R 하위 디렉터리 재귀

- -h 사람이 읽기 쉬운 크기


**예시**

```bash
ls -la
```
- 현재 디렉터리의 숨김 파일까지 상세 정보 출력

```bash
ls -lh /var/log
```
- /var/log의 파일 크기를 사람이 읽기 쉬운 단위로 표시

```bash
ls -R ~/projects
```
- projects 아래 하위 디렉터리까지 재귀적으로 나열

---

## 2. **pwd** — 현재 작업 디렉터리 경로 출력 Print Working Directory

**형식**

```bash
pwd [옵션]
```

**주요 옵션**

- -P 실제 경로(심볼릭 링크 해제)

- -L 논리 경로(기본값)


**예시**

```bash
pwd
```
- 현재 작업 디렉터리의 논리 경로 표시

```bash
pwd -P
```
- 심볼릭 링크를 해제한 실제 경로 표시

```bash
cd /var && pwd
```
- /var로 이동 후 경로 출력

---

## 3. **cd** — 디렉터리 이동 Change Directory

**형식**

```bash
cd [경로]
```

**주요 옵션**

- .. 상위 디렉터리

- ~ 홈 디렉터리

- - 이전 디렉터리로 전환


**예시**

```bash
cd ..
```
- 상위 디렉터리로 이동

```bash
cd ~/Downloads
```
- 홈의 Downloads 디렉터리로 이동

```bash
cd -
```
- 직전 작업 디렉터리로 이동

---

## 4. **mkdir** — 디렉터리 생성 Make Directory

**형식**

```bash
mkdir [옵션] 디렉터리
```

**주요 옵션**

- -p 하위 경로까지 일괄 생성

- -m 권한 지정

- -v 생성 과정 출력


**예시**

```bash
mkdir project
```
- 현재 위치에 project 디렉터리 생성

```bash
mkdir -p a/b/c
```
- 중간 경로가 없어도 a/b/c까지 모두 생성

```bash
mkdir -m 755 web
```
- 권한 755로 web 디렉터리 생성

---

## 5. **rmdir** — 비어 있는 디렉터리 삭제 Remove Directory

**형식**

```bash
rmdir [옵션] 디렉터리
```

**주요 옵션**

- -p 상위 디렉터리 연쇄 삭제

- -v 삭제 과정 출력


**예시**

```bash
rmdir empty
```
- empty 디렉터리를 삭제(비어 있어야 함)

```bash
rmdir -p a/b/c
```
- c 삭제 후 비어 있으면 b, a 순서로 삭제

```bash
rmdir -v old
```
- 삭제 과정을 출력하며 비어 있는 old 삭제

---

## 6. **rm** — 파일/디렉터리 삭제 Remove

**형식**

```bash
rm [옵션] 파일/디렉터리
```

**주요 옵션**

- -r 디렉터리 재귀 삭제

- -f 강제 삭제(확인 없음)

- -i 삭제 전 확인


**예시**

```bash
rm file.txt
```
- 일반 파일 file.txt 삭제

```bash
rm -rf build
```
- build 디렉터리와 내부 모든 항목 강제 삭제

```bash
rm -i data.csv
```
- 삭제 전 확인 프롬프트 표시

---

## 7. **cp** — 파일/디렉터리 복사 Copy

**형식**

```bash
cp [옵션] 원본 대상
```

**주요 옵션**

- -R 디렉터리 재귀 복사

- -p 권한/타임스탬프 보존

- -i 덮어쓰기 전 확인


**예시**

```bash
cp a.txt b.txt
```
- a.txt를 b.txt로 복사

```bash
cp -R src dst
```
- src 디렉터리 전체를 dst로 복사

```bash
cp -p config.ini /etc/app/
```
- 속성을 보존하며 복사

---

## 8. **mv** — 파일/디렉터리 이동 또는 이름 변경 Move

**형식**

```bash
mv [옵션] 원본 대상
```

**주요 옵션**

- -i 덮어쓰기 전 확인

- -n 덮어쓰기 방지

- -v 작업 로그 출력


**예시**

```bash
mv a.txt b.txt
```
- a.txt의 이름을 b.txt로 변경

```bash
mv /tmp/logs ./logs_old
```
- /tmp/logs를 현재 위치로 이동하며 logs_old로 변경

```bash
mv *.log logs/
```
- 현재 디렉터리의 로그 파일을 logs로 이동

---

## 9. **touch** — 빈 파일 생성 또는 타임스탬프 변경

**형식**

```bash
touch [옵션] 파일...
```

**주요 옵션**

- -t 지정한 타임스탬프 설정

- -a 접근 시간만 변경

- -m 수정 시간만 변경


**예시**

```bash
touch notes.md
```
- 빈 파일 생성(이미 존재 시 타임스탬프 갱신)

```bash
touch -t 202501010101 file
```
- 지정한 시각으로 타임스탬프 설정

```bash
touch -a -m data.bin
```
- 접근/수정 시간을 현재 시각으로 변경

---

## 10. **file** — 파일 타입 식별

**형식**

```bash
file [옵션] 파일...
```

**주요 옵션**

- -i MIME 타입 표시

- -b 파일명 없이 결과만

- -k 가능한 추가 정보 표시


**예시**

```bash
file image.png
```
- PNG 이미지 파일로 식별

```bash
file -i report.pdf
```
- MIME 타입(application/pdf) 표시

```bash
file -b archive.tar.gz
```
- 파일명 없이 타입 정보만 출력

---

## 11. **zip** — ZIP 압축 생성

**형식**

```bash
zip [옵션] 아카이브.zip 파일들
```

**주요 옵션**

- -r 디렉터리 재귀 압축

- -9 최대 압축률

- -q 조용한 모드


**예시**

```bash
zip logs.zip *.log
```
- 현재 디렉터리의 .log 파일을 logs.zip으로 압축

```bash
zip -r site.zip ./site
```
- site 디렉터리 전체를 압축

```bash
zip -9 backup.zip data/*
```
- 최대 압축률로 data 내용을 압축

---

## 12. **unzip** — ZIP 압축 해제

**형식**

```bash
unzip [옵션] 아카이브.zip [-d 대상]
```

**주요 옵션**

- -d 대상 디렉터리 지정

- -l 목록만 보기

- -o 덮어쓰기 강제


**예시**

```bash
unzip logs.zip
```
- 현재 디렉터리에 압축 해제

```bash
unzip -d out data.zip
```
- out 디렉터리로 압축 해제

```bash
unzip -l site.zip
```
- 압축 내부 파일 목록만 출력

---

## 13. **tar** — 아카이브 생성/해제 Tape ARchiver

**형식**

```bash
tar [옵션] [아카이브] [파일/디렉터리...]
```

**주요 옵션**

- -c 생성

- -x 해제

- -t 목록

- -f 파일 지정

- -z gzip 사용

- -v 진행 표시


**예시**

```bash
tar -cf backup.tar folder
```
- folder를 backup.tar로 묶기(압축 없음)

```bash
tar -czf backup.tar.gz folder
```
- gzip으로 압축한 tarball 생성

```bash
tar -xzf backup.tar.gz -C /restore
```
- /restore에 압축 해제

---

## 14. **nano** — 터미널 텍스트 편집기

**형식**

```bash
nano [옵션] 파일
```

**주요 옵션**

- -l 줄 번호 표시

- -B 백업 파일 생성

- -m 마우스 지원


**예시**

```bash
nano /etc/hosts
```
- hosts 파일을 편집

```bash
nano -l notes.md
```
- 줄 번호가 표시된 상태로 편집

```bash
nano -B config.ini
```
- 편집 시 config.ini~ 백업 생성

---

## 15. **vi** — 터미널 텍스트 편집기(Vim 포함)

**형식**

```bash
vi [+행] 파일
```

**주요 옵션**

- +N N번째 줄에서 시작

- -R 읽기 전용 모드

- -u NONE 기본 설정 없이


**예시**

```bash
vi app.py
```
- app.py 파일 편집

```bash
vi +10 main.c
```
- 10번째 줄에서 편집 시작

```bash
vi -R /etc/fstab
```
- 읽기 전용으로 열기

---

## 16. **cat** — 파일 내용 출력/결합 Concatenate

**형식**

```bash
cat [옵션] [파일...]
```

**주요 옵션**

- -n 줄 번호 출력

- -A 제어문자 표시

- -E 줄 끝 표시


**예시**

```bash
cat file.txt
```
- 파일 내용을 표준출력으로 표시

```bash
cat a.txt b.txt > all.txt
```
- 두 파일을 결합해 all.txt 생성

```bash
cat -n code.c | less
```
- 줄 번호를 붙여 페이지 단위로 보기

---

## 17. **tac** — 파일을 거꾸로 출력

**형식**

```bash
tac [옵션] 파일
```

**주요 옵션**

- -s 구분자 지정

- -b 구분자 뒤에 출력

- -r 정규식 구분자


**예시**

```bash
tac file.txt
```
- 마지막 줄부터 첫 줄까지 역순 출력

```bash
tac -s '---' parts.txt
```
- 지정 구분자로 블록 단위 역순 출력

```bash
tac -b -s "," csv_parts.txt
```
- 구분자 뒤에 이어 붙여 역순 출력

---

## 18. **grep** — 패턴 매칭 검색 Global Regular Expression Print

**형식**

```bash
grep [옵션] 패턴 [파일...]
```

**주요 옵션**

- -i 대소문자 무시

- -n 줄 번호

- -r 디렉터리 재귀

- -E 확장 정규식


**예시**

```bash
grep -n 'ERROR' app.log
```
- ERROR 포함 라인을 라인 번호와 함께 표시

```bash
dmesg | grep -i usb
```
- 커널 로그에서 usb 관련 메시지 필터링

```bash
grep -r 'TODO' src/
```
- src 디렉터리 전체에서 TODO 검색

---

## 19. **sed** — 스트림 편집기 Stream EDitor

**형식**

```bash
sed [옵션] '스크립트' [파일]
```

**주요 옵션**

- -n 선택 출력

- -E 확장 정규식

- -i 제자리 수정(in-place)


**예시**

```bash
sed -n '1,10p' file.txt
```
- 1~10행만 출력

```bash
sed 's/red/blue/g' colors.txt
```
- red를 blue로 전역 치환 후 출력

```bash
sed -i 's/DEBUG=false/DEBUG=true/' .env
```
- 파일을 직접 수정하여 값 변경

---

## 20. **head** — 파일 앞부분 출력

**형식**

```bash
head [옵션] 파일
```

**주요 옵션**

- -n N행 출력

- -c N바이트 출력


**예시**

```bash
head -n 20 access.log
```
- 앞 20줄 출력

```bash
head -c 1K big.bin
```
- 앞 1024바이트 출력

```bash
head README.md
```
- 기본 10줄 출력

---

## 21. **tail** — 파일 뒷부분 출력

**형식**

```bash
tail [옵션] 파일
```

**주요 옵션**

- -n N행

- -f 추가 내용 실시간 추적

- -c N바이트


**예시**

```bash
tail -n 50 access.log
```
- 마지막 50줄 출력

```bash
tail -f app.log
```
- 로그 파일을 실시간으로 추적

```bash
tail -c 512 data.bin
```
- 마지막 512바이트 출력

---

## 22. **awk** — 패턴 스캔과 처리 언어

**형식**

```bash
awk [옵션] '패턴{동작}' [파일]
```

**주요 옵션**

- -F 필드 구분자 지정

- NR 현재 레코드 번호

- NF 필드 개수

- $1..$N 필드 참조


**예시**

```bash
awk -F: '{print $1}' /etc/passwd
```
- 콜론 기준 1번째 필드(계정명) 출력

```bash
awk '{sum+=$1} END{print sum}' nums.txt
```
- 첫 필드 합계 계산

```bash
awk '$3>100 {print $0}' data.tsv
```
- 3번째 필드가 100 초과인 행 출력

---

## 23. **sort** — 정렬

**형식**

```bash
sort [옵션] [파일]
```

**주요 옵션**

- -r 내림차순

- -n 숫자 정렬

- -k N 정렬 키

- -t 구분자


**예시**

```bash
sort names.txt
```
- 기본 오름차순 정렬

```bash
sort -nr scores.txt
```
- 숫자 기준 내림차순 정렬

```bash
sort -t, -k2 data.csv
```
- 쉼표 구분, 2열 기준 정렬

---

## 24. **cut** — 필드/문자 범위 추출

**형식**

```bash
cut [옵션] 파일
```

**주요 옵션**

- -d 구분자

- -f 필드 리스트

- -c 문자 범위


**예시**

```bash
cut -d, -f2,4 data.csv
```
- 2, 4열만 추출

```bash
cut -c1-8 ids.txt
```
- 각 행의 1~8번째 문자 추출

```bash
cut -d: -f1 /etc/passwd
```
- 계정명 필드만 출력

---

## 25. **diff** — 두 파일의 차이 비교

**형식**

```bash
diff [옵션] 파일1 파일2
```

**주요 옵션**

- -u 통합 형식

- -r 디렉터리 재귀

- -q 차이 유무만


**예시**

```bash
diff a.txt b.txt
```
- 두 텍스트의 라인 단위 차이 표시

```bash
diff -u old.c new.c
```
- 패치에 적합한 통합 형식으로 표시

```bash
diff -r src_old src_new
```
- 디렉터리 간 차이를 재귀적으로 비교

---

## 26. **tee** — 출력을 화면과 파일에 동시에 기록

**형식**

```bash
cmd | tee [옵션] 파일
```

**주요 옵션**

- -a 파일에 이어쓰기

- -i SIGINT 무시


**예시**

```bash
echo hello | tee out.txt
```
- hello를 화면과 out.txt에 동시에 기록

```bash
dmesg | tee -a kernel.log
```
- 커널 메시지를 화면+파일(추가)로 기록

```bash
ls -l | tee list.txt | wc -l
```
- 목록을 저장하고 행 수 계산

---

## 27. **tr** — 문자 변환/삭제 Translate

**형식**

```bash
tr [옵션] 집합1 [집합2]
```

**주요 옵션**

- -d 삭제

- -s 반복 문자 압축


**예시**

```bash
echo 'Hello' | tr '[:upper:]' '[:lower:]'
```
- 대문자를 소문자로 변환

```bash
echo 'a,,b,,,c' | tr -s ','
```
- 연속된 쉼표를 하나로 압축

```bash
echo 'abc123' | tr -d '0-9'
```
- 숫자 문자 삭제

---

## 28. **chmod** — 파일 권한 변경 Change Mode

**형식**

```bash
chmod [옵션] 모드 파일
```

**주요 옵션**

- u/g/o 사용자/그룹/기타

- +/- 권한 추가/제거

- 숫자 표기 755 등

- -R 재귀 적용


**예시**

```bash
chmod 644 file.txt
```
- 소유자 읽기/쓰기, 그 외 읽기

```bash
chmod u+x script.sh
```
- 소유자에 실행 권한 추가

```bash
chmod -R 755 bin/
```
- 디렉터리 전체에 실행 권한 부여

---

## 29. **chown** — 파일 소유자/그룹 변경 Change Owner

**형식**

```bash
chown [옵션] 소유자[:그룹] 파일
```

**주요 옵션**

- -R 재귀 적용

- --from 기존 소유자 조건


**예시**

```bash
sudo chown user file.txt
```
- file.txt의 소유자를 user로 변경

```bash
sudo chown user:staff -R www
```
- www 디렉터리와 내부의 소유자/그룹 변경

```bash
sudo chown :www-data app.log
```
- 그룹만 www-data로 변경

---

## 30. **ln** — 하드/심볼릭 링크 생성

**형식**

```bash
ln [옵션] 원본 링크명
```

**주요 옵션**

- -s 심볼릭 링크

- -f 기존 링크 덮어쓰기

- -n 심볼릭 링크 대상 처리


**예시**

```bash
ln file.txt file_hard
```
- 하드 링크 생성

```bash
ln -s /opt/app/bin/run ./run
```
- 실행 파일의 심볼릭 링크 생성

```bash
ln -sf new.conf current.conf
```
- 기존 링크를 새 대상에 강제로 갱신

---

## 31. **find** — 파일 검색

**형식**

```bash
find 경로 [조건] [동작]
```

**주요 옵션**

- -name 이름 패턴

- -type 파일 타입

- -size 크기 조건

- -exec 명령 실행


**예시**

```bash
find . -name '*.log'
```
- 현재 디렉터리에서 .log 파일 검색

```bash
find /var -type d -name 'nginx'
```
- /var에서 디렉터리 nginx 검색

```bash
find . -size +100M -exec rm -i {} \;
```
- 100MB 초과 파일을 찾아 삭제 확인

---

## 32. **locate** — 인덱스 기반 빠른 파일 검색

**형식**

```bash
locate [패턴]
```

**주요 옵션**

- -i 대소문자 무시

- -n N개 결과 제한


**예시**

```bash
locate ssh_config
```
- 시스템 DB에서 ssh_config 경로 빠르게 검색

```bash
locate -i readme
```
- 대소문자 무시하고 README/Readme 등 검색

```bash
locate -n 5 nginx.conf
```
- 최대 5개 결과만 표시

---

## 33. **which** — 실행 파일의 경로 표시

**형식**

```bash
which 프로그램
```

**예시**

```bash
which python
```
- python 실행 파일의 절대 경로 출력

```bash
which ls
```
- ls 명령의 실제 경로 확인

```bash
which node
```
- node가 PATH에 있는지 확인

---

## 34. **whereis** — 명령의 바이너리/소스/매뉴얼 위치

**형식**

```bash
whereis 프로그램
```

**주요 옵션**

- -b 바이너리만

- -m 매뉴얼만

- -s 소스만


**예시**

```bash
whereis ls
```
- ls의 바이너리와 매뉴얼 위치 표시

```bash
whereis -b gcc
```
- gcc 바이너리 위치만 표시

```bash
whereis -m bash
```
- bash의 man 페이지 위치 표시

---

## 35. **du** — 디스크 사용량 추정 Disk Usage

**형식**

```bash
du [옵션] [경로]
```

**주요 옵션**

- -h 사람이 읽기 쉬운 단위

- -s 총합 요약

- -d 깊이 제한


**예시**

```bash
du -h .
```
- 현재 디렉터리의 각 항목 사용량 표시

```bash
du -sh /var/log
```
- /var/log의 총 사용량 요약

```bash
du -h -d1 /home/user
```
- /home/user 하위 1단계까지 사용량 표시

---

## 36. **df** — 파일시스템별 여유/전체 디스크 용량

**형식**

```bash
df [옵션] [경로]
```

**주요 옵션**

- -h 사람이 읽기 쉬운 단위

- -T 파일시스템 타입 표시

- -i inode 정보


**예시**

```bash
df -h
```
- 모든 마운트의 용량/사용량을 사람이 읽기 쉬운 단위로 표시

```bash
df -T /
```
- 루트 파티션의 파일시스템 타입과 용량 표시

```bash
df -i
```
- inode 사용 현황 표시

---

## 37. **free** — 메모리/스왑 사용량

**형식**

```bash
free [옵션]
```

**주요 옵션**

- -h 사람이 읽기 쉬운 단위

- -m MB 단위

- -s N초마다 갱신


**예시**

```bash
free -h
```
- RAM/스왑 총량과 사용량을 보기 좋게 표시

```bash
free -m
```
- 메모리 정보를 MB 단위로 표시

```bash
free -h -s 5
```
- 5초마다 갱신하여 메모리 사용 추적

---

## 38. **top** — 실시간 프로세스/리소스 모니터

**형식**

```bash
top
```

**주요 옵션**

- -p PID 특정 프로세스만

- -u USER 사용자 필터


**예시**

```bash
top
```
- CPU/메모리 사용량 상위 프로세스 실시간 표시

```bash
top -u www-data
```
- 특정 사용자 프로세스만 모니터링

```bash
top -p 1234
```
- PID 1234의 리소스 사용만 추적

---

## 39. **ps** — 프로세스 스냅샷

**형식**

```bash
ps [옵션]
```

**주요 옵션**

- aux 모든 프로세스 상세

- -ef 표준 포맷

- -o 출력 형식 지정


**예시**

```bash
ps aux | grep nginx
```
- nginx 관련 프로세스 찾기

```bash
ps -ef --forest
```
- 트리 형태로 프로세스 관계 표시

```bash
ps -o pid,cmd -p 1234
```
- 특정 PID의 정보만 출력

---

## 40. **kill** — 프로세스 종료 신호 전송

**형식**

```bash
kill [옵션] PID
```

**주요 옵션**

- -SIGTERM 정상 종료 요청

- -9 강제 종료 SIGKILL

- -l 신호 목록


**예시**

```bash
kill 1234
```
- PID 1234에 종료 요청

```bash
kill -9 5678
```
- PID 5678 강제 종료

```bash
kill -HUP 1111
```
- PID 1111 설정 재로딩(HUP) 유도

---

## 41. **pkill** — 이름/조건으로 프로세스에 신호 전송

**형식**

```bash
pkill [옵션] 패턴
```

**주요 옵션**

- -f 전체 명령줄 매칭

- -u 사용자 필터

- -9 SIGKILL


**예시**

```bash
pkill nginx
```
- nginx라는 이름의 프로세스 종료 요청

```bash
pkill -f 'python app.py'
```
- 명령줄에 패턴이 포함된 프로세스 종료

```bash
pkill -u www-data nginx
```
- 특정 사용자 소유 nginx만 종료

---

## 42. **xargs** — 표준입력을 인수로 변환해 명령 실행

**형식**

```bash
xargs [옵션] 명령
```

**주요 옵션**

- -0 널 구분자 입력

- -n N개씩 나눠 실행

- -I{} 자리표시자 사용


**예시**

```bash
printf 'a\nb\nc' | xargs echo
```
- a b c를 인수로 전달하여 한 줄 출력

```bash
find . -name '*.log' -print0 | xargs -0 rm -f
```
- 널 구분자로 안전하게 삭제

```bash
cat list.txt | xargs -n 1 wget -q
```
- URL 목록을 한 줄씩 wget 실행

---

## 43. **man** — 매뉴얼 페이지 보기 Manual

**형식**

```bash
man [섹션] 명령
```

**주요 옵션**

- -k 키워드 검색(apropos)

- -f 간단 설명(whatis)


**예시**

```bash
man grep
```
- grep의 공식 매뉴얼 페이지 열기

```bash
man 5 crontab
```
- 섹션 5 포맷 문서(crontab 파일 형식) 보기

```bash
man -k archive
```
- 아카이브 관련 명령 검색

---

## 44. **alias** — 명령 별칭 설정

**형식**

```bash
alias 이름='명령'
```

**예시**

```bash
alias ll='ls -alF'
```
- ll 입력만으로 상세/형식 표시 목록

```bash
alias gs='git status'
```
- git status를 gs로 단축

```bash
alias rm='rm -i'
```
- rm 사용 시 항상 확인 받기

---

## 45. **unalias** — 별칭 해제

**형식**

```bash
unalias [옵션] 이름
```

**주요 옵션**

- -a 모든 별칭 제거


**예시**

```bash
unalias ll
```
- ll 별칭 제거

```bash
unalias -a
```
- 현재 셸의 모든 별칭 제거

```bash
unalias gs || true
```
- 없어도 에러 무시하고 진행

---

## 46. **history** — 명령 기록 조회/관리

**형식**

```bash
history [옵션]
```

**주요 옵션**

- -c 지우기

- -d N 특정 항목 삭제


**예시**

```bash
history | tail
```
- 최근 실행한 명령 몇 줄 확인

```bash
history -d 100
```
- 100번째 기록 삭제

```bash
history -c
```
- 전체 히스토리 초기화

---

## 47. **env** — 환경 변수 조회/실행 환경 지정

**형식**

```bash
env [변수=값]... [명령]
```

**주요 옵션**

- -i 빈 환경으로 실행


**예시**

```bash
env | sort
```
- 현재 환경 변수 목록 표시

```bash
env PATH=/custom/bin:$PATH mycmd
```
- 일시적으로 PATH를 바꿔 실행

```bash
env -i sh -c 'echo $PATH'
```
- 빈 환경으로 셸을 실행

---

## 48. **export** — 환경 변수 내보내기(자식 프로세스에 전달)

**형식**

```bash
export 변수=값
```

**주요 옵션**

- -p 현재 내보낸 변수 표시


**예시**

```bash
export JAVA_HOME=/opt/jdk
```
- JAVA_HOME 환경 변수 설정

```bash
export PATH=$HOME/bin:$PATH
```
- 사용자 bin을 PATH 앞에 추가

```bash
export -p | grep JAVA_HOME
```
- 내보낸 변수 목록에서 JAVA_HOME 확인

---

## 49. **echo** — 문자열 출력

**형식**

```bash
echo [옵션] 문자열
```

**주요 옵션**

- -n 끝의 개행 생략

- -e 백슬래시 이스케이프 해석


**예시**

```bash
echo Hello
```
- Hello 출력 후 개행

```bash
echo -n 'No newline'
```
- 개행 없이 출력

```bash
echo -e 'A\nB'
```
- 이스케이프를 해석해 줄바꿈 포함 출력

---

## 50. **printf** — 포맷 지정 출력

**형식**

```bash
printf 포맷 [인수...]
```

**예시**

```bash
printf '%s %d\n' user 3
```
- 문자열과 정수를 형식에 맞게 출력

```bash
printf '%.2f\n' 3.14159
```
- 소수점 둘째 자리까지 반올림 출력

```bash
printf '%-10s | %5d\n' name 42
```
- 좌/우 정렬 폭 지정 출력

---

## 51. **date** — 시각 표시/설정

**형식**

```bash
date [옵션] [+포맷]
```

**주요 옵션**

- -u UTC 기준

- -d 입력 시각 해석

- -s 시스템 시각 설정


**예시**

```bash
date '+%Y-%m-%d %H:%M:%S'
```
- 지정 형식으로 현재 시각 출력

```bash
date -u
```
- UTC 기준 현재 시각 출력

```bash
date -d '2025-01-01 12:00' '+%s'
```
- 해당 시각의 epoch 초 계산

---

## 52. **cal** — 달력 출력

**형식**

```bash
cal [월] [년]
```

**주요 옵션**

- -y 전체 연도 달력

- -3 이전/다음 달 포함


**예시**

```bash
cal
```
- 현재 달의 달력 출력

```bash
cal 12 2025
```
- 2025년 12월 달력 출력

```bash
cal -y 2026
```
- 2026년 1~12월 달력 출력

---

## 53. **uname** — 시스템 정보 출력

**형식**

```bash
uname [옵션]
```

**주요 옵션**

- -a 모든 정보

- -r 커널 릴리스

- -m 머신 하드웨어


**예시**

```bash
uname -a
```
- 커널/호스트/아키텍처 등 전체 정보 표시

```bash
uname -r
```
- 커널 버전 표시

```bash
uname -m
```
- 머신 아키텍처(x86_64 등) 표시

---

## 54. **hostname** — 호스트명 조회/설정

**형식**

```bash
hostname [옵션] [이름]
```

**주요 옵션**

- -I IP 주소들

- -f FQDN 전체 도메인명


**예시**

```bash
hostname
```
- 현재 호스트명 출력

```bash
hostname -I
```
- 호스트에 할당된 IP 리스트 출력

```bash
sudo hostname new-host
```
- 호스트명을 일시적으로 변경

---

## 55. **ping** — 네트워크 연결 확인 ICMP Echo

**형식**

```bash
ping [옵션] 대상
```

**주요 옵션**

- -c 횟수 지정

- -i 간격

- -W 타임아웃


**예시**

```bash
ping -c 4 8.8.8.8
```
- 구글 DNS에 4회 패킷 전송 테스트

```bash
ping -c 3 example.com
```
- 도메인 이름으로 연결 확인

```bash
ping -i 0.2 -c 5 1.1.1.1
```
- 간격 0.2초로 5회 빠르게 테스트

---

## 56. **curl** — URL로 데이터 전송/다운로드 Client URL

**형식**

```bash
curl [옵션] URL
```

**주요 옵션**

- -L 리다이렉트 따라가기

- -o 파일로 저장

- -I 헤더만 요청

- -d 데이터 POST


**예시**

```bash
curl https://example.com
```
- HTML을 표준출력으로 가져오기

```bash
curl -L -o page.html http://example.com
```
- 리다이렉트 따라가 파일 저장

```bash
curl -X POST -d 'a=1&b=2' https://httpbin.org/post
```
- 폼 데이터를 POST

---

## 57. **wget** — 비대화식 네트워크 다운로드 World GET

**형식**

```bash
wget [옵션] URL
```

**주요 옵션**

- -O 파일명 지정

- -c 이어받기

- -r 재귀 다운로드

- --no-check-certificate 인증서 무시


**예시**

```bash
wget https://example.com/file.zip
```
- 현재 디렉터리에 파일 저장

```bash
wget -O latest.html https://example.com
```
- 파일명을 지정해 저장

```bash
wget -c big.iso
```
- 중단된 다운로드를 이어서 받기

---

## 58. **ssh** — 원격 셸 접속 Secure Shell

**형식**

```bash
ssh [옵션] 사용자@호스트 [명령]
```

**주요 옵션**

- -p 포트 지정

- -i 개인키 지정

- -L 로컬 포워딩


**예시**

```bash
ssh user@server
```
- 원격 서버에 셸 접속

```bash
ssh -i ~/.ssh/id_rsa user@server
```
- 특정 키로 인증하여 접속

```bash
ssh -L 8080:localhost:80 user@server
```
- 로컬 8080을 원격 80으로 포워딩

---

## 59. **scp** — SSH 기반 파일 복사 Secure Copy

**형식**

```bash
scp [옵션] 원본 대상
```

**주요 옵션**

- -P 포트

- -i 키 파일

- -r 디렉터리 재귀


**예시**

```bash
scp file.txt user@server:/tmp/
```
- 로컬 파일을 원격 /tmp로 업로드

```bash
scp -r site/ user@server:/var/www/
```
- 디렉터리를 재귀 업로드

```bash
scp -P 2222 user@server:/var/log/syslog .
```
- 특정 포트로 원격 파일 다운로드

---

## 60. **sudo** — 권한 상승하여 명령 실행 Superuser Do

**형식**

```bash
sudo [옵션] 명령
```

**주요 옵션**

- -v 자격 갱신

- -k 자격 무효화

- -u 사용자 지정


**예시**

```bash
sudo apt update
```
- 관리자 권한이 필요한 패키지 인덱스 갱신

```bash
sudo -u www-data ls /var/www
```
- 다른 사용자 권한으로 명령 실행

```bash
sudo -k && sudo whoami
```
- 캐시 무효화 후 다시 인증 요구

---

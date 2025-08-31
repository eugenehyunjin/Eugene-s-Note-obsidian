DB 버전: 10.10.2-MariaDB

1. 현재 열려있는 DB 연결 수 체크 명령어
	SHOW STATUS LIKE 'Threads_connected';
2. 실제로 얼마나 많은 연결이 이루어졌는지(과거 이력) 체크
	SHOW STATUS LIKE 'Connections';
3. 현재 연결 상태 확인(sleep 확인 가능)
	SHOW PROCESSLIST;
4. 새로운 연결 수락할 수 있는 상태인지 확인
	SHOW STATUS LIKE 'Aborted_connects';
5. 현재까지 최대 동시 연결 수 확인
	SHOW STATUS LIKE 'Max_used_connections'
6. 전체 설정 변수 확인
	SHOW VARIABLES;


02/12 DB 상태 확인
**최대 동시 접속자 수(`Max_used_connections`)**: **572**  
시스템 설정(`max_connections`): 10000
wait_timeout = 28800 (8시간)으로 설정
Aborted_connects(연결 실패 수) = 11,830개
총 연결 시도 횟수 (매우 많음) = 54,147,174번
02/12 오전 10시 기준, 연결 수(Threads_connected) 144개 중, 1개를 제외하고 모두 sleep 상태

 
기존 실 서버 코드
<?php
	@header('Content-Type: text/html; charset=utf-8');

	include (AC_ROOT."lib/connect.cab.php");		// 연결 정보 파일을 불러들인다
	$mysql = new mysqli($host,$sqlid,$sqlpw,$db);		// DB 연결
	if(is_object($mysql)){
		$mysql->query("SET names utf8");
	}
	//$mysql->query("SET charset utf8");

	$pdo = new PDO("mysql:host=".$host.";dbname=".$db, $sqlid, $sqlpw, [PDO::MYSQL_ATTR_COMPRESS => true]);

	// DB 내용이 유출되지 않도록 초기화
	$host = null;
	$sqlid = null;
	$sqlpw = null;
	$db = null;
?>


[12-Feb-2025 17:01:35 Asia/Seoul] PHP Fatal error:  Uncaught mysqli_sql_exception: You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near ')' at line 1 in D:\www\home\lib\config.php:312
Stack trace:
#0 D:\www\home\lib\config.php(312): mysqli->query('SELECT * from f...')
#1 D:\www\home\arch\danji_info.php(31): getQueryRows('SELECT * from f...')
#2 D:\www\home\arch\index.php(125): include('D:\\www\\home\\arc...')
#3 {main}
  thrown in D:\www\home\lib\config.php on line 312


03/05 오류 재 발생 - 처리 방법
[05-Mar-2025 09:01:58 Asia/Seoul] PHP Warning:  mysqli::__construct(): (HY000/2002): 각 소켓 주소(프로토콜/네트워크 주소/포트)는 하나만 사용할 수 있습니다 in D:\www\home\lib\mysql.php on line 6
[05-Mar-2025 09:01:58 Asia/Seoul] MySQL 연결 오류: 연결 실패: 각 소켓 주소(프로토콜/네트워크 주소/포트)는 하나만 사용할 수 있습니다

241 서버에서 netstat -ano | findstr :3306 명령어로 확인 결과 아래의 소켓 문제 발생
TCP 203.235.19.241:62494 203.235.19.245:3306 TIME_WAIT 0의 중복적인 발생

아래 명령어를 통해 실제 사용중인 소켓의 time wait 시간을 조정
netsh int tcp set global timewaittimeout=30

mysql 에서도 관련된 세팅 값 변경(5분)
SET GLOBAL wait_timeout = 300; 
SET GLOBAL interactive_timeout = 300;

평소의 netstat -ano | findstr :3306 명령어를 통해 time_wait이 이루어지고 있는지 체크해보고,  필요한 경우 위의 설정 값을 통해서 처리되는지 테스트 필요


0311 원인 확인
netstat -ano | findstr TIME_WAIT | find /c ":" 현재 time_wait 상태의 소켓 숫자 확인
netstat -ano | find /c ":" 현재 열려있는 소켓 숫자 확인
netsh int ipv4 show dynamicport tcp 현재 동적 포트 범위 확인

현재 241 서버의 동적 포트 범위: 시작 포트: 49152 포트 수: 16384
설정 값은 최적으로 보여짐
### **🔎 TIME_WAIT 개수가 많을 때 문제가 되는 경우**

1️⃣ **TIME_WAIT 개수가 동적 포트 수의 20~30% 이상이면 위험**

- 현재 동적 포트 범위: **49152 ~ 65535 (총 16384개)**
- 현재 TIME_WAIT 포트 개수: **2817개 (약 17%)** 03/11 오전 9시 30분 기준  
    → **아직 임계점(20~30%)까지는 아니지만, 포트 사용량이 많아지면 문제가 발생할 가능성이 있음.**

executed in


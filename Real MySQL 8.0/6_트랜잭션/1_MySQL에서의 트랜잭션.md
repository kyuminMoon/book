# MySQL에서의 트랜잭션

## InnoDB와 MyISAM의 차이
```SQL
    CREATE TABLE tab_myisam(fdpk INT NOT NULL, PRIMARY KEY(fdpk)) ENGINE=MyISAM;
    INAWER INRO tab_myisam (fdpk) VALUES (3);
    
    CREATE TABLE tab_innodb(fdpk INT NOT NULL, PRIMARY KEY(fdpk)) ENGINE=INNODB;
    INAWER INRO tab_innodb (fdpk) VALUES (3);
    
    -- AUTO-COMMIT 활성화
    SET autocoomit = ON;    
    
    INAWER INRO tab_myisam (fdpk) VALUES (1),(2),(3);
    INAWER INRO tab_innodb (fdpk) VALUES (1),(2),(3);
    
    -- ERROR 1062 Duplicate entry 3 for key PRIMARY
    
    SELECT * FROM tab_myisam; -- 1,2,3
    SELECT * FROM tab_innodb; -- 3

```

MyISAM에선 오류가 발생했음에도 1,2가 들어갔지만, InnoDB에선 3만 저장된다.
MEMORY 스토리지 엔진을 사용하는 테이블도 MyISAM 테이블과 동일하게 작동.
MyISAM 테이블에서 발생하는 이러한 현상을 부분 업데이트(Partial Update)라고 표현하며, 데이터 정합성을 맞추는데 어려운 문제를 만듦.

트랜잭션이 지원하지 않는 MyISAM에선 해당 기능은 꽤나 골치아픈 문제를 낳기도...

## 주의사항
트랜잭션 범뮈은 최소화 하는 것이 좋다.

> 1. 처리 시작 <br>
> => 데이터베이스 커넥션 생성 <br>
> => 트랜잭션 시작
> 2. 사용자의 로그인 여부 확인
> 3. 사용자의 글쓰기 내용의 오류 여부 확인
> 4. 첨부로 업로드된 파일 확인 및 저장
> 5. 사용자의 입력 내용을 DBMS에 저장
> 6. 첨부 파일 정보를 DBMS에 저장
> 7. 저장된 내용 또는 기타 정보를 DBMS에서 조회
> 8. 게시물 등록에 대한 알림 메일 발송
> 9. 알림 메일 발송 이력을 DBMS에 저장 <br>
> <= 트랜잭션 종료(COMMIT) <br>
> <= 데이터베이스 커넥션 반납
> 10. 처리 완료

다음 예제에서 DBMS의 트랜잭션 처리에 좋지 않은 영향을 미치는 부분을 살펴보자.

많은 사용자가 커넥션을 생성하는 코드를 1번과 2번 사이에 구현하며, 그와 동시에 START TRANSACTION한다.
그리고 9,10번 사이에서 트랜잭션을 COMMIT하고 커넥션 종료한다. 
실제 DBMS에 데이터를 저장하는 작업(트랜잭션)은 5번 부터 시작되는 것을 알 수 있다.
그래서 2번과 3번, 4번의 절차가 아무리 빨리 처리된다 하더라도 DBMS의 트랜잭션에 포함시킬 필요는 없다.

더 큰 위험은 8번 작업. 
메일 전송이나 FTP 파일 전송 작업 또는 네트워크를 통해 원격 서버와 통신하는 등과 같은 작업은 DBMS 트랜잭션 내에서 제거하는 것이 옳다.
실행되는 동안 메일 서버와 통신할 수 없는 상황이 발생한다면 웹 서버 뿐 아니라 DBMS 서버까지 위험해지는 상황이 발생할 것.

DBMS의 작업이 이 작업엔 크게 4개가 있다. 
사용자가 입력한 정보를 저장하는 5,6번 작업은 반드시 하나의 트랜잭션으로 묶어야하며, 7번은 저장된 데이터의 단순 확인 및 조회이므로 트랜잭션에 포함될 필요는 없다. 
9번은 작업 성격이 다르기 때문에 이전 트랜잭션(5,6)에 함께 묶지 않아도 무방해 보인다.
이러한 작업은 별도의 트랜잭션으로 분리하는 것이 좋다.
7번은 별도로 트랜잭션을 사용하지 않아도 무방해 보인다.


> 1. 처리 시작
> 2. 사용자의 로그인 여부 확인
> 3. 사용자의 글쓰기 내용의 오류 여부 확인
> 4. 첨부로 업로드된 파일 확인 및 저장 <br>
> => 데이터베이스 커넥션 생성<br>
> => 트랜잭션 시작
> 5. 사용자의 입력 내용을 DBMS에 저장
> 6. 첨부 파일 정보를 DBMS에 저장 <br>
> <= 트랜잭션 종료(COMMIT)
> 7. 저장된 내용 또는 기타 정보를 DBMS에서 조회
> 8. 게시물 등록에 대한 알림 메일 발송 <br>
> => 트랜잭션 시작
> 9. 알림 메일 발송 이력을 DBMS에 저장 <br>
> <= 트랜잭션 종료(COMMIT)
> <= 데이터베이스 커넥션 반납
> 10. 처리 완료


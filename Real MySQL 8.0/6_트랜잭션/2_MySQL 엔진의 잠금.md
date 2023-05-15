# MySQL 엔진의 잠금
## 스토리지 엔진 레벨 
스토리지 엔진 간 상호 영향을 주지 않음.

## MySQL 엔진 레벨 
- 스토리지 엔진을 제외한 나머지 부분.모든 스토리지 엔진에 영향을 미침
- 테이블 데이터 동기화를 위한 테이블 락
- 테이블 구조를 잠그는 메타데이터 락(Metadata Lock)
- 사용자의 필요에 맞게 사용할 수 있는 네임드 락(Named Lock) 잠금 기능 제공.

1. 글로벌 락
FLUSH TABLES WITH READ LOCK 명령으로 획득.
SELECT를 제외한 DDL, DML 문장을 실행하는 경우 글로벌 락이 해제될 때까지 해당 문장이 대기 상태로 남음.
MySQL 서버 전체에 영향을 미치며, 작업 대상 테이블, 데이터베이스가 다르더라도 동일하게 영향.
MyISAM이나 MEMORY 테이블에 대해 mysqldump로 일괄 백업을 받아야 할 때 글로벌 락 사용.

하지만 InnoDB 스토리지 엔진에선 트랜잭션을 지원하기 때문에 일관된 데이터 상태를 위해 모든 데이터 변경 작업을 멈출 필요는 없다.
8.0부턴 Xtrabackup이나 Enterprise Backup과 같은 백업 툴들의 안정적인 실행을 위해 백업 락이 도입됐다.

```sql
LOCK INSTANCE FOR BACKUP;
UNLOCK INSTANCE;
```

특정 세션에서 백업 락을 획득하면 모든 세션에서 다음과 같이 테이블의 스키마나 사용자의 인증 관련 정보를 변경할 수 없게 된다.

- 데이터베이스 및 테이블 등 모든 객체 생성 및 변경, 삭제
- REPAIR TABLE과 OPTIMIZE TABLE 명령
- 사용자 관리 및 비밀번호 변경

백업 락은 일반적인 테이블의 데이터 변경은 허용된다.

MySQL은 Source server와 Replica server로 구성되는데, 주로 백업은 Replica server에서 실행된다.
Xtrabackup이나 Enterprise Backup 툴들 모두 복제 도중 스키마 변경이 실행되면 백업은 실패.

2. 테이블 락
개별 테이블 단위로 설정되는 잠금. 명시적, 묵시적 방법으로 획득 가능. 
명시적 - LOCK TABLES table_name [READ | WRITE]
명시적 해제 - UNLOCK TABLES

특별한 상황이 아니면 명시적 락은 사용할 필요가 거의 없다.
글로벌 락과 동일하게 온라인 작업에 상당한 영향을 미치기 때문.

묵시적 락 - MyISAM이나 MEMORY 테이블에 데이터를 변경하는 쿼리를 실행하면 발생.
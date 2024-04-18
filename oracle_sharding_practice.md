오라클 데이터베이스19c 오라클 샤딩 길라잡이
========================================

순서
--------------------------------------
1. [오라클 샤딩 개요](#1.오라클-샤딩-개요)   
2. [오라클 샤딩 아키텍처 및 개념](#2.오라클-샤딩-아키텍처-및-개념)   
3. [샤딩 스키마 객체들](#3.샤딩-스키마-객체)   
4. [오라클 샤딩 데이터베이스의 토폴로지](#4.오라클-샤딩-데이터베이스의-토폴로지)   
5. [오리클 샤딩 데이터베이스 배포 로드맵](#5.오리클-샤딩-데이터베이스-배포-로드맵)   
6. [샤딩 스키마 설계](#6.샤딩-스키마-설계)   
7. [샤딩 DDL](#7.샤딩-DDL)   
8. [클라이언트 요청 라우팅 및 쿼리, DML 실행](#8.클라이언트-요청-라우팅-및-쿼리,-DML-실행)   
9. [애플리케이션 APIs](#9.애플리케이션-APIs)   
10. [샤드 데이터베이스 관리](#10.샤드-데이터베이스-관리)   
11. [샤드 샘플 시나리오](#11.오라클-샤딩-샘플-시나리오)

-------------------------------------

> ### 1.오라클 샤딩 개요

오라클 샤드 데이터베이스는 분산 데이터베이스 운영을 위한 통합 환경으로써 오라클 데이터베이스를 기반으로 여러개의 기술 컴포넌트들로 구성된 통합 아키텍처 및 토롤로지 환경임을 인식해야한다.

아울러 어플리케이션의 데이터 분산, HA 등의 요건에 맞추어 시스템 설계부터 샤딩 방식, 샤드 HA 설계가 이루어져야 함을 유념해야 한다. 뿐만 아니라 샤딩 구조의 효과를 최대화하기 위하여 샤딩 구조에 적합한 모델링, 싱글샤드 쿼리, 멀티샤드 쿼리 등과 같은 특성을 먼저 이해해야 할 것이다.

오라클 샤딩 데이터베이스는 여러 개의 물리적으로 분산된 데이터베이스에 데이터를 자동 분산, 운영할수 있는 특수한 오라클 데이터베이스 배포 환경을 말한다.

샤드된 데이터베이스(샤딩 데이터베이스)는 여러 샤드가 하드웨어나 소프트웨어를 공유하지 않는 비공유 아키텍처로 구성되며 모든 샤드는 샤딩된 데이터베이스 상에서 싱글의 논리적 데이터베이스 구조를 갖는다.

애플리케이션에서는 샤딩 데이터베이스는 싱글 데이터베이스처럼 보이며 모든 샤드에 분포되는 데이터는 애플리케이션에 완전히 투명하게 작동된다.

데이터베이스 관리자의 관점에서는 실제 물리적으로 분할된 다수의 데이터베이스(인스턴스)로 구성되지만 통합적으로 관리할 수 있는 환경을 제공한다.


> ### 2.오라클 샤딩 아키텍처 및 개념

#### 샤딩 주요 컴포넌트들

#### 오라클 샤딩 데이터베이스(Oracle Sharded Database)  

오라클 샤딩 데이터베이스(a.k.a 오라클 샤딩)은 서버 또는 소프트웨어의 공유없이 물리적으로 분리(지리, 공간)된 오라클 데이터베이스(샤드)로 구성된 풀 전체에 데이터가 수평으로 분할 저장된 싱글의 논리적 데이터베이스로써 디렉터를 포함한 모든 샤드들의 집합체를 뜻한다.

<img src="https://docs.oracle.com/en/database/oracle/oracle-database/19/shard/img/admin_3v_143d.png" title="오라클 샤딩 데이터베이스" height="" width=""></img>

#### 샤드

샤드는 독립적인 오라클 인스턴스이며 샤딩 데이터베이스에서 물리적으로 분리된 서버에 샤드 키 기반으로 분리된 데이터 셋을 관리하는 오라클 데이터베이스를 말한다. 샤드의 배치는 모드 한 지역에 배치될수 도 있고 지리적으로 다른 위치에 배치될 수 도 있다. 아울러 샤드별로 고가용성 및 재해복구를 위한 복제 샤드를 구성하여 샤딩 데이터베이스 환경에 통합할 수 있다.

#### 샤드 카탈로그 데이터베이스(또는 샤드 카탈로그)

샤딩 데이터베이스의 구성 정보를 영구 저장 및 유지하고 샤딩 데이터베이스의 중앙집중식 관리를 위한 특수목적용 오라클 데이터베이스이다. 샤딩 데이터베이스의 구성 및 변경, 샤드들의 중앙화 관리, 샤드의 제거 및 추가, 글로벌 서비스 변경 등의 정보관리 및 역할을 수행한다.


애플리케이션이 키 기반 쿼리를 사용하지 않을 경우 특정 샤드에 쿼리를 전달하려면 샤드 카탈로그가 필요하며, 샤드 키 기반 트랜잭션은 샤딩 데이터베이스에 의해 계속 라우팅되고 실행되며 카탈로그 중단의 영향을 받지 않는다.

샤드 카탈로그의 용도

- 전체 샤드 및 샤딩 토폴로지에 대한 관리 서버 역할 수행
- 데이터베이스의 모든 스키마의 골드 복사본 저장
- 모든 듀플리케이트 테이블 데이터의 골드 복사본 저장
- 샤드에서 실행되는 모든 DDL은 샤딩 데이터베이스에 연결하여 실행
- 멀티 샤드 쿼리와 샤딩키 없는 쿼리를 처리하는 코디네이터 역할 수행

#### 샤드 디렉터

데이터가 저장된 샤드로 연결 라우팅을 제공하는 네트워크 리스너로써 오라클 클라이언트가 연결 요청하면 전달된 샤딩 키 기반으로 적절한 샤드로 연결을 라우팅 한다. 오라클 샤딩에서는 글로벌 서비스 관리자(GSM, 12c부터 지원)를 샤드 디렉터라고 하며, 샤딩 데이터베이스의 현재 토폴로지 맵의 유지관리를 수행한다. 샤드 디렉터는 특정 리전에 최대 5개까지 배포가 가능하며 각 리전에 해당 리전용 디렉터를 구성 할 수 있다.

샤드 디렉터의 역할

- 샤드 데이터베이스 구성 및 가용한 샤드에 대한 런타임으로 데이터 유지
- 로컬과 타 리전 간의 네트워크 대기(latency)시간 측정
- 샤드 데이터베이스에 접속하는 클라이언트용 리전 리스너 수행
- 글로벌 서비스 관리
- 접속에 대한 로드발란싱 수행

#### 글로벌 서비스 메니지먼트

Oracle 12c에 도입 기능으로 데이터베이스의 역할, 부하, 복제 지연 및 지역성을 기반으로 데이터베이스의 연결을 라우팅하는 역할을 제공한다. 오라클 샤딩에서는  데이터 위치를 기반으로 연결 라우팅을 지원한다. 즉, 글로벌 서비스 관리자를 샤드 디렉터라고 한다. GSM 소프트웨어를 필요로 한다.

#### 쿼리 코디네이터

쿼리 코디네이터는 샤드 카탈로그의 컴포넌트로써 샤딩 데이터베이스에 대한 쿼리 처리를 제공한다. 즉, 프록시 기반 연결 및 쿼리, DML 처리를 조정한다. 샤드 카탈로그의 SQL 컴파일러에서 샤드를 자동 식별하고 참여하는 모든 샤드에서 쿼리 실행을 조정하게 된다.


쿼리 코디네이터가 관여하는 샤드 쿼리

1. 샤딩 키가 없는 싱글 샤드 쿼리 : 애플리케이션에서 샤딩키가 전달되지 않으면 쿼리에 필요한 데이터가 포함된 샤드를 찾고 실행을 위해 쿼리를 해당 샤드로 보내는 역할을 한다.

2. 멀티 샤드 쿼리 : 둘 이상의 샤드의 데이터 처리가 필요한 멀티 샤드 쿼리를 처리하는 역할을 한다.

3. 집계 쿼리 : SUM, AVG와 같은 일반적인 통합 집계쿼리를 처리하는 역할을 한다.

쿼리 코디네이터의 SQL 컴파일러는 쿼리에 관계되는 샤드를 자동으로 식별하고 참여하는 모든 샤드에서 쿼리 실행을 조정한다.

싱글 샤드 쿼리인 경우에는 전체 쿼리가 해당 싱글 샤드에서 실행되고 쿼리 코디네이터는 처리된 행을 클라이언트에 다시 전달하기만 한다.

멀티 샤드 쿼리인 경우에는 클라이언트에서 들어온 쿼리를 SQL 컴파일러는 쿼리를 분석하고 참여 샤드에서 실행되는 쿼리 조각으로 다시 만들어지고 만들어진 쿼리는 해당 샤드에서 수행된 다음 코디네이터에 의해 집계되도록 쿼리가 다시 작성된다.

쿼리 코디네이터 접속 방법

```os
sqlplus sdteat/Welcome1@sdcat:1521/GDS\$CATALOG.oradbcloud
```

쿼리 코디네이터 작동 개념

쿼리 Q가 들어오면 코디네이터가 CQ와 SQ로 두개의 쿼리로 재작성된다. SQ는 각 샤드에서 실행될 쿼리이며 CQ는 코디네이터 샤드로서 CQ와 Shard_Iterator로 재작성된다. SHard_Iterator는 샤드에 연결하고 SQ를 실행하는 연산자를 뜻한다.

<img src=https://docs.oracle.com/en/database/oracle/oracle-database/19/shard/img/cross_shard_query.png title="쿼리 코디네이터 작동 개념" hight="" width=""></img>

코디네이터에 의한 싱글 샤드 쿼리 처리

싱글 샤드 쿼리는 클라이언트가 해당 샤드에 연결하고 해당 샤드에 대해 쿼리를 실행하는 것과 유사하다. 싱글 샤드 쿼리는 해당 샤드에서만 데이터를 스캔하고 다른 샤드에서는 데이터를 조회할 필요가 없는 쿼리이다.  다음 쿼리는 CUSTNO를 샤드키로 사용한 경우이며 이 쿼리는 "123"이 있는 해당 샤드에서만 처리될 것이다.

```sql
SELECT count(*) FROM customers c, orders o WHERE c.custno = o.custno and c.custno = 123;
```

싱글 샤드 쿼리 지원 구문
- SELECT, UPDATE, DELETE, INSERT, FOR UPDATE 구문 지원
- = 및 in-list 지원
- 리터럴, 바인드 또는 리터럴 및 바인드의 조건 연산을 지원
- MERGE.UPSERT 미지원

```sql
Area = :bind

Area = CASE :bind <10 THEN ‘West’ ELSE ‘East’ END
```

코디네이터에 의한 멀티 샤드 쿼리 처리

멀티샤드 쿼리는 둘 이상의 샤드에서 데이터를 스캔해야 하는 쿼리이며 각 샤드의 처리는 다른 샤드와 독립적이다.

멀티샤드 쿼리는 둘 이상의 샤드에 매핑되며 코디네이터는 결과를 클라이언트에 보내기 전에 일부 처리를 수행해야 할 수도 있다.

다음 쿼리는 각 고객의 주문 수를 가져오는 쿼리인데 쿼리 코디네이터에 의해 변환 수행된다.

```sql
SELECT count(*), c.custno FROM customers c, orders o WHERE c.custno = o.custno
 GROUP BY c.custno;
```

```sql
SELECT sum(count_col), custno FROM (SELECT count(*) count_col, c.custno
 FROM customers c, orders o
 WHERE c.custno = o.custno GROUP BY c.custno) GROUP BY custno;
```

인라인 쿼리 블록은 원격 매핑 쿼리 블록과 마찬가지로 모든 샤드에 매핑된다. 코디네이터는 GROUP BY모든 샤드의 결과 집합을 바탕으로 추가 집계를 수행하게된다. 모든 샤드의 실행 단위는 인라인 쿼리 블록이다.

> ### 3.샤딩 스키마 객체

#### 샤드 테이블

오라클 샤드 데이터베이스의 샤드 테이블은 샤드 키로 파티션된 테이블로써 동일 샤드키의 로우 데이터 집합이 저장되는 테이블이다. 샤드 테이블의 각 파티션은 샤딩 키를 기반으로 각 샤드의 테이블스페이스에 분산된다.

샤딩 키로 사용 가능한 데이터 타입은 다음과 같다.

- NUMBER
- INTEGER
- SMALLINT
- RAW
- (N)VARCHAR
- (N)VARCHAR2
- (N)CHAR
- DATE
- TIMESTAMP

아래는 샤드 테이블을 생성하는 구문 예제이다. 파티션 키가 "gld", "slv"인 List 파티션으로 분산되는 샤드 테이블이다.

```sql
CREATE SHARDED TABLE customers
( cust_id     NUMBER NOT NULL,
    name        VARCHAR2(50),
    address     VARCHAR2(250),
    location_id VARCHAR2(20),
    class       VARCHAR2(3),
    signup_date DATE,
   CONSTRAINT cust_pk PRIMARY KEY(cust_id, class)
)
PARTITIONSET BY LIST (class)
PARTITION BY CONSISTENT HASH (cust_id)
PARTITIONS AUTO
(PARTITIONSET gold   VALUES (‘gld’) TABLESPACE SET ts2,
 PARTITIONSET silver VALUES (‘slv’) TABLESPACE SET ts1)
;
```

#### 샤드 테이블 패밀리

샤드 키로 동일하게 파티션된 테이블들의 집합으로써 상위 테이블의 기본키(Primary key)를 참조하는 하위 테이블(foreign key)의 참조 제약 조건을 사용하여 테이블간의 Parent-Child 관계로 이루어진다. 이러한 관계로 연결된 여러 테이블들은 모든 하위 항목이 싱글 상위 항목을 갖는 트리 구조를 형성하게 된다. 테이블 패밀리의 최상위 테이블을 루트 테이블이라고 하며 테이블 패밀리에는 로트 테이블 하나만 있을 수 있다.

오라클 샤딩에서 시스템 관리 샤딩 방식에서는 멀티 테이블 패밀리가 가능하고, 사용자지정 샤딩 방식 및 복합 샤딩방식에서는 오직 싱글 테이블 패밀리만 지원 된다.

새로운 테이블 패밀리를 생성하기 위해서는 루트 테이블을 생성할때 사용하지 않는 새로운 테으블스페이스를 지정해야 한다. 각 테이블 패밀리는 루트 테이블로 식별되며 서로 다른 테이블 패밀리에 있는 테이블이 서로 관련되어서는 안된다.

각 테이블 패밀리에는 자체 샤딩키가 지정되어야 하며, 테이블 패밀리 내의 하부 테이블에도 동일하게 적용된다. 즉, 서로 다른 테이블 계열의 모든 테이블은 일관된 해시를 사용하여 동일한 수의 청크로 샤딩되며, 각 청크에는 모든 테이블 계열의 데이터가 포함된다.

따라서 서로 다른 테이블 패밀리 간의 쿼리가 최소화되고 샤딩 코디네이터에서만 수행되도록 테이블 패밀리를 설계해야 한다.

멀티 테이블 패밀리 경우에는 서로 다른 글로벌 서비스와 연결되어야 한다. 어플리케이션은 자체 커넥션 풀과 서비스를 가지며 올바른 샤드로 라우팅하기 위해 샤딩 키를 사용한다.

아래 예문은 시스템관리 방식 테이블 패밀리 생성 예이다.

```sql
CREATE SHARDED TABLE Customers <=== Table Family #1
( CustId NUMBER NOT NULL
, Name VARCHAR2(50)
, Address VARCHAR2(250)
, region VARCHAR2(20)
, class VARCHAR2(3)
, signup DATE
)
PARTITION BY CONSISTENT HASH (CustId)
PARTITIONS AUTO
TABLESPACE SET ts1
;

CREATE SHARDED TABLE Orders
( OrderNo NUMBER
, CustId NUMBER
, OrderDate DATE
)
PARENT Customers
PARTITION BY CONSISTENT HASH (CustId)
PARTITIONS AUTO
TABLESPACE SET ts1
;

CREATE SHARDED TABLE Products <=== Table Family #2
( ProdId NUMBER NOT NULL,
  CONSTRAINT pk_products PRIMARY KEY (ProdId)
)
PARTITION BY CONSISTENT HASH (ProdId)
PARTITIONS AUTO
TABLESPACE SET ts_2
;
```

#### 듀플리케이트 테이블

오라클 샤딩에서 모든 샤드에 동일 데이터를 유지할 수 있도록 하는 테이블을 듀플리케이트 테이블이라고 한다. 모든 샤드에서 듀플리케이션 테이블의 데이터를 읽기 중심으로 사용할 수 있도록 구조화된다. 따라서 일반 적으로 업데이트 빈도가 낮으면서 샤드 테이블과 함께 자주 액세스되는 비교적 작은 테이블이 적합하다.  

튜플리케이트 테이블의 특징

- 비샤딩 테이블로써 모든 샤드에 복제 형태로 유지된다.
- 각 샤드에서 읽기, 쓰기가 가능하다.
- 샤드 카탈로그에서 Meterialized view를 기반으로 모든 샤드의 듀플리케이트 테이블의 변경 사항을 자동으로 복제하게 된다.

<img src="https://docs.oracle.com/en/database/oracle/oracle-database/19/shard/img/admin_3v_138e.png" title="오라클 샤드 테이블 및 듀플리케이트 테이블" width="" height=""></img>

> ### 4. 오라클 샤딩 데이터베이스의 토폴로지

샤딩 데이터베이스 토폴로지는 샤드 카탈로그 데이터베이스에 저장되는 샤딩 관련한 메타정보이다. 토폴로지 구성과 배포(deploy)를 통해 샤딩 데이터베이스 구성을 완료할 수 있게된다.

샤딩 데이터베이스 토폴로지는 샤딩 방식(메소드), 리플리케이션(HA), 디폴트 청크 수, 샤드 디렉터 갯수 및 위치, 샤드그룹의 수, 샤드 스페이스, 리전, 샤드, 샤드 데이터베이스 접속에 사용될 글로벌 서비스로 구성된다.

샤드 토폴로지 구성은  Global Data Services Control Utility (GDSCTL)를 사용한다. GDSCTL 유틸리티는 샤드 디렉터에 포함되어 있다. 이 GDSCTL 유틸리티를 사용하여 구성 절차에 따라 샤드 데이터베이스 메타정보를 등록하고 구성하게 된다.


#### 오라클 샤딩 데이터베이스 샤딩 방식

오라클 샤딩에서 지원하는 샤딩 방식(메소드)은 시스템관리 방식, 사용자지정 방식 및 이 두가지를 혼합한 복합샤딩 방식을 지원한다.

1. **시스템 관리 샤딩(System-managed shrding) 방식**

사용자가 샤드에 대한 데이터 매핑을 지정할 필요가 없는 샤딩방식이다. 버킷 수에 영향을 받지 않는 컨시스턴트 해시(전통적 해시 파티셔닝과 다름) 기반으로 DB엔진에 의해서 자동 파티션이 되고 데이터가 자동 분산된다.

파티셔닝 알로리즘에 의해 랜덤하게 데이터를 분산시켜 핫 스팟 제거 및 샤드 전체에 균일한 성능을 제공한다. 확장 가능형 분산시스템에 적합한 방식이다.

<img src="https://docs.oracle.com/en/database/oracle/oracle-database/19/shard/img/admin_3v_128a.png" title="" width="" height=""></img>

시스템관리 샤딩의 특징

- 각 샤드에 동일한 수의 청크가 할당된다.
- 샤드 추가시 일부 청크가 샤드간에 자동 재배치되어 데이터 균등성을 유지한다. 재해싱이 일어나지 않는다.
- 샤드의 총 청크수는 GDSCTL 명령으로 지정 가능하며 미지정시 샤드당 기본 120개 지정된다.
- 청크당 하나의 테이블스페이스를 사용한다.
- Tablespace SET을 사용하며 이 테이블 세트의 모든 테이블스페이스는 OMF 기반의 동일한 물리적 속성을 가진다.

2. **사용자 지정 샤딩 방식**

개별 샤드에 대한 데이터 맵핑을 사용자가 명시적을 지정하는 샤딩 방식이다.

사용자지정 샤딩 방식은 성능, 규제 및 기타 이유로 특정 위치나 서버에 특정 데이터를 분산된 샤드에 저장토록 제어가 필요한 경우에 사용하는 방식이다.

<img src="https://docs.oracle.com/en/database/oracle/oracle-database/19/shard/img/admin_3v_129a.png" title="" width="" height=""></img>

사용자지정 샤딩의 특징

- 오라클 데이터가드(DG, ADG) 복제 방식을 지원한다. OGG는 지원하지 않는다.
- 샤드 장애시 어떤 데이터를 사용할 수 없는지 인지가 가능하다.
- 관리자가 샤드들의 부하 모니터링 및 유지관리가 필요하다.
- 샤딩 테이블을 위한 Range, list 테이블 파티션을 지원한다.
- 샤딩 테이블 생성시 각 파티션을 별도의 테이블스페이스에 저장하도록 구문에 지정해야 한다. 이점을 제외하면 일반 테이블 생성 구문과 동일하다.
- Tablespace Set을 사용하지 않고 각 테이블스페이스는 개별적으로 생성되어야 하며 샤드스페이스와 명시적으로 연결되어야 한다.

3. **복합 샤딩 방식**

시스템관리 샤딩 방식과 사용자지정 방식을 조합한 방식이다.

<img src="https://docs.oracle.com/en/database/oracle/oracle-database/19/shard/img/admin_3v_131a.png" title="" width="" height="" ></img>

복합 샤딩방식의 특징

- Range, List로 분할된 샤드의 샤드스페이스에 추가적으로 컨시스턴트 해시 파티션으로 추가 분할한다.
- 각 샤드스페이스의 샤드 전체에 규등한 데이터 배포를 자동으로 유지하는 동시에 샤드 스페이스 전체에 데이터 분할이 가능하다.
- 테이블스페이스를 사용하여 샤드에 대한 파티션 맵핑을 지정한다.
- 샤딩된 테이블에 있는 데이터의 서브셋을 다른 샤드스페이스에 저장하기 위해서는 각 샤드스페이스에 별도의 테이블스페이스 세트를 생성해야한다.

#### 샤드 레벨 HA

오라클 샤딩은 샤드의 고가용성, DR과 같은 HA를 위하여 오라클 데이터베이스 복제 기술을 사용한다. 샤딩 데이터베이스의 복제 토톨로지는 GDSCTL을 사용하여 선언적으로 지정된다. 샤드 HA는 Data Guard 또는 GoldenGate를 중에 하나를 선택해야 한다.

아래는 사용자 지정 샤딩 방식에서 Data Guard를 사용한 복제 샤드를 자동 구성하는 예이다.

```sql

CREATE SHARDCATALOG -sharding user –database host00:1521:cat –region dc1,dc2,dc3

ADD GSM -gsm gsm1 -listener 1571 –catalog host00:1521:cat –region dc1
ADD GSM -gsm gsm2 -listener 1571 –catalog host00:1521:cat –region dc2
ADD GSM -gsm gsm3 -listener 1571 –catalog host00:1521:cat –region dc3
START GSM -gsm gsm1
START GSM -gsm gsm2
START GSM -gsm gsm3

ADD SHARDSPACE -shardspace shardspace_a
ADD SHARDSPACE -shardspace shardspace_b
ADD SHARDSPACE -shardspace shardspace_c

CREATE SHARD -shardspace shardspace_a –region dc1 -deploy_as primary  -destination
host01 -credential oracle_cred -netparamfile /home/oracle/netca_dbhome.rsp

CREATE SHARD -shardspace shardspace_a –region dc1 -deploy_as standby  -destination
host04 -credential oracle_cred -netparamfile /home/oracle/netca_dbhome.rsp

CREATE SHARD -shardspace shardspace_a –region dc2 -deploy_as standby  -destination
host06 -credential oracle_cred -netparamfile /home/oracle/netca_dbhome.rsp

CREATE SHARD -shardspace shardspace_a –region dc3 -deploy_as standby  -destination
host08 -credential oracle_cred -netparamfile /home/oracle/netca_dbhome.rsp

CREATE SHARD -shardspace shardspace_b –region dc1 -deploy_as primary  -destination
host08 -credential oracle_cred -netparamfile /home/oracle/netca_dbhome.rs
...

CREATE SHARD -shardspace shardspace_c –region dc3 -deploy_as standby  -destination
host10 -credential oracle_cred -netparamfile /home/oracle/netca_dbhome.rsp

DEPLOY

```
#### 파티션 및 테이블스페이스

샤드 테이블의 각 파티션은 각 샤드에 있는 지정된 테이블스페이스에 저장되며 이 테이블스페이스가 샤드 DB의 데이터 배포 단위가 된다. 멀티 샤드 조인 수를 최소화 할수 있도록 테이블 패밀리의 모든 테이블의 해당 파티션은 항상 같은 샤드에 저장된다.

#### 청크

샤드 간의 데이터 이동 단위로써 테이블 패밀리 내의 모든 테이블에서 해당 파티션들의 셋을 청크라고 한다. 시스템관리 및 복합방식의 샤딩에서 각 샤드 내의 청크 수는 샤딩 데이터베이스가 생성될때 지정된다.

<img src="https://docs.oracle.com/en/database/oracle/oracle-database/19/shard/img/admin_3v_126a.png" title="" height="" width=""></img>

#### 샤드스페이스

사용자지정 샤드 또는 복합샤드 방식에서 사용되는 샤드스페이스는 키값에 해당하는 데이터가 저장되는 샤드의 집합이다. 샤드 집합에는 복제된 샤드 세트도 포함한다. 하나의 샤드스페이스는 싱글 샤드 및 멀티 테이블스페이스로 구성될 수 있다. 생성된 테이블스페이스는 청크에 할당되지만 샤드 추가시 사용자가 직접 청크 마이그레이션을 해야한다. 각 테이블스페이스는 샤드스페이스와 명시적으로 연결되어야 한다. 아래 그림은 샤드스페이스와 테이블스페이스 관계를 나타내는 그림이다.

<img src=https://docs.oracle.com/en/database/oracle/oracle-database/19/shard/img/admin_3v_129a.png title="사용자지정 샤딩의 샤드스페이스와 테이블스페이스" height="" width=""></img>

#### 샤드그룹

샤드그룹은 시스템관리, 복합 샤딩에소 샤딩에서 복제(Replicate)의 논리 단위이다. 시스템관리 샤딩에서 샤드그룹은 샤딩 데이터베이스에 저장된 모든 데이터를 포함한다. 데이터는 샤드그룹을 구성하는 샤드 전체에 컨시스턴스 해시에 의해 분배된다. 샤드 그룹에 속하는 샤드는 일반적으로 동일한 데이터 센터에 위치한다. 전체 샤드그룹은 로컬 또는 다른 데이터 센터의 하나 이상의 샤드그룹에 복제가능하다. 

<img src="https://docs.oracle.com/en/database/oracle/oracle-database/19/shard/img/admin_3v_136a.png" title="샤드그룹" height="" width=""></img>


#### 테이블스페이스 세트

테이블스페이스 세트는 샤드 테이블 및 인덱스가 저장되는 논리적 스토리지 공간으로써 샤드스페이스의 샤드에 분산된 여러 테이블스페이스로 구성된다.
특히 시스템 관리 및 복합 샤딩 방식에서 샤드 카탈로그에 테이블스페이스 세트를 생성할 때 샤드 카탈로그와 각 샤드에 생성된 모든 테이블스페이스를 위한 충분한 공간이 있는지 확인해야 한다. 이는 계량 사용 환경에서 특히 중요하게 작용된다.

테이블스페이스 세트의 특징

- 시스템관리 샤딩인 경우 샤드 데이터베이스에 기본 샤드 공간이 하나만 존재한다.
- 테이블스페이스 수는 자동 결정되며 해당 샤드 스페이스의 청크 수와 동일하다.
- 테이블스페이스 세트의 모든 테이블스페이스는 bigfile 테이블스페이스이며 동일한 속성을 가지게 되며 속성은 TEMPLATE 구문으로 지정한다.
- 각 테이블스페이스의 데이터 파일 네임은 자동으로 지정된다.
- 시스템, 언두, 임시 테이블스페이스는 테이블스페이스 셋에 포함 되지 않는다.

#### 비샤딩 객체들의 복제

듀플리케이션 테이블 외에 user, role, view, index, synonym, fuction, procedure, package 및 tablespace, tablesacpe set, directory, context 등은 각 샤드에 복제 생성된다.

#### 글로벌 서비스

글로벌 서비스는 샤딩 데이터베이스의 데이터에 연결하는 데 사용되는 데이터베이스 서비스로써 기존 데이터베이스 서비스 뿐만 아니라 샤드 환경에서 데이터베이스 롤, 복제지연 허용(replication lag tolerance), 리전 선택 및 Primary(read-Write), standby(read only) 서비스 등을 수행한다.


> ### 5. 오리클 샤딩 데이터베이스 배포 로드맵

#### 1. 호스트 서버 준비 및 오라클 소프트웨어 설치
- 호스트 서버 준비
- 카탈로그 및 샤드 노드 서버에 오라클 데이터베이스 소프트웨어 설치
- 샤드 디렉터 노드 서버에 GSM 소프트웨어 설치

#### 2. . 카탈로그 DB 생성  
- 카탈로그 DB 생성은 DBCA 등 일반적인 방법으로 생성하면 된다.
- 카탈로드 DB는 Data guard 를 이용한 Standby DB를 구성한다.
- PDB를 이용한 카탈로그를 구성할 수 있다. 단, CDB는 부적합하다.
- 카탈로그 DB는 spfile을 반드시 사용해야 한다.
- 카탈로그, 샤드의 character set과 National character set는 도두 동일해야 한다.
- 초기화 파리미터 지정
   - OPEN_LINKS 및 OPEN_LINKS_PER_INSTANCE의 값은 샤드 수보다 같거나 커야 한다
   - DB_FILES은 청크 수, 테이블 스페이스 수 보다 같거나 커야 한다.
   - CREATE SHARD 방식으로 샤드를 추가할 경우에는 Remote Scheduler Agent를 위하여 SHARED_SERVERS 및 DISPATCHERS 값을 지정해야 한다.
   - DB_CREATE_FILE_DEST 파라미터 값을 정확히 지정한다.
   - STANDBY_FILE_MANAGEMENT 값은 AUTO로 지정한다.
- DB 기본 계정인 GSMCATUSER를 unlock 및 패스워드를 지정한다. 이 계정은 샤드 디렉터가 사용하며 카탈로그 DB 접속과 샤딩 명령어 처리를 수행한다.
- 샤드 카탈로그 관리용 계정을 만들고 권한을 부여 한다. 이 계정은 샤딩 메타 정보를 관리하는 계정으로써 샤딩 토폴로지를 변경하거나 다른 관리작업을 하는 계정이다.
- GSMCATUSER와 카탈로그 관리 계정의 패스워드는 Oracle Wallet에 저장관리된다.
- 리스너를 구성하고 카탈로그 DB 서비스를 등록한다.

#### 3. 샤드 DB 생성
- 오라클 샤딩은 샤딩 데이터베이스 구성을 위하여 샤드 및 복제 샤드를 포함한 전체 샤드를 자동 또는 수동으로 배포할 수 있다.
- 샤드를 생성하는데 사용할 수 있는 방법은 수동 추가 방법과 자동 생성 방식 2가지 중에 하나를 선택할 수 있다. 샤딩 토폴로지 구성을 진행하기 위해서는 토폴로지 구성 시작하기 전에 샤드 생성 방식을 우선 결정해야한다.   

샤드 자동 생성 방식

구성된 샤드 토폴로지를 배포할때 각 샤드(DB)를 원격에서 자동 생성하는 방식이다. 토폴로지 구성 단계에서 CREATE SHARD 명령어를 사용하여 구성하게 된다. 토폴로지가 배포되면서 카탈로그(GSM)는 Oracle Remote Scheduled Agent를 활용하여 각 샤드의 호스트에 원격으로 DBCA를 실행하여 데이터베이스를 생성하게 된다. Standby DB 생성도 가능하다. 이 방식은 PDB를 지원하지 않는다.

샤드 등록 방식(수동)

이 방식은 샤드 토폴로지 구성때 수동으로 각 샤드를 등록해주는 방법이다. 토폴로지를 구성하기 전에 각 샤드 호스트에 수동으로 DB를 생성해야 되며 필요시 Standby DB까지 구성할 수 있다. 수동방식은 샤드를 PDB로 사용할 수 있다.

샤드 DB 준비
- 샤드 DB 생성은 DBCA 등 일반적인 방법으로 생성하면 된다.
- 샤드 DB의 HA를 위한 Standby DB를 Data Guard 또는 OGG로 구성할 수 있다.
- PDB를 사용할 경우 GSMROOTUSER 계정을 활성화하고 패스워드를 지정한다. SYSDG, SYSBACKUP 권한도 부여한다. 이 계정은 샤드 디렉터 프로세스가 샤드에 접속하고 샤딩 명령어를 처리하는데 사용된다.
- 샤드 DB도 spfile을 반드시 사용해야 한다.
- 카탈로그, 샤드의 character set과 National character set는 도두 동일해야 한다.
- 초기화 파라미터 지정
   - COMPATIBLE 값은 12.2.0. 이상이어야 한다.
   - Standby DB 구성시에는 Flashback Database, FORCE LOGGING를 반드시 활성화한다.
   - DB_FILES은 청크 수, 테이블 스페이스 수 보다 같거나 커야 한다.
   - DB_CREATE_FILE_DEST 파라미터 값 정확히 지정한다.
   - DATA_PUMP_DIR은 반드시 지정하고, 오라클 기본 계정인 GSMADMIN_INTERNAL 계정에게 read, write 권한을 준다.
   - Standby DB의 디렉토리가 다른 경우에는 DB_FILE_NAME_CONVERT 를 지정한다.
   - STANDBY_FILE_MANAGEMENT 값은 AUTO로 지정한다.
- DB 기본 계정인 GSMUSER를 unlock 및 패스워드를 지정하고 SYSDG 및 SYSBACKUP 권한을 부여한다.
- 리스너를 구성하고 카탈로그 DB 서비스를 등록한다.

#### 4. 토폴로지 구성

샤딩 데이터베이스의 토폴로지 구성은 아래와 같은 순서에 따라서 구성한다.

  - 샤드 카탈로그 생성
  - 리전 등
  - 샤드 디렉터 생성 및 시작
  - 샤드스페이스 또는 샤드그룹 등록
  - 토폴로지 검증
  - 샤드 등록
  - 호스트 메타정보 등록


#### 5. 토폴로지 배포 실행 : 샤드 토폴로지 구성을 배포하는 단계

GDSCTL Deploy 명령어를 실행하면 토폴로지 각 서버에서 다음과 같은 처리가 자동 실행된다.

- GDS가 샤드 데이터베이스 토폴로지 구성을 검사하기 위한 카탈로그DB에서 PLSQL을 호출.
- CREATE SHARD 방식인 경우, 각 샤드 호스트에 원결 스케줄을 예약하여, 리스너를 생성, DBCA 이dydgks DB 생성, Standby DB 생성(구성 된 경우)을 수행.
- 카탈로그가 샤드 디렉터에 샤드들의 DB 파라미터 정보 업데이트, 샤드에 토폴로지 메타정보를 올리고, 샤드가 샤드 디렉터에 등록하도록 요청.
- DG를 사용하는 경우, Standby DB를 구성함. Fast Start Failovr 기능은 모든 샤드에서 활성화됨. 샤드 디렉터는 DG 프로세스 시작하여 DG 구성능 모니터링 시작.
- 새로운 샤드가 추가 될 경우, 이전에 실행된 모든 DDL 문이 새 샤드에 실행.
- System-Managed 또는 Composite 방식으로 샤딩된 구성에 샤드를 추가흐는 경우 자동으로 Chunk 이동이 백드라운드로 예약되며 GDSCTL CONFIG CHUNKS 명령 확인 가능.

#### 6. 글로벌 서비스 등록 : 각 샤드에 액세스를 위한 서비스 등록

#### 7. 샤드 상태 검증

> ### 6. 샤딩 스키마 설계

#### 샤딩 스키마 설계 계획

데이터가 샤드 데이터베이스에 채워지면 테이블의 샤딩, 복제 여부, 샤딩 키 같은 스키마 속성 변경이 불가능하다. 따라서 다음과 같은 요건을 우선 검토, 결정해야한다.

- 어떤 테이블을 샤딩할 것인가?
- 어떤 테이블을 듀플리케이트 테이블로 할 것인가?
- 어떤 샤드 테이블을 루트 테이블로 할 것인가?
  시스템관리방식 샤딩 경우 멀티 루트 테이블을 지원하지만 사용자지정 샤딩 방식 경우는 투트 테이블은 한개만 허용한다.
- 다른 테이블을 루트 테이블에 연결 하려면 어떻게 해야 하는가?
   DB 모델링 단계에서 parent-child 관계를 명확하게 설계하는 것이 바람직하다.
- 어떤 샤딩 방식을 사용 할 것인가?
   시스템관리 샤딩 방식 경우, 전 샤드에 데이터 분포가 골고루 분포되어 워크로드 분산이 필요하면서 데이터의 증가로 인해 향후 샤드가 지속적으로 추가가 되는 경우에 적합한 샤딩 방식일 것이다.
   사용자지정 샤딩 방식 경우, 소속, 조직등의 ,   
- 어떤 샤딩 키를 사용할 것인가?
- 어떤 슈퍼 샤딩 키를 사용할 것인가?(composite 샤딩 경우)

#### 스키마 설계

샤딩 데이터베이스에서 스키마 설계는 데이터베이스의 성능과 확장성에 큰 영향을 미친다. 부적절하게 설계된 스키마는 샤드 전반에 걸쳐 데이터의 불균형한 분배와 다수의 멀티 샤드 작업으로 이어지는 결과를 초래하기 때문에 적절한 설계가 필요하다.

데이터 모델은 가능한 루트 테이블이 있는 계층적 트리구조여야 하며, 이때 계층 레벨은 원하는 만큼 지원이 가능하다.

샤딩의 이점을 극대화 하기 위해서는 샤딩 데이터베이스의 스키마가 싱글 샤드에서 실행되는 데이터베이스 요청 수를 최대화할 수 있도록 설계되어야 한다.

#### 샤딩키 선택

샤드 테이블 파티션은 샤딩 키를 기반으로 테이블스페이스 레벨에서 샤드 전체에 분산된다. 고객ID, 계좌번호, 국가ID 등이 샤딩키로 사용이 가능할 것이다.

샤딩 키 준수사항
- 샤딩 키는 거의 변하지 않는 매우 안정적이어야 한다.
- 샤딩 키는 모든 샤드 테이블에 있어야 한다. 이를 통해 샤딩 키를 기반으로 동등하게 분할된 테이블 계열을 생성할 수 있다.
- 테이블 패밀리의 테이블 간 조인은 샤딩 키를 사용하여 수행되어야 한다.

시스템관리 방식에서의 샤딩 키
- 시스템관리 방식 샤딩을 위한 샤딩용 파티션 키는 카디널리티가 높은 컬럼을 기반으로 해야한다. 이 컬럼의 유니크한 값의 수는 샤드 수 보다 커야 한다.
  - 예, CUSTMER_ID는 적절할 수 있으나 US State의 주명은 적절하지 않을 수 있다.  
- 샤딩 키는 싱글 컬럼 또는 멀티 컬럼 지정 가능하다. 멀티 컬럼인 경우 열의 해시가 연결되어 샤딩 키를 형성한다.

사용자지정 방식에서의 샤딩 키
- List 파티션인 경우 싱글 샤딩키 컬럼만 지원한다.
- Range 파티션인 경우에는 멀티 컬럼 샤딩키를 지원한다.

복합 샤드 방식에서의 샤딩 키

- 복합샤딩 방식의 샤딩 키는 List + Consistent Hash, Range + Consistent Hash 두 가지를 지원한다.
- 슈퍼 샤딩키와 샤딩키를 제공하는 애플리케이션에 의해 수행된다.
- 멀티 컬럼 List 파티션은 지원하지 않는다.
- 멀티 컬럼 Rangre 파티션은 지원한다.

#### 샤딩 데이터베이스에서의 기본키 및 외래키 제약 규칙

기본키 제약 조건
- 기본키는 샤드 테이블의 고유 제약조건, 고유 인덱스 조건이다.
- 컬럼 리스트에는 샤딩키 컬럼이 반드시 포함 되어야 한다.

외래키 제약조건
- 하나의 샤드 테이블에서 다른 샤드 테이블로의 외래키에도 샤딩 키가 포함 되어야한다.
- 외래키가 참조된 테이블의 기본키 또는 유니크 컬럼을 참조할 경우 자동 적용된다.
- 샤드 테이블의 외래키는 동일한 테이블 패밀리 내에 있어야 한다. 다양한 테이블 패밀리에 서로 다른 샤딩 키가 있기 때문에 정확한 식별을 위해 필요하다.
- 로컬 테이블을 참조하는 샤딩된 테이블의 외래키는 허용안된다.
- 듀플리케이트 테이블을 참조하는 샤딩된 테이블의 외래키는 허용안된다.
- 샤딩된 테이블을 참조하는 중복 테이블의 외래키는 허용안된다.

샤드 테이블의 인덱스
- 샤드 테이블에는 로컬 인덱스만 생성이 가능하며 글로벌 인덱스는 허용안된다.
- 고유 로컬 인덱스에는 샤딩 키가 포함되어야 한다.

> ### 7. 샤딩 DDL

#### 샤딩 DDL 실행 툴

DDL 실행 툴은 글로벌데이터서비스 툴(GDSCTL) 또는 SQLPLUS를 사용할 수 있다.

#### GDS 유틸리티를 활용한 글로벌 스키마 생성

GDSCTL에서 DDL을 실행할 경우에는 글로벌하게 모든 샤드에 자동으로 DDL이 실행된다. GDSCTL SQL에서 DDL이 실행되면 마스터 복제본이 카칼로그 DB에 만들어 지고나서 모든 샤드에 샤드 객체가 생성되게 된다.

```gds
GDSCTL> sql "create tablespce set tblset"
```

#### SQLPLUS 유틸리티를 활용한 글로벌 및 로컬 스키마 생성

SQLPLUS로는 로컬 객체 또는 글로벌 객체 생성이 가능하다. 로컬 객체는 오직 샤드 카탈로그에만 만들어지는 객체로써 관리 목적용 또는 멀티샤드 쿼리의 보고서 생성/저장용으로 사용될수 있다.

로컬 테이블에서는 전체 샤드 뷰를 생성할 수 없는 것 처럼 샤딩 객체와 로컬 객체 간에는 종속성이 없다.

글로벌 객체 생성 경우는 다음과 같이 글로벌 샤드 DDL을 활성하고 DDL을 실행 해주어야 한다.

로컬 객체를 생성해야 하는 경우에는 글로벌 샤드 DDL을 비활성화 해준다.

```sql
SQL> alter session enable shard ddl
SQL> create tablespce set tblset ;

SQL> alter session disable shard ddl
SQL> create tablespce set tblset ;
```

#### 샤딩 DDL 구문

샤딩 DDL 구문은 일반 DDL 구문과 다소 상이한 부분이 있으며 샤드 데이터베이스에서만 작동된다.

샤딩 데이데베이스 객체 생성 및 DDL 규칙

- 샤딩 데이터베이스에서 스키마를 생성하려면 DDL은 반드시 카탈로그 DB에서 수행되어야 한다.
- 카탈로그 DB에는 샤드 데이터베이스의 모든 객체들의 로컬 복제본을 유지하고 있다.
- 샤드 DDL을 실행하면 카탈로그 검증이 먼저 일어나게 되고 DDL 실행이 성공하면 자동으로 모든 샤드에 전파되는 과정으로 이루어진다.
- DDL 전파중 샤드가 다운되거나 액세스할 수 없을 경우에는 카탈로에 정보를 유지했다가 샤드가 가용할 때 적용된다.
- 새로운 샤드가 추가될 경우에는 기존 샤드에 배포와 동일하게 모든 DDL을 자동 실행하게된다.

#### 샤드 객체 생성

#### 사용자 계정 생성

- 카탈로그 DB의 로컬 계정은 샤드 데이터베이스 내의 객체 생성 권한이 없다.
- 샤드 데이터베이스 객체 생성 전에 all shard 계정을 만들어야 한다..
- 생성 절차
   - 샤드 카탈로그 DB 시스쳄 계정 로그인 -> 샤드 DDL 활성화 -> CREATE USER 구분 실행
   - all-shards 계정으로 카탈로그 DB에 로그인 할 경우에는 디폴트로 샤드 DDL이 활성화된다.

#### 테이블스페이스 세트 생성

이 명령문은 오라클 샤딩을 위한 특수 DDL로써 테이블스페이스 세트를 생성한다. 이 테이블스페이스 세트는 여러 샤드에 분산된 여러 오라클 테이블스페이스로 구성된다.

테이블스페이스 세트 구분 사용 예)

```sql
CREATE TABLESPACE SET TSP_SET_1 IN SHARDSPACE sgr1
USING TEMPLATE
( DATAFILE SIZE 100m
  EXTEND MANAGEMENT LOCAL
  SEGMENT SPACE MANAGEMENT AUTO
);
```

#### 샤딩 테이블 패밀리 생성

SQL CREATE TABLE 구문에서 Parent-Child 관계를 지정하여 샤드 테이블 패밀리를 생성할 수 있다.

참조 파티션(Reference partitioning)사용한 테이블 패밀리 지정

샤딩 패밀리 파티션을 생성하는 방법으로 참조 파티션을 사용한 P-C 관계를 지정하는 방법이 권고된다. 이 방법은 루트 테이블에만 지정되기 때문에 DDL 구문을 단순화할 수 있다. 아울러 루트 테이블에서 수행되는 파티션 관리작업이 해당 하위 테이블에 자동전파되어 관리 복잡도를 해소할 수 있다.

샤딩 테이블 패밀리 생성 예

```sql

루트테이블(Customers) 생성

CREATE SHARDED TABLE Customers
( CustNo      NUMBER NOT NULL
, Name        V ARCHAR2(50)
, Address     VARCHAR2(250)
, CONSTRAINT RootPK PRIMARY KEY(CustNo)
)
PARTITION BY CONSISTENT HASH (CustNo)
PARTITIONS AUTO
TABLESPACE SET ts1
;

하위 테이블(Orders, LineItems) 생성

CREATE SHARDED TABLE Orders
( OrderNo   NUMBER NOT NULL
, CustNo    NUMBER NOT NULL
, OrderDate DATE
, CONSTRAINT OrderPK PRIMARY KEY (CustNo, OrderNo)
, CONSTRAINT CustFK  FOREIGN KEY (CustNo) REFERENCES Customers(CustNo)
)
PARTITION BY REFERENCE (CustFK)
;


CREATE SHARDED TABLE LineItems
( CustNo    NUMBER NOT NULL
, LineNo    NUMBER(2) NOT NULL
, OrderNo   NUMBER(5) NOT NULL
, StockNo   NUMBER(4)
, Quantity  NUMBER(2)
, CONSTRAINT LinePK  PRIMARY KEY (CustNo, OrderNo, LineNo)
, CONSTRAINT LineFK  FOREIGN KEY (CustNo, OrderNo) REFERENCES Orders(CustNo, OrderNo)
)
PARTITION BY REFERENCE (LineFK)
;

```

이 예문에서 테이블 패밀리에 있는 모든 테이블의 해당 파티션은 모드 동일한 테이블스페이스세트(TS1)에 저장된다. 그러나 각 테이블에 대한 별도의 테이블스페이스 세트를 지정할 수도 있다.

참조 파티션 기반 테이블 패밀리 생성 규칙

- 샤드 테이블의 기본 키는 샤딩 키와 동일하거나 샤딩 키를 포함해야 한다. 이는 다른 샤드와의 조정 없이 기본 키의 전역 고유성을 적용하는 데 필요하다.
- 하위 테이블을 상위 테이블에 연결하려면 하위 테이블의 외래 키 제약 조건에 기본 키를 지정해야 하기 때문에 참조 파티션에는 상위 테이블의 기본 키가 필요하다.
- Primary 키가 없는 상위 테이블에 UNIQUE 제약조건만 있어도 가능하다.
- 샤딩 키는 NOT NULL을 허용하지 않는다.

동일 파티션(Equi-partitioning) 기반의 테이블 패밀리 지정

Reference partitioning을 위한 기본키 및 외래키 조건을 만드는 것이 불가능할때 사용가능한 방법이다. 이 경우 테이블 패밀리에 Parent-Child 관계를 지정하혀면 모든 테이블이 명시적으로 동등하게 분할 되어야 한다.
하위 테이블 생성 위한 CREATE SHARDED TABLE의 PARENT 절에 상위 테이블 이름을 사용하여 생성한다.

동일 파티션(Equi-partitioning) 기반의 테이블 패밀리 생성 사용 예

```sql
CREATE SHARDED TABLE Customers
( CustNo      NUMBER NOT NULL
, Name        VARCHAR2(50)
, Address     VARCHAR2(250)
, region      VARCHAR2(20)
, class       VARCHAR2(3)
, signup      DATE
)
PARTITION BY CONSISTENT HASH (CustNo)
PARTITIONS AUTO
TABLESPACE SET ts1
;

CREATE SHARDED TABLE Orders
( OrderNo   NUMBER
, CustNo    NUMBER NOT NULL
, OrderDate DATE
)
PARENT Customers
PARTITION BY CONSISTENT HASH (CustNo)
PARTITIONS AUTO
TABLESPACE SET ts1
;

CREATE SHARDED TABLE LineItems
( LineNo    NUMBER
, OrderNo   NUMBER
, CustNo    NUMBER NOT NULL
, StockNo   NUMBER
, Quantity  NUMBER
)
PARENT Customers
PARTITION BY CONSISTENT HASH (CustNo)
PARTITIONS AUTO
TABLESPACE SET ts1
;

```

#### 샤드 테이블 생성

시스템관리 방식에서 샤드 테이블 생성할 경우에는 샤드 테이블을 생성하기 전에 테이블 파티션이 저장될 테이블스페이스 세트를 우선 만들어야 한다.

테이블 생성 구문은 일반 DDL 구문에 SHARDED, DUPLICATED 및 테이블 패밀리(PARENT)를 지정할 수 있다.

샤드 테이블 생성 구문
```sql
CREATE [ { GLOBAL TEMPORARY | SHARDED | DUPLICATED} ]
        TABLE [ schema. ] table
      { relational_table | object_table | XMLType_table }
         [ PARENT [ schema. ] table ] ;
```
- "PARENT" 절은 해당 테이블 패밀리의 Root 테이블에 연결함.
- 시스템관리, 복합샤딩 방식에는 tablespace 절이 아닌 "TABLRESPACE SET" 를 사용해야 함.

샤드 테이블 생성 구문 사용 예

```sql

CREATE SHARDED TABLE customers
( cust_id     NUMBER NOT NULL,
    name        VARCHAR2(50),
    address     VARCHAR2(250),
    location_id VARCHAR2(20),
    class       VARCHAR2(3),
    signup_date DATE,
   CONSTRAINT cust_pk PRIMARY KEY(cust_id, class)
)
PARTITIONSET BY LIST (class)
PARTITION BY CONSISTENT HASH (cust_id)
PARTITIONS AUTO
(PARTITIONSET gold   VALUES (‘gld’) TABLESPACE SET ts2,
 PARTITIONSET silver VALUES (‘slv’) TABLESPACE SET ts1)
;


CREATE DUPLICATED TABLE Products
(   ProductId  INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    Name       VARCHAR2(128),
    DescrUri   VARCHAR2(128),
    LastPrice  NUMBER(8,2)
  ) TABLESPACE products_tsp;

```

샤드 테이블 생성 제약 사항

- 샤드 테이블에는 디폴트 테이블스페이스가 설정되지 않는다.
- Temporary 테이블은 샤딩 되거나 복제될 수 없다.
- IOT 샤드 테이블은 지원되지 않는다.
- 샤드 테이블은 nested 컬럼, identity 컬럼을 포함 할 수 없다.
- Primary 키는 제약조건에 반드시 명시 해야한다.
- Duplicated 테이블 컬럼을 참조하는 샤드 테이블 컬럼에 대한 외래키 제약 조건은 지원되지 않는다.
- 가상 컬럼 파티션 방식은 지원되지 않는다

현 버전에서 Duplicated 테이블 생성 제약 사항

- 시스템과 참조 파티션 테이블 미지원
- LONG, REF 타입 미지원, abstract(MDSYS 타입은 지) 미지원
- 최대 컬럼수 999(primary 키 제외)개 지원
- nologging, inmemory option 미지원
- XML 타입 열은 ASSM이 아닌 테이블스페이스에서 사용할 수 없다.

> ### 8. 클라이언트 요청 라우팅 및 쿼리, DML 실행

오라클 샤딩은 오라클 클라이언트 드라이버 및 커넥션 풀(gdspool)을 통한 클라이언트 요청 라우팅을 제공한다. 데이터 기반 라우팅에는 키기반 라우팅 및 프록시 라우팅을 지원하고 미들티어와 데이터베이스 간에 라우팅을 위한 미들티어 라우팅을 지원한다.

#### 쿼리 및 DML 라우팅

#### 키 기반 라우팅(다이렉트 라우팅)
오라클 클라이언트 드라이버를 사용하며, DB 커넥션 스트링에 명시한 샤딩 키를 드라이버가 자동 인식하여 해당 샤드로 직접 연결하여 쿼리를 처리한다.

키 기반 라우팅의 특징

- 더 나은 성능 제공 :프록시(샤드 코디네이터)를 경유하지 않기 때문에 프록시 라우팅에 비해 더 나은 성능을 경험할 수 있다.  
- 데이터의 지리적 분포 자동 제공 : 애플리케이션은 해당 지역에 국한된 데이터에만 액세스가 가능하다.
- 간편한 로드 발란싱 : 청크 기반 데이터 이동을 통해 애플리케이션 요청 로드발란싱이 가능하다.
- 샤드 테이블 쿼리 및 DML : 접속된 샤드 테이블 데이터에 한정된 쿼리 및 INSERT, UPDATE 실행이 수행된다.  
- 듀플리케이션 테이블 쿼리 및 DML : 듀플리케이션 테이블의 모든 데이터에 대한 SELECT, INSERT, UPDATE 실행이 수행된다. DML은 코디네이터 데이터베이스로 재라우팅 되어 처리된다.

듀플리케이션 테이블에 대한 다이렉트 라우팅  

<img src="https://docs.oracle.com/en/database/oracle/oracle-database/19/shard/img/dml_duptable_direct.png" title="듀플리케이션 테이블에 대한 다이렉트 라우팅" width="" height=""></img>

#### 프록시 라우팅
샤딩 키를 명시하지 않은 애플리케이션에 대한 라우팅에 사용된다. 쿼리를 수행할 샤드를 지정하지 않아도 싱글 샤드 쿼리 및 멀티 샤드 쿼리를 처리해준다. 샤드 카탈로그 코디네이터가 프록시 라우팅을 처리한다.

듀플리케이션 테이블에 대한 프록시 라우팅

<img src="https://docs.oracle.com/en/database/oracle/oracle-database/19/shard/img/dml_duptable_coord.png" title="듀플리케이션 테이블에 대한 프록시 라우팅 DML" height="" width=""></img>

#### 미들티어 라우팅
웹 계층과 애플리케이션 계층을 샤딩하여 특정 데이터베이스 샤드 세트를 서비스하기 위해 중간 계층의 샤드를 배포하고 수영 레인이라는 패턴을 생성할 수 있습니다.
스마트 라우터는 특정 샤딩 키를 기반으로 클라이언트 요청을 적절한 수영 레인으로 라우팅할 수 있으며, 그러면 샤드 하위 집합에 대한 연결이 설정된다. // 재해석

#### 멀티 샤드 쿼리의 읽기 일관성

멀티샤드 쿼리는 모든 샤드에서 가장 높은 공통 SCN에서 쿼리를 실행하여 전역 읽기 일관성(CR)을 유지해야 한다. 예를 들어 샤드 간 SCN 동기화 비용을 피하기 위해 일부 쿼리를 원할 수 있으며 이러한 샤드는 전역적으로 분산될 수 있다. 또 다른 사용 사례는 복제를 위해 대기를 사용하고 기본 및 해당 대기에서 결과를 가져올 수 있으므로 다중 샤드 쿼리에 약간 오래된 데이터가 허용되는 경우이다

기본 모드는 모든 샤드에서 SCN 동기화를 수행하는 강력 모드이며 다른 모드에서는 SCN 동기화를 건너 뛴다. Delayed_standby_allowed 레벨은 로드 밸런싱 및 기타 요소에 따라 대기 서버에서도 데이터를 가져오는 것을 허용하며 오래된 데이터를 포함할 수 있다.

#### 멀티 샤드 쿼리의 읽기 일관성 설정 파라미터

멀티 샤드 쿼리의 읽기 일관성 조절은 **MULTISHARD_QUERY_DATA_CONSISTENCY** 파라미터 사용하여 다양한 일관성 수준을 설정 할수 있다. 이 파라미터는 시스템 또는 세션 수준에서 설정할 수 있다.

- STRONG(Default) : 이 설정값은 SCN 동기화가 모든 샤드에서 수행되고 데이터가 모든 샤드에서 일관성이 유지 된다. 이 설정 값은 전역적으로 일관된 읽기를 제공한다.
- SHARD_LOCAL : 이 설정값은 SCN 동기화가 모든 샤드에서 수행되지 않고 각 샤드 내에서 데이터의 읽기 일관성을 유지된다. 이 설정값은 최신 현재 데이터를 제공한다.
- DELAYED_STANDBY_ALLOWED : 이 설정 값은 역시 SCN 동기화가 모든 샤드에서 수행되지 않고 데이터는 각 샤드 내에서 일관된다. 이 설정을 사용하면 가능한 경우(예: 로드 밸런싱에 따라) Data Guard 대기 데이터베이스에서 데이터를 가져올 수 있으며 대기 데이터베이스에서 오래된 데이터를 반환할 수 있다.

#### 쿼리 구조 제한사항

- CONNECT BY 쿼리 미지원
- MODEL Clause 미지원
- 사용자 지정 PL/SQL은 SELECT 멀티샤드 쿼리만 지원
- XLATE, XML 쿼리 타입 미지원
- Object 타입 제한적 지원

#### 멀티 샤드 DML 제한사항

- 멀티샤드 Parallel DML 미지원
- 멀티샤드 Error Logging 미지원
- 멀티샤드 Array DML 미지원
- 멀티샤드 RETURINNG 절 미지원(ORA-22816)
- MERGE 및 UPSERT 부분 지원(싱글 샤드에만 지원)
- 멀티 테이블 삽입 미지원
- 업데이트 테이블되는 조인 뷰 제한(ORA-1779)

#### 샤드 테이블만 쿼리

싱글 테이블 쿼리 경우 샤드를 한정하는 샤딩 키에 대한 동등(=) 필터가 가능하며, 조인 쿼리의 경우 샤딩 키의 동등성(=)을 사용하여 모든 테이블을 조인해야 한다.

다음 쿼리는 샤딩 테이블만 참여하는 조인 쿼리의 예이다.

```sql
SELECT … FROM s1 INNER JOIN s2 ON s1.sk=s2.sk
WHERE any_filter(s1) AND any_filter(s2)               // inner join

SELECT … FROM s1 LEFT OUTER JOIN s2 ON s1.sk=s2.sk    // left Outer Join

SELECT … FROM s1 RIGHT OUTER JOIN s2 ON s1.sk=s2.sk   // Right Outer join

SELECT … FROM s1 FULL OUTER JOIN s2 ON s1.sk=s2.sk    // Full Outer join
WHERE any_filter(s1) AND any_filter(s2)

```

#### 샤드 테이블과 듀플리케이트 테이블 간의 쿼리

샤드 테이블과 듀플리케이트 테이블 모두를 포함하는 쿼리는 샤드 키의 조건자를 기반으로 싱글 샤드 또는 멀티 샤드 쿼리일 수 있다. 유일한 차이점은 쿼리에 비 샤드 테이블이 포함되어 있다는 것이다.

샤드 테이블과 듀플리케이트 테이블 간의 조인은 비교 연산자, = < > <= >= 또는 임의의 조인 표현식을 사용하여 모든 열에서 이루어질 수 있습니다.

#### 멀티 샤드 쿼리의 힌트 전달

코디네이터의 원래 쿼리에 지정된 힌트는 모든 샤드에 전파된다.

#### 느리게 실행되는 멀티샤드 쿼리 추적 및 문제 해결

쿼리 재작성 및 샤드 프루닝을 추적하려면 코디네이터에서 trace 이벤트(shard_sql)를 설정하면 된다. 일반적인 성능 문제 중 하나는 특정 샤딩의 제한으로 인해 GROUP BY가 샤드로 푸시되지 않는 경우이다. 가능한 모든 작업이 샤드에 푸시되고 코디네이터가 샤드의 결과를 통합하기 위해 최소한의 작업만 수행하는지 확인이 필요하다.


#### 샤딩 데이터베이스에서 PLSQL 실행

구성의 모든 샤드에서 DDL 문을 실행할 수 있는 것과 마찬가지로 Oracle 제공 PL/SQL 프로시저도 실행할 수 있다. 이러한 특정 프로시저 호출은 모든 샤드에 전파되고 카탈로그에 의해 추적되며 새 샤드가 구성에 추가될 때마다 실행된다는 점에서 샤딩된 DDL문인 것처럼 동작한다.

PLSQL 실행 전에 "alter session enable shard ddl"을 실행해줘야 한다.

지원 PLSQL 패키지

- DBMS_FGA의 모든 프로시저
- DBMS_RLS의 모든 프로시저

DBMS_STATS의 다음 패키지
- GATHER_INDEX_STATS
- GATHER_TABLE_STATS
- GATHER_SCHEMA_STATS
- GATHER_DATABASE_STATS
- GATHER_SYSTEM_STATS

DBMS_GOLDENGATE_ADM 의 다음 패키지
- ADD_AUTO_CDR
- ADD_AUTO_CDR_COLUMN_GROUP
- ADD_AUTO_CDR_DELTA_RES
- ALTER_AUTO_CDR
- ALTER_AUTO_CDR_COLUMN_GROUP
- PURGE_TOMBSTONES
- REMOVE_AUTO_CDR
- REMOVE_AUTO_CDR_COLUMN_GROUP
- REMOVE_AUTO_CDR_DELTA_RES


> ### 9. 애플리케이션 APIs

#### 키기반 라우팅(또는 다이렉트 라우팅)

특정 사드에 직접 접속 및 쿼리 수행을 위한 방식으로 커넥션 스트링에 데이터 종속적인 라우팅을 위한 샤당 키를 기술하면 오라클 클라리언트 드라이버 및 커넥션 풀이 자동으로 지정한 샤드로 접속하여 처리할 수 있도록 해준다.

Oracle JDBC 환경인 경우

애플리케이션에 샤드 인식을 위한 오라클 제공 클래스 및 메소드를 사용하여 데이터소스를 만들어준다.

- OracleDataSource : 오라클 샤딩 접속 데이터 소스 클래스이며 createConnectionBuilde, createShardingKeyBulider 메소드 지원.
- OracleShardingKey : 오라클 샤딩키 객체를 지정하는 인터페이스.
- OracleShardingKeyBuilder : 샤딩키 빌더 인터페이스이며 다양한 데이터 타입을 지원. 새로운 JDK8 빌더 패턴 사용. 
- OracleConnection : 오라클 샤딩 키 세팅 클래스이며 setShardingKeyIfValid, setShardingKey 메소드 지원. 
- OracleConnectionBuilder : 계정/패스워드 이외의 추가 파라미터 사용한 연결 객체 생성 인터페이스.
- OracleXADataSource : XA용 오라클 샤딩 데이터소스 클래스이며 createConnectionBuilder 메소드 지원.
- OracleXAConnection : XA용 오라클 샤딩키 세팅 클래스이며 setShardingKeyIfValid, setShardingKey 메소드 지원.

JDBC를 사용하는 데이터소스 샘플 코드

```java
OracleDataSource ods = new OracleDataSource();
   ods.setURL("jdbc:oracle:thin:@(DESCRIPTION=(ADDRESS=(HOST=myhost)(PORT=1521)(PROTOCOL=tcp))(CONNECT_DATA=(SERVICE_NAME=myorcldbservicename)))");
   ods.setUser("hr");
   ods.setPassword("hr");

  // Employee name is the sharding Key in this example.
  // Build the Sharding Key using employee name as shown below.

   OracleShardingKey employeeNameShardKey = ods.createShardingKeyBuilder()
                                               .subkey("Mary", JDBCType.VARCHAR)// First Name
                                               .subkey("Claire", JDBCType.VARCHAR)// Last Name
                                               .build();

   OracleShardingKey locationSuperShardKey = ods.createShardingKeyBuilder() // Building a super sharding key using location as the key
                                                .subkey("US", JDBCType.VARCHAR)
                                                .build();

   OracleConnection connection = ods.createConnectionBuilder()
                                    .shardingKey(employeeNameShardKey)
                                    .superShardingKey(locationSuperShardKey)
                                    .build();
```

OCI, UCP 및 .NET API는 오라클 메뉴얼을 참고 바란다.

[APIs for Application](https://docs.oracle.com/en/database/oracle/oracle-database/19/shard/developing-applications-sharded-database1.html#GUID-2BF7AD3E-B8C5-4A75-8DCF-D13AB208A46B)


> ### 10. 샤드 데이터베이스 관리

#### 통계정보 수집

COORDINATOR_TRIGGER_SHARD = 'TRUE'를 설정하면 코디네이터 데이터베이스가 샤드에 수집된 통계를 가져올 수 있다.

수동 통계정보 수집

connect / as sysdba
EXECUTE DBMS_STATS.SET_SCHEMA_PREFS('SHARDUSER','COORDINATOR_TRIGGER_SHARD','TRUE');

자동 통계정보 수집

connect / as sysdba
EXECUTE DBMS_STATS.SET_SCHEMA_PREFS('SHARDUSER','COORDINATOR_TRIGGER_SHARD','TRUE');

#### DDL 실행 상태 모니터링 및 관리 명령

샤드 DDL 수행 결과에 대한 모니터링은 GDSCTL의 "show ddl" 및 "config shard" 명령어을 사용하여 샤드등에 대한 DDL 전파 상태를 확인할 수 있다.
```gds
GDSCTL> show ddl

GDSCTL> config shard -shard sd1
Name: sd1
Shard space: sdspc1
Status: Ok
State: Deployed
Region: dc2
Connection string: (DESCRIPTION=(ADDRESS=(HOST=sdlab3.labs.woko.oraclevcn.com)(PORT=1521)(PROTOCOL=tcp))(CONNECT_DATA=(SID=sd1)))
SCAN address:
ONS remote port: 0
Disk Threshold, ms: 20
CPU Threshold, %: 75
Version: 19.0.0.0
Failed DDL:
DDL Error: ---
Failed DDL id:
Availability: ONLINE

Supported services
------------------------
Name                                                            Preferred Status
----                                                            --------- ------
ro_srvc                                                         Yes       Enabled
rw_srvc                                                         Yes       Enabled

```
SHOW DDL 결과에서 DDL 구문에 대한 전체 텍스트 조회는 다음과 같이할 수 있다.

```sq
SELECT ddl_text FROM gsmadmin_internal.ddl_requests;
```
선행 DDL 실패로 후행 DDL이 실행되지 못할 경우 선행 실패 DDL을 건너 뛰려면 -ignore_first 옵션을 실행할 수 있다.

–ignore_first 옵션을 사용하여 샤드 복구를 실행하면 실패한 DDL은 증분 배포 중에 무시되도록 표시된다.

```gds
GDSCTL> show ddl
id      DDL Text                                Failed shards
--      --------                                -------------
1       create user sidney identified by *****
2       create tablespace set tbsset             
3       create tablespace set tbs_set            
4       drop tablespace set tbs_set             shard01  
5       create tablespace set tbsset2

GDSCTL> recover shard -shard shard01 –ignore_first

GDSCTL> show ddl
id      DDL Text                                Failed shards
--      --------                                -------------
1       create user sidney identified by *****
2       create tablespace set tbsset             
3       create tablespace set tbs_set          
4       drop tablespace set tbs_set
5       create tablespace set tbsset2
```

#### 테이블스페이스 세트 확인

```sql
sqlplus / as sysdba

SQL> select TABLESPACE_NAME, BYTES/1024/1024 MB from sys.dba_data_files
 order by tablespace_name;
```

#### 청크 생성 및 분산 획인

```gds
GDSCTL> config chunks
Chunks
------------------------
Database                      From      To        
--------                      ----      --        
sh1                           1         6         
sh2                           1         6         
sh3                           7         12        
sh4                           7         12

```

```sql
SQL> show parameter db_unique_name

NAME             TYPE        VALUE
---------------- ----------- ------------------------------
db_unique_name   string      sh1

SQL> select table_name, partition_name, tablespace_name
 from dba_tab_partitions
 where tablespace_name like 'C%TSP_SET_1'
 order by tablespace_name;

$ sqlplus / as sysdba

SQL> SELECT a.name Shard, COUNT(b.chunk_number) Number_of_Chunks
  FROM gsmadmin_internal.database a, gsmadmin_internal.chunk_loc b
  WHERE a.database_num=b.database_num
  GROUP BY a.name
  ORDER BY a.name;
```

#### 샤딩 컴포넌트 확인

GDSCTL config sdb 명령어를 실행하면 샤딩 방식, 복제 방식, 샤드스페이스, 글로벌 서비스들의 리스트들을 확인할 수 있다.

```gds

gdsctl> condig sdb
Catalog connection is established
GDS Pool administrators
------------------------

Replication Type
------------------------
Data Guard

Shard type
------------------------
User-defined

Shard spaces
------------------------
sdspc1
sdspc2

Services
------------------------
ro_srvc
rw_srvc

```
#### 샤딩키 값이 포함된 청크 확인

```gds
GDSCTL> config chunks
Chunks
------------------------
Database                      From      To
--------                      ----      --
sd1                           1         1
sd1_stby                      1         1
sd2                           2         2
sd2_stby                      2         2

GDSCTL> config chunks -key 1
Range Definition
------------------------
Chunks    Range Definition
------    ----------------
1         -

Databases
------------------------
sd1
sd1_stby

GDSCTL> config chunks -key 300
Range Definition
------------------------
Chunks    Range Definition
------    ----------------
2         -

Databases
------------------------
sd2
sd2_stby
```

#### 샤드 오퍼레이션 모드 확인  // 샘플 샤딩 구성 확인 필요

```gds
GDSCTL> config chunks -cross_shard
Read-Only cross shard targets
------------------------
Database                      From To
--------                      ---- --

Chunks not offered for cross-shard
------------------------
Shard space    From To
-----------    ---- --
sdspc1         1    1
sdspc2         2    2

Read-Write cross-shard targets
------------------------
Database                      From To
--------                      ---- --

Chunks not offered for Read-Write cross-shard activity
------------------------
Data N/A
```

#### GSM 모니터
```gsm

GDSCTL> status gsm
Alias                     SDDIRECTOR1
Version                   19.0.0.0.0
Start Date                06-FEB-2024 05:42:06
Trace Level               off
Listener Log File         /home/oracle/orabase/diag/gsm/sdlab1/sddirector1/alert/log.xml
Listener Trace File       /home/oracle/orabase/diag/gsm/sdlab1/sddirector1/trace/ora_8229_140190149553216.trc
Endpoint summary          (ADDRESS=(HOST=sdlab1.labs.woko.oraclevcn.com)(PORT=1522)(PROTOCOL=tcp))
GSMOCI Version            3.0.180702
Mastership                N
Connected to GDS catalog  Y
Process Id                8231
Number of reconnections   0
Pending tasks.     Total  0
Tasks in  process. Total  0
Regional Mastership       TRUE
Total messages published  36
Time Zone                 +00:00
Orphaned Buddy Regions:
     None
GDS region                dc1
Network metrics:
   Region: dc2 Network factor:0

GDSCTL> config gsm
Name      Region    ENDPOINT
----      ------    --------
sddirector1 dc1       (ADDRESS=(HOST=sdlab1.labs.woko.oraclevcn.com)(PORT=1522)(PROTOCOL=tcp))
sddirector2 dc2       (ADDRESS=(HOST=sdlab7.labs.woko.oraclevcn.com)(PORT=1522)(PROTOCOL=tcp))
sddirector3 dc3       (ADDRESS=(HOST=sdlab8.labs.woko.oraclevcn.com)(PORT=1522)(PROTOCOL=tcp))

GDSCTL> config gsm -gsm sddirector1
Name: sddirector1
Endpoint 1: (ADDRESS=(HOST=sdlab1.labs.woko.oraclevcn.com)(PORT=1522)(PROTOCOL=tcp))
Local ONS port: 6123
Remote ONS port: 6234
ORACLE_HOME path: /home/oracle/19c/gsm
GSM Host name: sdlab1.labs.woko.oraclevcn.com
Region: dc1

```

#### DDL Text 확인
```gds
GDSCTL> show ddl -ddl 34
DDL Text: create sharded table customers
( CustId      varchar2(60) NOT NULL,
    Gender varchar2(1),
    Cust_email  varchar2(60),
    CustProfile VARCHAR2(4000),
    CONSTRAINT pk_customers PRIMARY KEY (CustId)
)
 PARTITION BY RANGE(CustId)
  ( PARTITION cst_1 values less than ('250') tablespace tbs1,
    PARTITION cst_2 values less than (maxvalue) tablespace tbs2
  )
Owner: SDTEST
Object name: CUSTOMERS
DDL type: C
Obsolete: 0
Failed shards:
```

#### 계정 타입 확인(Local 또는 all shard)

```sql
SQL> select USERNAME,ALL_SHARD from dba_users where username = 'SDTEST';

USERNAME                                 ALL
---------------------------------------- ---
SDTEST                                   YES


```

#### 샤드된 테이블스페이스 확인

```sql
SQL> select TABLESPACE_NAME,CHUNK_TABLESPACE from user_tablespaces;

TABLESPACE_NAME                C
------------------------------ -
SYSTEM                         N
SYSAUX                         N
UNDOTBS1                       N
TEMP                           N
USERS                          N
TBS1                           Y
TBS2                           Y
PRODUCTS_TSP                   N

```

#### 듀플리케이트 테이블 리프레시 주기 확인 및 변경

```sql
SQL> show parameter SHRD_DUPL_TABLE_REFRESH_RATE

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
shrd_dupl_table_refresh_rate         integer     60

SQL> alter session enable shard ddl;

SQL> alter system set SHRD_DUPL_TABLE_REFRESH_RATE = 200 scope=both;

System altered.

SQL> SQL> show parameter SHRD_DUPL_TABLE_REFRESH_RATE

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
shrd_dupl_table_refresh_rate         integer     200

```
#### 샤드 디버깅

```sql
ALTER SYSTEM SET EVENTS 'immediate trace name GWM_TRACE level 7';

ALTER SYSTEM SET EVENT='10798 trace name context forever, level 7' SCOPE=spfile;

```

sharding alert log 위치 확인

```gds
GDSCTL> status
Alias                     SDDIRECTOR1
Version                   19.0.0.0.0
Start Date                01-FEB-2024 01:50:14
Trace Level               off
Listener Log File         /home/oracle/orabase/diag/gsm/sdlab1/sddirector1/alert/log.xml
Listener Trace File       /home/oracle/orabase/diag/gsm/sdlab1/sddirector1/trace/ora_6492_140358916582464.trc
Endpoint summary          (ADDRESS=(HOST=sdlab1.labs.woko.oraclevcn.com)(PORT=1522)(PROTOCOL=tcp))
GSMOCI Version            3.0.180702
Mastership                Y
Connected to GDS catalog  Y
Process Id                6494
Number of reconnections   0
Pending tasks.     Total  0
Tasks in  process. Total  0
Regional Mastership       TRUE
Total messages published  904
Time Zone                 +00:00
Orphaned Buddy Regions:
     None
GDS region                dc1
Network metrics:
   Region: dc2 Network factor:0
```


#### 샤드 및 듀플리케이트 테이블 확인

SQL> select TABLE_NAME,SHARDED,DUPLICATED from user_tables;

```sql
TABLE_NAME                     S D
------------------------------ - -
CUSTOMERS_EXT                  N N
CUSTOMERS                      Y N
ORDERS                         Y N
PRODUCTS_EXT                   N N
PRODUCTS                       N Y
MLOG$_PRODUCTS                 N N
RUPD$_PRODUCTS                 N N

```


#### 글로벌 샤드 시퀀스 생성

오라클 샤딩은 기본키가 아닌 컬럼에 대하여 샤딩 데이터베이스에 의해 처리되는 글로벌 유니크한 시퀀스 번호를 생성할 수 있다. 이 기능은 샤드 전체에서 고유한 시퀀스 번호를 얻을 수 있다.

샤드 시퀀스에 의해 생성된 숫자는 이 샤드에 삽입되는 새 행에 대한 샤딩 키로 즉시 사용될 수 없다. 키 값이 다른 샤드에 속할 수 있고 삽입 시 오류가 발생할 수 있기 때문이다.

새 행을 삽입하려면 애플리케이션이 먼저 샤딩 키 값을 생성한 다음 이를 사용하여 적절한 샤드에 연결하여 사용해야 한다. 샤딩 키의 새 값을 생성하는 일반적인 방법은 샤드 카탈로그에서 일반(샤딩되지 않은) 시퀀스를 사용하는 것이다.

```sql
CREATE | ALTER SEQUENCE [ schema. ]sequence
   [ { INCREMENT BY | START WITH } integer
   | { MAXVALUE integer | NOMAXVALUE }
   | { MINVALUE integer | NOMINVALUE }
   | { CYCLE | NOCYCLE }
   | { CACHE integer | NOCACHE }
   | { ORDER | NOORDER }
   | { SCALE {EXTEND | NOEXTEND} | NOSCALE}
   | { SHARD {EXTEND | NOEXTEND} | NOSHARD}
   ]
```

> ### 11. 오라클 샤딩 샘플 시나리오

#### 서버 준비

시나리오를 위한 호스트 정보
샤드 시나리오를 위하여 총 7대의 호스트를 사용합니다.

- sdlab1 : 샤드 디렉터(sddirector1) for Master
- sdlab2 : 카탈로그(sdcat) DB 서버
- sdlab3 : sd1 DB, sd2_stby DB용 서버
- sdlab4 : sd2, sd1_stby DB용 서버
- sdlab5 : 카탈로그(sdcat_stby) 스켄드바이 DB 서버
- sdlab7 : 샤드 디렉터(sddirector2) for DC2 리전
- sdlab8 : 샤드 디렉터(sddirector3) for DC3 리전

#### 샤드 DB Primary 및 standby 구성

카탈로그 primary : DB_NAME = sdcat, DB_UNIQUE_NAME = sdcat

카탈로그 standby : DB_NAME = sdcat, DB_UNIQUE_NAME = sdcat_stby

샤드1 primary : DB_NAME = sd1, DB_UNIQUE_NAME = sd1

샤드1 standby : DB_NAME = sd2, DB_UNIQUE_NAME = sd1_stby

샤드2 primary : DB_NAME = sd2, DB_UNIQUE_NAME = sd2

샤드2 stanbdy : DB_NAME = sd2, DB_UNIQUE_NAME = sd2_stby


#### 오라클 소프트웨어 설치

샤딩 데이터베이스 설치 할 모든 서버에 Oracle EE 소프트웨어 설치
- 샤드 서버 및 HA 구성할 서버

Standby DB 구성은 ADG 구성 방법 참조

#### 디렉터 서버에 오라클 GSM 소프트웨어 설치

오라클 다운로드 센터에서 적절한 GSM 다운로드

오라클 데이터베이스 다운로드에서 indivisual component 클릭해서  Global Service Manager (GSM/GDS) (19.3) for Linux x86-64를 다운로드
https://www.oracle.com/database/technologies/oracle19c-linux-downloads.html

카탈로그 데이터베이스 서버와 같이 사용할 경우에는 Oracle database의 홈과는 다른 디렉토리에 설치 해야함.

```linx cmd
mkdir -p /home/oracle/19c/gsm

cd /home/oracle/downloads

unzip LINUX.X64_193000_gsm.zip

/home/oracle/downloads/gsm

./runInstaller

software location : /home/oracle/19c/gsm

export PATH=$PATH:/home/oracle/19c/gsm/bin

[oracle@sdlab1 gsm]$ gdsctl

GDSCTL: Version 19.0.0.0.0 - Production on Thu Jan 04 03:18:43 GMT 2024

Copyright (c) 2011, 2019, Oracle.  All rights reserved.

Welcome to GDSCTL, type "help" for information.

Warning:  GSM  is not set automatically because gsm.ora does not contain GSM entries. Use "set  gsm" command to set GSM for the session.
Current GSM is set to GSMORA
GDSCTL>exit

```

#### 파이월 및 OCI의 Port 개방
- 샤드 디렉터 : 1522
- ONS(사용시) : 6123, 6234
- TNS : 1521
- xdb : 8080 (Create Shard 방식 사용시)

#### catalog 데이터베이스 설치

샤드 카탈로그용 오라클 DB 인스턴스를 생성한다. 일반 DB 인스턴스 설치와 동일하며 ADG 또는 OGG를 활용한 standby DB 구성을 강력히 권고한다. 이 시나리오는 ADG를 기준으로 작성되었으며 OGG를 이용한 구성은 ADG와 상이하니 메뉴얼을 확인한다.

##### DB Compatible 설정

COMPATIBLE parameter는 12.2.0. 이상이어야 한다.

```sql
SQL> show parameter COMPATIBLE
```
##### 플래시백 데이터베이스 설정

샤딩 데이터베이스 환경을 위하여 Primary, Standby DB를 운영할 경우에는 플래시백 기능을 활성화 하여 운영하는 것을 권고한다.

```sql
$ sqlplus / as sysdba

SQL> REM run the following command if using a CDB
SQL> select flashback_on from v$database;

FLASHBACK_ON
------------------
YES
```

##### 카탈로그 DB 생성후 조치사항

카탈로드 데이터베이스는 server parameter file(SPFILE) 반드시 사용해야 한다.

```sql
SQL> show parameter spfile

NAME                                 TYPE        VALUE
----------- ----------- ------------------------------
spfile      string      /home/oracle/19c/dbs/spfilesdcat.ora
```

##### 문자 세트 통일

카탈로그, 샤드 DB의 DB character set과 national character set은 동일해야한다.
```sql
SQL> select * from nls_database_parameters where parameter like '%CHARACTERSET';

shard catalog db

PARAMETER                      VALUE
------------------------------ --------------------------------------------------
NLS_NCHAR_CHARACTERSET         AL16UTF16
NLS_CHARACTERSET               AL32UTF8

shard db

PARAMETER                                VALUE
------------------------------ ---------------------------------------------------
NLS_NCHAR_CHARACTERSET                   AL16UTF16
NLS_CHARACTERSET                         AL32UTF8
```
##### 멀티 샤드 쿼리용 DB link 셋업

open_links, open_links_per_instance > 샤드 수를 고려하여 지정하면 된다.

```sql
SQL> show parameter open_links

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
open_links                           integer     4
open_links_per_instance              integer     4

SQL> alter system set open_links=16 scope=spfile;
SQL> alter system set open_links_per_instance=16 scope=spfile;

```
##### DF_FILES 파라미터 값 설정

DF_FILES 값은 샤드의 총 chunk 수 또는 테이블스페이스 수 보다 커야한다.

```sql
SQL> show parameter db_files

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------
db_files                             integer     200

카탈로그 DB(primary) 에서

SQL> alter system set db_files=1024 scope=spfile;

SQL> startup force

SQL> show parameter db_files

```

##### 원격 샤드 생성시 참고 사항(* ADD SHARD 방식은 불필요)

CREATE SHARD 방식으로 샤드 추가할 경우에는 "SHARDED_SERVERS", "DISPATCHERS" 파라미터 설정이 필요하다.
SHARDED_SERVERS는 0보다 커야한다.

DISPATCHERS는 SID기반의 XDB 서비스를 반드시 포함해야한다. (alter system resister 명령어 실행하여 확인)

```sql
SQL> show parameter shared_servers

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------
shared_servers                       integer     5

SQL> show parameter dispatchers

NAME                                 TYPE        VALUE
------------------------------------ ----------- ----------------------
Dispatchers                          string      (PROTOCOL=TCP), (PROTO
                                                 COL=TCP)(SERVICE=mysid
                                                 XDB)
```
##### chunk 이동용 디렉토리 지정

OMF 지원을 위한 파라미터 세팅이 필요하다. 샤드 카탈로그 DB, shard primary DB, standby DB에 설정하면 된다.

```sql
SQL> show parameter db_create_file_dest

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
db_create_file_dest                  string


SQL> alter system set db_create_file_dest='/home/oracle/orabase/oradata' scope=both;

```
##### STANDBY_FILE_MANAGEMENT 파라미터 지정

STANDBY_FILE_MANAGEMENT 파라미터를 "AUTO"로 설정한다.

```SQL
SQL> show parameter standby_file_management

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
db_create_file_dest                  string      /home/oracle/orabase/oradata

NAME TYPE VALUE
------------------------------------ ----------- ------------
standby_file_management stirng MANUAL

SQL> alter system set standby_file_management='AUTO' scope=spfile;

```

##### GSM 관리자 계정 활성화

GSMCATUSER는 오라클이 제공하는 기본 계정으로서 샤드 디렉터 프로세스가 사용하는 계정이다. PDB경우에는 이 계정은 common 유저이다.

패스워드는 ADD GSM 때 사용되니 기억해둬야 햐며 샤드 디렉터가 오라클 월넷에 저장 관리하게된다.
GSMCATUSER의 패스워드가 변경 될 경우에누 MODIFY GSB으로 업데이터 해주면 된다. 이작업은 카탈로그 DB에서만 해주면 된다.

카탈로그 DB에서

```SQL
SQL> alter user gsmcatuser account unlock;
SQL> alter user gsmcatuser identified by Welcome1;                

```

##### 카탈로그 관리자 계정 생성

샤드 카탈로그 관리자 계정은 사용자 지정 계정으로서 반드시 생성이 되어야 하며, 관리자 권한이 부여 되어야 한다. 이 계정은 샤딩 메타정보를 관리하는 계정이며 샤딩 데이터베이스 토폴로지 변경 및 관리 작업을 수행하게 된다.

카탈로그 DB에서
```SQL
SQL> CREATE USER catadmin IDENTIFIED BY Welcome1;
SQL> GRANT connect, create session, gsmadmin_role to catadmin;
SQL> grant inherit privileges on user SYS to GSMADMIN_INTERNAL;
```
##### GSMUSER 활성화 및 셋업

GSMUSER는 반드시 샤드 카탈로그, 샤드 DB에 모두 조치가 필요하다.

```SQL
SQL> alter user gsmuser account unlock;
SQL> alter user gsmuser identified by Welcome1;
SQL> grant SYSDG, SYSBACKUP to gsmuser;
```

##### DATA_PUMP_DIR 디렉토리 생성

Shard catalog, shard db 모두에 조치가 필요하다.
```SQL
SQL> create or replace directory DATA_PUMP_DIR as '/home/oracle/orabase/oradata';

SQL> grant read, write on directory DATA_PUMP_DIR to gsmadmin_internal;

Grant succeeded.

```

#### 오라클 샤드 준비 요건 검증

카탈로그 DB에서
```SQL
sqlplus / as sysdba

SQL> set serveroutput on
SQL> execute dbms_gsm_fix.validateShard
INFO: Data Guard shard validation requested.
INFO: Database role is PRIMARY.
INFO: Database name is SDCAT.
INFO: Database unique name is sdcat.
INFO: Database ID is 4054837795.
INFO: Database open mode is READ WRITE.
INFO: Database in archivelog mode.
INFO: Flashback is on.
INFO: Force logging is on.
INFO: Database platform is Linux x86 64-bit.
INFO: Database character set is AL32UTF8. This value must match the character
set of the catalog database.
INFO: 'compatible' initialization parameter validated successfully.
INFO: Database is not a multitenant container database.
INFO: Database is using a server parameter file (spfile).
INFO: db_create_file_dest set to: '/home/oracle/orabase/oradata'
INFO: db_recovery_file_dest set to: '/home/oracle/orabase/fast_recovery_area'
INFO: db_files=1024. Must be greater than the number of chunks and/or
tablespaces to be created in the shard.
INFO: dg_broker_start set to TRUE.
INFO: remote_login_passwordfile set to EXCLUSIVE.
WARNING: db_file_name_convert is not set.
INFO: GSMUSER account validated successfully.
INFO: DATA_PUMP_DIR is '/home/oracle/orabase/oradata'.

PL/SQL procedure successfully completed.

```

"Warrning"부분은 스킵 가능하지만 "Error"항목인 있는 경우에는 찾아서 해결 조치해야 한다.

##### Remote Schedule Agent 포트 개방

CREATE SHARD방식(자동 샤드 생성) 사용할 경우 agent 포트 활설화 필요하다.

```sql
execute dbms_xdb.sethttpport(8080);
commit;

 @?/rdbms/admin/prvtrsch.plb
SQL> @?/rdbms/admin/prvtrsch.plb
SQL>  exec DBMS_SCHEDULER.SET_AGENT_REGISTRATION_PASS('Welcome1');
```

#### 샤드 데이터베이스 토폴로지 구성

sdlab1 서버(for dc1 region)에서 샤드 카탈로그 DB 접속정보를 확인 및 조치한다.

tnsnames.ora 파일 확인 및 수정
```text
SDCAT =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = sdlab2.labs.woko.oraclevcn.com)(PORT = 1521))
    (ADDRESS = (PROTOCOL = TCP)(HOST = sdlab5.labs.woko.oraclevcn.com)(PORT = 1521))   // 추가
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = sdcat.labs.woko.oraclevcn.com)
    )
  )
```

##### 샤드 카탈로그 생성
```gds
$> gdsctl
GDSCTL> connect catadmin/Welcome1@sdlab2.labs.woko.oraclevcn.com:1521:sdcat
Catalog connection is established

GDSCTL> create shardcatalog -database sdlab2.labs.woko.oraclevcn.com:1521:sdcat -user catadmin/Welcome1 -protectmode maxperformance -sharding user

Catalog is created
```

"- sharding" 옵션 : 샤딩 방법을 의미하며 여기서는 사용자지정 방식 사용할 것이므로 "user"를 지정한다.

##### 샤드 디렉터 등록 및 시작

```gds
sdlab1 서버(for dc1 region)에서

GDSCTL> add gsm -gsm sddirector1 -catalog sdlab2.labs.woko.oraclevcn.com:1521:sdcat -listener 1522 -pwd Welcome1  
GSM successfully added

GDSCTL> start gsm -gsm sddirector1
GSM is started successfully
```

#### 리전 및 인증 정 등록

```gds
GDSCTL> add region -region dc1, dc2, dc3
The operation completed successfully

GDSCTL> modify gsm -gsm sddirector1 -region dc1
GSM modified

GDSCTL> add credential -credential oracle_cred -osaccount oracle -ospassword Odnsdyd#!
The operation completed successfully
```

##### 샤드스페이스 등록

샤드스페이스는 user-defiend 방식 샤드에서 필요한 구성이다.

```gds
GDSCTL> add shardspace -SHARDSPACE sdspc1 -PROTECTMODE MAXPERFORMANCE
The operation completed successfully

GDSCTL> add shardspace -SHARDSPACE sdspc2 -PROTECTMODE MAXPERFORMANCE
The operation completed successfully
```

##### 샤드 디렉터 추가

```gds
sdlab7서버에서
$> gdsctl
GDSCTL: Version 19.0.0.0.0 - Production on Thu Jan 04 06:51:04 GMT 2024

Copyright (c) 2011, 2019, Oracle.  All rights reserved.

Welcome to GDSCTL, type "help" for information.

Warning:  GSM  is not set automatically because gsm.ora does not contain GSM entries. Use "set  gsm" command to set GSM for the session.
Current GSM is set to GSMORA
GDSCTL> connect catadmin/Welcome1@sdlab2.labs.woko.oraclevcn.com:1521:sdcat
Catalog connection is established

GDSCTL> add gsm -gsm sddirector2 -catalog sdlab2.labs.woko.oraclevcn.com:1521:sdcat -listener 1522 -pwd Welcome1 -region dc2
GSM successfully added
GDSCTL> start gsm -gsm sddirector2
GSM is started successfully

sdlab8 서버에서
$> gdsctl
GDSCTL: Version 19.0.0.0.0 - Production on Thu Jan 04 06:51:04 GMT 2024

Copyright (c) 2011, 2019, Oracle.  All rights reserved.

Welcome to GDSCTL, type "help" for information.

Warning:  GSM  is not set automatically because gsm.ora does not contain GSM entries. Use "set  gsm" command to set GSM for the session.
Current GSM is set to GSMORA

GDSCTL> connect catadmin/Welcome1@sdlab2.labs.woko.oraclevcn.com:1521:sdcat
Catalog connection is established
GDSCTL> add gsm -gsm sddirector3 -catalog sdlab2.labs.woko.oraclevcn.com:1521:sdcat -listener 1522 -pwd Welcome1 -region dc3
GSM successfully added
GDSCTL> start gsm -gsm sddirector3
GSM is started successfully
```

##### 샤딩 토폴로지 확인
```gds
GDSCTL> config

Regions
------------------------
dc1
dc2
dc3
regionora

GSMs
------------------------
sddirector1
sddirector2
sddirector3

Sharded Database
------------------------
orasdb

Databases
------------------------

Shard spaces
------------------------
sdspc1
sdspc2

Services
------------------------

GDSCTL pending requests
------------------------
Command                       Object                        Status
-------                       ------                        ------

Global properties
------------------------
Name: oradbcloud
Master GSM: sddirector1
DDL sequence #: 0
```

##### 샤드 DB 등록
```gds
GDSCTL> add shard -connect sdlab3.labs.woko.oraclevcn.com:1521:sd1 -pwd Welcome1 -shardspace sdspc1 -region dc2 -deploy_as PRIMARY

INFO: Data Guard shard validation requested.
INFO: Database role is PRIMARY.
INFO: Database name is SD1.
INFO: Database unique name is sd1.
INFO: Database ID is 730890083.
INFO: Database open mode is READ WRITE.
INFO: Database in archivelog mode.
INFO: Flashback is on.
INFO: Force logging is on.
INFO: Database platform is Linux x86 64-bit.
INFO: Database character set is AL32UTF8. This value must match the character set of the catalog database.
INFO: 'compatible' initialization parameter validated successfully.
INFO: Database is not a multitenant container database.
INFO: Database is using a server parameter file (spfile).
INFO: db_create_file_dest set to: '/home/oracle/orabase/oradata'
INFO: db_recovery_file_dest set to: '/home/oracle/orabase/fast_recovery_area'
INFO: db_files=200. Must be greater than the number of chunks and/or tablespaces to be created in the shard.
INFO: dg_broker_start set to TRUE.
INFO: remote_login_passwordfile set to EXCLUSIVE.
WARNING: db_file_name_convert is not set.
INFO: GSMUSER account validated successfully.
INFO: DATA_PUMP_DIR is '/home/oracle/orabase/oradata'.
DB Unique Name: sd1
The operation completed successfully


GDSCTL> add shard -connect sdlab4.labs.woko.oraclevcn.com:1521:sd2 -pwd Welcome1 -shardspace sdspc2 -region dc3 -deploy_as PRIMARY

메시지 생략

GDSCTL> add shard -connect sdlab4.labs.woko.oraclevcn.com:1521:sd1 -pwd Welcome1 -shardspace sdspc1 -region dc3 -deploy_as STANDBY

메시지 생략

GDSCTL> add shard -connect sdlab3.labs.woko.oraclevcn.com:1521:sd2 -pwd Welcome1 -shardspace sdspc2 -region dc2 -deploy_as STANDBY
INFO: Data Guard shard validation requested.
INFO: Database role is PHYSICAL STANDBY.
INFO: Database name is SD2.
INFO: Database unique name is sd2_stby.
INFO: Database ID is 415987369.
INFO: Database open mode is READ ONLY WITH APPLY.
INFO: Database in archivelog mode.
INFO: Flashback is on.
INFO: Force logging is on.
INFO: Database platform is Linux x86 64-bit.
INFO: Database character set is AL32UTF8. This value must match the character set of the catalog database.
INFO: 'compatible' initialization parameter validated successfully.
INFO: Database is not a multitenant container database.
INFO: Database is using a server parameter file (spfile).
INFO: db_create_file_dest set to: '/home/oracle/orabase/oradata'
INFO: db_recovery_file_dest set to: '/home/oracle/orabase/fast_recovery_area'
INFO: db_files=200. Must be greater than the number of chunks and/or tablespaces to be created in the shard.
INFO: dg_broker_start set to TRUE.
INFO: remote_login_passwordfile set to EXCLUSIVE.
WARNING: db_file_name_convert is not set.
INFO: GSMUSER account validated successfully.
INFO: DATA_PUMP_DIR is '/home/oracle/orabase/oradata'.
DB Unique Name: sd2_stby
The operation completed successfully

GDSCTL> config

Regions
------------------------
dc1
dc2
dc3
regionora

GSMs
------------------------
sddirector1
sddirector2
sddirector3

Sharded Database
------------------------
orasdb

Databases
------------------------
sd1
sd1_stby
sd2
sd2_stby

Shard spaces
------------------------
sdspc1
sdspc2

Services
------------------------

GDSCTL pending requests
------------------------
Command                       Object                        Status
-------                       ------                        ------

Global properties
------------------------
Name: oradbcloud
Master GSM: sddirector1
DDL sequence #: 0

GDSCTL> config shard
Name                Shard space         Status    State       Region    Availability
----                -----------         ------    -----       ------    ------------
sd1                 sdspc1              U         none        dc2       -
sd1_stby            sdspc1              U         none        dc3       -
sd2                 sdspc2              U         none        dc3       -
sd2_stby            sdspc2              U         none        dc2       -

```

##### 호스트 정보 등록

모든 샤드의 호스트명과 IP 주소는 카탈로그에 등록해야함
```gds
GDSCTL> config vncr

Name                          Group ID
----                          --------
10.0.19.112
sdlab3
sdlab4
```
노드가 등록 안되어 있으면 다음 명령어로 등록 해줌. 샤드 토폴로지 안에 있는 모든 서버간(샤드 카탈로그, 샤드, 디렉터 등)에 ping 프로토콜로 host명 연결 확인이 가능해야한다.

```gds
GDSCTL> add invitednode sdlab1
GDSCTL> add invitednode sdlab2
GDSCTL> add invitednode sdlab5
GDSCTL> add invitednode sdlab7
GDSCTL> add invitednode sdlab8

GDSCTL> config vncr
Name                          Group ID
----                          --------
10.0.19.112
sdlab1
sdlab2
sdlab3
sdlab4
sdlab5
sdlab7
sdlab8
```
##### 샤드 구성 배포

샤드 데이터베이스 토폴로지 구성이 완료됐다면 샤드 데이버베이스 구성을 배포해야한다.

add shard 방식으로 추가한 경우는 다음과 같은 메시지가 출력된다. CREATE SHARD 방식인 경우는 메뉴얼을 참고 바란다.
```gds
GDSCTL> deploy
deploy: examining configuration...
deploy: requesting Data Guard configuration on shards via GSM
deploy: shards configured successfully
The operation completed successfully

샤드 배포 검증

GDSCTL> config shard

Name                Shard space         Status    State       Region    Availability
----                -----------         ------    -----       ------    ------------
sd1                 sdspc1              Ok        Deployed    dc2       ONLINE
sd1_stby            sdspc1              Ok        Deployed    dc3       READ ONLY
sd2                 sdspc2              Ok        Deployed    dc3       ONLINE
sd2_stby            sdspc2              Ok        Deployed    dc2       READ ONLY

```

##### GDS pool 등록

사용자 지정 샤드 방식에서는 기본 샤드로 등록된 gdspool을 사용하게된다. 이 지정 방식에서는 추가 gdspool 이 필요치 않는다.
시스템관리 샤드 방식에서는 테이블 패밀리별로 gdspool 을 만들준다.

#GDSCTL> add gdspool -gdspool labs_pool -users sdtest

#GSM-45046: Add gdspool failed
#ORA-45581: cannot mix sharded and non-sharded pools in the same catalog
#ORA-06512: at "GSMADMIN_INTERNAL.DBMS_GSM_CLOUDADMIN", line 2933
#ORA-06512: at "SYS.DBMS_SYS_ERROR", line 79
#ORA-06512: at "GSMADMIN_INTERNAL.DBMS_GSM_CLOUDADMIN", line 2912
#ORA-06512: at line 1

#####  글로벌 데이터 서비스 생성 및 등록

샤드의 Primary, Standyby 특성에 맞게 서비스를 등록해줌

```gds
GDSCTL> add service -service rw_srvc -role primary
The operation completed successfully

GDSCTL> start service -service rw_srvc

GDSCTL> add service -service ro_srvc -role physical_standby
The operation completed successfully

GDSCTL> start service -service ro_srvc

GDSCTL> config service

Name           Network name                  Pool           Started Preferred all
----           ------------                  ----           ------- -------------
ro_srvc        ro_srvc.orasdb.oradbcloud     orasdb         Yes     Yes
rw_srvc        rw_srvc.orasdb.oradbcloud     orasdb         Yes     Yes

GDSCTL> status service
Service "ro_srvc.orasdb.oradbcloud" has 2 instance(s). Affinity: ANYWHERE
   Instance "orasdb%21", name: "sd1", db: "sd1_stby", region: "dc3", status: ready.
   Instance "orasdb%31", name: "sd2", db: "sd2_stby", region: "dc2", status: ready.
Service "rw_srvc.orasdb.oradbcloud" has 2 instance(s). Affinity: ANYWHERE
   Instance "orasdb%1", name: "sd1", db: "sd1", region: "dc2", status: ready.
   Instance "orasdb%11", name: "sd2", db: "sd2", region: "dc3", status: ready.

```

##### 샤드 상태 검증

샤딩 구성 배포 단계를 완료한 후에는 각 샤드의 세부 상태를 확인한다.
```gds
GDSCTL> config shard -shard sd1
Name: sd1
Shard space: sdspc1
Status: Ok
State: Deployed
Region: dc2
Connection string: (DESCRIPTION=(ADDRESS=(HOST=sdlab3.labs.woko.oraclevcn.com)(PORT=1521)(PROTOCOL=tcp))(CONNECT_DATA=(SID=sd1)))
SCAN address:
ONS remote port: 0
Disk Threshold, ms: 20
CPU Threshold, %: 75
Version: 19.0.0.0
Failed DDL:
DDL Error: ---
Failed DDL id:
Availability: ONLINE
Rack:


Supported services
------------------------
Name                                                            Preferred Status
----                                                            --------- ------
ro_srvc                                                         Yes       Enabled
rw_srvc                                                         Yes       Enabled

GDSCTL> config shard -shard sd1_stby

GSM-45063: Database "sd1-stby" is not found. Database does not exist or you do not have access rights to the pool.
GDSCTL> config shard -shard sd1_stby
Name: sd1_stby
Shard space: sdspc1
Status: Ok
State: Deployed
Region: dc3
Connection string: (DESCRIPTION=(ADDRESS=(HOST=sdlab4.labs.woko.oraclevcn.com)(PORT=1521)(PROTOCOL=tcp))(CONNECT_DATA=(SID=sd1)))
SCAN address:
ONS remote port: 0
Disk Threshold, ms: 20
CPU Threshold, %: 75
Version: 19.0.0.0
Failed DDL:
DDL Error: ---
Failed DDL id:
Availability: READ ONLY
Rack:


Supported services
------------------------
Name                                                            Preferred Status
----                                                            --------- ------
ro_srvc                                                         Yes       Enabled
rw_srvc                                                         Yes       Enabled
```
--------------------------------------


#### 샤드 스키마 생성

##### 샤드 계정 생성

시나리오를 위한 계정(sdtest)을 생성한다.

```sql
카탈로그 서버에서

$ sqlplus / as sysdba

SQL> alter session enable shard ddl;
SQL> create user sdtest identified by Welcome1;   // 샤드 글로벌하게 계정이 만들어진다
SQL> grant all privileges to sdtest;
SQL> grant gsmadmin_role to sdtest;
SQL> grant select_catalog_role to sdtest;
SQL> grant connect, resource to sdtest;
##SQL> grant dba to app_schema;
##SQL> grant execute on dbms_crypto to app_schema;

```
##### 샤드 테이블스페이스 생성

시나리오를 위한 테이블스페이스는 tbs1, tb2 2개를 만들 것이며, tbs1은 샤드스페이스 sdspc1에 tbs2는 샤드스페이스 sdspc2에서 관리가 될 것이다.

```sql
SQL> conn sys/Welcome1@sdcat as sysdba

SQL> ALTER SESSION ENABLE SHARD DDL;

SQL> CREATE TABLESPACE tbs1 DATAFILE SIZE 100M autoextend on next 10M maxsize unlimited extent
management local segment space management auto in shardspace sdspc1;

SQL> CREATE TABLESPACE tbs2 DATAFILE SIZE 100M autoextend on next 10M maxsize
unlimited extent management local segment space management auto in
 shardspace sdspc2;

SQL> select tablespace_name,status, ALLOCATION_TYPE, CHUNK_TABLESPACE from dba_tablespaces;
```

##### 싱글 루트 패밀리 샤드 테이블 생성

다음과 같이 Parent-Child 관계가 없는 루트 테이블을 생성한다.

```sql
SQL> connect sdtest/sdtest@sdcat
SQL> ALTER SESSION ENABLE SHARD DDL;

CREATE SHARDED TABLE accounts
( id             NUMBER not null,
account_number NUMBER,
customer_id    NUMBER,
branch_id      NUMBER,
state          VARCHAR(4) NOT NULL,
status         VARCHAR2(2),
CONSTRAINT acct_pk PRIMARY KEY(id, state)   
) PARTITION BY LIST (state)
( PARTITION p_northwest VALUES ('OR', 'WA', 'GG') TABLESPACE tbs1,
 PARTITION p_southwest VALUES ('AZ', 'UT', 'NM', 'JJ') TABLESPACE tbs2)
;
```

파티션 키에 제약조건을 지정하지 않을 경우에는 다음과 같은 에러를 만난다.

예)
```sql
~
CONSTRAINT acct_pk PRIMARY KEY(id)    // Primary 키에 파티션 키를 지정하지 않은 경우

ERROR at line 1:
ORA-14039: partitioning columns must form a subset of key columns of a UNIQUE
index
```

##### 샤드 테이블 생성 확인

테이블(파티션)이 모든 샤딩 DB에서 조회가 되나 카탈로그DB, 샤드DB(sd1, sd2)에서 테이블스페이스를 정보가 차이가 있음을 확인할 수 있음.

sdcat, sd1, sd2 db에서 테이블 리스트 확인


```sql
SQL> connect sdtest/Welcome1@sdcat
Connected.

select table_name, partition_name, tablespace_name from user_tab_partitions;
TABLE_NAME                     PARTITION_NAME                 TABLESPACE_NAME
------------------------------ ------------------------------ ------------------------------
ACCOUNTS                       P_NORTHWEST                    TBS1
ACCOUNTS                       P_SOUTHWEST                    TBS2

SQL> connect sdtest/Welcome1@sd1
Connected.

SQL> select table_name, partition_name, tablespace_name from user_tab_partitions;
TABLE_NAME                     PARTITION_NAME                 TABLESPACE_NAME
------------------------------ ------------------------------ ------------------------------
ACCOUNTS                       P_NORTHWEST                    SYS_SHARD_TS
ACCOUNTS                       P_SOUTHWEST                    TBS2

SQL> connect sdtest/Welcome1@sd2

SQL> select table_name, partition_name, tablespace_name from user_tab_partitions;
TABLE_NAME                     PARTITION_NAME                 TABLESPACE_NAME
------------------------------ ------------------------------ ------------------------------
ACCOUNTS                       P_NORTHWEST                    TBS1
ACCOUNTS                       P_SOUTHWEST                    SYS_SHARD_TS

```
##### 데이터 삽입

sd1 샤드에서 sd1에 해당하는 샤드키를 포함하는 데이터를 삽입할 경우에 정상적으로 삽입이 되지만, 다른 샤드키 값에 해당하는 데이터를 삽입할 경우에는 ORA-14466 에러 메시지를 리턴한다.  

```sql
connect sdtest/Welcome1@sd1

SQL> insert into accounts values(255,2762,5572,22,'GG','Y');

1 row created.

SQL> commit;

Commit complete.

SQL> insert into accounts values(622,1470,7878,57,'JJ','Y');
insert into accounts values(622,1470,7878,57,'JJ','Y')
            *
ERROR at line 1:
ORA-14466: Data in a read-only partition or subpartition cannot be modified.

샤드에서 사용하는 샤딩키와 다른 데이터가 입력됐기 떄문에 에러를 반환한다.

connect sdtest/Welcome1@sd2

SQL> insert into accounts values(622,1470,7878,57,'JJ','Y');

1 row created.

SQL> commit;

Commit complete.

```

##### 멀티 샤드 쿼리 실행

sd1, sd2 샤드의 모든 데이터에 대한 결과값이 리턴된다.

```sql
conn sdtest/Welcome1@sdcat

select * from accounts;

        ID ACCOUNT_NUMBER CUSTOMER_ID  BRANCH_ID STAT ST
---------- -------------- ----------- ---------- ---- --
       255           2762        5572         22 GG   Y
       622           1470        7878         57 JJ   Y

```
##### 싱글 샤드 쿼리 실행

sd1 샤드에서 쿼리를 실행하면 해당 키 값의 데이터만 리턴된다.

```sql
conn sdtest/Welcome1@sd1

SQL> select * from accounts;

        ID ACCOUNT_NUMBER CUSTOMER_ID  BRANCH_ID STAT ST
---------- -------------- ----------- ---------- ---- --
       255           2762        5572         22 GG   Y

sd2 샤드에서 쿼리를 실행하면 해당 키 값의 데이터만 리턴된다.

conn sdtest/Welcome1@sd2

SQL> select * from accounts;

        ID ACCOUNT_NUMBER CUSTOMER_ID  BRANCH_ID STAT ST
---------- -------------- ----------- ---------- ---- --
       622           1470        7878         57 JJ   Y

```
##### 듀플리케이션 테이블 생성

```sql
conn sdtest/Welcome1@sdcat

SQL> alter session enable shard ddl;

SQL> CREATE TABLESPACE products_tsp datafile size 100m autoextend
 on next 10M maxsize unlimited extent management local uniform size 1m;

  CREATE DUPLICATED TABLE Products
  (
    ProductId  INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    Name       VARCHAR2(128),
    DescrUri   VARCHAR2(128),
    LastPrice  NUMBER(8,2)
  ) TABLESPACE products_tsp;

Table created.
```

##### 패밀리 테이블 생성

다음 시나리오는 Parent-Child 관계를 가진 루트 패밀리 테이블을 생성한다. 앞의 시나리오에서 생성한  루트 테이블이 있는 상태에서 새로운 루트 테이블 생성을 하면 ORA-02530 에러를 만난다. 사용자지정 샤딩 방식에서는 오직 한개의 루투테이블만 만들 수 있다.

```sql
alter session enable shard ddl;

create sharded table customers
(
    CustId      Number(3) NOT NULL,
    Gender varchar2(1),
    Cust_email  varchar2(60),
    CustProfile VARCHAR2(4000),
    CONSTRAINT pk_customers PRIMARY KEY (CustId)
)
 PARTITION BY Range(CustId)
  ( PARTITION cst_1 values less than (250) tablespace tbs1,
    PARTITION cst_2 values less than (maxvalue) tablespace tbs2
  );

ORA-02530: parent table not specified

먼저 생성한 account 테이블을 삭제해주고 다음 시나리오를 이어간다.

root 패밀리 테이블 생성

conn sdtest/Welcome1@sdcat

alter session enable shard ddl;

create sharded table customers
(
    CustId      Number(3) NOT NULL,
    Gender varchar2(1),
    Cust_email  varchar2(60),
    CustProfile VARCHAR2(4000),
    CONSTRAINT pk_customers PRIMARY KEY (CustId)
)
 PARTITION BY Range(CustId)
  ( PARTITION cst_1 values less than (250) tablespace tbs1,
    PARTITION cst_2 values less than (maxvalue) tablespace tbs2
  );

Table created.

create sharded table orders
  ( OrderId     INTEGER NOT NULL,
    CustId      Number(3) NOT NULL,
    OrderDate   TIMESTAMP NOT NULL,
    SumTotal    NUMBER(8,2),
    Status      CHAR(4),
    CONSTRAINT  pk_orders PRIMARY KEY (OrderId, CustId),
    CONSTRAINT  fk_orders_parent FOREIGN KEY (CustId) REFERENCES Customers ON DELETE CASCADE
  )
  PARTITION BY REFERENCE (fk_orders_parent);

Table created.

```

#### 데이터 로딩 및 마이그레이션 방법

##### 비샤드 데이터베이스에서 샤드데이터베이스 마이그레이션

1. 스키마 레벨 마이그레이션

샤드 테이터베이스에서 샤드 테이블 생성 - 비샤드 DB에서 Pump로 원하는 테이블을 metadata_only로 추출 - 샤드 테이블 생성 스크립트 작성 및 저장 - sqlfile 옵션을 사용하여 샤디 DB에 import 실행  - GDSCTL VALIDATE 실행 및 확인


2. 데이터 마이그레이션

듀플리케이트 테이블 경우, 카탈로그 DB에 각종 DB 툴(Data Pump, SQLLoader, SQL 등)을 사용하여 데이터 로딩 실행하면 된다.

샤드 테이블인 경우, Data Pump 사용할 경우에 Impdp 실행은 각 샤드에서 실행하면된다. 데이터가 로딩 될 때 해당 샤드의 샤드 키 값에 해당하는 데이터만 로딩될 것이다.

3. external table 이용한 데이터 로딩

##### 샘플 데이터 생성

샘플데이터는 오라클 샘플데이터의 OE 계정이 가지있는 테이블 데이터를 활용할 것이다. 관련한 샘플데이터 만드는 방법은 오라클 메뉴얼을 참고 바란다.

```sql

conn  oe/oe

create table b_customers as
select a.customer_id as CUSTID, a.cust_address.STREET_ADDRESS as CUSTPROFILE, a.cust_email as cust_email, a.gender as gender from customers a;

create or replace view b_orders as
select order_id as ORDERID, customer_id as CUSTID, order_date as ORDERDATE, order_total as SUMTOTAL, order_status as STATUS from orders;

create table b_lineitems as
select order_id as ORDERID, product_id as PRODUCTID, unit_price as PRICE, quantity as QTY from order_items;

REM create or replace view b_products as
REM select product_id as PRODUCTID, product_name as NAME, catalog_url as DescrUri, list_price as LASTPRICE
REM from products;

create or replace view b_products as
select to_number(product_id) as PRODUCTID, product_name as NAME, list_price as LISTPRICE, catalog_url as DescrUri
from products;

select a.orderid from b_orders a, b_customers b
where a.custid = b.custid and a.custid = 101;

set heading off
set line 200
set pagesize 30000

 select e.customer_id, a.cust_address.STREET_ADDRESS, b.cust_address.CITY, c.cust_address.COUNTRY_ID, d.cust_address.POSTAL_CODE
 from customers a, customers b, customers c, customers d, customers e
 where
 e.customer_id = a.customer_id and
 e.customer_id = b.customer_id and
 e.customer_id = c.customer_id and
 e.customer_id = d.customer_id and
 e.customer_id = d.customer_id ;


select c.cust_address.COUNTRY_ID as state from customers c;



set line 200
set pagesize 2000
set colsep ','
set heading off
spool

~

spool off

```
만들어진 샘플데이터를 샤드 카탈로드, 각 샤드의 디렉토리로 옮긴다. 그리고 샤드 카랄로그, 샤드에서 Extername table 방식으로 데이터를 로딩한다.

```sql
CREATE OR REPLACE DIRECTORY shard_dir AS '/home/oracle/data';
GRANT ALL on DIRECTORY shard_dir TO sdtest;

spool

select custid, gender, cust_email, custprofile from b_customers;

sdlab3에서

sqlplus sdtest/Welcome1;

ALTER SESSION DISABLE SHARD DDL;

CREATE TABLE customers_ext (
    CustId      Number(3) NOT NULL,
    Gender varchar2(1),
    Cust_email  varchar2(60),
    CustProfile VARCHAR2(4000)
)
ORGANIZATION EXTERNAL
(TYPE ORACLE_LOADER DEFAULT DIRECTORY shard_dir
   ACCESS PARAMETERS
      (RECORDS delimited by newline
       badfile 'customers.bad'
       logfile 'customers.log'
       FIELDS TERMINATED BY ',' (CustId, Gender, Cust_email, CustProfile )
   )LOCATION ('customers.csv')
 );

ALTER SESSION ENABLE PARALLEL DML;

INSERT INTO Customers (select * from customers_ext where custid < 250);
commit;


sdlab4에서

ALTER SESSION DISABLE SHARD DDL;

CREATE TABLE customers_ext (
    CustId      Number(3) NOT NULL,
    Gender varchar2(1),
    Cust_email  varchar2(60),
    CustProfile VARCHAR2(4000)
)
ORGANIZATION EXTERNAL
(TYPE ORACLE_LOADER DEFAULT DIRECTORY shard_dir
   ACCESS PARAMETERS
      (RECORDS delimited by newline
       badfile 'customers.bad'
       logfile 'customers.log'
       FIELDS TERMINATED BY ',' (CustId, Gender, Cust_email, CustProfile )
   )LOCATION ('customers.csv')
 );


ALTER SESSION ENABLE PARALLEL DML;

INSERT INTO Customers (select * from customers_ext where custid >= 250);
commit;
```

##### 듀플리케이트 테이블 Products 데이터 로딩

```sql
sdlab2서버(카탈로그 DB)에서

sqlplus sdtest/Welcome1@sdcat

ALTER SESSION DISABLE SHARD DDL;

CREATE TABLE products_ext (
    ProductId      Number(10) NOT NULL,
    Name varchar2(60),
    DescrUri VARCHAR2(70),
    ListPrice  Number(8,2)
)
ORGANIZATION EXTERNAL
(TYPE ORACLE_LOADER DEFAULT DIRECTORY shard_dir
   ACCESS PARAMETERS
      (RECORDS delimited by newline
       badfile 'products.bad'
       logfile 'products.log'
       FIELDS TERMINATED BY ',' (ProductId, Name, DescrUri, Listprice)
   )LOCATION ('products.csv')
 );

ALTER SESSION ENABLE PARALLEL DML;

INSERT INTO products (select * from products_ext);

```

#####  데이터 수동 입력

```sql
카탈로그 서버 DB에서

insert into customers values(982,'M','test@oracle.com','test region')
commit;

sdlab4 서버에서

select * from customers where custid = 981;

sdlab3 서버 DB에서
insert into customers values(983,'F','test@oracle.com','test region')

ORA-14466: Data in a read-only partition or subpartition cannot be modified.
```
##### 카탈로그에서 truncate table 할 경우

```sql
conn sdtest/Welcome1@sdcat

SQL> truncate table accounts;

Table truncated.

GDSCTL> show ddl
id      DDL Text                                 Failed shards
--      --------                                 -------------
3       create user sdtest identified by *****
4       grant all privileges to sdtest
5       grant gsmadmin_role to sdtest
6       grant select_catalog_role to sdtest
7        grant connect, resource to sdtest
8       CREATE SHARDED TABLE accounts ( id   ...
9       drop table accounts
10      purge recyclebin
11      CREATE SHARDED TABLE accounts ( id   ...
12      truncate table accounts
```

##### 멀티샤드 쿼리 일관성 파라미터 적용

DB 파리미터 MULTISHARD_QUERY_DATA_CONSISTENCY = 'STRONG' 적용할 경우 sd1 primary db 다운시 쿼리 결과

sd1 pirmary db shutdown 후에 카탈로그 DB에서

```sql
conn sdtest/Welcome1@sdcat

SQL> select count(*) from customers
*
ERROR at line 1:
ORA-02519: cannot perform cross-shard operation. Chunk "1" is unavailable
ORA-06512: at "GSMADMIN_INTERNAL.DBMS_GSM_POOLADMIN", line 22562
ORA-06512: at "SYS.DBMS_SYS_ERROR", line 86
ORA-06512: at "GSMADMIN_INTERNAL.DBMS_GSM_POOLADMIN", line 22527
ORA-06512: at "GSMADMIN_INTERNAL.DBMS_GSM_POOLADMIN", line 22579
ORA-06512: at line 1


SQL> select count(*) from customers where custid >= 250;

  COUNT(*)
----------
       171
```

DB 파리미터 MULTISHARD_QUERY_DATA_CONSISTENCY = 'DELAYED_STANDBY_ALLOWED' 경우

sd1 primary db 다운시 쿼리 결과가 에러없이 멀티쿼리 결과가 리턴된다.

```sql
conn sdtest/Welcome1@sdcat

SQL> select count(*) from customers;

  COUNT(*)
----------
       320
```
----------------------------------------------------------------------------------------
참고 문헌

[Oracle® Database Using Oracle Sharding 19c](https://docs.oracle.com/en/database/oracle/oracle-database/19/shard/index.html)

------------------------------------------------------------------------------------------
end of document


# 데이터 분석을 위한 SQL 레시피
- 갖고 싶었던 책인데 선물받았다 :)
- 괜찮은 부분 쿼리 돌려보면서 익히고, 본 페이지에는 중요 함수 or 개념만 정리해보자.

<p align="center">
  <img width="250" height="350" src="https://user-images.githubusercontent.com/45617225/91926983-76b5aa00-ed13-11ea-97af-d60fae42d0ad.png">
</p>


## 공부하고 싶었던 점 
- SQL 이용해서 유저 단위 분석, 총 매출 관련 등등 `실무에 도움되는 분석법`을 적용하여 데이터 추출해 보고 싶다.
- 새로운 함수 학습 및 다양하게 쿼리 작성하는 연습을 하면서 현재 업무를 하면서 더 좋은 쿼리를 작성하는 방법을 익히고 싶다.

## 기초 문법 정리
```sql
SELECT 
FROM 
WHERE 
GROUP BY
HAVING 
ORDER BY 
```
##### 처리 순서 
- ```FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY```
- WHERE 절이 SELECT 보다 선행하기 때문에 SELECT 절에 정의된 값을 WHERE 절에서 참조할 수 없음 

##### SELECT 
- ```SELECT count(distinct 변수1)```
  - 이렇게 count distinct는 한 번만 사용 가능함, 두번의 count distinct는 불가능 
- ```SELECT 변수1 + 3 AS 변수2, 변수3```
  - 변수1로부터 변수2를 생성한 후, 변수2를 활용해 변수3을 생성하지 못함

##### WHERE
- ```NOT IN ()```
    - ()안에 null값이 존재하는 경우, 인식할 수 없으므로 주의가 필요함 
    
##### FROM
- FROM 뒤에 alias 써주기 
    - ```FROM () as A```

##### GROUP BY 
- group by 변수 having cnt <=2 조건 시 cnt = 1,2인 조건만 추출되며 cnt=0유저는 추출되지 않는다!
- 당연한 일을 착각함. 내가 원하는 유저가 이 조건 때문에 추출에서 제외되었다. OMG


#### 기타 함수
##### Date 다루기
- 'yyyymmdd' 형태는 연산이 불가능하기 때문에 'yyyy-mm-dd' 형태로 변경해 줘야 연산이 가능함
- ```concat_ws('-', substr(dt,1,4), substr(dt,5,2), substr(dt,7,2))```
- ```substr(변수명, 문자열 시작번째 수, 갯수)```
- ```from_timestamp(date_sub(concat_ws('-', substr('20200107',1,4), substr('20200107',5,2), substr('20200107',7)), 1), 'yyyyMMdd')``` 계산과정
    - [input value]'20200107'
    - [concat_ws function]'2020-01-07'
    - [date_sub]'2020-01-06'
    - [from_timestamp function]'20200106'
- ```dayname('2019-01-01')```
  - 요일 산출 함수, 결과는 Monday, Tuesday ,... 
- dt between '기간1' and '기간2'
  - 이렇게 between 조건을 사용할 때 기간1 < 기간2 관계가 성립해야 쿼리가 정상적으로 작동함

- ```from_timestamp( cast(to_timestamp('20191201000000','yyyyMMddHHmmss') - interval 86400 second as string) ,'yyyyMMddHHmmss')'```
     - to_timestamp : timestamp 형식으로 'yyyyMMddHHss' 형태로 변환 
     - interval : to_timestamp 형식으로 변환해주면, 날짜 계산이 가능 
     - cast : 형식 변환
     - from_timestamp : 원하는 형식으로 변환
- ```date_part('year',concat_ws('-',substr(dt,1,4),substr(dt,5,2),substr(dt,7,2))) as years```, ```year(concat_ws('-',substr(dt,1,4),substr(dt,5,2),substr(dt,7,2))) as years```
   - year 만 추출 

- ```from_timestamp(date_trunc('week',concat_ws('-',substr(dt,1,4),substr(dt,5,2),substr(dt,7,2))),'yyyy-MM-dd' ) as stdt```
   - week 단위로 끊어서 한 주 시작일자만을 남기고 싶을 때 사용
   




##### RANK 함수
```
1. RANK() OVER(partition by 변수명, order by)
- 동순위 같은 RANK, 최종 랭크는 최종 행 수가 되는 방식
- Ex) RANK : 1 2 2 4

2. DENSE_RANK() OVER(partition by 변수명, order by)
- 동순위 같은 RANK, 순위를 차례로 모두 사용해 나타냄
- Ex) RANK : 1 2 2 3

3. ROW_NUMBER() OVER(partition by 변수명, order by)
- 동순위 랜덤 RANK부여, 동순위 내에서는 랜덤으로 순위 부여됨 
- Ex) RANK : 1 2 3 4 
```
- ```RANK() OVER (PARTITION BY user_id ORDER BY count(distinct product_id) desc) as rk```
  - 유저 id 내에서 가장 많이 구입한 product_id 의 rank 를 생성할 수 있음
  
- 주로 중복 허용할 때는 rank(), 중복 허용하지 않을 때는 row_number() 를 사용함.

##### JOIN
- 중첩조인
```sql
select *
from table1 as A 
left outer join table2 as B on A.user_id = B.user_id
left outer join table3 as C on A.user_id = C.user_id
```
  - 중첩 JOIN 가능하지만, WHERE 조건을 사용하지 못하며 SELECT 절에서 처리해줘야 함  

- FULL OUTER JOIN
  - 테이블 A, B 에 있는 모든 유저를 가져오고 싶다면 기준 key를 어느 한 테이블(ex.A)에서 가져오면 B 유저 추출되지 않기 때문에 새로운 key를 만들어줘야 함.
  - full outer join 시 A테이블이 week1의 구독 작품 수가 0,1,2라고 하고 B테이블이 week2의 구독 작품 수가 0,1,2라고 하면 full outer join을 하게되면 B에 포함되지 않고 A에만 있는 유저 중에 week2에서 구독 작품 수가 2개를 초과한 유저가 있을 수 있다. 반대의 경우도 마찬가지. 처음에 이렇게 쿼리를 작성해서 우량 유저가 구독 수 0 으로 변경되어 추출되었다.


##### WITH 테이블명 as (쿼리) 
- 코드 복잡성 감소, 연산 속도상 차이 없음

##### RANDOM SAMPLING
```sql
SELECT * FROM target ORDER BY RAND(100)-- LIMIT 1000
```


*용어정리
https://www.slideshare.net/keunbongkwak/ss-127085496

*KPI서비스 용어집
https://docs.adjust.com/ko/kpi-glossary/



## Chapter1. 


## Chapter2.

## Chapter3. 

## Chapter 8. 데이터를 무기로 삼기 위한 분석 기술




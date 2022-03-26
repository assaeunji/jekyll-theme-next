---
layout: post
title: "PySpark 함수 정리"
date: 2022-03-26
categories: [Python]
tag: [pyspark, easy-guide, python]
comments: true
photos:
    - "../../images/.png"
---

* 최근 Pyspark로 코딩을 하는 일이 많은데, 계속 문법이 헷갈려 오류가 나는 경우가 많았습니다. 이 하나의 포스트로 기본 Pyspark 함수들을 정리하는 것이 목표입니다.
* [Spark By Examples](https://sparkbyexamples.com/pyspark-tutorial/){:target='_blank'}에 pyspark 튜토리얼이 잘 나와있기에 이를 활용하였습니다. (~~이 페이지에 광고만 많지 않으면 정리할 필요도 없을텐데~~)

--
## 들어가기 전에

### Pyspark는 관용적인 언어이다.

Pyspark를 사용하면서 가장 헷갈렸던 점은 컬럼을 부르는 방법입니다. 참 기본적인 것인데 헷갈렸던 이유를 생각해보면 pyspark는 **똑같은 질의이지만 여러 표현으로 가능한 관용적인 언어**였기 때문입니다.

지금까지 찾아낸 "똑같은 질의를 하고 있지만 여러 표현으로 가능한 부분"은 다음과 같습니다.
1. 컬럼을 참조할 때 다음의 세 가지 표현이 가능합니다. 마치 Python에서 컬럼을 불러올 때 `.`이나 `[]`을 통해 불러올 수 있듯이 a와 b의 표현이 가능합니다. c의 경우 `pyspark.sql.functions` 모듈을 불러왔을 때 `col()`함수를 사용하여 불러올 수 있습니다.
    a. "col"
    b. `df.col`
    c. `df["col"]`
    d. `col("col")`
2. `orderBy` 와 `sort`는 같은 결과를 냅니다. 아마도 SQL 기반의 `ORDER BY`와 Python 기반의 `sort`을 모두 고려했기 때문이라 생각합니다.
3. `filter`와 `where`는 같은 결과를 냅니다. 이 또한 `where`는 SQL 기반의 함수를 고려하였기 때문입니다.
4. `union`과 `unionAll`은 같은 결과를 냅니다. 이건 좀 의외인데 SQL에서는 `UNION` 연산은 distinct한 데이터만 가져와 합쳐주는 것인 반면, Pyspark에서는 `union`, `unionAll` 모두 `UNION ALL`을 의미한다 합니다. 
`unionAll` 함수는 없어질 예정이라 하나, 개인적으로 명시적으로 나타내는 함수가 좋아서 `union` 함수 대신 `unionAll`을 사용하고자 합니다.


### 기본 함수



| 함수명  | 설명  |
|:---:|:---:|
|`printSchema()`|테이블의 스키마를 보여주는 함수|
|`collect()`| 테이블에서 행을 가져오는 함수|
|`show(truncate=False)`| 테이블 결과를 보여주는 함수, `truncate = False`를 사용하면 테이블 내용이 잘리지 않도록 보여줍니다.|
|`describe()`|서머리 결과를 보여주는 함수|


---
## Pyspark에서 쓰는 SQL 질의용 함수

Pyspark의 함수는 대부분 SQL 언어와 비슷하게 구성되어 있습니다. 
SQL 언어와 비교하며 다음의 함수들에 대해 설명하겠습니다.

1. `select` / `drop`
2. `withColumn` / `withColumnRenamed
3. `filter`
4. `groupBy` / `orderBy`
5. `join`
6.`union` / `unionByName`
7. `pivot` 
8. `rownumber`


먼저 관련한 테이블은 다음과 같이 불러옵니다.

```python
from pyspark.sql.types import StructType,StructField, StringType, IntegerType, ArrayType
from pyspark.sql.functions import *
data = [
    (("James","","Smith"),["Java","Scala","C++"],"OH","M"),
    (("Anna","Rose",""),["Spark","Java","C++"],"NY","F"),
    (("Julia","","Williams"),["CSharp","VB"],"OH","F"),
    (("Maria","Anne","Jones"),["CSharp","VB"],"NY","M"),
    (("Jen","Mary","Brown"),["CSharp","VB"],"NY","M"),
    (("Mike","Mary","Williams"),["Python","VB"],"OH","M")
 ]
        
schema = StructType([
     StructField('name', StructType([
        StructField('firstname', StringType(), True),
        StructField('middlename', StringType(), True),
         StructField('lastname', StringType(), True)
     ])),
     StructField('languages', ArrayType(StringType()), True),
     StructField('state', StringType(), True),
     StructField('gender', StringType(), True)
 ])

df = spark.createDataFrame(data = data, schema = schema)
df.show(truncate=False)
```

```
+----------------------+------------------+-----+------+
|name                  |languages         |state|gender|
+----------------------+------------------+-----+------+
|{James, , Smith}      |[Java, Scala, C++]|OH   |M     |
|{Anna, Rose, }        |[Spark, Java, C++]|NY   |F     |
|{Julia, , Williams}   |[CSharp, VB]      |OH   |F     |
|{Maria, Anne, Jones}  |[CSharp, VB]      |NY   |M     |
|{Jen, Mary, Brown}    |[CSharp, VB]      |NY   |M     |
|{Mike, Mary, Williams}|[Python, VB]      |OH   |M     |
+----------------------+------------------+-----+------+

```

### select과 drop

`select`는 다음과 같은 SQL 질의를 표현합니다.
> As-is
```sql
SELECT name
FROM df
```

> To-Be
```python
df.select("name").show()
```


`drop`은 특정 컬럼만 제외하고 불러올 때 사용합니다.

> As-Is
```sql
SELECT languages, state, gender
FROM df
```

> To-Be
```python
df.drop("name").show() ## name을 제외한 컬럼을 불러오기
```


### withColumn 과 withColumnRenamed

`withColumn`은 컬럼의 정보를 바꾸고자 할 때 혹은 새로운 컬럼을 추가할 때 사용합니다.

> As-Is 

```sql
SELECT name, language, CAST(state as string) as state, gender
FROM df
```

```python
df.withColumn("state", df.state.cast("String")).show()
````


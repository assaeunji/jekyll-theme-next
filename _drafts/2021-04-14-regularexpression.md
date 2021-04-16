---
layout: post
title: 정규 표현식에 대한 쉬운 설명 + PySpark 실습
date: 2021-04-14
categories: [Python]
tag: [python, easy-guide, regular-expression, pyspark]
comments: true
photos:
    - "../../images/regex-title.png"
---

* 이번 포스팅은 **정규식**의 네 가지 종류에 대해 알아보고 이를 PySpark에서 실습해보는 글을 작성하고자 합니다. 
* 주요 내용은 [정규표현식 더 이상 미루지 말자](https://www.youtube.com/watch?v=t3M6toIflyQ) YouTube 강의를 참조하였습니다.

---
## 글을 쓰게 된 계기

며칠 전 업무를 하던 중 SQL 쿼리의 `LIKE` 연산자로 쓰면 여러 줄을 꼭 써줘야 하는 쿼리를 정규식으로 쉽게 표현한 적이 있었습니다.
직접적으로 언급하긴 어렵지만, 제가 처했던 난관과 비슷한 예로 **이메일 주소에서 ID와 .com이나 .co.kr 전의 도메인을 추출**하는 것을 들 수 있습니다. 
만약 `email`이라는 컬럼에 다음과 같이 세 값이 저장되어 있다 가정합시다.
* hello@gmail.com
* hello.eunji@kakao.com
* hello.eunji12@brunch.co.kr

id와 메일 도메인을 뽑아 각각 `id` 컬럼과 `domain` 컬럼에 저장하면 다음과 같은 결과를 얻을 수 있을 것입니다.

| id  |  domain |
|:---:|:---:|
|hello|gmail|
|hello.eunji|kakao|
|hello.eunji12|brunch|

이를 무식이 용감하다고 **하드 코딩** <sup>하드 코딩이란 소스 코드에 데이터를 직접 입력해서 저장한 경우를 지칭합니다.</sup> 을 하게 되면
다음과 같습니다.

```sql
select
    case 
        when email like '%@gmail%' then replace(email, '@gmail.com','') 
        when email like '%@kakao%' then replace(email, '@kakao.com','') 
        when email like '%@brunch%' then replace(email, '@brunch.co.kr','') 
    end as id
    , case 
        when email like '%@gmail%' then 'gmail' 
        when email like '%@kakao%' then 'kakao'
        when email like '%@brunch%' then 'brunch'
    end as domain
from email
```
비추천할 코드이죠. 
* 만약에 새로운 메일 도메인이 들어와 `@naver.com`이 추가 된다면 `case ... when...` 구문에 해당하는 경우가 추가되어야 하고,
* 혹은 이메일에 오타라도 나서 `@gmail.com`대신 `@gamil.com`이라 입력되었을 경우 어떠한 케이스에도 속하지 않기 때문에 NULL값을 반환하게 됩니다.

쿼리를 간단하게 작성해보자니 `@` 앞, 뒤로 아이디와 도메인 문자열 길이가 제각각이기 때문에 `substring`과 같은 문자열을 자르는 함수도 쓰기 어려웠습니다.

이렇기 때문에 **각 케이스를 일반화할 코드는 없을까?**하여 정규 표현식을 찾게 되었습니다. 
정규 표현식은 `@` 앞, 뒤를 기준으로 **패턴**을 찾아주면 아이디와 도메인 문자열 길이가 제각각이여도 적절하게 찾을 수 있기 때문입니다.

**위 예시에 대한 정규표현식 답은 정규 표현식의 네 가지 종류를 먼저 알아보고, 실습에서 다시 언급하도록 하겠습니다!**
---
## 정규 표현식의 네 가지 종류

정규 표현식이 어려운 이유는 정규 표현식이 어떻게 분류되는지 모르기 때문이라 생각합니다.
저도 이전부터 정규 표현식을 구글링한 경험이 많았지만, 항상 이해하지 못한 채 제가 원하는 정규 표현식만 찾아서 복붙하곤 했습니다.
[위키 피디아](https://ko.wikipedia.org/wiki/정규_표현식)에서 찾아봐도 이 네 가지 종류에 대한 언급이 없이 나열만 하고 있다 보니 어렵기만 합니다.

그러나 [정규표현식 더 이상 미루지 말자](https://www.youtube.com/watch?v=t3M6toIflyQ) 강의에서는 **정규 표현식의 네 가지 종류**에 대해 명확히 설명하시고, 
각각에 대해 예제를 보여줘서 처음 접하는 저도 쉽게 이해할 수 있었습니다. (강추!)

여기서 언급하고 있는 네 가지 종류는 다음과 같습니다.
* 그룹과 범위에 대한 표현 (Groups and ranges)
* 수량에 대한 표현 (Quantifiers)
* 경계의 종류 (Boundary types)
* 캐릭터 클래스 (Character classes)

차근차근 하나씩 알아보겠습니다.

---
### 잠깐만요! 그 전에 기본은 숙지합시다.

정규 표현식의 기본은 **슬래시, 패턴, 플래그** 이 세 가지를 기억하시면 됩니다.

![](../../images/regex-basic.png)

위의 이미지는 `Text`에서 `abc`라는 글자를 찾기 위한 정규식을 나타낸 것입니다.

여기서 
* **/ (슬래시)**는 정규 표현식의 시작과 끝을 나타내는 기호로, 정규 표현식인 `abc` 앞과 뒤에 붙여 줍니다.
* **패턴**은 정규 표현식으로 찾고자 하는 대상으로 `abc`에 해당합니다.
* **플래그**는 검색 옵션으로 여기선 마지막 / 뒤의 `gm`이 플래그입니다.

위 예시와 같이 정규 표현식의 패턴에 어떠한 기호 없이 문자열만 있어도 이에 해당하는 글자를 검색할 수 있습니다.
또한 플래그의 경우 다음과 같이 6개인데 자주 쓰이는 플래그인 g, m, i 만 설명드리겠습니다.

![](../../images/regex-flag.png)

* g (**g**lobal):  패턴과 일치하는 다수의 결과값들을 찾습니다. g 플래그가 없으면 패턴과 일치하는 첫 번째 결과만 반환됩니다.
* m (**m**ultiline): 다중행 모드 (multiline mode)를 활성화합니다.
* i (case **i**nsensitive): 대·소문자 구분 없이 검색합니다. 따라서 A와 a에 차이가 없습니다.


자 이제 정규 표현식의 네 가지 종류에 대해 설명하고자 하는데요. [이 사이트](https://regexr.com/5mhou){:target="_blank"}에서 함께 실습하면 더욱 좋습니다.

실습할 텍스트는 다음과 같습니다.

```
Hi there, Nice to meet you. And Hello there and hi.
I love grey(gray) color not a gry, grdy, graay and graaay.
Ya ya YaYaYa Ya

abcdefghijklmnopqrstuvwxyz
ABSCEFGHIJKLMNOPQRSTUVWZYZ
1234567890

.[]{}()\^$|?*+

010-898-0893
010-405-3412
02-878-8888

dream.coder.ellie@gmail.com
hello@daum.net
hello@daum.co.kr

https://www.youtu.be/-ZClicWm0zM
https://youtu.be/-ZClicWm0zM
youtu.be/-ZClicWm0zM
```

---
### 그룹과 범위에 대한 표현

| Chracter | 뜻                                     |
| -------- | -------------------------------------- |
| `|`     | 또는                                   |
| `()`     | 그룹                                   |
| `[]`     | 문자셋, 괄호안의 어떤 문자든                 |
| `^`    | 부정 문자셋, 괄호안의 어떤 문자를 제외하고          |
| `(?:)`   | 찾지만 기억하지는 않음                 |  

위 표현식들을 설명하기 위해 이 텍스트를 활용하겠습니다.

```
Hi there, Nice to meet you. And Hello there and hi.
I love grey(gray) color not a gry,grdy, graay, and graaay.
Ya ya YaYaYa Ya
```

1. `|`는 일반적으로 사용하는 **"OR"** 연산자입니다. 만약 Hi 또는 Hello를 검색할 경우 다음과 같이 작성합니다.
   * `Hi | Hello`

   ![](../../images/regex-g1ex1.png)
2. `()`는 그룹을 의미합니다. 만약 gray나 grey를 검색하고 싶을 경우, 그룹을 이용해서 a또는 e를 하나의 그룹으로 묶어줄 수 있습니다.
   * `gray | grey`대신 그룹을 사용하면 `gr(e|a)y`로 더 간결하게 표현할 수 있겠죠?

   ![](../../images/regex-g1ex2.png)
3. `[]`는 대괄호에 있는 모든 문자열을 검색합니다. gray, grey, grdy를 모두 검색한다 할 때 다음과 같이 쓸 수 있습니다.
   * `gr(a|e|d)y`로 그룹을 묶고 `|`연산자를 써도 되지만 `gr[aed]y`로 하면 `[]`연산자만 있어도 해당 패턴을 찾을 수 있게 됩니다.

   ![](../../images/regex-g1ex3.png)

   * `[]`연산자에서 영어 대문자와 소문자 모두, 숫자 모두를 검색하려면 다음과 같이 작성합니다.
     * `[a-zA-Z0-9]`: `-`는 ~부터 ~까지를 나타내는 걸 의미합니다.

    ![](../../images/regex-g1ex4.png)

4. `^`는 뒤에 있는 문자가 아닐 때를 반환합니다. 3번의 예에서 `^`를 붙이면 다음과 같습니다.
   * `[^a-zA-Z0-9]`: 영어 대소문자, 숫자를 제외한 나머지인 "공백"만 검색되게 됩니다.

   ![](../../images/regex-g1ex5.png)

5. `(?:)`는 그룹 `()`안에 있는 내용을 찾긴 하지만 기억하지 않는다는 의미입니다. 예를 들어 `gr(e|a)y`에서 `?:`를 그룹 앞에 적어준다면 해당 그룹을 찾아주진 않지만 그룹이 생성되지 않음을 확인할 수 있습니다.
   * `gr(?:e|a)y`

   ![](../../images/regex-g1ex6.png)

   * 위 내용이 이해가 잘 가지 않는데요. 이 예시라면 더 쉽게 이해하실 수 있습니다. 만약 `https://youtu.be/-ZClicWm0zM`라는 유투브 주소에서 `youtu.be/`뒤에 오는 아이디 **-ZClicWm0zM**만을 가져온다 가정할 때, `http`나 `www`와 같은 패턴의 도움을 받아야하지만 찾아야 하는 대상은 아닙니다. 이 때 다음과 같이 정규 표현식을 작성할 수 있습니다. (이 표현식은 어려우므로 상세 설명은 생략하고 후에 다시 말씀드리겠습니다)

   ![](../../images/regex-g1ex7.png)


---
###  수량에 대한 표현 (Quantifiers)



* `regexp_replace(컬럼, '@(.*)', '')`




https://regexr.com/5mhou

며칠 전 [퓨처 스킬](https://futureskill.io)이라는 곳을 접하게 되었는데요. 해당 사이트는 강의 클립을 짧게 보여주고 몇 개의 문제를 내어 스스로 문제를 풀고 피어 리뷰, 토론 등을 자유롭게 할 수 있는 사이트입니다. 아직 매

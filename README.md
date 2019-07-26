<!--
작성 팁:
 ![https://img.shields.io/badge/works%20with-~5.2-blue](works with ~5.2)
-->
# CTF PHP Trick
CTF에서 자주 쓰이는 PHP 트릭

## 언어 구조
PHP는 상당히 *독특하게* 작동한다. 본 목차에서는 이러한 작동 방식에 대해서 다룬다.

### ==
PHP의 비교 연산자 `==`는 *느슨하다*. 즉, 일반적으로 달라보이는 값도 같은 값으로 처리할 때가 가끔 있다. 아래는 ==의 작동 방식을 나타낸 표이다.  

|==|true|false|1|0|-1|"1"|"0"|"-1"|null|array()|array(1)|array("php")|"php"|""|NAN|
|--- |--- |--- |--- |--- |--- |--- |--- |--- |--- |--- |--- |--- |--- |--- |--- |
|true|true|false|true|false|true|true|false|true|false|false|true|true|true|false|true|
|false|false|true|false|true|false|false|true|false|true|true|false|false|false|true|false|
|1|true|false|true|false|false|true|false|false|false|false|false|false|false|false|false|
|0|false|true|false|true|false|false|true|false|true|false|false|false|true|true|false|
|-1|true|false|false|false|true|false|false|true|false|false|false|false|false|false|false|
|"1"|true|false|true|false|false|true|false|false|false|false|false|false|false|false|false|
|"0"|false|true|false|true|false|false|true|false|false|false|false|false|false|false|false|
|"-1"|true|false|false|false|true|false|false|true|false|false|false|false|false|false|false|
|null|false|true|false|true|false|false|false|false|true|true|false|false|false|true|false|
|array()|false|true|false|false|false|false|false|false|true|true|false|false|false|false|false|
|array(1)|true|false|false|false|false|false|false|false|false|false|true|false|false|false|false|
|array("php")|true|false|false|false|false|false|false|false|false|false|false|true|false|false|false|
|"php"|true|false|false|true|false|false|false|false|false|false|false|false|true|false|false|
|""|false|true|false|true|false|false|false|false|true|false|false|false|false|true|false|
|NAN|true|false|false|false|false|false|false|false|false|false|false|false|false|false|false[^NAN===NAN은 false이다.]|

...사실 `==`에서 수많은 취약점이 도래하기도 한다.

#### Magic Hash
MD5, SHA 등의 해시 함수를 이용해서 비밀번호 등을 비교하고, `==`다면, Magic Hash 트릭을 사용할 수 있다. 아래를 보자.

```php
"0e1354" == "0e87453" // true
"0" == "0e7124511451155" //true
"0" == md5("240610708") // true
"0" == sha1("w9KASOk6Ikap") // true
md5("QLTHNDT") == md5("QNKCDZO") // true
```

### 문자열 함수 호출
PHP에서는 함수를 호출할 때 *문자열*로 호출할 함수를 정할 수 있다. 아래 코드를 보자.

```php
"phpinfo"();
```

위와 같은 코드에서는, 정상적으로 `phpinfo();`가 작동하게 된다.  
당연히 정상적인 개발자는 문자열에다가 대고 함수를 호출하는 방식으로 코드를 짜지 않기 때문에, 보통 PHP jail을 풀 때 자주 쓰인다. `system`이 필터링되고 있을 때, `system` 없이 `system('ls')`를 호출할 수 있을까?

```php
"sys"."tem"('ls');
```

따옴표를 쓰지 못한다면?

```php
getallheaders()[chr(65)](getallheaders()[chr(66)]); // 헤더에 A: system, B: ls를 붙여서 요청해야 한다.
```

### register_globals
![https://img.shields.io/badge/works%20with-~5.3-blue](works with ~5.3)
굳이 PHP 5.3 이하임을 강조하고 있거나, 어쨌든 그렇다면, `register_globals`가 활성화되어있을 수 있다. *입력을 그대로 전역 변수에 선언*하는 설정이다.

```php
$FLAG = "FLAG{You are king-god}";
if(isset($admin)) {
   echo $FLAG;
}
```

위와 같은 코드가 있을 때, `register_globals`가 활성화되어있다면, `https://victim.com?admin=1`로 들어갈 수 있다. 그러면 `$admin`이 선언되고, 결과적으로, 위 코드에서는 플래그가 출력될 것이다.

## 취약한 함수
PHP의 수많은 함수는 생각보다 **더** 취약하다. 아래는 그의 목록이다.

WIP

## 기타

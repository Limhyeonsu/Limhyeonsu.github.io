---
title: 2장 JUnit과 Hamcrest
date: 2022-05-06 15:27:32 +0900
categories: [BOOK, TDD 실천법과 도구]
tags: [tdd]  # TAG는 반드시 소문자로 이루어져야함!
---

### 2.1 JUnit
JUnit은 현재 전 세계적으로 가장 널리 사용되는 Java 단위 테스트 프레임워크이다. 기본적으로 다음과 같은 기능을 제공한다.
* 테스트 결과가 예상과 같은지를 판별해주는 단정문
* 여러 테스트에서 공용으로 사용할 수 있는 테스트 픽스처
* 테스트 작업을 수행할 수 있게 해주는 테스트 러너

꼭 알아두어야할 개념으로 테스트 픽스처(테스트 기반 환경 또는 테스트를 위한 구조물)라는 개념이다.

#### 테스트 픽스처
테스트 픽스처는 테스트를 반복적으로 수행할 수 있게 도와주고 매번 동일한 결과를 얻을 수 있게 도와주는 '기반이 되는 상태나 환경'을 의미한다. (일관된 테스트 실행환경이라고도 한다.)

예) 테스트 케이스에서 사용할 객체의 인스턴스를 만든다, 데이터베이스와 연동할 수 있는 참조를 선언한다 등등

그리고 테스트 픽스처를 만들고 정리하는 작업을 수행하는 메소드를 '테스트 픽스처 메소드'라고 한다.

#### 테스트 케이스와 테스트 메소드
테스트 케이스는 테스트 작업에 대한 시나리오적인 의미가 강하고, 테스트 메소드는 JUnit의 메소드를 지칭한다. 하지만 두 단어는 종종 혼용되어 사용되기 때문에 동일한 뜻으로 이해해도 된다.

#### JUnit3
JUnit은 junit.org 사이트에서 파일을 내려받아 클래스 경로내에 junit.jar 파일을 포함시켜 놓으면 바로 사용할 수 있다. JUnit3은 크게 두 개의 규칙과 네 개의 구성요소를 갖는다.

[규칙]
* TestCase를 상속받는다.
* 테스트 메소드의 이름은 반드시 test로 시작해야 한다.

[구성요소]

1)테스트 픽스처 메소드
* setUp() : 각각의 테스트 메소드가 실행되기 전에 공통으로 호출되는 메소드다. 주로 테스트 환경 준비에 해당하는 자원 할당, 객체 생성, DB 연결 등의 작업이 이루어진다.
* tearDown() : 각각의 테스트 메소드가 실행된 후 수행되는 메소드다. 자원 해제, 연결 해제, 객체 초기화 등 뒷정리 작업을 한다.
```java
public class DaoTest extends TestCase{
  Connection connection;

  protected void setUp() throws Exception {
    connection = Connection.getConnection();
  }

  public void testA() throws Exception {....}
  public void testB() throws Exception {....}

  protected void tearDown() throws Exception {
    account = Connection.releaseConnection();
  }
}
```
setUp -> testA -> tearDown, setUp -> testB -> tearDown 식으로 실행된다.

2)단정문
* assertEquals([message], expected, actual) : 예상 값과 실제 테스트 결과 값이 서로 일치하는지를 비교한단 한다. double, float 형의 경우 정확하게 일치하는 값을 찾기 어려워 delta라는 오차 값을 이용해 적절한 범위내의 값은 동일한 값으로 판단해준다. 단 `float 타입끼리 비교는 제공하지 않음`
* assertSame / assertNotSame([message], expected, actual) : 두 객체가 동일한 객체인지 `주소값`으로 비교하는 단정문이다. 주로 동일 객체임을 증명하거나 싱글톤으로 만들어진 객체를 비교할 때 쓰이기도 한다.
* assertTrue / assertFalse([message], expected) : 예상 값의 참, 거짓을 판별하는 단정문이다.
* assertNull([message], expected) : null 여부를 판단하는 단정문이다.
* fail([message]) : 호출 즉시 테스트 케이스는 실패한다. 단정문을 사용하지 않으면 예외가 발생하지 않은 이상 무조건 성공하는 테스트 케이스가 되므로 사용한다. __만일 테스트 케이스를 작성 중 완료하지 못한 상태에서 구현을 중단해야 하는 경우 끝 부분에 fail()을 추가해 놓으면 도움이 된다.__

3)테스트 러너

JUnit 프레임워크는 엄연히 독립적인 소프트웨어이고, 때문에 명령행 프롬프트에서 실행하거나 셸 스크립트 등을 이용해 실행할 수도 있다. 이를 위해 JUnit은 테스트 러너라는 테스트 실행 클래스를 제공한다. (Swing UI, 텍스트, AWT UI 제공)
```
junit.swingui.TestRunner.run(Test.Class);
junit.textui.TestRunner.run(Test.Class);
junit.awtui.TestRunner.run(Test.Class);
```

4)테스트 스위트
* 여러 개의 테스트 케이스를 한꺼번에 수행하고자 할 때
* 테스트 스위트는 테스트 케이스와 다른 테스트 스위트를 포함시킬 수 있다.
* 메소드는 반드시 __publid static Test suite()__ 여야 한다.
* 테스트 추가는 suite.addTestSuite(테스트 클래스.class) 형식을 갖는다.

```java
Class SuiteTest {
  public static void main(String[] args) {
    junit.swingui.TestRunner.run(SuiteTest.class);
  }

  public static Test suite() {
    TestSuite suite = new TestSuite();
    suite.addTestSuite(SuiteTest.class);      // 테스트 케이스 추가
    suite.addTestSuite(SuiteTest2.class);     // 테스트 케이스 추가
    suite.addTest(AnotherSuiteTest.suite());  // 다른 테스트 스위트를 포함

    return suite;
  }
}
```
테스트 스위트는 여러 개의 테스트 케이스를 함께 수행할 때 사용하나 현재는 잘 사용하지 않는다.

#### JUnit3으로 테스트 케이스 작성하기
```java
public class AccountTest extends TestCase{
    Account account;

    protected void setUp() throws Exception{
        account = new Account(10000);
    }

    public void testGetBalance() {
        assertEquals(10000, account.getBalance());
    }

    public void testDeposit() {
        account.deposit(1000);
        assertEquals(11000, account.getBalance());
    }

    public void testWithdraw() {
        account.withdraw(1000);
        assertEquals(9000, account.getBalance());
    }
}
```
특징으로 TestCase를 상속한 점과 @로 시작하는 애노테이션이 없다는 점이다.

**JUnit 테스트 클래스에는 각 테스트 메소드 별로 생성자가 호출된다. 이는 Java의 리플렉션을 사용해서 테스트 메소드를 실행할 때마다 테스트 클래스를 강제로 인스턴스화 하는 것이다.
그 이유는 좋은 테스트 케이스는 기본적으로 다른 테스트 케이스의 수행이나 수행 결과에 영향을 받지 않아야 한다. 이것이 테스트의 기본 원칙이다. 따라서 테스트 케이스를 독립적으로 수행하기 위해 테스트 메소드 수행 전 테스트 클래스 자체를 리셋하는 것이다.

#### JUnit4
[특징]
* Java5 애노테이션 지원
* test 로 method 이름을 시작해야 한다는 제약 해소 : @Test 사용
* 좀 더 유연한 픽스처 : @Before, @After, @BeforeClass, @AfterClass
* 예외 테스트 : @Test(expected=NumberFormatException.class)
* 시간 제한 테스트 : @Test(timeout=1000)
* 테스트 무시 : @Ignore("this method isn't working yet")
* 배열 지원 : assertArrayEquals([message], expected, actual)
* @RunWith(클래스이름.class) : 테스트 클래스 실행 전 러너를 명시적으로 지정한다.
* @SuiteClasses(Class[]) : 여러개의 테스트 클래스를 수행하기 위해 쓰인다.
* 파라미터를 이용한 테스트

1)애노테이션

JDK 1.5에서 많은 변화가 있었는데 대표적으로 애노테이션, 제네릭스, 향상된 for문, 타입세이프한 열거형 타입 등이 있다. 애노테이션의 가장 큰 장점은 프레임워크의 내부 모델에 대한 자세한 이해 없이도 각 메소드의 사용 의도를 명확하게 문서화한다는 점이다.
JUnit 3버전에서는 애노테이션을 사용할 수 없었으나 TestNG라는 애노테이션 기반의 xUnit 테스트 프레임워크가 JDK 1.4 이하에서도 애노테이션을 쓸 수 있게 해줬다.

JUnit 4는 TestNG의 상당 기능을 그대로 차용해왔고, 인지도 측면에서 JUnit이 월등이 높아 JUnit 4가 나오면서 애노테이션을 이용한 테스트 케이스 작성시 JUnit이 주로 사용되게 됐다.

2)@Test

테스트 케이스에 해당하는 메소드로 지정하기 위해서 메소드 이름을 소문자 test로 시작해야한다는 규칙이 있었으나 JUnit 4에서는 메소드 이름과 상관없이 @Test 애노테이션만 붙이면 테스트 메소드로 인식한다.

3)테스트 픽스처 메소드 추가 지원

버전3에서는 setUp, tearDown이라는 두 개의 테스트 픽스처 메소드를 제공했는데 버전 4에서는 각각 @Before, @After라는 이름의 애노테이션으로 지원한다. 또 단 한 번만 실행할 수 있는 기능을 제공하지 않았으나
@BeforeClass, @AfterClass 라는 두 개의 애노테이션을 이용해 하나의 테스트 클래스 내에서 한 번만 실행하는 메소드를 만들 수 있게 한다.

4)예외 테스트

3버전에서는 예외를 테스트하는 공식적인 방법을 제공하지 않았다. 대신 try/catch 문과 assert 단정문을 일종의 트릭처럼 사용했으나 4에서는 애노테이션을 이용해 작성한다.
`@Test(expected=NumberFormatException.class)` expected 값으로 예외 클래스를 지정했을 때, 테스트 메소드 내에서 해당 예외가 발생하지 않는다면 테스트 메소드를 실패로 간주하낟.

5)테스트 시간 제한

`@Test(timeout=5000)` 밀리초 단위의 시간을 정해준 후 해당 시간내에서 테스트 메소드가 수행완료되지 않으면 실패한 테스트 케이스로 간주한다.

6)테스트 무시

`@Ignore` 애노테이션을 붙이면 지우기 전까지 수행하지 않는다.

7)배열 지원

원소의 자리 순서 기준으로 equals 비교가 이뤄지기 때문에 배열안의 값이 동일하더라도 순서가 다르면 테스트가 실패한다.
```java
@Test
public void testArrayAssertEquals() throws Exception {
  String [] names = {"Tom", "JIMMY", "JOHIN"}
  String [] anotherNames = {"Tom", "JIMMY", "JOHIN"}
  assertArrayEquals(name, anotherNames);
}
```

8)@RunWith

각각의 테스트 메소드 실행을 담당하고 있는 클래스를 테스트 러너라고 한다. @RunWith 애노테이션은 JUnit에 내장된 기본 테스트 러너인 BlockJUnit4ClassRunner 대신 @RunWith 로 지정된 클래스를 이용해 클래스 내의 테스트 메소드들을 수행하도록 지정해주는 애노테이션이다.
예를 들면 스프링 프레임워크에서 제공하는 SpringJUnit4ClassRunner.class를 지정하면 스프링에서 자체적으로 만들어놓은 추가적인 테스트 기능을 이용할 수 있게 된다.

9)@SuiteClasses

여러개의 테스트 클래스를 일괄적으로 수행할 수 있다.

```java
//4버전
@RunWith(Suite.class)
@SuiteClasses(ATest.class, BTest.class, CTest.class)
public class ABCSuite{...}

//3버전
public class ABCSuite extends TestCase{
  public static Test suite() {
    TestSuite suite = new TestSuite();
    suite.addTestSuite(ATest.claa);
    suite.addTestSuite(BTest.claa);
    suite.addTestSuite(CTest.claa);
    return suite;
  }
}
```

10)파라미터화된 테스트

하나의 메소드에 대해 다양한 테스트 값을 하꺼번에 실행시키고자 할 때 사용한다.

11)룰(Rule)

하나의 테스트 클래스 내에서 각 테스트 메소드의 동작 방식을 재정의하거나 추가하기 위해 사용하는 기능이다.
* TemporaryFolder : 테스트 메소드 내에서만 사용 가능한 임시 폴더나 임시 파일을 만들어 준다.
* ExternResource : 외부 자원을 명시적으로 초기화한다.
* ErrorCollector : 테스트 실패에도 테스트를 중단하지 않고 진행할 수 있게 도와준다.
* Verifier : 테스트 케이스와는 별개의 조건을 만들어서 확인할 때 사용한다.
* TestWatchman : 테스트 실행 중간에 사용자가 끼어들 수 있게 도와준다.
* TestName : 테스트 메소드의 이름을 알려준다.
* Timeout : 일괄적인 타임아웃을 설정한다.
* ExpectedException : 테스트 케이스 내에서 예외와 예외 메시지를 직접 확인할 때 사용한다.

12)이론(Theory)

테스트 데이터와 상관없이 작성 대상 메소드를 항상 유지해야 하는 논리적인 규직을 표현할 때 사용한다.


### 2.2 비교표현의 확장 : Hamacrest(햄크레스트)
jMock이라는 Mock 라이브러리 저자들이 참여해 만들고 있는 Matcher 라이브러리로 테스트 표현식을 작성할 때 좀 더 문맥적으로 자연스럽고 우아한 문장을 만들 수 있게 도와준다.
Matcher는 이름 그대로 어떤 값들의 상호 일치 여부나 특정한 규칙 준수 여부 등을 판별하기 위해 만들어진 메소드나 객체를 지칭한다.

Hamacrest는 다양한 Matcher들이 모인 Matcher 집합체다. 단위 테스트와 함께 사용하면 테스트 케이스 작성시 문맥적으로 좀 더 자연스러운 문장을 만들어준다.

기본적으로 assertEquals 보다는 assertThat이라는 구문 사용을 권장한다.

예) assertThat(account.getBalance(), is(equalTo(10000)));, assertThat(resource.newConnection(), is(notNullValue())); assertThat(account.getBalance(), isGreaterThan(0));

위 예제에서 is, equalTo, isGreaterThan 등의 메소드가 Matcher 구문에 해당한다. 이 구문은 static으로 선언되어 있고, 리턴 값은 Matcher 클래스로 되어있다.
따라서 사용하려면 `import static org.junit.Assert.*;`, `import static org.hamcrest.CoreMatchers.*;`

Hamacrest 라이브러리를 사용시 실패 메시지는 예상 값이 이것인데 실제로는 이 값이 나왔음의 형태로 보여준다.

[Hamacrest 패키지]
* org.hamcrest.core : 오브젝트나 값들에 대한 기본적인 Matcher
* org.hamcrest.beans : Java Bean과 그 값 비교에 사용되는 Matcher
* org.hamcrest.collection : 배열과 컬렉션 Matcher
* org.hamcrest.number : 수를 비교하기 위한 Matcher
* org.hamcrest.object : 오브젝트와 클래스를 비교하는 Matcher
* org.hamcrest.text : 문자열 비교
* org.hamcrest.xml : XML 문서 비교

[Matcher 종류에 따른 분류]

<img src="/assets/img/posting_img/book/TDD/matcher1.jpeg" width="700px">
<img src="/assets/img/posting_img/book/TDD/matcher2.jpeg" width="700px">
<img src="/assets/img/posting_img/book/TDD/matcher3.jpeg" width="700px">

#### 사용자 정의 Matcher 만들기
자신만의 비교 구문 Matcher를 만들고 싶다면 TypeSageMatcher를 상속받아서 matchesSafely, describeTo를 재정의하면 된다.

```java
package main;

import org.hamcrest.Description;
import org.hamcrest.Factory;
import org.hamcrest.Matcher;
import org.hamcrest.TypeSafeMatcher;

public class IsNotANumber extends TypeSafeMatcher<Double> {

    @Override
    protected boolean matchesSafely(Double number) {
        return number.isNaN();
    }

    @Override
    public void describeTo(Description description) {
        description.appendText("not a number");
    }

    @Factory
    public static <T> Matcher<Double> notANumber() {
        return new IsNotANumber();
    }
}
```
```
public void testSquareRootOfMinusOneIsNotANumber() {
  assertThat(Math.sqrt(-1), is(notANumber()));
}
```

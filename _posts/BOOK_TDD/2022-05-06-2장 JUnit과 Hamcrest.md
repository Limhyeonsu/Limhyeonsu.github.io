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
```
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

```
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
```
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

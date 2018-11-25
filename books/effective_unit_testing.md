## Part 1. 기반 다지기

### Chapter 3. 테스트 더블

#### 의미

테스트 더블은 테스트를 실행하기 위한 스텁, 더미와 같은 다양한 객체들을 의미하며 얻어지는 효과는 아래와 같다. 

- 테스트 대상 코드를 격리한다.
- 테스트 속도를 개선한다.
- 예측 불가능한 실행 요소를 제거한다.
- 특수한 상황을 시뮬레이션한다.
- 감춰진 정보를 얻어낸다.

#### 종류

##### Stub

원래의 구현을 최대한 단순한 것으로 대체하는 것, 아무런 기능이 없고 아무런 행위를 하지않는다.

테스트에서 대상 협력 객체에 대해서 아무런 관심이 없을 때 사용한다.

##### Fake Object 

원래 기능의 행위를 단순하게 모방함. 테스트간에 재사용이 가능하며 많은 테스트코드를 줄여주는 효과가 있다.

예를 들어서 `RedisCache` 클래스에서 Key/Value를 저장하거나 읽어오는 일은 Stub, Mock보다는 `FakeCache`를 구현하는 편이
훨씬 가독성에도 활용성도 좋을 수 있다.

##### Spy

행위에 대해서 **검증**하는 역할. 주로 아래와 같은 상황에서 사용한다. 

협력 객체에 대해서 
- 메서드 호출 여부 검증
- 메서드에 전달된 파라미터 값 검증
- 메서드 호출 횟수 검증

##### Mock

어떻게보면 Stub의 발전 형태도 띄는데, 예를 들어 stub으로 구현할 때 특정 조건에 의해서 어떤 값을 반환할지만 설정한다면,
Mock은 지정된 값이 들어오지 않은 경우 테스트를 실패시키거나 아무런 행위를 하지 않는 것, 호출할 때마다 다른 반환 값을
지정할 수도 있다.

Spy를 만들기위해 사용되기도 한다. Spy를 손수 하나씩 만들기에는 보통 중복되고 혹은 더 많은 코드를 요구하지만 Mock Library를
사용하다보면 행위 검증에 대해서 훨씬 쉽게 작성할 수 있는 장점이 있다. 아래는 주로 사용하는 C# 코드다


```csharp
doMock.Verify(d => d.Something(It.IsAny<string>()), Times.Onece);
```

#### 활용 지침

- 두 객체간 상호작용으로 특정 메서드 호출 여부를 알고 싶다면 `Mock`
- `Mock`으로 코드가 깔끔하게 정리되지 않는다면 `Spy`
- 협력 객체가 자리만 지키면 되고 응답 값을 테스트에서 통제 가능하다면 `Stub`
- 테스트 코드의 관리가 어려워지거나 시나리오가 복잡하다면 `Fake Object`
- 위 케이스에서 구분이 불가능하다면 동전을 던져서 선택해보자.

> Stub은 질문하고 Mock은 행동한다.

#### 준비하고, 시작하고 단언하라.

> AAA : Arrange-Act-Assert, GIVEN-WHEN-THEN

1. 준비 : 테스트 협력 객체 + 테스트 더블을 준비
1. 시작 : 실제 동작을 호출
1. 단언 : 반환 값이 옳바르고, 적절한 행위를 했는지 검증


#### 구현이 아니라 동작을 검증해라

---
## Part 2. Test Smell

### Chapter 4. 가독성

#### 기본 타입 단언 (Primitive Assertion)

의미를 알 수 없는 단어, 숫자(매직 넘버)에 가려진 상황. 아래 상황의 경우 **포함하고 있는 경우**를 `!= -1`의 indexOf 반환 타입을 통해서 유추할 수 있음. 이것은 추가적인 인지 부하가 생기므로 추상화된 메서드를 정의하거나 사용하는 편이 좋음.


```csharp
// What?
assertTrue(sentance.indexOf("CONTIAN_STRING") != -1);

// Fix
assertTrue(sentance.contains("CONTAIN_STRING"));
assertTrue(isContainsString(sentance, "CONTAIN_STRING"));
```

#### 광역 단언

너무 범위가 크게 단언하는 것, 테스트 케이스가 실패할 확률이 비교적 높고 하려고 하는 것이 무엇인지 알 수 없어 유지보수가 힘듦

##### 예시

```csharp
[Fact]
public void AdVenderParser_GetReport_WhenParametersIsValidated()
{
    // Given
    var parser = new FacebookReportParser();

    // When
    var report = parser.GetReports();

    // Then
    report.Should().NotBeNull().And.HaveAnyItems();
}
```

1. 파라미터가 유효한 것이 무엇을 의미하는지 전혀알 수 없음
2. `NotBeNull`은 어차피 `HaveAnyItems()`에서 확인할 수 없음
3. 네트워크 에러나 파서 버전 등 실패하는 이유가 너무 다양함.

SRP 원칙처럼 테스트가 실패하는 이유는 단 하나여야함. 하지만 위 예제에서는 테스트가 너무 많이 실패할 수 있고, 실제로 값이 반환된다고 하더라도 유의미한 결과 검증에 대해선 누락되어 있어 실패할 가능성이 높은 무의미한 테스트임.


##### 해결책

- 하나의 테스트를 상세하게 여러 테스트로 쪼갬. 아래와 같은 테스트를 예상할 수 있음.
    - 파라미터 검증 테스트
    - 리포트 값 검증 테스트
    - 리포트 갯수 테스트
- 의미 없는 단언 `NotNull` 혹은 다음 단언에서 확인이 가능한 단언은 하나로 축소하거나 다시 작성할 것.


#### 비트 단언

비트 단언의 경우 추상화의 수준이 너무 낮아 테스트의 의도나 의미를 파악하기 어려움

```csharp
[Fact]
public void PlatformBitTest()
{
    (Platform.IS_32_BIT ^ Platform.IS_64_BIT).Should().BeTrue();
}
```

`^`는 XOR연산자로 `A이거나 B`를 의미함. 즉 둘 다 다른 값인 경우에만 참인 논리 연산자. 하지만 대부분의 개발자의 경우 이 연산자를 쓰는 경우가 적고 의미가 저수준의 연산자에 가려진 상황이라 더 많은 인지부하가 필요한 테스트이다.

이를 아래와 같이 읽기 편하게 풀어쓸 수 있다.

```csharp
(Platform.IS_32_BIT || Platform.IS_64_BIT).Should().BeTrue();
(Platform.IS_32_BIT && Platform.IS_64_BIT).Should().BeFalse("Can't be 32bit and 64bit at the same time.");
```

#### 부차적 상세정보

테스트 코드에 테스트 내용과 관련없는 부수적인 정보가 넘쳐 흐르는 경우.

```csharp
[Fact]
public void Test()
{
    // Given
    var a = new Something(1);
    var b = new Something(2);
    var c = new Something(3);
    var d = new Something(4);
    var e = new Something(5);

    var composite = new CompositeSomething();
    composite.AddChild(a);
    composite.AddChild(b);
    composite.AddChild(c);
    composite.AddChild(d);
    composite.AddChild(e);

    // ....
}
```
위 예제에서 테스트를 위해 상황을 조합하는 코드는 실제로 테스트와는 크게 상관없는 코드. 이를 `[BeFore]` 혹은 생성자에 위임하는 것도 하나의 방법.

#### 다중 인격(Split Personality)

여러 테스트 목적이 하나의 테스트에 결합되어 있는 형태, 광역 단언과 다른 점은 단언 자체를 너무 크게하는 것과 여러가지 테스트 케이스를 길게 검증하거나 시나리오 형태로 검증하는 형태로 방법의 차이가 비교적 명확함.

이 경우 테스트 케이스를 여러개로 분리하고 하나의 테스트가 하나의 이유로 실패하도록 유도해야함

#### 쪼개진 논리

테스트 클래스는 한 곳에 있느나 오직 그 테스트 케이스를 위한 픽스쳐나 테스트 리소스가 다른 장소(파일)에 있는 경우 인지부하로 이어짐. 따라서 이를 한 곳으로 모아야함. 하지만 이는 상황에 따라서 안하는 것만 못한 경우가 있으므로 아래와 같은 가이드를 따를 것

1. 짧다면 통함
1. 통합하기 너무 길다면 팩토리 메서드나 테스트 데이터 생성기를 통해 제작
1. 위 방법 또한 쉽지 않다면 독립파일로 남겨둘 것

#### 매직 넘버

의미를 알 수 없는 단어 숫자에 가려진 상황, 예를 들면 아래 코드에서 `10`은 어떠한 정보로 전달하고 있지 않음.

```csharp
// When
var reports = reporter.GetReport();

// Then
reports.Should().HaveCount(10);
```

이를 해결하자면 의미 있는 숫자를 상수로 제공하거나 아래와 같이 읽기 좋게 만드는 방법이 필요함

```csharp
reports.Should().HaveCount(reportItem(10));
```

#### 셋업 설교

이전에 부차적인 상세 정보와 같이 테스트에서 실제로 관심이나 연관이 적은 맥랙 생성 코드를 셋업이나 생성자에 옮겨 테스트를 행위와 단언에 집중할 수 있도록 만들었다면 셋업 설교는 너무 상세하고 긴 준비작업을 의미함.

1. 셋업에서 핵심을 제외한 상세 정보는 private 메서드로 추출한다.
1. 알맞은 서술적 이름 사용
1. 셋업 내의 추상화 수준을 통일

예시 코드로 방법을 알아보자.

```csharp
public TestClass()
{
    _mockA = new Mock<IA>();
    _mockA
        .Setup(a => a.DoSomething())
        .Return(100);

    _mockB = new Mock<IB>();
    _mockB
        .Setup(b => b.DoSomething())
        .Return("RESULT OF B");

    _mockC = new Mock<IC>();
    _mockC
        .Setup(c => c.DoSomething())
        .Return(true);
}

// Fix

public TestClass()
{
    MockA();
    MockB();
    MockC();
}
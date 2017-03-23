# C#에서의 Rhino Mock 활용 (Observer pattern 기반)

[옵저버 패턴](https://ko.wikipedia.org/wiki/%EC%98%B5%EC%84%9C%EB%B2%84_%ED%8C%A8%ED%84%B4) 예제를 구현하는 과정에서 Rhino Mock을 활용해본다.
우리가 사용할 예제는 `Head First Design Pattern`에서 나오는 예제인데,
Observer 인터페이스만 있는 상태에서
Subject 인터페이스를 구현하는 WeatherDataSubject를 구현하는 예제이다.

 ```c#
using System;
using System.Collections.Generic;

namespace ObserverTest2
{
    public interface Subject
    {
        void registerObserver(Observer o);
        void removeObserver(Observer o);
        void notifyObservers();
    }

    public interface Observer
    {
        void update(TemperatureInfo temperaturInfo);
    }

    public class TemperatureInfo
    {
    }
}
```

우선 `registerObserver()` 기능을 구현하기 전에 실패하는 테스트 코드를 작성한다.

``` C#
using NUnit.Framework;
using Rhino.Mocks;
using System.Collections.Generic;

namespace ObserverTest2
{
    [TestFixture]
    public class UnitTest1
    {
        [Test] 
        public void TestRegisterObserver()
        {
            // given
            ISet<Observer> set = new HashSet<Observer>();
            Subject subject = WeatherDataSubjectFactory.getDefaultSubjectWithHashSet(set);
            Observer observerStub = MockRepository.GenerateStub<Observer>();

            // when
            subject.registerObserver(observerStub);

            // then
            Assert.AreEqual(1, set.Count);

        }
    }
}
```

Visual Studio 를 활용하여 컴파일에러를 해결한다.
컴파일 에러를 모두 해결하면 다음과 같은 코드가 추가되어 있을 것이다.
`Observer` 인터페이스를 `MockRepository` 를 통해서 해당 인터페이스를 상속받는
임의의 객체를 만들므로 우리가 `Observer`를 구현한 *concrete* 클래스를 만들 필요가 없다.
> 여기에서는 Observer 인터페이스를 구현한 객체의 행위를 모니터링할 필요가 없으므로 stub로 만든다.

``` C#
    class WeatherDataSubjectFactory
    {
        internal static Subject getDefaultSubjectWithHashSet(ISet<Observer> set)
        {
            return new WeatherDataSubject(set);
        }
    }

    internal class WeatherDataSubject : Subject
    {
        private ISet<Observer> set;

        public WeatherDataSubject(ISet<Observer> set)
        {
            this.set = set;
        }

        public void notifyObservers()
        {
            throw new NotImplementedException();
        }

        public void registerObserver(Observer o)
        {
            throw new NotImplementedException();
        }

        public void removeObserver(Observer o)
        {
            throw new NotImplementedException();
        }
    }
```

(실패하기 위한) 테스트를 수행해본다. 
다음과 같이 테스트가 실패한다.

```
테스트 이름:	TestRegisterObserver
테스트 전체 이름:	ObserverTest2.UnitTest1.TestRegisterObserver
테스트 소스:	C:\Users\idept\Documents\Visual Studio 2015\Projects\ObserverTest2\ObserverTest2\UnitTest1.cs : 줄 12
테스트 결과:	실패
테스트 지속 시간:	0:00:00.223

Result StackTrace:	
위치: ObserverTest2.WeatherDataSubject.registerObserver(Observer o) 파일 C:\Users\idept\Documents\Visual Studio 2015\Projects\ObserverTest2\ObserverTest2\Observer.cs:줄 46
   위치: ObserverTest2.UnitTest1.TestRegisterObserver() 파일 C:\Users\idept\Documents\Visual Studio 2015\Projects\ObserverTest2\ObserverTest2\UnitTest1.cs:줄 19
Result 메시지:	System.NotImplementedException : 메서드 또는 연산이 구현되지 않았습니다.
```

테스트를 성공하게 하기 위하여 `registerObserver()` 를 아래와 같이 구현한다.

```C#
        public void registerObserver(Observer o)
        {
            set.Add(o);
        }
```

테스트가 성공한다.
```
테스트 이름:	TestRegisterObserver
테스트 전체 이름:	ObserverTest2.UnitTest1.TestRegisterObserver
테스트 소스:	C:\Users\idept\Documents\Visual Studio 2015\Projects\ObserverTest2\ObserverTest2\UnitTest1.cs : 줄 12
테스트 결과:	성공
테스트 지속 시간:	0:00:00.154
```

이제 `notifyObservers()`를 위한 실패하는 테스트 케이스를 만들어 본다.
이제는 드디어 rhino mock의 기능을 활용하여 실패하는 테스트코드를 작성할 차례이다.	
여기서 확인하고 싶은 것은 `Subject` 에서 `notifyObservers()`를 클릭하면
`Subject`에 등록되어 있는 `Observer` 의 `update()` 메소드가 호출되는지 여부이다.
이를 위해서 `MockRepository`의 `GenerateMock()`을 호출한다.
**observerMock.Expect(o => o.update(new TemperatureInfo()))** 의미가 바로 `update()`메소드가 불려졌는지를 확인할 수 있도록 행위를 추적하라는 의미이다.
그리고 `observerMock.VerifyAllExpectations()`를 통해서 우리가 기대한 **expectation**들이 모두 수행되었는지를 확인한다.

```C#
        [Test]
        public void TestAfterSubjectNotifyAllThenObserverUpdateCalled()
        {
            // given
            Subject subject = WeatherDataSubjectFactory.getDefaultSubject();
            Observer observerMock = MockRepository.GenerateMock<Observer>();
            subject.registerObserver(observerMock);

            observerMock.Expect(o => o.update(new TemperatureInfo())).IgnoreArguments();

            // when
            subject.notifyObservers();

            //then
            observerMock.VerifyAllExpectations();
        }


```		


`WeatherDataSubjectFactory` 에 getDefaultSubject() 메소드를 추가한다.
(Visual Studio의 기능을 활용해서 컴파일 에러를 해결하면 더 쉽다.)

```C#
    class WeatherDataSubjectFactory
    {
        internal static Subject getDefaultSubjectWithHashSet(ISet<Observer> set)
        {
            return new WeatherDataSubject(set);
        }

        internal static Subject getDefaultSubject()
        {
            return new WeatherDataSubject();
        }
    }
```

그리고 `WeaderDataSubject`에 인수를 받지 않는 Default 생성자를 추가한다.
```C#
public WeatherDataSubject() : this(new HashSet<Observer>()) { }
```

이제 (실패하도록 설계된) 테스트를 수행해본다.
역시 테스트가 실패한다.

```
테스트 이름:	TestAfterSubjectNotifyAllThenObserverUpdateCalled
테스트 전체 이름:	ObserverTest2.UnitTest1.TestAfterSubjectNotifyAllThenObserverUpdateCalled
테스트 소스:	C:\Users\idept\Documents\Visual Studio 2015\Projects\ObserverTest2\ObserverTest2\UnitTest1.cs : 줄 48
테스트 결과:	실패
테스트 지속 시간:	0:00:00.185

Result StackTrace:	
위치: Rhino.Mocks.Impl.ReplayMockState.Verify()
   위치: Rhino.Mocks.MockRepository.Verify(Object obj)
   위치: Rhino.Mocks.RhinoMocksExtensions.VerifyAllExpectations(Object mockObject)
   위치: ObserverTest2.UnitTest1.TestAfterSubjectNotifyAllThenObserverUpdateCalled() 파일 C:\Users\idept\Documents\Visual Studio 2015\Projects\ObserverTest2\ObserverTest2\UnitTest1.cs:줄 60
Result 메시지:	Rhino.Mocks.Exceptions.ExpectationViolationException : Observer.update(any); Expected #1, Actual #0.


```

테스트를 성공시키기 위해 `notifyObservers()` 메소드를 아래와 같이 수정한다.

```C#
        public void notifyObservers()
        {
            IEnumerator<Observer> e = set.GetEnumerator();
            while (e.MoveNext())
            {
                e.Current.update(new TemperatureInfo());
            }
        }
```

그리고 테스트를 돌려 본다.
이제 테스트가 모두 성공하는 것을 볼 수 있다.



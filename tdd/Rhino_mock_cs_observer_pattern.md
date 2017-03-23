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

```C#
        [Test]
        public void TestRemoveObserver()
        {
            // given
            ISet<Observer> set = new HashSet<Observer>();
            Subject subject = WeatherDataSubjectFactory.getDefaultSubjectWithHashSet(set);
            Observer observerStub = MockRepository.GenerateStub<Observer>();
            subject.registerObserver(observerStub);
            Assert.True(set.Contains(observerStub));

            // when
            subject.removeObserver(observerStub);

            // then
            Assert.False(set.Contains(observerStub));

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
            throw new NotImplementedException();
        }
    }
```

---
title: "Java - 객체지향 디자인 패턴1 (Singleton, Strategy, State)"
exerpt: "객체지향 디자인 패턴 중 Singleton Pattern, Strategy Pattern, State Pattern 을 자바 기반으로 공부해보았다."
layout: single
author_profile: true
read_time: true
related: true
categories:
- Java
tags:
- Java
- Study Note
---



# Singleton Pattern

프로세스에서 특정 객체가 하나만 존재해야 할 때 쓰는 패턴.

예를 들어 앱에서 다크모드를 한 번 설정했을 때 여러 페이지에 걸쳐서 지속적으로 적용이 되어야 하는데, 각 페이지 소스코드마다 다크모드 세팅 객체가 다 개별적으로 존재할 경우 특정 페이지에 접근할 때마다 매번 다크모드를 켜줘야 한다.

### 방법

1. 클래스 안의 생성자를 private으로 만들어서 다른 클래스에서 new 로 새 객체를 만들지 못하게 한다.
2. 클래스 객체 자기 자신을 static으로 클래스 안에서 생성되도록 메서드를 만든다. → 객체가 얼마나 많이 생성되든 메모리에는 딱 하나만 존재하게 된다.
3. 클래스 메서드가 최초 호출될 때 단 하나의 클래스 객체가 생성되고 이후에는 어떤 페이지에서라도 이미 생성된 객체를 메서드를 통해 접근하게 된다.

**예) 다크모드 설정용 싱글턴 클래스**

```java
public class Settings {

  private Settings () {};    // private 생성자
  private static Settings settings = null;    // 객체 자신을 static으로 생성되도록 함. 초기값은 null

  public static Settings getSettings () {    // 객체 접근용 메서드
    if (settings == null) {    // 객체가 처음 호출되어 초기값이 null인 상태라면,
      settings = new Settings();    // 클래스 내에서 클래스 생성자를 사용하여 객체 생성
    }
    return settings;
  }
	
	// 다크모드를 켰을 때의 private 세팅
  private boolean darkMode = false;
  private int fontSize = 13;
	
	// getSettings를 통해 이미 생성된 객체를 호출하고 나면 아래 메서드들을 사용해서 다크모드 사용이 가능하다.
  // 이때 객체는 메모리에 하나만 존재하므로 모든 페이지에서 공통된 하나의 객체에만 접근할 수 있다.
  public boolean getDarkMode () { return darkMode; }
  public int getFontSize () { return fontSize; }

  public void setDarkMode (boolean _darkMode) { 
    darkMode = _darkMode; }
  public void setFontSize (int _fontSize) { 
    fontSize = _fontSize; }
}
```

### 유의할 점

- 위 방식대로만 사용하면 멀티스레딩 등에서 오류가 발생할 가능성이 있다. (Thread-safe 가 보장되어야 한다.)
- Lazy loading, serialization 등 싱글턴 패턴을 활용하고 안전한 싱글턴을 만들 수 있는 방법들이 있으므로 각 언어별 싱글턴 활용법을 익히자.

# Strategy Pattern

객체가 할 수 있는 기능들을 각각 전략 클래스로 만들어놓고 기능을 실행하는 상황이나 행위에 따라 전략을 바꿔주는 식으로 기능을 확장하는 방법.

예를 들어 검색창이 하나가 있는데 사용자가 검색하려는 주제(사이트, 뉴스, 이미지, 지도 등)를 선택할 때마다 검색창의 기능을 바꿔 특정 주제만 검색하도록 만들 수 있다.

각 주제별 변수를 만들어놓고 if 문을 통해 변수값에 따라 동작이 바뀌도록 할 수도 있지만 소프트웨어가 커질수록 관리하기가 어려워지고 클래스마 역할지정을 뚜렷이 해서 모듈화 된 소프트웨어를 구축하고자 하는 객체지향의 철학과도 어긋난다.

### 방법

1. 기능이 동작하는 방식, 전략에 따라 Strategy 클래스를 생성한다.
2. 공통된 메서드로 된 인터페이스를 만들어 각 전략 클래스에 implement 하고 캡슐화한다.
3. 상황에 따라 다른 전략 클래스를 선택할 수 있도록 strategy setter를 만든다.
4. 선택된 전략 클래스에 따라 공통된 메서드의 전략이 수정된다.

**예) 검색 기능을 위한 전략 패턴**

MyProgram.java

```java
package strategy.after;

public class MyProgram {
  private SearchButton searchButton = new SearchButton(this);

  public void setModeAll () { 
    searchButton.setSearchStrategy(new SearchStratagyAll());
  }
  public void setModeImage () {
    searchButton.setSearchStrategy(new SearchStratagyImage());
  }
  public void setModeNews () {
    searchButton.setSearchStrategy(new SearchStratagyNews());
  }
  public void setModeMap () {
    searchButton.setSearchStrategy(new SearchStratagyMap());
  }

  public void testProgram () {
    searchButton.onClick();   // "SEARCH ALL" 출력
    setModeImage();           // 이미지검색 모드로
    searchButton.onClick();   // "SEARCH IMAGE" 출력
    setModeNews();            // 뉴스검색 모드로
    searchButton.onClick();   // "SEARCH NEWS" 출력
    setModeMap();             // 지도검색 모드로
    searchButton.onClick();   // "SEARCH MAP" 출력
  }
}
```

SearchStrategy.java

```java
package strategy.after;

interface SearchStrategy {
  public void search ();
}

class SearchStratagyAll implements SearchStrategy {
  public void search () {
      System.out.println("SEARCH ALL");
      // 전체검색하는 코드
  }
}

class SearchStratagyImage implements SearchStrategy {
  public void search () {
      System.out.println("SEARCH IMAGE");
      // 이미지검색하는 코드
  }
}

class SearchStratagyNews implements SearchStrategy {
  public void search () {
      System.out.println("SEARCH NEWS");
      // 뉴스검색하는 코드
  }
}

class SearchStratagyMap implements SearchStrategy {
  public void search () {
      System.out.println("SEARCH MAP");
      // 지도검색하는 코드
  }
}
```

SearchButton.java

```java
package strategy.after;

public class SearchButton {

  private MyProgram myProgram;

  public SearchButton (MyProgram _myProgram) {
    myProgram = _myProgram;
  }

  private SearchStrategy searchStrategy = new SearchStratagyAll();
  
  public void setSearchStrategy (SearchStrategy _searchStrategy) {
    searchStrategy = _searchStrategy;
  }

  public void onClick () {
    searchStrategy.search();
  }
}
```

# State Pattern

스테이트 패턴은 전략 패턴과 얼핏 유사해보인다. 하지만 전략 패턴이 특정 작업의 방식, 모드를 바꿔준다면 스테이트 패턴은 스위치처럼 특정 상태마다 다르게 할 일과 바꿀 상태 자체를 모듈화해서 지정해 놓는다.

예를 들어 다크모드를 껐다 켰다 하는 스위치를 프로그래밍 할 때 사용할 수 있다.

### 방법

1. 모드의 상태를 나타내고 스위치를 on/off 할 toggle 메서드를 가진 ModeState 인터페이스를 만든다.
2. ModeState 인터페이스를 implement 하여 스위치를 눌렀을 때 필요한 기능을 수행하고 모드의 상태까지 바꿔줄 toggle 메서드를 갖는 클래스들을 만든다.
3. 상태에 따라 toggle 메서드를 사용할 setter 객체를 만들어준다.

**예) 다크모드 스위치**

MyProgram.java

```java
public class MyProgram {
  public static void main(final String[] args) {
    final ModeSwitch modeSwitch = new ModeSwitch();

    modeSwitch.onSwitch(); // "FROM LIGHT TO DARK" 출력
    modeSwitch.onSwitch(); // "FROM DARK TO LIGHT" 출력
    modeSwitch.onSwitch(); // "FROM LIGHT TO DARK" 출력
    modeSwitch.onSwitch(); // "FROM DARK TO LIGHT" 출력
  }
}
```

ModeState.java

```java
public interface ModeState {
  public void toggle (ModeSwitch modeSwitch);
}

class ModeStateLight implements ModeState {
  public void toggle (ModeSwitch modeSwitch) {
    System.out.println("FROM LIGHT TO DARK");
    // 화면을 어둡게 하는 코드
    // ..
    // ..
    modeSwitch.setState(new ModeStateDark());
  }
}

class ModeStateDark implements ModeState {
  public void toggle (ModeSwitch modeSwitch) {
    System.out.println("FROM DARK TO LIGHT");
    // 화면을 밝게 하는 코드
    // ..
    // ..
    modeSwitch.setState(new ModeStateLight());

  }
}
```

ModeSwitch.java

```java
public class ModeSwitch {
  private ModeState modeState = new ModeStateLight();

  public void setState (ModeState _modeState) {
    modeState = _modeState;
  }

  public void onSwitch () {
    modeState.toggle(this);
  }
}
```

------

해당 포스트는 유튜브 채널 '얄팍한 코딩사전'의 강좌를 기반으로 공부를 진행하고 내용을 기록했습니다.  (참고: https://www.yalco.kr/29_oodp_1/)


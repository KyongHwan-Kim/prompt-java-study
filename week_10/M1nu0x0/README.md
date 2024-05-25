- [1. Thread 클래스와 Runnable 인터페이스](#1-thread-클래스와-runnable-인터페이스)
  - [1.1. Thread 클래스](#11-thread-클래스)
  - [1.2. Runnable 인터페이스](#12-runnable-인터페이스)
- [2. 쓰레드의 상태](#2-쓰레드의-상태)
- [3. 쓰레드의 우선순위](#3-쓰레드의-우선순위)
- [4. Main 쓰레드](#4-main-쓰레드)
  - [4.1. main 쓰레드의 제어](#41-main-쓰레드의-제어)
- [5. 동기화](#5-동기화)
  - [synchronized keyword](#synchronized-keyword)
  - [wait(), notify(), notifyAll()](#wait-notify-notifyall)
- [6. 데드락](#6-데드락)

# 1. [Thread 클래스와 Runnable 인터페이스](https://mangkyu.tistory.com/258)
## 1.1. Thread 클래스
쓰레드 생성을 위해 java에서 미리 구현해둔 클래스이다. 다음과 같은 메소드를 가지고 있다.
- sleep
  - 현재 쓰레드 정지
  - 자원의 해제 없이 제어권을 넘겨주므로 데드락 발생 가능
- interupt
  - 다른 쓰레드를 깨워서 interruptedException을 발생
  - interupt가 발생한 쓰레드는 예외를 catch하여 다른 작업을 할 수 있음
- join
  - 다른 쓰레드의 작업이 끝날 때까지 기다림
  - 쓰레드의 순서를 제어할 때 사용
```java
public class ThreadExample {
    static class MyThread extends Thread {
        MyThread(String threadName) {
            super(threadName);
        }

        @Override
        public void run() {
            System.out.println("Thread: " + Thread.currentThread().getName());
        }
    }

    public static void main(String[] args) {
        MyThread th1 = new MyThread("first thread");
        MyThread th2 = new MyThread("second thread");
        th1.start(); th2.start();
    }
}
```

## 1.2. Runnable 인터페이스
하나의 메소드만을 갖는 함수형 인터페이스이다. 따라서 람다식으로 사용할 수 있다.
```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```
```java
public class RunnableExample {
    static class MyRunnable implements Runnable {
        @Override
        public void run() {
            System.out.println("Thread: " + Thread.currentThread().getName());
        }
    }

    public static void main(String[] args) {
        MyRunnable r = new MyRunnable();

        Thread th1 = new Thread(r, "first thread");
        Thread th2 = new Thread(r, "second thread");
        th1.start(); th2.start();
    }
}
```
이는 쓰레드를 사용한 것과 마찬가지로 동시성을 잘 나타내고 있다.

# 2. 쓰레드의 상태
- new
  - 쓰레드 객체 생성
  - start() 메소드가 호출되지 않은 상태
  - 실행 가능한 상태: 0
  - 따라서 new가 0이 아니면 `IllegalThreadStateException` 예외를 발생시킴
- runnable
  - 실행 상태로 언제든지 갈 수 있는 상태
- waiting
  - 다른 쓰레드가 통지할 때까지 기다리는 상태
- time waiting
  - 주어진 시간 동안 기다리는 상태
- blocked
  - 사용하고자 하는 객체의 락이 풀릴 때까지 기다리는 상태
- terminated
  - 실행을 마친 상태
# 3. 쓰레드의 우선순위
쓰레드 기본 우선순위 값이 큰 순서대로 먼저 실행된다. 쓰레드의 기본 우선 순위는 5로 설정되며, 최소는 1, 최대는 10이다. 이 값들은 Thread.class에 상수값으로 저장 되어있다.
- public static int NORM_PRIORITY
- public static int MIN_PRIORITY
- public static int MAX_PRIORITY

자바에서 쓰레드의 우선 순위를 가져오고 설정하는 메소드가 있다.
- public final int getPriority()
  - 주어진 쓰레드의 우선 순위를 반환
- public final void setPriority(int newPriority)
  - 쓰레드의 우선 순위를 newPriority 값으로 변경.
  - newPriority 값이 1미만, 10초과면 IllegalArgumentException을 발생시킴

# 4. [Main 쓰레드](https://velog.io/@paki1019/Java-%EB%A9%94%EC%9D%B8-%EC%8A%A4%EB%A0%88%EB%93%9CMain-thread)
자바는 다중 쓰레드 프로그래밍을 지원한다. 다중 쓰레드는 2개 이상의 쓰레드로 구성되며, 쓰레드는 별도의 실행 경로를 가진다.
자바 프로그램 시작과 동시에 하나의 쓰레드가 즉시 동작하게 된다. 이를 `Main Thread`라 한다.

메인 쓰레드는 다음과 같은 속성을 가진다.
- 메인 쓰레드에서 자식 쓰레드가 생긴다
- 다양한 종료 작업을 수행하기 때문에 대체로 마지막으로 실행 종료되는 쓰레드여야 함
![mainThread](assets/mainThread.png)

## 4.1. main 쓰레드의 제어
자식 쓰레드는 부모 쓰레드의 우선순위를 그대로 상속한다.
```java
import java.io.*;
import java.util.*;

public class MainThreadExample extends Thread {
    public static void main(String[] args) {
        Thread mainThread = Thread.currentThread();

        Thread childThread = new Thread();
    }
}
```

# 5. 동기화
쓰레드를 동기화하지 않으면 같은 공유 자원에 접근할 때 예상하지 못한 일이 발생할 수 있다. 따라서 동기화를 해주어야 해당 문제가 해결된다.

쓰레드를 동기화하기 위해서는 Critical section(임계 영역)과 Lock(락)의 개념이 필요하다.
- Critical section
  - 둘 이상의 쓰레드가 동시에 접근해서는 안되는 코드의 영역
- Lock
  - 임계 영역을 포함하고 있는 객체에 접근할 수 있는 권한

## synchronized keyword
임계 영역으로 지정할 메소드의 반환 타입 앞에 입력하여 메소드 전체를 Critical section으로 설정할 수 있다. 설정한 메소드가 호출되면, 메소드가 포함된 객체의 Lock을 얻으며, Lock을 반납하기 전까지 다른 쓰레드는 해당 메소드를 실행하지 못한다.
```java
public synchronized void threadExample() throws InterruptedException {
    // do something
}
```
멀티 쓰레딩 시 Critical section은 성능을 좌우하기 때문에 가능하면 메소드 전체에 락을 거는 것보다 블럭으로 임계영역을 최소화해서 효율적인 프로그램을 작성하는 것을 추천.
```java
public void threadExample() throws InterruptedException {
    synchronized (this) {
        // do something
    }
}
```
## wait(), notify(), notifyAll()
wait 메소드는 Object의 인스턴스 메소드이다. 따라서 어떤 객체에서도 호출이 가능하지만, 오직 synchronized 블록 내에서만 호출이 가능하다. 그렇지 않으면 에러를 발생시킨다.

wait() 메소드의 기능은 객체에 대한 lock을 release 하는 것이다.

wait()을 걸게 되면 Lock을 release하고 WAITING, TIMED_WAITING 상태로 바뀌게 된다. 이는 Non-Runnable 영역이다.

wait() 상태로 들어간 쓰레드는 notify() 혹은 notifyAll() 메소드를 호출함으로써 RUNNABLE 상태로 변경할 수 있다.

TIMED_WAITING 상태의 쓰레드의 경우 wait(1000)과 같이 생성자를 이용하면 된다. 이는 시간이 지나면 자동으로 RUNNABLE 상태로 변경된다.

[참조 블로그](https://velog.io/@destiny1616/자바의-wait와-notify-메서드)

# 6. 데드락
데드락의 발생 조건이 있다.
- Mutual Exclusion
  - 한 자원에 대해 여러 쓰레드 동시 접근 불가
- Hold and Wait
  - 자원을 가지고 있는 상태에서 다른 쓰레드가 사용하고 있는 자원 반납을 기다리는 것
- Non Preemptive
  - 다른 쓰레드의 자원을 실행 중간에 강제로 가져올 수 없음
- Circle Wait
  - 각 쓰레드가 순환적으로 다음 쓰레드가 요구하는 자원을 가지고 있는 것

만약 데드락이 발생하면 아무런 실행도 하지 못하고 무한정 대기를 한다.

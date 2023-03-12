# synchronized

**Intro**

자바와 같은 멀티스레드 환경에서는 실행 단위가 동시에 여러개 존재 합니다.

여러 스레드가 한 자원을 공유해서 변경한다면 원하지 않은 결과가 발생할 수 있습니다.

예를들어 재고를 차감시키는 경우를 살펴보겠습니다.

```java
class StockManager{
		private int stock = 2;
    public void decrease() {
        int s = this.stock;
        System.out.println(Thread.currentThread().getName() + "가 "+ s +"개의 재고를 확인 함. 비지니스 로직을 수행함...");
        sleep(500);
        if(s > 0) {
            stock--;
        } else {
            System.out.println("재고가 없습니다.");
        }
        System.out.println(Thread.currentThread().getName() + "이 재고 차감 완료. 남은 재고 : " + stock);
    }
	...
}
```

위의 예제는 재고의 개수를 확인 후 어떤 비지니스 로직을 수행 한다고 가정 후 재고를 차감시키는 로직 입니다.

```java
class RunThread implements Runnable {
    public StockManager sm = new StockManager();
    @Override
    public void run() {
        sm.decrease();
    }
}

public class nonSync {
			RunThread r = new RunThread();
      for(int i = 0; i < 3; i++) {
          int randomTime = (int)(Math.random() * 100);
          try {
              Thread.sleep(randomTime);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
          new Thread(r).start();
      }

			/**
			...
			결과를 출력하기 위해 5초 sleep
			**/

			System.out.println("남은 재고 : " + r.sm.getCounter());
}

```

여러 스레드가 실행시키는 환경을 위해 Runnable 인터페이스를 구현하고 3개의 스레드가 실행하도록 했습니다.

불규칙한 실행 순서를 위해 최대 0.1초의 실행 대기 시간을 가지고 실행시켰습니다.

결과는 어떻게 될까요?

일반적으로 생각한다면 원래 재고 2개, 실행 단위 3개 이므로 한 개의 스레드는 “재고가 없습니다.” 메세지를 받고 재고는 0개가 될것 같습니다.

```java
Thread-0가 2개의 재고를 확인 함. 비지니스 로직을 수행함...
Thread-1가 2개의 재고를 확인 함. 비지니스 로직을 수행함...
Thread-2가 2개의 재고를 확인 함. 비지니스 로직을 수행함...
Thread-0이 재고 차감 완료. 남은 재고 : 1
Thread-1이 재고 차감 완료. 남은 재고 : 0
Thread-2이 재고 차감 완료. 남은 재고 : -1
남은 재고 : -1
```

예상한 결과와 다르게 나왔습니다.

여기서 어떤 문제가 나왔는지 정의 해보겠습니다.

- 기대한 로직은 한 스레드가 재고를 확인하고 재고를 차감시켜야 한다. 입니다.
- 0번 스레드가 재고를 확인했을 때 1, 2번 스레드 모두 같은 양의 재고를 확인 했다.
- 3개의 스레드 모두 0개 이상의 재고를 확인 했으므로 stock를 한 개 씩 차감 시킨다.
- stock를 세 번 차감했으므로 -1이 된다.

혹은 재고를 차감시키는 로직이 다음과 같다면 아래의 문제가 발생할 수 있습니다.

```java
if(s > 0) {
	  s--;
	  this.stock = s;
}
// 남은 재고 : 1
```

```java
Thread-0가 2개의 재고를 확인 함. 비지니스 로직을 수행함...
Thread-1가 2개의 재고를 확인 함. 비지니스 로직을 수행함...
Thread-0이 재고 차감 완료. 남은 재고 : 1
Thread-2가 1개의 재고를 확인 함. 비지니스 로직을 수행함...
Thread-1이 재고 차감 완료. 남은 재고 : 1
Thread-2이 재고 차감 완료. 남은 재고 : 0
남은 재고 : 0
```

- 0, 1번 스레드가 재고를 하나씩 차감 했을 때 남은 재고는 0이 아닌 1이 되었습니다.
- 가장 먼저 실행한 0번 스레드가 재고를 차감한 값을 지역변수 s에 담고 stock를 s로 갱신했습니다.
- 거의 비슷한 시간에 1번 스레드가 재고를 읽은 시점에 stock은 2였고 0번 스레드와 같은 로직을 수행해 stock을 1개 차감한 값을 stock에 갱신했습니다.
- 원하지 않은 값의 덮어쓰기가 발생했습니다.

**왜 이런 문제가 발생하는 것 일까요?**

결론은 공유 데이터에 대해 한 스레드가 작업을 마치기 전 까지 다른 스레드가 방해를 해서는 안된다는 것 입닌다.

기대한 decrease 로직은 stock을 읽는다 → 비지니스 로직 수행 → stock 한 개 차감 입니다.

한 스레드가 stock을 읽고 재고를 차감하기 전 다른 스레드가 같은 양의 stock을 읽고 재고를 차감시켜 이런 문제가 발생한 것 입니다.

**문제 해결 방법?**

한 스레드가 작업중일 때 다른 스레드가 방해받지 못하도록 막으려면 동기화(Synchronized)를 사용해야 합니다.

동기화를 얘기하기 전 용어 두 개를 정의하고 가겠습니다.

- 임계 영역(Critical Section) : 공유 데이터를 사용하는 코드 영역. 이 경우 decrease 메서드
- 잠금(Lock) : 락을 획득한 스레드만 임계 영역에서 코드를 수행할 수 있음

애플리케이션 레벨에서는 synchronized 예약어를 사용해 임계영역을 설정할 수 있습니다.

```java
//synchronized 메서드
public synchronized void decrease() {//임계 영역 시작
...
}//임계 영역 끝

//synchronized 블럭
public void decrease() {
	synchronized() {//임계 영역 시작

	}//임계 영역 끝
}
```

```java
Thread-0가 2개의 재고를 확인 함. 비지니스 로직을 수행함...
Thread-0이 재고 차감 완료. 남은 재고 : 1
Thread-2가 1개의 재고를 확인 함. 비지니스 로직을 수행함...
Thread-2이 재고 차감 완료. 남은 재고 : 0
Thread-1가 0개의 재고를 확인 함. 비지니스 로직을 수행함...
재고가 없습니다.
Thread-1이 재고 차감 완료. 남은 재고 : 0
남은 재고 : 0
```

이제 원하던 결과가 나왔습니다.

1개의 스레드는 재고가 없다는 메세지를 받았고 재고는 1개씩 차감되어 0개가 되었습니다.

**synchronized**

synchronized 블럭에 들어간 스레드는 락을 획득하고 블럭에서 벗어날 때 락을 반납합니다.

락을 획득한 스레드를 제외하고 다른 스레드들은 락을 획득할 때 까지 대기하고 있습니다.

락 획득과 반납은 임계 영역에 들어가는 순간 자동으로 수행되므로 영역의 경계를 잘 설정해 주면 됩니다.

하지만 synchronized를 사용할 때는 다음을 주의해야 합니다.

- 임계 영역에 설정한 공유 데이터는 private로 설정해야 한다.
  - synchronized는 특정 로직 수행에 대해 원자성을 보장하는 것이지 공유 데이터에 대해 원자성을 보장하는 것은 아닙니다. private로 설정해 외부에서 변수 변경에 대해 원천적으로 막아야 합니다.
  - 그만큼 공유 변수를 사용하고자 한다면 철저하게 관리해야 합니다.
- synchronized는 하나의 프로세스 안에서만 보장 됩니다.
  - 일반적으로 한 프로세스 안에서 메모리 공유가 가능하지만 프로세스간 메모리 공유는 가능하지 않습니다.(별도의 방법이 필요)
  - 서버가 2개 이상일 때에는 공유 변수를 사용하더라도 synchronized를 사용하더라도 동시 접근을 막을 수 없습니다.(그렇다 하더라도 다른 프로세스간 이라면 static 변수더라도 다른 변수이기 때문에 프로세스간 공유 변수가 될 수 없습니다.)

# fork & join 프레임워크

- JDK 1.7부터 도입된 fork join 프레임워크는 분할 정복 방식으로 하나의 작업을 작은 단위로 나눠서 여러 스레드가 동시에 처리 하는 것을 쉽게 해줍니다.
- ForkJoinPool

  - ExecuterService의 구현체로 worker 스레드를 관리하고 스레드 풀 상태, 성능 정보를 얻을 수 있는 도구를 제공합니다.
  - invoke()
    - ForkJoinPool을 생성해 invoke 메서드를 호출해 작업을 시작합니다.

- Work-Stealing 알고리즘

  - 기본적으로 worker 스레드는 duque의 헤드에서 작업을 가져옵니다.
  - 한 스레드가 바쁘게 헤드에서 가져온 작업을 수행하고 있을 때, 작업을 하지 않는 스레드는 deque의 끝에서 작업을 가져와 수행합니다.
  - 이렇게 서로 한 작업을 가지고 경쟁해야 할 가능성을 최소화 함으로써 작업 효율을 향상시킬 수 있습니다.

- ForkJoinTask<V>
  - ForkJoinTask는 ForkJoinPool 내에서 사용되는 작업의 기본 유형으로 수행할 작업에 따라 두 클래스 중 하나를 상속받아 구현해야 합니다.
    - Recursive Action : 반환값이 없는 작업을 구현
    - Recursive Task : 반환값이 있는 작업을 구현
  - compute()
    - 사용할 때는 두 클래스가 가지고 있는 compute(반환 값이 void, V)인 메서드를 구현해야 합니다.
    - 작업을 나눌 범위를 정해주고 실제로 수행할 작업을 지정해야 합니다.
    - compute() 는 재귀호출 방식과 동일합니다.
- fork()
  - 재귀호출 방식처럼 작업이 단순해질 때 까지 하위 작업으로 나누고 작업을 스레드 풀의 작업 큐에 넣습니다.
  - 비동기 메서드 입니다.
- join()

  - 작업의 수행이 끝날 때 까지 기다렸다가 수행이 끝나면 결과를 반환합니다.
  - return a.compute() + b.join(); 이라면 compute()가 재귀호출 되고, join()은 호출되지 않은 상태에서 compute()가 작업을 더 이상 나눌 수 없다면 join() 결과를 기다렸다가 더해서 반환합니다.
  - 동기 메서드 입니다.

- Recursive Action

  - 리턴값이 없는 작업입니다.
  - Task가 호출되면 실제 작업인 compute()를 실행합니다.
  - fork()로 작업을 분할해 다른 스레드에서 작업을 수행할 수 있습니다.

  ```java
  public class Upper {

      public static void main(String[] args) {
          ForkJoinPool forkJoinPool = new ForkJoinPool(4);
          CustomRecursiveAction action = new CustomRecursiveAction("bananana");
          forkJoinPool.invoke(action);
          sleep(1000000);

      }

      public static void sleep(long millis) {
          try {
              Thread.sleep(millis);
          } catch (Exception e) {
              e.printStackTrace();
          }

      }
  }

  class CustomRecursiveAction extends RecursiveAction {
      public String workload = "";
      private static final int THREASHOLD = 2;

      private static Logger logger = Logger.getAnonymousLogger();

      public CustomRecursiveAction(String workload) {
          this.workload = workload;
      }

      @Override
      protected void compute() {
          String threadName = Thread.currentThread().getName();
          if (workload.length() > THREASHOLD) {
              System.out.println("The result [" + workload + "] working by " + threadName);
              sleep(1000);
              List<CustomRecursiveAction> subtasks = createSubtasks();

              for(CustomRecursiveAction subtask : subtasks) {
                  subtask.fork();
              }
          } else {
              processing(workload);
          }
      }

      private List<CustomRecursiveAction> createSubtasks() {

          List<CustomRecursiveAction> subtasks = new ArrayList<>();

          String partOne = workload.substring(0, workload.length() / 2);
          String partTwo = workload.substring(workload.length() / 2, workload.length());

          subtasks.add(new CustomRecursiveAction(partOne));
          subtasks.add(new CustomRecursiveAction(partTwo));

          return subtasks;
      }

      private void processing(String work) {
          String result = work.toUpperCase();
          logger.info("This result - (" + result + ") - was processed by " + Thread.currentThread().getName());
      }

  }
  ```

- Recursive Task
  - 리턴값이 있는 작업 입니다.
  - Parent의 join()으로 Child의 작업을 완료할 때 까지 기다렸다가 결과를 반환합니다.
    package test.forkjoin;

```java
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.RecursiveTask;
import java.util.logging.Logger;

public class UpperTask {
    public static void main(String[] args) {
        ForkJoinPool forkJoinPool = new ForkJoinPool(4);
        CustomRecursiveTask banana = new CustomRecursiveTask("bananaapple");
        String invoke = forkJoinPool.invoke(banana);
        System.out.println(invoke);
    }

}

class CustomRecursiveTask extends RecursiveTask<String> {
    private String workload;

    private static final int THREASHOLD = 2;

    private static Logger logger = Logger.getAnonymousLogger();

    public CustomRecursiveTask(String workload) {
        this.workload  = workload;
    }


    @Override
    protected String compute() {
        if( workload.length() > THREASHOLD) {
            List<CustomRecursiveTask> subtasks = createSubtasks();
            String result = "";
            for(CustomRecursiveTask task : subtasks) {
                task.fork();
                result += task.join();
            }
            return result;

        } else {
            String result = processing(workload);
            return result;
        }
    }

    private List<CustomRecursiveTask> createSubtasks() {

        List<CustomRecursiveTask> subtasks = new ArrayList<>();

        String partOne = workload.substring(0, workload.length() / 2);
        String partTwo = workload.substring(workload.length() / 2, workload.length());

        subtasks.add(new CustomRecursiveTask(partOne));
        subtasks.add(new CustomRecursiveTask(partTwo));

        return subtasks;
    }

    private String processing(String work) {
        String result = work.toUpperCase();
        logger.info("This result - (" + result + ") - was processed by " + Thread.currentThread().getName());
        return result;
    }
}
```

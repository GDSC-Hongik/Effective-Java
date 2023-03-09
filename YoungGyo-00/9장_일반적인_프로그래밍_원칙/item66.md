## 66. 네이티브 메서드는 신중히 사용하라

### 네이티브 메서드

> C, C++ 같은 네이티브 프로그래밍 언어로 작성된 메서드
>
- 레지스트리 같은 플랫폼 특화 기능을 사용할 경우
    - 자바가 진화하며, 하위 플랫폼의 기능을 흡수하고 있음
    - 네이티브 메서드를 사용할 일이 줄어들고 있음
    - JAVA 9 에서는 Process API 가 추가되어 OS 프로세스도 접근 가능

    ```java
    public class JavaProcess {
        public static void printProcessInfo(){
            ProcessHandle processHandle = ProcessHandle.current();
            ProcessHandle.Info processInfo = processHandle.info();
        
            System.out.println("processHandle.pid(): " + processHandle.pid());
            System.out.println("processInfo.arguments(): " + processInfo.arguments());
            System.out.println("processInfo.command(): " + processInfo.command());
            System.out.println("processInfo.startInstant(): " + processInfo.startInstant());
            System.out.println("processInfo.user(): " + processInfo.user());    
        }
        public static void main(String[] args) {
            printProcessInfo();
        }
    }
    
    // 출력 예시
    processHandle.pid(): 18008
    processInfo.arguments(): Optional.empty
    processInfo.command(): Optional[C:\Program Files\Java\jdk-11.0.9\bin\java.exe]
    processInfo.startInstant(): Optional[2021-03-16T05:43:42.416Z]
    processInfo.user(): Optional[DESKTOP-3MKBM0C\jiae]
    ```

- 네이티브 코드로 작성된 기존 라이브러리를 사용할 경우
    - `레거시 데이터`를 사용하는 레거시 라이브러리
        - 기존에 운영하던, 구형 시스템에서 생성 저장된 데이터
    - 자바로 된 라이브러리가 없는 경우 네이티브 라이브러리 사용
- 성능 개선을 목적으로 하는 경우
    - JVM은 엄청난 속도로 발전
    - 다른 플랫폼에 견줄만한 성능

### Process API

> “Process is a program in execution” : 실행 중인  프로그램
>
- **Process 실행 과정**

    1. 코드와 데이터를 디스크로부터 메모리로 불러온다(**Fetch**)

    2. 스택(Stack), 힙(Heap) 메모리를 할당(**Allocate**)한다

    3. I/O Setup 등의 사전 작업을 수행한다

    4. 시작점(main)으로 진입한다

    5. 명령어를 해석(**Decode**)하여 수행(**Execute**)하는 작업을 반복한다

- **Process 상태**
    - `Ready` : 프로세스가 실행을 준비중이 단계
    - `Running` : 프로세서 위에서 실행중인 단계
    - `Blocked` : 프로세스가 다른 프로세스나 여러 이유로 멈춰 있는 단계

- **OS가 Application 에게 제공하는 interface**
    - 프로세스를 생성, 종료하거나 정지, 재개 또는 `프로세스의 상태를 알려주는 기능 제공`

### 네이티브 메서드 단점

- 네이티브 언어가 안전하지 않으므로, 네이티브 메서드를 사용하는 애플리케이션도 메모리 훼손 오류 문제
- 플랫폼 종속성이 있어, 이식성이 좋지 못함
- 디버깅 어려움
- 주의하지 않으면, 속도 저하
- 코드의 경계를 넘나들 때마다 비용 추가
- 자바 코드와 네이티브 코드 사이에 접착 코드(glue code) 작성해야 함 → 가독성 떨어짐
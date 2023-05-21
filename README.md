# Garbage-Collection



👉**가비지 컬렉션이란(GC)?**

 * 자바의 메모리 관리 방법 중의 하나로 JVM(Java virtual Machine)의 Heap영역에서 동적으로 할당 했던 메모리 영역 중 필요없게 된 메모리 영역을 주기적으로 삭제하는 프로세스

 * C나C++는 프로그래머가 수동으로 메모리 할당과 해제를 일일이 해야하는 반면,JAVA는 JVM에 재되어 있는 가비지 컬렉터가 메모리 관리를 대행 해준다.

👉**왜 사용할까?**

 * 개발자들이 메모리를 할당 하고 해제해야 하는데 까먹어서 메모리 누수가 생기거나,해제한 메모리를 실수로 다시 사용하거나,해제 했던 메모리를 또다시 해제하는 등 여러 실수가 일어남.

 * 메모리 관련 버그는 한참 떨어진 곳에서 발생되고,재현이 불가능한 경우도 많아서,개발자에게는 지옥의 디버깅을 선사
 

👉**GC의 장점(버그를 줄이거나 완전히 막을 수 있음)**
 * 이미 해제된 메모리에 접근하는 버그
 * 이미 해제된 메모리를 또다시 해제하는 버그
 * 더이상 필요하지 않은 메모리가 해제되지 않고 남아 있는 버그

👉**GC의 단점**
 * 개발자가 메모리가 언제 해제되었는지 정확하게 알 수가 없다.
 * GC가 동작하는 동안 다른 동작이 멈추기에 오버헤드 발생! 
 **(전문적인 용어로 stop-the-world)**

##### 오버헤드:프로그램의 실행흐름에서 나타나는 현상으로 실행흐름 도중에 동떨어진 코드를 실행할 때,추가적으로 시간,메모리,자원이 사용되는 현상

 
 
👉**STW (Stop The World)**

 * GC를 수행하기 위해 JVM이 프로그램 실행을 멈추는 현상을 의미.
 * GC가 작동하는 동안 GC 관련 Thread를 제외한 모든 Thread는 멈추게 되어 서비스 이용에 차질이 생길 수 있다. 따라서 이 시간을 최소화 시키는 것이 쟁점이다
 ![image](https://github.com/BombJuun/Garbage-Collection/assets/133941695/387ba701-3f86-4b31-a0dc-497466787c4c)






👉**GC의 대상**

 * 객체들은 Heap에 생성되며,Method area이나 Stack Area에서 Heap Area에 생성된 객체의 주소를 참조하는 형식으로 구성된다.
 * 특정이벤트들로 인하여 Heap Area객체의 메모리 주소를 참조 변수가 삭제되는 현상이 발생하면
Heap Area에 생성되었던 객체가 어떠한 참조 변수에게 참조되고 있지 않는 상태가 된다.
이러한 상태의 객체들은 GC의 대상이 된다.


	

👉**GC청소 방식**

**① Mark And Sweep알고리즘**

 * GC의 동작하는 원리이며,루드에서부터 해당 객체에 접근 가능한지에 따라 메모리 해제의 기준을 삼는다.

 * **Mark And Sweep**은 3가지의 과정으로 나뉜다.

  * **Mark**:그래프 순회를 통해 연결된 객체들을 찾아내어 각각 어떤 객체를 참조하고 있는지 찾아 2가지 정보 중 하나로 표시된다.
  * **Sweep**:참조하고 있지 않은 객체들을 Heap에서 제거
  * **Compact:Sweep** 과정을 거친 후 아직 까지 Heap영역에 있는 객체들을 Heap 시작주소로 모아 메모리가 할당된 부분과 그렇지 않은 부분으로 압축

**② 삼색 표시 기법**
 * Mark And Sweep 기법의 단점을 보완하였다.
 * Mark And Sweep과 기본적으로 같은 기법이지만,Mark 단계에서 2가지가 아닌 3가지(흰,회,검)정보 중 하나로 메모리를 표시한다.

	**1**.각각의 객체를 흰색,회색,검은색으로 분류한다.

	**흰색**:더이상 접근 불가능한 객체

	**회색**:접근 가능한 객체이지만,이 객체에서 가리키는 객체들은 아직 검사되지 않았음을 의미

	 **검은색**:영역에서 가리키는 개체들이 흰색 객체를 가리키지 않음을 의미

	**2**. 회색으로 표시된 객체 가운데 하나를 선택하여 회색으로 표시하고,이 객체가 가리키는 모든 객체를 회색으로 표시

	**3**.회색 객체가 하나도 남지 않을 때까지 위 과정 반복

	**4**.남은 흰색 객체는 접근 불가능한 객체이므로, 해제

삼색 표시 기법은 Mark And Sweep과 달리 프로그램 실행 중에도 병행하여 수행할 수 있다.

### 동작과정

👉**Heap이란?**

 * 자바 프로그램이 실행되면서 동적으로 생성된 객체가 저장되는 공간
 * Heap영역은 처음 설계될 때 2가지 전제로 설계되었다.
 	* 1.대부분의 객체는 금방 접근 불가능한 상태가 된다.
 	* 2.오래된 객체에서 새로운 객체로의 참조는 아주 적게 존재한다.

❗
**객체는 메모리에 오랫동안 남아있는 경우는 드물다.**

**이러한 특성을 이용해 객체의 생존기간에 따라 물리적인 Heap영역을 나누게 되었고,
 young과 Old 총 2가지 영역으로 설계하였다.**

**young**
 * 새로운객체들이  할당되는 영역
 * 대부분의 객체들은 금방 접근 불가능한 상태가 되기에 Young영역에서 생성되었다 사라진다.

**Minor GC**:Young영역에 대한 가비지 컬렉션

**Old**
 * young generation에서 오랫동안 살아남은 객체들이 존재하는 영역
 * young영역보다 크게 할당되며,영역의 크기가 큰 만큼 가비지는 적게 발생

**Major GC**:Old 영역에 대한 가비지 컬렉션

다시 heap 영어에서 더욱 효율적인 GC를 위해 Young영역을 3가지 영역으로 나눈다

**Eden**
새로 생성된 객체가 위치
정기적인 쓰레기 수집 후 살아남은 객체들은 Survivor영역으로 보냄

**Survivor0/Survivor1**
 * 최소 1번의 GC이상 살아남은 객체가 존재하는 영역
 * Survivor 영역에는 특별한 규칙이 있는데, Survivor 0 또는 Survivor 1 둘 중 하나에는 꼭 비어 있어야 하는 것이다.

👉**Minor GC 과정**

Young Generation 영역은 짧게 살아남는 메모리들이 존재하는 공간이다.
모든 객체는 처음에는 Young Generation에 생성되게 된다.

Young Generation의 공간은 Old Generation에 비해 상대적으로 작기 때문에 메모리 상의 객체를 찾아 제거하는데 적은 시간이 걸린다. (작은 공간에서 데이터를 찾으니까)

이 때문에 Young Generation 영역에서,발생되는 GC를 Minor GC라 불린다.

**①** 처음 생성된 객체는 Eden 영역에 위치

**②** 객체가 계속 생성되어 Eden 영역이 꽉차게 되고 Minor GC가 실행


**③** Mark 동작을 통해 접근 가능한 객체 탐색


**④** Eden 영역에서 살아남은 객체는 1개의 Survivor 영역으로 이동

![image](https://github.com/BombJuun/Garbage-Collection/assets/133941695/51eaabe0-118e-4e10-ba04-e2c37a3d4531)

**⑤** Eden 영역에서 사용되지 않는 객체(unreachable)의 메모리를 해제(sweep) 및 살아남은 모든 객체들은 age 값이 1씩 증가

❓
**age란?**

**Survivor 영역에서 객체의 객체가 살아남은 횟수를 의미하는 값이며, Object Header에 기록된다.**


**⑥** 또다시 Eden 영역에 신규 객체들로 가득 차게 되면 다시한번 minor GC 발생하고 mark 한다.


**⑦** 비어있는 Survival 1으로 이동하고 sweep

![image](https://github.com/BombJuun/Garbage-Collection/assets/133941695/164f7eed-daf6-4a27-bcb2-d42804cba82b)



**⑧** 다시 살아남은 모든 객체들은 age가 1씩 증가


**⑨** 이러한 과정을 반복


👉**major GC 과정**

**①**객체의 age가 임계값에 도달하게 되면 **Old**로 이동된다. 이를 **promotion**이라 부름

![image](https://github.com/BombJuun/Garbage-Collection/assets/133941695/039120c5-32e3-4756-b25b-23b27e53aa80)

**②**위의 과정이 반복되어 Old 영역의 공간(메모리)가 부족하게 되면 **Major GC**가 발생되게 된다.


**Major GC**는 Old영역은 데이터가 가득 차면 GC를 실행하는 단순한 방식이다.

Old 영역에 할당된 메모리가 허용치를 넘게 되면, Old 영역에 있는 모든 객체들을 검사하여 참조되지 않는 객체들을 한꺼번에 삭제하는 Major GC가 실행되게 된다.

하지만 Old Generation은 Young Generation에 비해 상대적으로 큰 공간을 가지고 있어, 이 공간에서 메모리 상의 객체 제거에 많은 시간이 걸리게 된다.

young 영역에서는 Old영역보다 크기가 작기 때문에 GC가 보통 1초이내에 끝나지만
Old영역의 Major GC는 일반적으로 Minor GC보다 시간이 오래걸리며 10배이상의 시간을 사용한다.

여기서 Stop-The-World 문제가 발생하게 된다.

Major GC가 일어나면 Thread가 멈추고 Mark and Sweep 작업을 해야 해서 CPU에 부하를 주기 때문에 멈추거나 버벅이는 현상이 일어나기 때문이다.

### GC가 제대로 동작하려면 다음 규칙을 준수해야 한다.

 * Reachability(도달 가능성): GC는 도달 가능한 객체만을 수거. 즉, 어떤 객체도 다른 객체를 참조하고 있거나, 직접 또는 간접적으로 어떤 경로를 통해 참조할 수 있는 경우 도달 가능한 것으로 간주된다.

 * Strong Reference(강한 참조): 기본적인 참조 유형이며, 참조하고 있는 동안 GC의 수거 대상이 되지 않는다. 객체에 최소한 하나 이상의 강한 참조가 있는 경우 GC의 대상이 되지 않는다.

 * Weak Reference(약한 참조): 약한 참조는 GC의 수거 대상이 될 수 있다. GC가 발생할 때 약한 참조로만 참조되고 있는 객체는 수거된다.

 * Soft Reference(소프트 참조): 약한 참조와 유사하지만, JVM이 메모리 부족 상황에서만 수거된다. 메모리 부족이 없을 경우 GC의 대상이 되지 않는다.

 * Phantom Reference(유령 참조): 객체 수거 직전에 알림을 받을 수 있는 참조. 유령 참조는 get() 메서드를 호출하여 실제 객체를 가져올 수 없다. 단지 참조되는 객체가 GC에 의해 수거되었음을 알린다.


### 메모리 누수(Memory Leak)

메모리 누수는 GC에 의해 수거되지 않는 객체들이 계속해서 쌓이는 상황을 말한다.

```java
import java.util.ArrayList;
import java.util.List;

public class MemoryLeakExample {
    private static List<Object> list = new ArrayList<>();

    public static void main(String[] args) {
        while (true) {
            Object obj = new Object();
            list.add(obj);
            // obj = null;  //  메모리 누수 발생:obj는 참조하고 있어 GC의 대상이 되지 않음
        }
    }
}

```

 GC는 이들을 도달할 수 없는 객체로 인식하지 않으므로 수거되지 않는다. 이렇게 계속해서 객체를 생성하고 유지하는 경우, 메모리용량이 커지게 된다.
 
 **메모리 누수 방지 방법**
 
 **①** 객체의 참조를 명시적으로 해제: 객체를 사용한 후에는 해당 객체의 참조를 명시적으로 null로 설정하여 GC의 수거 대상으로 만든다.
 
 ```java
 public class MemoryLeakExample {
    public static void main(String[] args) {
        Object obj = new Object();
        // obj 사용
        obj = null;  // 사용이 끝난 후에 참조 해제하여 GC의 대상으로 만듦
    }
}

 
 ```
 
 **②**자원의 적절한 해제: 메모리 외의 자원(파일, 네트워크 연결 등)을 사용한 경우에는 자원을 명시적으로 해제해야 한다. finally 블록을 사용하여 자원 해제를 보장할 수 있다.
 
 ```java
 import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;

public class MemoryLeakExample {
    public static void main(String[] args) {
        FileInputStream fis = null;
        try {
            File file = new File("data.txt");
            fis = new FileInputStream(file);
            // 파일 읽기 작업 수행
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (fis != null) {
                try {
                    fis.close();  // 파일 입력 스트림 해제
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

 ```
 
 위의 예제에서는 FileInputStream을 사용하여 파일을 읽는 작업을 수행한다. finally 블록에서 close() 메서드를 호출하여 파일 입력 스트림을 해제하여 자원을 제대로 반환한다.


[출처1:Inpa Dev](https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EA%B0%80%EB%B9%84%EC%A7%80-%EC%BB%AC%EB%A0%89%EC%85%98GC-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%F0%9F%92%AF-%EC%B4%9D%EC%A0%95%EB%A6%AC#heap_%EB%A9%94%EB%AA%A8%EB%A6%AC%EC%9D%98_%EA%B5%AC%EC%A1%B0)

[출처2:Chat GPT](https://chat.openai.com/)

[출처3:위키백과](https://ko.wikipedia.org/wiki/%EC%93%B0%EB%A0%88%EA%B8%B0_%EC%88%98%EC%A7%91_(%EC%BB%B4%ED%93%A8%ED%84%B0_%EA%B3%BC%ED%95%99))














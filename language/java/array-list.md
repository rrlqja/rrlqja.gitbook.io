# Array List

### Array List

자바 컬렉션 프레임워크의 List 구현체

* 연속적인 데이터 리스트 (요소 **중간에 공백이 없음**)
* Array List 내부에서 Object배열을 사용하고있음
* 내부 공간을 가변적으로 늘리거나 줄인다(가변 사이즈)
* 공간을 바꿀땐 기존 배열을 copy 하여 새로운 배열을 생성하는 과정이기때문에 지연이 발생함
* 데이터를 중간에 삽입, 삭제시 기존 요소들의 위치를 재배열하기 때문에 속도가 느림
* 조회성능은 좋음

#### 배열의 장단점

* 배열은 크기를 변경할수없다(정적 할당).
* 크기가 정해져있어서 메모리 관리가 편함
* 요소들이 메모리에 연속적으로 할당되어있어서 접근속도가 빠름
* index 위치의 요소를 제거해도 index위치는 빈공간으로 그대로 남아있음
* 크기를 변경할수없기때문에 크기가 작으면 공간이 부족해지거나 크기가 크면 메모리 낭비가 심함

#### ArrayList의 장단점

* 크기가 가변적이다(동적 할당).&#x20;
* 데이터 사이의 빈 공간을 허용하지 않음
* 객체로만 데이터를 다루기때문에 차지하는 메모리가 클 수 있음

ArrayList의 capacity는 리스트의 총 크기, size는 현재 들어있는 요소들의 총 개수\
ArrayList에 데이터 삽입시 삽입 위치(index)가 capacity보다 클경우 예외 발생, 삽입 위치(index)와 기존의 요소들위치에 공백이 있을경우에도 예외 발생한다.

#### List 생성

```java
List<String> list1 = new ArrayList<>();

List<String> list2 = Arrays.asList();

List<String> list3 = List.of();
```

`new ArrayList<>();` 처럼 생성자로 list를 생성하는 것 외에 `Arrays.asList()`, `List.of();`은 모두 **불변 리스트**이다. 즉 요소를 추가/삭제 할 수 없다.

Arrays.asList()는 set()매서드에 한해서는 변경이 가능하고, List.of()는 완전 불변\
Arrays.asList(), List.of()의 반환 리스트는 java.util.ArrayList를 상속받는 **다른 List**이다.

Arrays.asList(), List.of()는 여러가지 이유때문에 불변의 list를 반환하는것.

* 스레드의 안전성
* 코드 간소화
* 향상된 성능

Arrays.asList(), List.of()로 반환된 list로 직접 collection의 새로운 객체로 만들수있다.

원본 배열을 복사하여 List.of()를 생성하면 원본 배열의 값이 바뀌어도 List.of()의 list는 바뀌지않음\
원본 배열을 Arrays.asList()로 복사하여 생성하면 원본 배열이 바뀌면 Arrays.asList()의 list도 값이 바뀌고 Arrays.asList()의 리스트를 바꿔도 원본 배열의 값이 바뀜\
즉 List.of()는 원본 배열을 깊은 복사를 하고, Arrays.asList()는 얕은 복사를 한다.

Arrays.asList()는 null값을 가질수 있고, List.of()는 null값을 가질수 없다.

변경 불가능한 컬렉션은 jvm내부에서 메모리를 더 적게 사용한다. 그러니 반 불변인 Arrays.asList()보다 List.of()를 사용하는게 권장됨.

#### Arrays.asList() vs List.of()

Arrays.asList()는 변경이 가능하기때문에 thread-safe하지 않음. List.of()는 스레드 안전\
Arrays.asList()는 null 허용, List.of()는 null 허용하지 않음\
List.of()가 메모리는 덜 사용함\
두개 모두 변경할수 없기때문에 별도로 collections를 생성해서 요소를 복사해서 사용해야함



### ArrayList vs LinkedList

|           |                             ArrayList                             |                 LinkedList                |
| :-------: | :---------------------------------------------------------------: | :---------------------------------------: |
|     구성    |                                 배열                                |                     노드                    |
|   접근 시간   |                            모든 데이터 상수 접근                           |               위치에 따라 이동시간 발생              |
| 삽입, 삭제 시간 | <p>상수 시간 or 데이터 이동이 필요시 추가 시간 발생<br>공간 부족시 새로운 배열을 복사하는 시간 발생</p> | 상수 시간 or 삽입, 삭제 위치에 따라 해당 위치까지 이동하는 시간 발생 |
|     검색    |                        최악의 경우 최대 요소갯수만큼 발생                        |            최악의 경우 최대 요소갯수만큼 발생            |
|     캐시    |                              캐시 이점 활용                             |                                           |

#### ArrayList 단점

ArrayList의 배열 공간이 부족하여 새로운 배열을 복사할때, 요소 중간에 새로운 요소를 삽입할때 기존 배열을 복사하여 요소를 뒤로 한칸씩 일일히 이동해야하므로 시간이 많이 소요됨

#### LinkedList 장단점

요소들을 서로 연결한 구조이기때문에 공간에 제약이없음.\
삽입, 삭제 속도 빠름\
요소를 가져오는 과정은 ArrayList보다 현저히 느림.(ArrayList는 연속적인 메모리주소에 위치해있고 LinkedList는 서로다른 위치에있는 요소들을 순차적으로 접근해야하기때문에)

#### 속도 비교

|                 |  ArrayList | LinkedList(doubly) |
| :-------------: | :--------: | :----------------: |
|    get(index)   |    O(1)    |        O(n)        |
| add(index, obj) | O(1), O(n) |     O(1), O(n)     |
|  remove(index)  | O(1), O(n) |        O(n)        |
|     get(obj)    | O(n), O(1) |        O(n)        |
|     add(obj)    |    O(1)    |        O(n)        |
|   remove(obj)   |    O(n)    |        O(n)        |

ArrayList의 첫번째 삽입이 O(n)인 이유는 첫번째로 삽입시 뒤에있는 모든 요소를 뒤로 이동시켜야 하기때문이다.\
ArrayList의 마지막 삽입이 O(1), O(n)인 이유는 배열 공간 부족시 배열 복사가 일어나기때문이다.



#### 결론

삽입, 삭제가 빈번할땐 LinkedList를 사용, 검색이 빈번할땐 ArrayList를 사용하라고 되어있지만 큰 차이없음. 대부분 ArrayList를&#x20;



***

#### 참고

[https://www.cs.cmu.edu/\~mrmiller/15-121/Slides/09-BigO-ArrayList.pdf](https://www.cs.cmu.edu/\~mrmiller/15-121/Slides/09-BigO-ArrayList.pdf)\
[https://www.programcreek.com/2013/03/arraylist-vs-linkedlist-vs-vector/](https://www.programcreek.com/2013/03/arraylist-vs-linkedlist-vs-vector/)\
[https://inpa.tistory.com/entry/JCF-%F0%9F%A7%B1-ArrayList-vs-LinkedList-%ED%8A%B9%EC%A7%95-%EC%84%B1%EB%8A%A5-%EB%B9%84%EA%B5%90](https://inpa.tistory.com/entry/JCF-%F0%9F%A7%B1-ArrayList-vs-LinkedList-%ED%8A%B9%EC%A7%95-%EC%84%B1%EB%8A%A5-%EB%B9%84%EA%B5%90)


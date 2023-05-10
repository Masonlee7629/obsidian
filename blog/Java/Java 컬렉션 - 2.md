태그: #Java 
연결문서: [Java 심화 - 1](Java%20심화%20-%201.md)

# Java 컬렉션

## Java의 컬렉션 프레임워크

### 컬렉션 프레임워크

-   여러 데이터들을 그룹으로 묶어놓은 것을 컬렉션이라 하며 컬렉션 크레임워크는 컬렉션을 다루는데 편리한 메서드를 미리 정의한 것
-   특정 자료 구조에 데이터를 추가, 삭제, 수정, 검색 등의 동작을 수행하는 편리한 메서드들을 제공
-   주요 인터페이스로 List, Set, Map을 제공

-   **List**
    -   데이터의 순서가 유지되며 중복 저장이 가능한 컬렉션을 구현할 때 사용
    -   ArrayList, Vector, Stack, LinkedList등이 List 인터페이스를 구현

-   **Set**
    -   데이터의 순서가 유지되지 않으며, 중복 저장이 불가능한 컬렉션을 구현할 때 사용
    -   HashSet, TreeSet 등이 Set 인터페이스를 구현

-   **Map**
    -   키(key)와 값(value)의 쌍으로 데이터를 저장하는 컬렉션을 구현할 때 사용
    -   데이터의 순서가 유지되지 않으며 키는 중복 저장이 불가능하고 값은 중복 저장이 가능
    -   HashMap, HashTable, TreeMap, Properties 등

---

### Collection 인터페이스

-   List와 Set의 공통점이 추출되어 추상화된 것

|   기능   |   리턴타입   |   메서드   |   설명   |
| :-: | :-: | :-: | :-: |
|   객체 추가   |   boolean   |   add(Object o) / addAll(Collection c)   |   주어진 객체 및 컬렉션의 객체들을 컬렉션에 추가   |
|   객체 검색   |   boolean   |   contains(Object o) / containsAll(Collection c)   |   주어진 객체 및 컬렉션의 저장 여부를 리턴   |
|   |   iterator   |   iterator()   |   컬렉션의 iterator를 리턴   |
|   |   boolean   |   equals(Object o)   |   컬렉션이 동일한 지 여부를 확인   |
|   |   boolean   |   isEmpty()   |   컬렉션이 비어있는 지 여부를 확인   |
|   |   int   |   size()   |   저장되어 있는 전체 객체 수를 리턴   |
|   객체 삭제   |   void   |   clear()   |   컬렉션에 저장된 모든 객체를 삭제   |
|   |   boolean   |   remove(Object o) / removeAll(Collection c)   |   주어진 객체 및 컬렉션을 삭제하고 성공 여부를 리턴   |
|   |   boolean   |   retainAll(Collection c)   |   주어진 컬렉션을 제외한 모든 객체 삭제하고 변화 여부를 리턴   |
|   객체 변화   |   Object\[\]   |   toArray()   |   컬렉션에 저장된 객체를 객체배열로 반환   |
|   |   Object\[\]   |   toArray(Object o)   |   주어진 배열에 컬렉션의 객체를 저장해서 반환   |

---

### List 인터페이스

-   List 인터페이스는 배열처럼 객체를 일렬로 늘어놓은 구조
-   객체를 인덱스로 관리하므로 객체를 저장하면 자동으로 인덱스가 부여되며 인덱스로 객체를 검색, 추가, 삭제 등 가능
-   List 인터페이스를 구현한 클래스로 ArrayList, Vector, LinkedList, Stack 등이 존재

|   기능   |   리턴타입   |   메서드   |   설명   |
| :-: | :-: | :-: | :-: |
|   객체 추가   |   void   |   add(int index, Object o)   |   주어진 인덱스에 객체를 추가   |
|   |   boolean   |   addAll(int index, Collection c)   |   주어진 인덱스에 컬렉션을 추가   |
|   |   Object   |   set(int index, Object o)   |   주어진 위치에 객체를 저장   |
|   객체 검색   |   Object   |   get(int index)   |   주어진 인덱스에 저장된 객체를 반환   |
|   |   int   |   indexOf(Object o) / lastIndexOf(Object o)   |   순방향/역방향으로 탐색하여 주어진 객체의 위치를 반환   |
|   |   ListIterator   |   listIterator()   listIterator(int index)   |   List의 객체를 탐색할 수 있는 ListIterator 반환   주어진 index부터 탐색할 수 있는 ListIterator 반환   |
|   |   List   |   subList(int fromIndex, int toIndex)   |   fromIndex부터 toIndex에 있는 객체를 반환   |
|   객체 삭제   |   Object   |   remove(int index)   |   주어진 인덱스에 저장된 객체를 삭제하고 삭제한 객체를 반환   |
|   |   boolean   |   remove(Object o)   |   주어진 객체를 삭제   |
|   객체 정렬   |   void   |   sort(Comparator c)   |   주어진 비교자(comparator)로 List를 정렬   |

-   **ArrayList**
    -   List 인터페이스를 구현한 클래스로 컬렉션 프레임워크에서 가장 많이 사용
    -   기능적으로 Vector와 동일하지만 기존의 Vector를 개선한 것으로 주로 ArrayList를 사용
    -   객체가 인덱스로 관리된다는 점에서 배열과 유사하며 ArrayList는 저장 용량을 초과하여 객체를 추가하면 자동으로 저장 용량이 늘어남
    -   List 계열 자료구조의 특성을 이어받아 데이터가 연속적으로 존재하며 데이터의 순서를 유지

```
List<타입 매개변수> 객체명 = new ArrayList<타입 매개변수>(초기 저장 용량);

List<String> box1 = new ArrayList<String>();
// String 타입의 객체를 저장하는 ArrayList 생성
// 초기 용량이 인자로 전달되지 않으면 기본적으로 10으로 지정

List<String> box2 = new ArrayList<String>(20);
// 초기 용량을 20으로 지정
```

-   **LinkedList**
    -   데이터를 효율적으로 추가, 삭제 변경하기 위해 사용
    -   데이터가 불연속적으로 존재하며 이 데이터는 서로 연결되어 있음
    -   LinkedList의 요소(node)들은 자신과 연결된 이전 요소 및 다음 요소의 주소값과 데이터로 구성
    -   배열처럼 데이터를 이동하기 위해 복사할 필요가 없으므로 처리 속도가 빠름

-   **ArrayList와 LinkedList의 차이점**
    -   ArrayList
        -   중간에 위치한 객체를 추가 및 삭제할 때는 데이터 이동이 많이 일어나므로 속도가 저하
            -   검색(읽기) 측면에서 인덱스를 통해 바로 데이터에 접근할 수 있으므로 유리함
            -   데이터를 순차적으로 추가하거나 삭제하는 경우 유리함
    -   LinkedList
        -   데이터를 중간에 추가하거나 삭제하는 경우 ArrayList보다 속도가 빠름
        -   데이터 검색에 있어서는 ArrayList보다 상대적으로 속도가 느림

-   **Iterator**
    -   컬렉션에 저장된 요소들을 순차적으로 읽어오는 역할
    -   Collection 인터페이스에 정의된 iterator()를 호출하면 Iterator 타입의 인스턴스를 반환
    -   Collection 인터페이스를 상속받는 List와 Set 인터페이스를 구현한 클래스들은 iterator() 메서드 사용 가능
    -   iterator()로 생성된 인스턴스는 hasNext(), next(), remove() 메서드 사용 가능

|   메서드   |   설명   |
| :-: | :-: |
|   hasNext()   |   읽어올 객체가 남아있으면 true를 리턴, 없으면 false를 리턴   |
|   next()   |   컬렉션에서 하나의 객체를 읽음   이 때, next()를 호출하기 전에 hasNext()를 통해 읽어올 다음 요소가 있는지 먼저 확인 필요   |
|   remove()   |   next()를 통해 읽어온 객체를 삭제   next()를 호출한 다음 remove()를 호출해야 함   |

---

### Set 인터페이스

-   요소의 중복을 허용하지 않고, 저장 순서를 유지하지 않는 컬렉션
-   대표적인 Set을 구현한 클래스로는 HashSet, TreeSet이 있음

|   기능   |   리턴타입   |   메서드   |   설명   |
| :-: | :-: | :-: | :-: |
|   객체 추가   |   boolean   |   add(Object o)   |   주어진 객체를 추가하고 성공하면 true, 중복 객체면 false를 반환   |
|   객체 검색   |   boolean   |   contains(Object o)   |   주어진 객체가 Set에 존재하는지 확인   |
|   |   boolean   |   isEmpty()   |   Set이 비어있는지 확인   |
|   |   Iterator   |   iterator()   |   저장된 객체를 하나씩 읽어오는 반복자를 리턴   |
|   |   int   |   size()   |   저장되어 있는 전체 객체의 수를 리턴   |
|   객체 삭제   |   void   |   clear()   |   Set에 저장되어 있는 모든 객체를 삭제   |
|   |   boolean   |   remove(Object o)   |   주어진 객체를 삭제   |

-   **HashSet**
    -   Set 인터페이스를 구현한 대표적인 컬렉션 클래스
    -   Set 인터페이스의 특성을 물려받아 중복된 값을 허용하지 않으며 저장 순서를 유지하지 않음
    -   HashSet에 값을 추가할 때 중복된 값을 판단하는 과정
        1.  add(Object o)를 통해 객체를 저장하고자 한다
        2.  저장하고자 하는 객체의 해시코드를 hashCode() 메서드로 얻어낸다
        3.  Set에 저장하고 있는 모든 객체들의 해시코드를 hashCode()로 얻어낸다
        4.  저장하고자 하는 객체와 Set에 저장되어 있는 객체들의 해시코드를 비교하며 같은 해시코드가 있는지 검사한다
            -   만약 같은 해시코드를 가진 객체가 있다면 5번으로 넘어간다
                -   같은 해시코드를 가진 객체가 없다면 Set에 객체가 추가되며 add(Object o) 메서드가 true를 리턴한다
        5.  equals() 메서드로 객체를 비교한다
            -   true가 리턴되면 중복 객체로 간주되어 Set에 추가되지 않고 add(Object o)가 false를 리턴한다
            -   false가 리턴되면 Set에 객체가 추가되며 add(Object o) 메서드가 true를 리턴한다

-   **TreeSet**
    -   이진 탐색 트리 형태로 데이터를 저장하며 데이터의 중복 저장을 허용하지 않고 저장 순서를 유지하지 않음
    -   하나의 부모 노드가 최대 두 개의 자식 노드와 연결되는 이진 트리(Binary Tree)의 일종으로 정렬과 검색에 특화된 자료 구조
    -   모든 왼쪽 자식의 값이 루트나 부모보다 작고, 모든 오른쪽 자식의 값이 루트나 부모보다 큰 값을 가짐

---

### Map 인터페이스 (Map<K, V>)

-   Map 인터페이스는 키(key)와 값(value)으로 구성된 객체를 저장하는 구조
-   해당 객체를 Entry객체라고 하며 키와 값을 각각 Key객체와 Value객체로 저장
-   키는 중복 저장될 수 없지만 값은 중복 저장이 가능하며 기존에 저장된 키와 동일한 키로 값을 저장하면 기존의 값이 새로운 값으로 대체

|   기능   |   리턴타입   |   메서드   |   설명   |
| :-: | :-: | :-: | :-: |
|   객체 추가   |   Object   |   put(Object key, Object value)   |   주어진 키로 값을 저장   해당 키가 새로운 키면 null을 리턴   동일한 키가 있으면 기존의 값을 대체하고 대체되기 이전의 값을 리턴   |
|   객체 검색   |   boolean   |   containsKey(Object key)   |   주어진 키가 있으면 true, 없으면 false를 리턴   |
|   |   boolean   |   containsKey(Object value)   |   주어진 값이 있으면 true, 없으면 false를 리턴   |
|   |   Set   |   entrySet()   |   키와 값의 쌍으로 구성된 모든 Map.Entry객체를 Set에 담아 리턴   |
|   |   Object   |   get(Object key)   |   주어진 키에 해당하는 값을 리턴   |
|   |   boolean   |   isEmpty()   |   컬렉션이 비어있는지 확인   |
|   |   Set   |   keySet()   |   모든 키를 Set객체에 담아 리턴   |
|   |   int   |   size()   |   저장된 Entry객체의 총 개수를 리턴   |
|   |   Collection   |   values()   |   저장된 모든 값을 Collection에 담아 리턴   |
|   객체 삭제   |   void   |   clear()   |   모든 Map.Entry(키와 값)을 삭제   |
|   |   Object   |   remove(Object key)   |   주어진 키와 일치하는 Map.Entry를 삭제하고 값을 리턴   |

-   **HashMap**
    -   Map 인터페이스를 구현한 대표적인 클래스로 키와 값으로 구성된 Entry객체를 저장
    -   해싱(Hashing)을 사용하여 많은 양의 데이터를 검색하는 것에 뛰어난 성능을 보임
    -   HashMap을 생성할 때는 키와 값의 타입을 따로 지정해야 함
    -   Entry객체를 Map 인터페이스의 내부 인터페이스인 Entry 인터페이스를 구현하여 아래의 표와 같은 메서드가 정의되어 있음

|   리턴타입   |   메서드   |   설명   |
| :-: | :-: | :-: |
|   boolean   |   equals(Object o)   |   동일한 객체인지 비교   |
|   Object   |   getKey()   |   Entry 객체의 Key 객체를 반환   |
|   Object   |   getValue()   |   Entry 객체의 Value 객체를 반환   |
|   int   |   hashCode()   |   Entry 객체의 해시코드를 반환   |
|   Object   |   setValue(Object value)   |   Entry 객체의 Value 객체를 인자로 전달한 Value 객체로 변경   |

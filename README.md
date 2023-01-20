# ios-calculator

# Step1, Step 2
- **`트러블 슈팅 ☄️`**
    - 큐, 스택, 자료구조 스터디
        
        [자료 구조의 이용](https://lee-mandu.tistory.com/462?category=633568)
        
    1.  입력값과 연산자를 함께 저장하기 위해 LinkedList, Array중 무엇을 사용하면 좋을까?
        1. 둘 다 기술적으로는 가능할 것이라고 판단
    2. 학습을 목적으로 LinkedList를 활용해보기로 결정
    3. LinkedList에는 어떤 종류의 저장 프로퍼티들이 필요할까 고민
        1. number, operator, isPositive
    4. 구현 방식을 고민 중, 결국 계산을 위해서는 input을 한 곳에 전부 담았다가
    숫자(number)와 operator(연산자)를 분리하는 과정이 필수적이라고 생각
        1. 처음부터 숫자 큐, 연산자 큐 두 가지로 나눠서 관리하는 것이 좋지 않을까?
    5. 이를 위한 방법은 아래와 같다.
        - 연산자 큐, 숫자 큐 각각의 타입으로 2가지를 만들기
        - 제네릭을 활용해 큐를 하나만 만들고 각각 찍어내기
    6. 제네릭을 활용한 방식을 채택
    7. 링크드 리스트를 활용해 계속 구현하는 것도 가능하지만, 현재 로직상 꼭 링크드 리스트를 활용해야 하는 이유가 뚜렷하지 않고, 코드나 로직의 간결성에 있어서는 Array를 활용하는 것이 더 합리적이라고 판단
    8. Array를 활용해 큐를 구현하는 방식으로 전환
    9. 최종적으로 Array를 활용한 큐 방식으로 프로젝트를 진행하기로 결정(제네릭도 활용)

**`학습내용`**

- 자료구조 스터디
    - 리스트(순차, 연결), 스택, 큐, 데크
- 제네릭
- 코드리뷰 방식과 장점은?

**`다음 프로젝트 계획`**

- Step1 리뷰 반영 수정
- Step2 완료 후 PR
- Step2 리뷰 반영 수정
- Step3 구현

`UML`
<p align="center"><img src="https://user-images.githubusercontent.com/99641242/213641971-46d1304b-8c51-4fa2-9f1d-0f90ea8ca5d1.png" width="350" height="500"></p>

`구현사항`

| Name | Type | 책임 |
| --- | --- | --- |
| CalculatorItemQueue | Class | LinkedList를 통한 데이터 저장소 |
| LinkedList | Class | Node 창고 |
| Node | Class | 데이터 창고 |
| Operation | enum | 연산기호 목록 |
| CalculateItem | protocol | 연산기호 함수 정의 |

### **`CalculatorItemQueue`**

- LinkedList를 생성한 객체로써 데이터 저장소의 역할
    
    ```
    class CalculatorItemQueue {
        var linkedList = LinkedList()
    }
    
    ```
    

### **`LinkedList`**

- Array 배열로도 현재 구현가능하지만(더 적절해 보이지만) LinkedList(단방향 연결 리스트) 공부를 위해 활용해 보았어요.
    - 처음에 Array로 구현하려고 했던 이유
        - 계산 과정에서 중간 과정을 삭제, 수정하는 기능이 없기 때문에 append만 있어도 충분
        - 계산 과정에 있어서 엄청난 크기의 데이터 공간이 필요 하지는 않을 것 같기 때문
        - 데이터 접근 속도는 Array가 더 빠르기 때문(계산 속도)


- `Head`를 기준으로 `enqueue` 함수를 통해 다음 `Node` 추가기능을 구현했어요. `dequeue`는 현재 로직상으로는 필요하지 않다고 생각하여 따로 구현하진 않았어요.

```
class LinkedList {
    var head: Node?

    func enqueue(number: Double, operation: Operation) {
        // 처음 head 정하기
	if head == nil {
            head = Node(number: number, operation: operation)
            return
        }
       // 헤드 지정 후 Node 추가하기
        var node = head
        while node?.next != nil {
            node = node?.next
        }
        node?.next = Node(number: number, operation: operation)
    }
}

```

### **`Node`**

- 연산에 필요한 데이터를 담는 창고인 `Node`를 만들었어요
- `Node`에 필요한 요소들을 고민해보니, 필요한 것은 아래와 같다고 생각하여 프로퍼티를 지정해줬어요
    - 숫자 : 소수점 숫자들도 포함할 수 있게 `Double`타입으로 지정했어요
    - 다음 노드 : 단방향 연결 리스트는 `Head`를 기준으로 계속 연결되는 형태로 다음 노드를 담을 수 있도록 해주었어요
    - 연산 기호 : 연산기호는 정해진 것 이외에는 크게 추가될 것이 없다고 판단하여, `enum`타입으로 구현했어요
    - +,- 구분 : 두 가지의 선택권이 있으므로, toggle과 bool타입을 고민하던 중 On,Off보다는 `postive → true, nagative → false`에 매칭하는 것이 어울려서 bool타입으로 최종 선택했습니다.
    
    ```
    class Node: Equatable {
        var number: Double? // 숫자
        var next: Node? // 다음 노드
        var operation: Operation? // 연산기호
        var isPostive: Bool = true // +,- 구분
        //...
    }
    
    ```
    

### **`Operation`**

- Operation이 관리하는 연산기호 종류 : add, subtract, divide, mulitply, calculate
- 향후 button에 text와 매칭하고자 rawValue를 주었어요.
- 요구사항에 명시한데로, CalculateItem 프로토콜을 채택해주었고 프로토콜의 요구사항에 따라 각 연산 함수를 추가해 주었어요.

```
enum Operation: String, CalculateItem {
    case add = "+"
    case subtract = "-"
    case divide = "÷"
    case multiply = "×"
    case calculate = "="

    func add(previousData: Double, nextData: Double) -> Double {}
    // ...
}

```

### **`CalculateItem`**

- 연산기호별 함수를 구현해둔 프로토콜

```
protocol CalculateItem {
    func add(previousData: Double, nextData: Double) -> Double
    // ...
}

```

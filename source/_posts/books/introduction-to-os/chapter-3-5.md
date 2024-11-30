---
title: 3-5. [심화학습] 동적 할당 영역과 시스템 호출
date: 2024-11-30 21:33:27
categories:
  - Books
  - 쉽게 배우는 운영체제
#tags:
---
## 프로세스의 동적 할당 영역

![프로세스의 상세 구조](/images/process_stack_heap.png)

- 프로세스의 코드 영역과 데이터 영역은 프로세스가 실행되기 직전에 위치와 크기가 결정되고 변하지 않기 때문에 정적 할당 영역이라고 부릅니다.
- 스택, 힙 영역은 프로세스가 실행되는 동안 만들어지는 영역으로, 그 크기가 변하는 동적 할당 영역입니다.

### 스택 영역

- 호출한 함수가 종료되면 함수를 호출하기 전 코드로 되돌아와야 하는데, 되돌아올 메모리의 주소를 스택에 저장합니다.
- 변수에 접근할 수 있는 범위에 영향을 미치는 영역인 scope를 구현합니다.
- 스레드가 작업을 진행하면서 임의의 함수를 호출한 횟수가 곧 스택 영역의 크기가 됩니다.

### 힙 영역

- 대부분의 데이터는 데이터 영역에 할당되고 그 크기가 결정되지만, 일부 데이터는 프로세스가 실행되는 동안 동적으로 할당 즉, 조건에 따라 메모리 공간을 할당하고 반환합니다.

```c
int main()
{
    int sarr[50];
    int *darr;
    
    darr = (int*)malloc(sizeof(int)*50); // 할당
    
    free(darr); // 반환(회수)
    
    return 0;
}
```

## exit()와 wait() 시스템 호출

### exit() 시스템 호출

```c
void main()
{
    printf("Hello \n"):
    exit(0);
}
```

- 작업의 종료를 알려주는 시스템 호출입니다.
- C에서 함수 마지막 줄에 `exit(0)` 또는 `return 0`을 사용하는 것은 자식 프로세스가 끝났음을 부모 프로세스에 알려주기 위함입니다.
- 부모-자식 관계를 만드는 이유는 자식 프로세스가 사용하던 자원을 부모 프로세스가 회수하여 자원 관리가 용이하기 때문입니다.

### wait() 시스템 호출

- 부모 프로세스가 먼저 종료됨으로써 미아 프로세스가 생기는 것을 방지하기 위해 자식 프로세스가 종료될 때까지 대기하는 시스템 호출입니다.
- 부모 프로세스와 자식 프로세스 간의 동기화에 사용됩니다.
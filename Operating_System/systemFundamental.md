# 컴퓨터 시스템의 동작 원리

1. [주요 컴퓨터 시스템의 요소](#주요-컴푸터-시스템의-요소)
2. [Memory address section](#Memory-address-section)
3. [Interrupt Handling](#interrupt-Handling)
4. [I/O Structure](#IO-Structure)
5. [I/O 처리 단계](#IO-처리-단계)
6. [Storage Structure](#Storage-Structure)
7. [Memory, CPU Protection](#memory-and-CPU-Protection)

## 주요 컴퓨터 시스템의 요소

### Controller
- I/O를 CPU연산과 동시에 수행하기 위해 작은 CPU가 각각 내장되어있다.
- Main Memory와 I/O는 동시에 수행이 가능하며, Main Memory참조는 CPU만이 할 수 있다
- 따라서, Controller는 Device가 주어진 연산이 끝나면 CPU에게 인터럽트를 주어 주어진 인터럽트 벡터 루틴을 통해 I/O데이터를 Main Memory에 참조할 수 있게 한다.
- Device 혹은 Main Memory의 데이터를 임시로 저장할 수 있는 local Buffer를 가지고 있다.

### CPU
- 옆에 Interrupt Line을 두어 Interrupt Signal이 들어오는지 파악
- CPU는 명령 하나를 수행할 때마다, 인터럽트를 체크한다.

### Kernel: 
- 운영 체제의 일부분이 항상 메모리에 상주하는 것
- 인터럽트 처리 루틴: 다양한 인터럽트에 대한 처리 업무를 정의 한다.
    - 인터럽트 처리 루틴은 운영 체제의 개발자가 미리 프로그래밍하여 커널 내에 포함시켜 둔다.
- OS가 인터럽트의 대응 루틴을 쉽게 찾기 위해 Interrupt Vector를 갖는다
- Interrupt Vector: 인터럽트 종류마다, 번호를 정해 번호에 따라 처리할 코드(Interrupt Service Routine)의 포인터를 가진 자료구조
- Kernel mode, User mode 두 가지의 Operation mode가 존재해 중요한 명령 및 연산을 Kernel mode로 운영 체제에게 맡긴다.
- 사용자 프로그램이 CPU를 가지고 있는 동안 운영 체제가 CPU를 선점할 수 없어 CPU내부에 Mode bit를 두어 0(kernel),1(user)mode를 둔다.
- 사용자 프로그램이 System Call을 사용하면 Mode bit가 0이 되어 OS에게 위임한다.

## Memory address section

1. Data
- 프로그램의 전역 변수 저장
2. Stack
- 실행 중인 함수A가 다른 함수B를 호출하면 호출 함수를 수행하는데, B의 수행이 끝나면 A로 돌아가기 위해 복귀 주소를 저장한다.
- A의 수행과 B의 수행은 다른 메모리 주소가 바뀐다.
3. Code
- 프로그래머가 작성한 코드가 기계어 명령 형태로 저장되는 영역
- CPU가 매 시점 Code영역의 명령을 하나씩 수행한다.

## Interrupt Handling
### Interrupt시 발생하는 CPU의 현상
- CPU가 명령을 수행할 때 CPU내부의 레지스터에 데이터를 읽거나 씀
- 인터럽트가 발생해 새로운 명령을 수행하면 기존의 레지스터값들이 지워짐

### PCB 
- 수행되는 프로그램들을 관리하기 위해 Kernel에서 저장되는 자료구조
- 인터럽트가 발생했을 때 레지스터 값, 메모리 주소 등 작업내용이 없어지므로 그 내용들을 저장하는 자료구조를 내포하고 있다.
- 새로 생긴 Interrupt의 서비스가 끝나고 기존 작업으로 돌아갈 때 사용된다.

### Handling 내용
- 프로그램 내부의 다른 함수호출은 작업 내용을 Stack에 저장한다.
- 인터럽트가 발생하면, 실행 중이던 작업 내용은 PCB에 저장되며 인터럽트 루틴을 수행하는 도중 발생하는 함수는 커널안의 stack영역 중 직전까지 수행했던 프로그램과 연관된 Kernel Stack이 수행함
- 인터럽트의 처리 중 호출되는 함수 또한, 이전 작업을 그 Stack에서 수행됨
- 수행중인 프로그램 수 n만큼 Stack도 n개의 독립적인 공간을 만든다.

### 인터럽트 중 인터럽트가 발생할 때
- 원칙적으로 데이터의 일관성 때문에, 허용하지 않음
- 하지만 중요도가 높은 인터럽트를 허락할 수 있다.
- 이 때, Kernel의 stack에 작업 중이던 인터럽트 작업을 저장한다.

### Software Interrupt 종류
- 예외 사항
    - 프로그램이 수행하면 안되는 부분을 접근할 때 발생
- System Call
    - 운영 체제에 정의된 함수를 호출하는 것
    - ex)printf라는 함수를 호출하면 그 안에 있는 write System Call을 호출해 운영 체제가 그 작업을 수행하게 시킴

## I/O Structure

### Synchronous I/O: 
- 프로그램이 I/O 요청을 하면 CPU는 그 프로그램의 I/O가 끝날 때 까지 제어권을 주지 않음
- I/O가 끝날때 까지, CPU가 그 프로그램에 상주하면 낭비이므로 다른 프로그램에게 제어권을 넘김
- CPU가 다른 프로그램에게 양도하는 결과, 같은 Device에 I/O 요청이 두개 이상이 될 수 있으며 의도치 않는 결과가 발생한다 
- 따라서, Device별로 queue를 두어 순서대로 I/O를 수행하게 한다.
- I/O가 끝나면 Device Controller는 CPU에게 프로그램의 봉쇄상태를 Unlock하라고 인터럽트를 보낸다.

### Asynchronous I/O: 
- I/O 요청 후에도 CPU제어권을 다시 그 프로그램에게 부여하는 방식
- 해당 I/O와 상관없는 작업을 하다가, I/O작업이 끝나면 다시 그 작업을 재개한다.

### Direct Memory Access:
- I/O 연산을 Device마다 Controller가 존재해도 모두 CPU를 거치게 되면 효율성이 떨어짐
- 그래서 DMA라는 일종의 Controller를 두어, local Buffer에서 메모리로 읽어오는 작업을 대신해, 작업을 마친 Interrupt만을 CPU가 처리해 CPU의 효율성을 증대시킨다. 

## I/O 처리 단계

1. 프로그램 A가 Device에게 입력을 하라고 CPU에게 Interrupt를 보낸다.
2. CPU가 수행 중인 프로그램 A의 상태와 레지스터값을 PCB에 저장 후, 커널의 루틴으로 이동
3. 처리 루틴에서 CPU는 Controller에게 I/O를 요청한다.
4. Device는 local Buffer로 데이터를 입력을 받으면서, 프로그램 A(Asynchronous)나 차례상 다른 프로세스(Synchronous)가 CPU의 제어권을 얻는다.
5. 데이터를 모두 입력 받으면, Controller는 CPU에게 Interrupt를 주어 처리를 한다.
6. 인터럽트를 처리하기 전에 현재 프로그램 상태를 저장한다.

## Storage Structure

### 주 기억 장치 
- 휘발성 메모리에 주로 속해있다.
1. Register: CPU내부에 존재하며, CPU작업 내용들을 저장한다.
2. Cache Memory: Register와 Main Memory사이에 존재하며, 캐슁 기법으로 빈번히 사용되는 정보를 선별적으로 저장함<br>
용량이 적고 빠른 저장 장치의 성능 향상을 위한 캐싱 기법 사용
3. Main Memory: 실행할 프로그램 및 시스템을 유지할 운영 체제 Kernel, 시스템 영역이 올라감

### 보조 기억 장치
- 비휘발성 메모리에 주로 속해있다.
1. File System용
    - 전원이 나가도 유지해야할 정보를 파일 형태로 저장
2. Swap Area
    - 한정된 메모리를 효율있게 사용하기 위해 프로그램 내용을 저장하고 당장 사용할 내용들만 Main Memory로 Swap in시킨다.
    - 필요 없는 내용들은 다시 보조 기억 장치의 Swap Area로 Swap in시킨다.

## Memory and CPU Protection

### Memory

- 메모리 경우, 여러 프로그램이 동시에 올라가므로 다른 사용자나 프로그램이 C언어의 포인터등으로 메모리 주소를 잘못 참조하여 침범할 수 있다.
- 그래서 Base Register, Limit Register 두개를 사용해 프로그램의 메모리 범위를 명시해 다른 프로그램의 메모리를 참조하지 못하게 한다.
    - Base Register: 프로그램의 시작 주소
    - Limit Register: 프로그램이 사용할 수 있는 메모리 공간
    - 이 경우 Page기법을 사용한 경우 일반적으로 보호가 보장되지 않는다
- Memory연산은 사용자 프로그램에서도 사용이 가능하지만, Base와 Limit Register값 세팅은 Kernel Mode에서 이루어져야 한다.

### CPU
- CPU를 부적절하게 독점을 할 경우 OS가 CPU를 선점할 수가 없게 된다.
- Timer: 정해진 시간이 지나면 OS에서 CPU 제어권을 다른 프로그램으로 양도 시킨다.
- Timer가 지나면 인터럽트를 보내어 CPU제어권을 바꾸게 한다.
- 시분할 시스템에서 현재 시간을 계산하기 위해서도 사용한다
- Load Timer: 타이머 값을 세팅하는 명령, Kernel Mode에서 수행되어야 한다.

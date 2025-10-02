## Process 
- 프로세스란 실행중인 프로그램(디스크에 저장된 실행파일)이라고 보면됨
- 프로세스의 메모리 배치는 텍스트(실행 코드), 데이터(전역변수), 힙(프로그램 실행중 동적 할당 메모리), 스택(지역변수, 함수 매개변수..등등)으로 배치됨

## PCB(process control block)
하나의 프로세스 데이터 저장소
- 프로세스의 상태
- 프로그램 카운터(이 프로세스가 다음 실행할 명령어 주소를 포인터)
- CPU 레지스터
- CPU 스케쥴링 정보
- 메모리 관리 정보
- 회계/정보
- 입출력 상태 정보

### 프로세스 스케쥴링
1. process system call 되면 ready queue(대부분 연결리스트) 삽입
   ready queue header에는 첫번째 PCB에 대한 포인터 저장되고, 준비큐의 다음 PCB 포인터 필드 저장
2. CPU core 할당되면, 프로세스는 어떻게든 인터럽트 되거나 I/O 요청 완료, 특정 이벤트 시스템 콜을 기다려야하기 때문에 wait queue 대기 
   여기서 I/O 요청 -> i/o wait queue 대기, fork() 후 자식 프로세스 종료할때 까지 wait queue, 타임 슬라이스나, 인터럽트로 인한 ready queue 로 돌아갈 수 있음
3. 종료되면 모든 큐에서 제거되고 PCB 반환된다.


### Context Switch 
- 인터럽트가 뭔지 생각해보자. 인터럽트는 커널에서 CPU 코어를 현재 사용자 작업에서 뺏어 가는 걸 말하는데 여기서 사용자 프로그램의 작업을 그냥 뺏어가는게 아니라 어디에 저장하고 커널단으로 뺏어가는 시간이라고 보면됨
- 여기서 문맥은 Process 의 PCB 임
- CPU 코어를 다른 프로세스로 교환하려면 이전의 프로세스 상태를 보관하고 새로운 프로세스의 보관된 상태를 복구하는 작업이 필요한데 이걸 Context Switch 라고 하고 이 시간은 시스템이 유용한 일을 하지 못하기 때문에 순수한 오버헤드임

### process fork(), exec()
``` c
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

int main() {
    pid_t pid;

    pid = fork();   // 1. 자식 프로세스 생성

    if (pid < 0) {
        // fork 실패
        perror("fork failed");
        return 1;
    }
    else if (pid == 0) {
        // 자식 프로세스 실행 부분
        printf("자식 프로세스: exec 실행 전 (PID: %d)\n", getpid());

        // exec 함수로 "ls -l" 실행
        execlp("ls", "ls", "-l", NULL);

        // exec()은 성공하면 절대 이 줄로 안 돌아옴
        // 실패하면 perror 출력
        perror("exec failed");
    }
    else {
        // 부모 프로세스 실행 부분
        printf("부모 프로세스: 자식 기다리는 중 (PID: %d)\n", getpid());

        // 자식이 끝날 때까지 대기
        wait(NULL);
        printf("부모 프로세스: 자식 종료 확인!\n");
    }

    return 0;
}

```

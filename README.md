# EcoServerTest



동기 IO 
- 이벤트 데이터 들어온후 데이터 전송받음
비동기 IO 
- 데이터를 모두 전송 받은후 마지막에 이벤트 데이터가 들어옴

===========================================================

출처 : https://blog.naver.com/zxwnstn/221511584116

1. overlapped IO 
- 기본적으로 비동기 IO기반
- 하나의 서버가 2개 3개 이상의 IO작업을 중첩시켜 작업 할 수 있음

2. overlapped 함수를 사용할때 이용 구조체 
- WASBuf, WSAOverlapped 
1) WASBuf 

typedef struct _WSABUF
{
ULONG len;  //the length of the buffer 
CHAR *buf;  //the pointer to the buffer 
}

- 실재로 함수에 이용될때에는 LPWSABUF 형태로 이용되는데 이는 WSABUF를 배열로 만들어 사용해도 된다는 소리 
- 하나의 IO작업에 동시에 여러개의 버퍼가 사용될수 있음

2) WSAOverlapped - Overlabpped 구조체와 같음

typedef struct _OVERLAPPED
{
ULONG_PTR Internal;
ULONG_PTR InternalHigh;
union
	struct
	{
		DWORD Offset;
		DWORD OffsetHigh;
	} DUMMYSTRUCTNAME;
	PVOID Pointer;
}DUMMYUNIONNAME;

- event를 제외한 모든 인자는 시스템 내부적으로 이용되는 것이라 별로 신경쓸 필요가 없음
- 단 0으로 초기화는 해주어야 한다
- event 가 이용됨을 알수 있는데, 이는 비통기 IO작업이 완료됬었을때 signaled 상태가 되어 알람 역활을 해주는 아주 중요한 요소임 -> 꼭기억

3. 필요 함수 
- WSASocket, WSARecv, WSASend, WSAGetOverlappedResult 

1) WSASocket  - 소켓을 만들때 이용
- 반환형은 SOCKET

int af;
int type; 
int protocol;
int LPWSPAROTOCOL_INFO lpProtocolInfo;
GROUP g;

- 1~3인자는 기본 소켓 생성 인자와 같음
- 3~4인자는 사용하지 않음
- 5번째인자 중요 
- 5번째 인에 WSA_FLAG_OVERLAPPED 라는 인자를 전달하면 비동기 IO 수행 가능 소켓이 됨

2) WSASend - 패킷을 전송하는 함수
- 반환형 int 

Socket s;
LPWASBUF lpBuffers;
DWORD dwBufferCount;
LPDWORD lpNumberOfBytesRecvd;
LPDWORD lpFlags;
LPWSAOVERLAPPED lpOverlapped;
LPWSAOVERLAPPED_COMPLETION_ROUTINE lpCompletionRoutine;

- LPWSAOVERLAPPED lpOverlapped 이를 지정해주지 않으면 일반적인 blocking 함수가 되어버림
- LPWSAOVERLAPPED_COMPLETION_ROUTINE lpCompletionRoutine : IO 작업이 끝나면 실행될 함수의 주소 완료루팅 방식에서 이용, 완료루팅 방식을 사용하지 않는다면 NULL을 전달
- LPDWORD lpNumberOfBytesRecvd : 비동기 방식이라고 해서 무조건 함수를 바로 반환하고 IO작업을 진행하는 것은 아님, 전송될 패킷의 크기가 작은 경우 함수를 실행시키는 동시에 작업을 완료해버릴수도 있다. 이런 경우를 대비하는 인자값.

3) WSARecv - 패킷을 수신하는 함수
반환형 int

SOCKET s
DWORD dwBufferCount;
LPDWORD lpNumberOfByteSent;
DWORD dwFlags;
LPWSAOVERLAPPED lpOverlapped;
LPWSAOVERLAPPED_ONMPLETION_ROUTINE lpCompletionRoutine;

- Send와 크게 다를바없음

4) WSAGetOverlappedResult - IO작업 완료를 확인하는 함수
- 반환형 bool

HENDLE hFile;
LPOVERLAPPED lpOverlapped;
LPDWORD lpNumberOfBytesTransferred;
BOOL bWait;

- BOOL bWait : IO작업을 완료할때까지 이 함수를 블로킹 상태에 둘 것인지를 설정하는 인자. true 로 설정하면 IO작업이 완료될때까지 반환하지 않고 blocking 상태가 됨.

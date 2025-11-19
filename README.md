# RC-CAR
web 기반 RC CAR 제어
제어 보드는 STM32 NUCLEO F103RB

## 환경
STM32CubeIDE

Java script

HTML

## 필요 장비
NUCLEO F103RB
DC Motor
BlueTooth module
ultrasonic distancement
Buzzer
ETC...

## STM32 ioc setting

### 1. RCC Disable
외부 클럭을 사용하지 않으므로 Disable

<img width="675" height="768" alt="image" src="https://github.com/user-attachments/assets/b1b6597c-3db2-4f73-a52d-30b83fc23c07" />


### 2. ultrasonic distancement와 dc motor PWM 제어를 위해 timers 활성화

Clock source -> Internal clock

Channel -> PWM Generation // DC Motor PWM 제어를 위한 설정

Prescale -> 63 // 현재 보드의 클럭은 64MHz, Prescale은 clock divider의 역할을 함 --> timer의 output은 1MHz의 클럭

counter period -> 25000 // 클럭의 한 사이클을 세고 저장하는 기능. 0번부터 세기 시작해서 최대 설정 값까지 저장. 여기서는 최대 25001번까지 세고, 이후로 다시 0번부터 세기 시작.

<img width="664" height="693" alt="image" src="https://github.com/user-attachments/assets/78949359-f868-437b-9fa0-89f5a4b7a91b" />

나머지 timer도 이와 동일하게 적용

Clock Configuration에서 아래와 같은 설정인지 확인

<img width="1544" height="773" alt="image" src="https://github.com/user-attachments/assets/c62a082e-250d-473c-a887-bd97f8af3c78" />


### 3. UART 설정

Bluetooth를 통한 통신을 하므로, UART3 활성화 필요. // 기존 UART2는 USB통신

Mode -> Asynchronous

사용할 Baud Rate 및 Word Length 등 확인

<img width="490" height="695" alt="image" src="https://github.com/user-attachments/assets/2b2a244c-51f9-4cee-9ba9-09373119abbd" />

### 4. ioc pinout

<img width="636" height="555" alt="image" src="https://github.com/user-attachments/assets/d0a59857-854d-4047-8728-fbe319868330" />


모든 설정이 완료되었으면, Project -> Generate Code 클릭.

main.c 변경 및, inc와 src 폴더에 uart3_printf 파일 추가.

##  Java script

여기서, Java script는 node.js를 통한 서버 구축을 담당.

최종적으로, HTML에서 신호를 송신 혹은 수신할 수 있도록 해주는 역할.


### 실행법
window - cmd - 디렉토리../node app.js 실행

***주의***

app.js에서, 실제 Bluetooth가 연결된 port가 다르므로, COM을 변경해야만 함. 


연결되지 않았을 때 

<img width="1017" height="484" alt="image" src="https://github.com/user-attachments/assets/c198a37d-0852-433a-99ca-056cf8e0e868" />


연결되었을 때 

<img width="621" height="480" alt="image" src="https://github.com/user-attachments/assets/e4564c02-fb97-4c46-a34f-389d843fcd81" />


정상적으로 연결이 되면, localhost:3000 혹은 서버와 연결된 ip:3000으로 연결 가능.


## HTML

여기서, HTML은 제어 신호 송수신을 담당.

기본적으로 DC Motor 조작은 URL 클릭을 이용, ultrasonic distancement 값은 node.js에서 값을 읽고, 이를 html에 띄우는 방식.

키보드 조작 및 마우스를 통한 버튼 클릭을 통하여 방향을 제어

숫자 0~5까지의 클릭을 통해 DC Motor의 회전 속력 제어 및 Buzzer 작동

W : 전진, S : 후진, A : 좌회전, D : 우회전

0 : Buzzer, 1~5 : DC Motor 회전 속력 제어(20~100%)

<img width="727" height="808" alt="image" src="https://github.com/user-attachments/assets/7ac30965-3009-4a58-9efe-b565eaf7e0ea" />


추가적으로, 스마트폰 터치를 통해서도 제어가 가능하도록 추가.



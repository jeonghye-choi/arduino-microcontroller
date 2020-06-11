# 📙 Serial통신

UART, SPI, I2C

### ✔ UART

- 비동기 통신 -> 별도의 클록 사용X (start bit -> data bit -> stop bit)

- 저수준 통신방법

    - 하드웨어 수준 (통신을 위한 전용 하드웨어 핀 제공)

- 4개 통신 채널 

<br><br>

### ✔ UART vs SoftwareSerial 클래스

- UART 

    - 하드웨어 지원

    - 메가4개, 우노1개

    - 1:1통신 = 하나의 채널에는 하나의 장치 연결 가능

- SoftwareSerial 클래스

    - 하드웨어로 지원되는 UART통신을 소프트웨어로 에뮬레이션

    - 우노는 1개채널 -> 부족하니까 SoftwareSerial클래스 흔히 사용

    - 메가는 잘 사용X

    - 채널 생성과정에서 데이터 핀을 지정해야하는 차이 이외엔 사용방법 동일

- UART 연결 방법

    ![image](https://user-images.githubusercontent.com/54584063/84385421-8056d680-ac2a-11ea-9f83-6a94563e1c00.png)

- LAB

    - 마스터 코드
        ```c
        #include <SoftwareSerial.h>

        // SoftwareSerial(RX, TX) 형식으로 slave의 핀과 교차 연결
        SoftwareSerial mySerial(4, 5);
        int button_pin = 2;
        boolean previous_state = false;

        void setup(){
            mySerial.begin(9600);
            Serial.begin(9600);
            pinMode(button_pin, INPUT);
        }

        void loop(){
            boolean state = digitalRead(button_pin);
            if(state) {
                if(previous_state==false){
                    mySerial.write('1');
                    Serial.println("Button is pressed..");
                }
                previous_state = true;
                delay(20);
            }
            else{
                previous_state = false;
            }
        }

        ```
    - slave 코드

        ```c
        int LED_pins[] = {2,3,4,5};
        int num_LED = 0;
        int delta = 1;

        void setup(){
            Serial.begin(9600);
            Serial1.begin(9600);

            for (int i=0; i<4; i++){
                pinMode(LED_pins[i], OUTPUT);
                digitalWrite(LED_pins[i], LOW);
            }
        }

        void loop(){
            if(Serial1.available()){
                char data = Serial1.read();
                if (data == '1'){
                    num_LED += delta;
                    if(num_LED==5){
                        delta = -1;
                        num_LED = 3;
                    }
                    else if(num_LED == -1){
                        delta = 1;
                        num_LED = 1;
                    }
                    Serial.print("Currently");
                    Serial.print(num_LED);
                    Serial.println("LEDs are ON.");

                    for(int i=0; i<4; i++){
                        if(i<num_LED){
                            digitalWrite(LED_pins[i], HIGH);
                        }else{
                            digitalWrite(LED_pins[i], LOW);
                        }
                    }

                }
            }
        }
        ```

<br><br>

### ✔ SPI 

<br><br>

### ✔ I2C


<br><br>

### 🔎 정리

- UART

    - 별도의 클록X

    - 1:1 통신만 지원

    - 2개의 데이터 선: RX, TX - 교차연결!

- 아두이노 UART

    - 전용핀이 아닌 데이터 핀으로 UART통신을 하기에 -> SoftwareSerial클래스 사용!
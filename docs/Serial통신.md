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

- 고속 직렬 통신 방식

- 1:n 통신 (cf UART는 1:1)

- 4개의 연결선

    - SCK : 동기통신을 위한 클록

    - MOSI, MISO : 데이터 송수신
        
        Master Output Slave Input/ Master Input Slave Output

    - SS : 하드웨어적으로 슬레이브 중 하나 선택

        선택된 슬레이브의 SS만 LOW상태

- 아두이노 SPI

    - 하드웨어적으로만 지원(cf UART의 SoftwareSerial클래스 같은 에뮬레이션 라이브러리 없음)

    - 4개의연결선 중 3개의 연결선(SS제외)은 전용핀 사용

    - **"SPI"**라이브러리

    - 시작: begin(), 끝: end(), 데이터전송: transfer()

- 연결방법

    ![image](https://user-images.githubusercontent.com/54584063/84387232-a5007d80-ac2d-11ea-8f92-59dbbca1fc02.png)

- LAB

    - 마스터 -> 슬레이브

        마스터 코드
        ```c
        #include <SPI.h>

        int SS_arduino = 53;

        void setup(void) {
            pinMode(SS_arduino, OUTPUT);
            digitalWrite(SS_arduino, HIGH); //슬레이브가 선택되지 않은 상태

            SPI.begin();

            //안정적인 전송을 위해 분주비를 높여 전송속도를 낮춤
            SPI.setClockDivider(SPI_CLOCK_DIV16);
        }

        void loop(void){
            const char *p = "Hello, world\n";
            digitalWrite(SS_arduino, LOW);
            for (int i=0; i<strlen(p); i++){
                SPI.transfer(p[i]);
            }
            digitalWrite(SS_arduino, HIGH);
            delay(1000);
        }
        ```
    - 슬레이브: 마스터로부터 수신된 데이터 출력
        
        슬레이브 코드
        - 슬레이브 모드 설정

            - SPI에는 슬레이브 모드X -> 컨트롤 레지스터인SPCR 직접 조작해야함

            - SPI통신 사용하도록 : SPE 비트1

            - SPI통신에서 슬레이브로 동작하도록 : MSTR 비트0

            - SPI통신을 통해 데이터가 수신된 경우, 인터럽트가 발생하도록 : SPIE 비트1

                ![image](https://user-images.githubusercontent.com/54584063/84387791-83ec5c80-ac2e-11ea-95ff-9eb5fc6f65e9.png)


        ```c
        #include <SPI.h>

        char buf[100];
        
        // pos와 process_it은 인터럽트 처리 루틴에서 값을 바꾸는 변수이므로
        // volatile 선언을 통해 매번 값을 참조하도록 설정
        volatile byte pos = 0;
        volatile boolean process_it = false;

        void setup(){
            Serial.begin(9600);

            //MOSI, SCLK, SS는 입력으로 설정하지 않아도 된다.
            //MISO 역시 이 예에서는 사용하지 않으므로 생략할 수 있다.

            pinMode(MISO, OUTPUT);
            pinMode(MOSI, INPUT);
            pinMode(SCK, INPUT);
            pinMode(SS, INPUT);

            SPI.setClockDivider(SPI_CLOCK_DIV16);

            //SPI 통신을 위한 레지스터 설정
            //SPCR : SPI Control Register
            SPCR |= (1 << SPE);
            SPCR &=~ (1 << MSTR);
            SPCR |= (1 << SPIE);
        }

        ISR(SPI_STC_vect){
            byte c = SPDR;
            if(pos<sizeof(buf)){
                buf[pos++] = c;

                if(c=='\n'){
                    process_it = true;
                }
            }
        }

        void loop(){
            if(process_it){
                buf[pos] = 0;
                Serial.print(buf);

                pos = 0;
                process_it = false;
            }
        }
        ```




<br><br>

### ✔ I2C

- 저속 직렬 통신 (cf SPI: 고속 직렬)

- 1:n 통신

- 2개의 연결선: SDA(데이터 송수신), SCL(동기통신을 위한 클록)

- N개의 슬레이브는 '주소'에 의해 구별됨

    - SPI의 ss와 같은 연결선X -> 슬레이브 수에 따라 연결선 증가X

- **"Wire"**라이브러리 : 마스터, 슬래이브 모두 지원

    cf. SPI라이브러리는 마스터모드만 지원: SPCR직접 조작

    - 마스터모드

        - 송신:begin Transmission -> write -> end Transmission

        - 수신:requestFrom
    
    - 슬레이브 모드: 인터럽트 기반

        - 송신: onRequest

        - 수신: onReceive

- 연결방법

    ![image](https://user-images.githubusercontent.com/54584063/84389830-9ddb6e80-ac31-11ea-8e05-30b2257126c3.png)

- LAB

    - 마스터

        ```c
        #include <Wire.h>

        #define SLAVE 4  //슬레이브 주소

        void setup(){

            //I2C통신을 위한 Wire 라이브러리 초기화, 마스터로 참여하는 경우 주소 불필요
            wire.begin();
            Serial.begin(9600)
        }

        void loop(){
            i2c_communication();
            delay(1000);
        }

        void i2c_communication(){
            int reading = analogRead(A0);

            Wire.beginTransmission(SLAVE);

            Serial.print("I send");
            Serial.println(reading);
        }
        ```
    - 슬레이브

        ```c
        #include <Wire.h>

        #define SLAVE 4

        void setup(){
            //Wire 라이브러리 초기화, 슬레이브로 참여하기 위해서는 주소 지정
            Wire.begin(SLAVE);

            //마스터의 데이터를 수신했을 때 처리할 함수 등록
            Wire.onReceive(receiveFromMaster);
            
            Serial.begin(9600);
        }

        void loop(){
        }

        void receiveFromMaster(int bytes){
            byte r1 = Wire.read();   //2바이트 정수값 읽기
            byte r2 = Wire.read();

            int value = (r1 << 8) + r2;

            Serial.print("I got");
            Serial.println(value); 
        }

        ```


<br><br>

### 🔎 정리

![image](https://user-images.githubusercontent.com/54584063/84389431-f9592c80-ac30-11ea-9274-02feeed5e80e.png)


- UART

    - 별도의 클록X

    - 1:1 통신만 지원

    - 2개의 데이터 선: RX, TX - 교차연결!

- 아두이노 UART

    - 전용핀이 아닌 데이터 핀으로 UART통신을 하기에 -> SoftwareSerial클래스 사용!

- SPI

    - 고속 직렬 통신 (많은 데이터 고속 전송)

    - 1:n

    - 4개의 연결선: 3개전용선 + 슬레이브 선택 선(LOW가 선택)

    - "SPI"라이브러리, 슬레이브모드는 지원X -> SPCR조작

- I2C

    - 저속 직렬 통식 (적은 데이터 전송에 사용 ex: 센서 데이터 전송)

    - 1:n

    - 2개의 연결선 (슬레이브는 주소로 선택)

    - "Wire"라이브러리 : 마스터 슬레이브 모두 지원

    - 슬레이브는 인터럽트 방식

        인터럽트 : 다른 연산을 수행하고 있는 중이더라도 특정 이벤트가 발생하면 즉시 연산을 멈추고
        발생한 이벤트를 처리하기 위한 연산을 수행
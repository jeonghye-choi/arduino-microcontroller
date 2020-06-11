# 📌 text LCD

LCD : 액정(Liquid Crystal)을 이용해 문자 출력

- 16핀 = 제어3 + 데이터8 + 전원5

    ![image](https://user-images.githubusercontent.com/54584063/84368010-00237780-ac10-11ea-9829-c7f1031a516a.png)

    ![image](https://user-images.githubusercontent.com/54584063/84368372-732cee00-ac10-11ea-90c8-1ce365ad7b56.png)

- 제어핀

    - 4. RS : 데이터 제어(RS=HIGH), 명령 제어(RS=LOW)

    - 5. R/W : 읽기(R/W=HIGH), 쓰기(R/W=LOW)

    - 6. E : enable(처리시작)하도록 신호

### ✔ 연결방법

![image](https://user-images.githubusercontent.com/54584063/84368828-0bc36e00-ac11-11ea-8167-861e4b184fd8.png)

- Vss : 그라운드 연결

- Vcc : 전원 연결

- RS : 출력핀

- R/W : write로만 사용 -> GND

- 상위 4비트만 출력핀에 연결

- LED 백라이트 전원 연결

<br><br>

### ✔ 제어 방법

1. **"LiquidCrystal"**라이브러리

2. 객체 생성

    LiquidCrystal lcd(44, 45, 46, 47, 48, 49);

3. 라이브러리 초기화

    lcd.begin(열, 행)

    lcd.begin(16, 2) : 16열 2행 


<br><br>

### ✔ 출력 함수

- lcd.write()

- lcd.print()

- lcd.clear() : 화면 지움

- lcd.setCursor(col, row) : 커서 위치

    16열 2행 = (0,0) ~ (15,1)

- 🔎 write / print

    print(1) : 1, write(1) : 아스키코드


<br><br>

### ✔ LAB

- Hello World 표시

    ```c
    #include <LiquidCrystal.h>

    // 핀번호 (RS, E, DB4, DB5, DB6, DB7)
    LiquidCrystal lcd(44, 45, 46, 47, 48, 49); // LCD연결

    void setup() {
        lcd.begin(16, 2);
        lcd.print("Hello World!");
    }

    void loop(){}
    ```
- 임의의 위치 표시

    ```c
    #include <LiquidCrystal.h>

    LiquidCrystal lcd(44, 45, 46, 47, 48, 49);

    void setup(){
        lcd.begin(16, 2); // LCD 초기화
        lcd.setCursor(0,0);
        lcd.write('1');

        lcd.setCursor(5,0);
        lcd.write('2');

        lcd.setCursor(0, 1);
        lcd.write('3');

        lcd.setCursor(5,1);
        lcd.write('4');
    }

    void loop(){}
    ```
- 사용자 정의 문자

    ![image](https://user-images.githubusercontent.com/54584063/84373409-7f687980-ac17-11ea-8b58-9126ee7c8ea1.png)

    - void createChar(위치, 문자데이터)

    ```js
    #include <LiquidCrystal.h>
    // 핀 번호 (RS, E, DB4, DB5, DB6, DB7) 
    LiquidCrystal lcd(44, 45, 46, 47, 48, 49); // LCD 연결
    // 사용자 정의 문자 데이터 
    byte user1[8] = { B00000, B10001, B00000, B00000, B10001, B01110, B00000, B00000 };
    byte user2[8] = { B00000, B10001, B01010, B10001, B00000, B10001, B01110, B00000 };
    void setup() {
        lcd.createChar(0, user1); // 사용자 정의 문자 1 생성 
        lcd.createChar(1, user2); // 사용자 정의 문자 2 생성
        lcd.begin(16, 2); // LCD 크기 설정
        lcd.clear(); // LCD 화면 지우기
        lcd.write(byte(0)); // 사용자 정의 문자 1 출력
        lcd.setCursor(0, 1); // 두 번째 행으로 이동
        lcd.write(byte(1)); // 사용자 정의 문자 2 출력
    }
    void loop() { }

    ```

<br><br>

### ✔ Text LCD 내부구조

![image](https://user-images.githubusercontent.com/54584063/84373698-e6862e00-ac17-11ea-8f62-7fa22979ce86.png)

- 하나의 모듈 = LCD panel + 제어기(controller)

- 데이터 버스로 데이터 이동 : 전송된 문자 데이터 지정된 방식으로 출력

    - DD RAM

        : LCD화면과 1:1 대응
    - CG RAM

        : LCD에 Display
    
    - CG ROM

        : LCD화면에 display될 글자데이터 저장하고 있는 ROM
    
    - controller

        : display 문자코드를 받아서 DDRAM 에 write, display변경 명령 처리, 주기적으로 DDRAM 데이터 읽어서
        그에 따른 글자를 ROM에서 읽어와 display하는 일 수행
    
    - Power회로

        : High level 신호를 만드는 Boost DC-DC converter; 필요한 여러가지의 전압을 만들어 전압분배회로 등의
        전원 회로


<br><br>

### 🔎 정리

- text LCD

    - 액정을 사용한 출력장치

    - 16칸 2줄

    - "LiquidCrystal" 라이브러리

- 단점

    - 고정된 위치

    - 문자 출력

    - 흑백
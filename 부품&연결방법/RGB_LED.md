# 📌 RGB LED

![image](https://user-images.githubusercontent.com/54584063/84186198-27bcf780-aacb-11ea-9ffc-2e3c64176262.png)

공통핀과 3개의 R,G,B 제어핀으로 구성

공동 양극방식(common anode)의 경어 제어판에 GND로 출력하면 켜짐

### ✔ 연결방법

![image](https://user-images.githubusercontent.com/54584063/84186350-5f2ba400-aacb-11ea-98c0-bd737cc6c70d.png)

### ✔ 밝기 조절

- 최대 밝기 : analogWrite(0)

- 최소 밝기 : analogWrite(255)


### ✔ 예시

- RGB LED 밝기 조절
    ```js
    int RGB_LED[] = {6,7,8};

    void setup(){
        for (int i=0; i<3; i++){
            pinMode(RGB_LED[i], OUTPUT);
        }
    }

    void loop(){
        // Adjusting Blue color
        digitalWrite(RGB_LED[1], HIGH);
        digitalWrite(RGB_LED[2], HIGH);
        for (int i=255; i>=0; i--){
            analogWrite(RGB_LED[0], i);
            delay(10);
        }
    }

    ```

- 가변저항사용 -> LED밝기조절 ; 방법1

    ```js
    int pins_LED[] = {2, 3, 4, 5}; // Pins connected to LEDs

    void setup() { 
        Serial.begin(9600); 
        for (int i = 0; i < 4; i++) {
            pinMode(pins_LED[i], OUTPUT);
        } 
        pinMode(A0, INPUT); // Pin A0 connected to a Potentiometer 
    }

    void loop() {
        int ADC_value = analogRead(A0); // ADC value 
        int PWM_value = ADC_value >> 2; // PWM value 
        Serial.print(String("ADC value : ") + ADC_value);
        Serial.println(String(", PWM value : ") + PWM_value);
        for (int i = 0; i < 4; i++) {
            // Adjusting the brightness of LEDs 
            analogWrite(pins_LED[i], PWM_value);
        } 
        delay(1000);
    }

    ```
- 가변저항사용 -> LED밝기조절 ; 방법2

    ```js
    int pins_LED[] = {2, 3, 4, 5}; // Pins connected to LEDs

    void setup() { 
        Serial.begin(9600);
        for (int i = 0; i < 4; i++) {
            pinMode(pins_LED[i], OUTPUT);
        } // Pin A0 connected to a Potentiometer
        pinMode(A0, INPUT);
    }

    void loop() {
        int ADC_value = analogRead(A0); // ADC value
        int PWM_value[4] = {0, };
        Serial.println(String("ADC value : ") + ADC_value);
        for (int i = 3; i >= 0; i--) { 
            // Calculating the brightness of each LED 
            if (ADC_value >= 256 * i) {
                PWM_value[i] = ADC_value - 256 * i;
                ADC_value -= (PWM_value[i] + 1);
            } // Adjusting the brightness of LEDs 
            analogWrite(pins_LED[i], PWM_value[i]);
            Serial.print(" ");
            Serial.print(PWM_value[i]);
        } 
        Serial.println();
        delay(500);
    }
    ```



# 📙 String Class와 디지털 입력

## 🌟 String Class

### ✔ String Class

```
String string1 = "Hello String";

String stringOne = String(13);

String stringOne = String(45, HEX); //16진법

String stringOne = String(5.689, 3); //소숫점 3째자리
```

String("abc") + 123 : (가능) String객체 + 정수 

"abc" + 123 : (불가능) 문자열 + 정수

<br>
<br>

### ✔ String Class 함수

string.compareTo(string2)  : 두 문자 아스키코드 비교 -> 음수(string < string2), 0(=), 양수(string > string2)

string.equals()  :  대소문자 구분 -> T/F

string.equlsIgnoreCase  :  대소문자 구분X

<br>
<br>

### ✔ 예시

- String 객체 생성 및 출력

    ```js
    void setup() {
        Serial.begin(9600);
    }

    void loop() {
        String str1 = "One string", str2 = "Another string;
        int n = 1234;
        float f = 3.14;
        char c = 'A';

        Serial.println()
        Serial.println(str1);   // String출력 
        Serial.println(str1 + " " + str2);   // 문자열 연결 
        Serial.println(String(n));   // 정수로부 10진 문자열 생성 
        Serial.println(String(n, BIN));   // 정수로부 2진 문자열 생성 
        Serial.println(String(n, HEX));   // 정수로부 16진 문자열 생성

        Serial.println(String(f)); 
        Serial.println(f); 
        
        // 다른 데를 연결여 새로운 String 객체를 생성다. 
        Serial.println("String + integer : " + String(n)); 
        String str3 = "String + character : "; 
        str3 += n; 
        Serial.println(str3);   //Serial.println("String + character : " + 1234) --> 오류!(문자열 + 정수)
        
        while (true);
    }
    ```

- Serial로부터 문자열 입력 받기1

    ```js
    void setup() { 
        Serial.begin(9600); // 시리얼 기 
    }
    void loop() { 
        int state = 1, len = 0;
        char buffer[128];

        while (true) { 
            if (state == 1) { 
                Serial.print("Enter a String --> "); 
                state = 2; 
            } 
            while (Serial.available()) { 
                char data = Serial.read(); 
                if (data == '\n') { 
                    // 개 문 ‘\n’ 만날 때까지 읽음 
                    buffer[len] = '\0'; 
                    String in_str = buffer; 
                    Serial.println(in_str + " [" + in_str.length()+"]");
                    state = 1;
                    len = 0;
                    break; 
                } 
                buffer[len++] = data;
            }
        }
    }
    ```

- Serial로부터 문자열 입력 받기2

    ```js
    void setup() { 
        Serial.begin(9600); // 시리얼 포트 초기화
    }
    void loop() { 
        int state = 1;
        char buffer[128];

        while (true) { 
            if (state == 1) { 
                Serial.print("Enter a String --> ");
                state = 2;
            } 
            while (Serial.available()) { 
                int len = Serial.readBytesUntil('\n', buffer, 127);
                if (len > 0) {
                    buffer[len] = '\0';
                    String in_str = String(buffer);
                    Serial.println(in_str + " [" + in_str.length()+"]");
                    state = 1;
                    break;
                }
            }
        }
    }
    ```

## 🌟 디지털 입력; 버튼

Floating Input 문제(1이나 0이 아닌 미결정 상태) 발생 -> 풀다운 저항 사용

|  <center>풀업/풀다운 저항 사용</center> |  <center>버튼off</center> |  <center>버튼on</center> |
|:--------|:--------:|--------:|
|**사용안함** | <center>플로팅 </center> |*1* |
|**풀다운** | <center>0 </center> |*1* |
|**풀업** | <center>1 </center> |*0* |


### ✔ 풀다운 저항

버튼 on : 1
버튼 off : 0

![image](https://user-images.githubusercontent.com/54584063/84107874-c872cf00-aa59-11ea-93cc-25de7d5ab7e2.png)

<br>
<br>

### ✔ 풀업 저항

버튼 on : 0
버튼 off : 1

![image](https://user-images.githubusercontent.com/54584063/84107899-d7598180-aa59-11ea-9b83-7d347f46acf0.png)

```js
int buttons[] = {14, 15, 16, 17}; // 버튼 연결 핀
void setup() { 
    Serial.begin(9600); // 시리얼 통신 초기화
    for (int i = 0; i < 4; i++) { // 버튼 연결 핀을 입력으로 설정
        pinMode(buttons[i], INPUT);
    }
} 
void loop() { 
    for (int i = 0; i < 4; i++) {
        Serial.print(digitalRead(buttons[i])); // 버튼 상태 출력
        Serial.print(" "); 
    } 
    Serial.println(); // 줄바꿈
    delay(1000); 
}

```

<br>
<br>

### ✔ 내장 풀업 저항 사용

![image](https://user-images.githubusercontent.com/54584063/84108841-230d2a80-aa5c-11ea-9525-2737c85793f6.png)

```js
int button = 21; // 버튼 연결
void setup() {
    Serial.begin(9600); // 시리얼 통신 초기화
    pinMode(button, INPUT_PULLUP); // 버튼 연결 핀을 입력으로 설정
}
void loop() {
    Serial.println(digitalRead(button)); // 버튼 상태 출력 
    delay(1000);
}

```
<br>
<br>

### ✔ 버튼 누른 횟수 세기

- 버튼을 누른 상태로 있으면 횟수 계속 증가

- 이전 상태와 현재 상태 비교 -> 횟수 증가

- 바운싱(bouncing) : 기계적 진동에 의해 on/off 반복해서 나타남

    -> 디바운싱 (delay이용)

    ```js
    int pin_button = 15; // 버튼 연결 핀
    boolean state_previous = false; // 버튼의 이전 상태
    boolean state_current; // 버튼의 현재 상태
    int count = 0; // 버튼 누른 횟수
    void setup() { 
        Serial.begin(9600); // 시리얼 통신 초기화
        pinMode(pin_button, INPUT); // 버튼 연결 핀을 입력으로 설정
    }
    void loop() {
        state_current = digitalRead(pin_button); // 버튼 상태 읽기
        if (state_current) { // 버튼 누른 경우 
            if (state_previous == false) { // 이전 상태와 비교
                count++; // 상태가 바뀐 경우에만 횟수 증가 
                state_previous = true;
                Serial.println(count); }

                delay(50); // 디바운싱 
            } else { 
                state_previous = false; 
            } 
        }
    }

    ```

### 🔎 정리

- 데이터 핀을 통해 디지털 데이터 입력

    pinMode -> digitalRead

- 플로팅 상태 -> 풀업저항, 풀다운저항, 내장 풀업 저항


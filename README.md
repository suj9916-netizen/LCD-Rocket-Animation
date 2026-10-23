# LCD Rocket Animation

## 1. 프로젝트 개요

LCD 16x2의 첫 번째 줄, 첫 번째 글자 위치에 Custom Character를 출력하여  
Arduino가 동작 중임을 나타내는 애니메이션을 구현하였다.

캐릭터는 **로켓**을 사용하였으며,

1. 로켓 대기
2. 작은 불꽃 점화
3. 큰 불꽃 점화
4. 불꽃이 반복되며 발사 준비
5. 로켓이 위로 이동
6. 화면 밖으로 사라짐
7. 다시 처음부터 반복

하는 방식으로 동작한다.

이를 통해 Arduino가 정상적으로 동작 중임을 직관적으로 확인할 수 있다.

---

## 2. 아이디어

일반적인 로켓이 발사되는 모습을 표현하였다.

로켓의 엔진이 바로 점화되는 것이 아니라,

`작은 불꽃 → 큰 불꽃 → 작은 불꽃 → 큰 불꽃 → 발사`

순서로 동작하도록 하여 실제 로켓이 점화되는 것과 비슷한 느낌을 만들었다.

여러 개의 Custom Character를 연속적으로 출력하여 애니메이션처럼 보이도록 구현하였다.

---

## 3. LCD 연결

Arduino UNO와 I2C LCD를 사용하는 경우 다음과 같이 연결한다.

| LCD | Arduino UNO |
|---|---|
| GND | GND |
| VCC | 5V |
| SDA | A4 |
| SCL | A5 |

Arduino UNO에서 A4는 SDA, A5는 SCL의 역할을 한다.


<img width="733" height="419" alt="image" src="https://github.com/user-attachments/assets/bf81c7e9-5543-40d6-a470-57ddb7641f17" />


---

## 4. Arduino Code

```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

LiquidCrystal_I2C lcd(0x27, 16, 2);

// 로켓 대기
byte rocket0[] = {
  0x00,
  0x00,
  0x04,
  0x0E,
  0x1F,
  0x0E,
  0x04,
  0x00
};

// 작은 불꽃
byte rocket1[] = {
  0x00,
  0x04,
  0x0E,
  0x1F,
  0x0E,
  0x04,
  0x04,
  0x00
};

// 큰 불꽃
byte rocket2[] = {
  0x00,
  0x04,
  0x0E,
  0x1F,
  0x0E,
  0x04,
  0x0E,
  0x04
};

// 발사 시작
byte rocket3[] = {
  0x04,
  0x0E,
  0x1F,
  0x0E,
  0x04,
  0x0E,
  0x04,
  0x00
};

// 위로 이동
byte rocket4[] = {
  0x04,
  0x0E,
  0x1F,
  0x0E,
  0x04,
  0x04,
  0x00,
  0x00
};

// 더 위로 이동
byte rocket5[] = {
  0x04,
  0x0E,
  0x1F,
  0x0E,
  0x04,
  0x00,
  0x00,
  0x00
};

int motion[] = {0, 1, 2, 1, 2, 3, 4, 5};

int frame = 0;
unsigned long previous = 0;

void setup() {
  lcd.init();
  lcd.backlight();

  lcd.createChar(0, rocket0);
  lcd.createChar(1, rocket1);
  lcd.createChar(2, rocket2);
  lcd.createChar(3, rocket3);
  lcd.createChar(4, rocket4);
  lcd.createChar(5, rocket5);
}

void loop() {

  if (millis() - previous >= 400) {

    previous = millis();

    lcd.setCursor(0, 0);
    lcd.write(motion[frame]);

    frame++;

    if (frame >= 8) {

      delay(400);

      lcd.setCursor(0, 0);
      lcd.print(" ");

      delay(700);

      frame = 0;
    }
  }
}
```

---

## 5. 코드 설명

### Custom Character

LCD Custom Character는 하나의 문자를 5×8 픽셀 형태로 직접 정의할 수 있다.

예를 들어 다음 코드는 로켓의 모양을 정의한다.

```cpp
byte rocket0[] = {
  0x00,
  0x00,
  0x04,
  0x0E,
  0x1F,
  0x0E,
  0x04,
  0x00
};
```

각 값은 LCD의 한 행을 나타내며 총 8개의 행을 이용하여 하나의 캐릭터를 만든다.

---

### 애니메이션 순서

```cpp
int motion[] = {0, 1, 2, 1, 2, 3, 4, 5};
```

Custom Character를 위 순서대로 출력한다.

따라서 로켓의 동작은 다음과 같다.

```text
대기
 ↓
작은 불꽃
 ↓
큰 불꽃
 ↓
작은 불꽃
 ↓
큰 불꽃
 ↓
발사
 ↓
위로 이동
 ↓
화면 밖으로 이동
```

---

### 프레임 속도

```cpp
if (millis() - previous >= 400)
```

400ms마다 다음 Custom Character를 출력하도록 하였다.

따라서 너무 빠르게 캐릭터가 바뀌지 않고 로켓의 점화 및 이동 모습을 확인할 수 있다.

---

## 6. 결과


https://github.com/user-attachments/assets/652772cf-0ac8-45b4-823c-7dbb852853ed



LCD 첫 번째 줄 첫 번째 위치에서 로켓의 불꽃이 점화된 후 위쪽으로 발사되는 애니메이션이 반복된다.

이 애니메이션이 지속적으로 반복되므로 Arduino가 현재 정상적으로 동작하고 있음을 쉽게 확인할 수 있다.

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


`frame` 변수는 현재 출력할 애니메이션 프레임의 번호를 저장하고, `previous` 변수는 마지막으로 프레임을 변경한 시간을 저장한다.

`setup()`에서는 LCD를 초기화하고 백라이트를 켠 뒤, `createChar()`를 이용해 `rocket0`부터 `rocket5`까지 총 6개의 Custom Character를 LCD에 등록한다.

`loop()`에서는 `millis()`를 이용하여 마지막 프레임 출력 후 400ms가 지났는지 확인한다.

```cpp
if (millis() - previous >= 400)
```

400ms가 지나면 `previous`에 현재 시간을 저장하고,

```cpp
lcd.setCursor(0, 0);
lcd.write(motion[frame]);
```

LCD의 첫 번째 줄 첫 번째 칸에 `motion[]` 배열의 현재 프레임에 해당하는 로켓 캐릭터를 출력한다.

이후

```cpp
frame++;
```

을 통해 다음 프레임으로 이동한다.

모든 8개의 프레임이 출력되어 `frame` 값이 8이 되면 마지막 로켓 모습을 400ms 동안 유지한 뒤,

```cpp
lcd.print(" ");
```

공백을 출력하여 로켓을 화면에서 지운다. 이를 통해 로켓이 화면 밖으로 완전히 날아간 것처럼 표현하였다.

그 후 700ms 동안 빈 화면을 유지한 다음,

```cpp
frame = 0;
```

으로 설정하여 다시 첫 번째 프레임부터 로켓 발사 애니메이션을 반복한다.

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


## 7. 고찰

처음에는 로켓 모양만 만들면 쉽게 끝날 줄 알았는데, 한 칸 안에서 움직이는 것처럼 보이게 만드는 게 생각보다 어려웠다. 특히 로켓이 위로 올라갈 때 몸통이 잘려 보이는 문제가 있어서 여러 번 모양을 바꿔가며 수정했다.

또 처음에는 프레임 전환 속도가 너무 빨라서 불꽃이 켜지는 장면이나 로켓이 올라가는 모습이 거의 안 보였다. 그래서 시간을 조금씩 늘려보면서 눈으로 봤을 때 가장 자연스러운 속도로 조절했다.

불꽃도 한 번에 크게 켜지는 것보다 작은 불꽃과 큰 불꽃을 번갈아 보여주니까 실제로 점화되는 느낌이 더 잘 났다. 단순히 위치만 바꾸는 것보다 모양 자체를 조금씩 바꾸는 것도 자연스러운 움직임을 만드는 데 중요하다는 걸 알게 되었다.

이번 실습을 하면서 Custom Character는 크기가 작고 표현할 수 있는 범위도 제한적이지만, 여러 모양을 순서대로 바꾸면 생각보다 다양한 움직임을 만들 수 있다는 점이 재미있었다.

그리고 처음에는 `delay()`만 쓰면 되는 줄 알았는데, `millis()`를 이용하면 일정 시간마다 화면만 바꾸는 방식으로 만들 수 있다는 것도 알게 되었다. 나중에 센서나 다른 기능과 같이 사용할 때는 이런 방식이 더 유용할 것 같다.

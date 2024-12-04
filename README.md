# Эхолокатор.
Мой проект Эхо-локатора с подробной инструкцией по сборке. 

### Сборка.
> Для сборки станции понадобятся следующие компоненты:
1. Микроконтроллер (В моем проекте использовался Arduino UNO)
2. Провода-перемычки (очевидно, но на всякий)
3. Ультразвуковой дальномер (HC-04)
4. Сервопривод (SG90)
5. Корпус (здесь проявите ваш креатив, я собрал его из коробки, гвоздей, скотча и зубочисток)

## Подключение компонентов.

Схема подключения будет приложена ниже.

![Schematic](https://github.com/user-attachments/assets/899a5675-2cef-47da-aa22-21bd8cb46a9f)

## Код для Arduino IDE и Processing.
> Необходимое ПО.
1. Arduino IDE
2. Processing

### Код для Arduino IDE ![image](https://github.com/user-attachments/assets/daf8ae91-5a96-4ab3-8c98-e53255c62b72)

```
#include <Servo.h>

const int trigPin = 9;
const int echoPin = 10;

long duration;
int distinCM;

Servo radarServo;

void setup() 
{
  pinMode(trigPin, OUTPUT); 
  pinMode(echoPin, INPUT);
  Serial.begin(9600);
  radarServo.attach(6);
}
void loop() 
{
  for(int i=0;i<=180;i++)
  {
    radarServo.write(i);
    delay(1);
    
    digitalWrite(trigPin, LOW); 
    delayMicroseconds(1);
    digitalWrite(trigPin, HIGH); 
    delayMicroseconds(10);
    digitalWrite(trigPin, LOW);
    duration = pulseIn(echoPin, HIGH);
    distinCM = duration*0.034/2;
    
    Serial.print(i);
    Serial.print("*");
    Serial.print(distinCM);
    Serial.print("#");
  }
  
  for(int i=180;i>=0;i--)
  {
    radarServo.write(i);
    delay(1);
    digitalWrite(trigPin, LOW); 
    delayMicroseconds(1);
    digitalWrite(trigPin, HIGH); 
    delayMicroseconds(5);
    digitalWrite(trigPin, LOW);
    duration = pulseIn(echoPin, HIGH);
    distinCM = duration*0.034/2;
    
    Serial.print(i);
    Serial.print("*");
    Serial.print(distinCM);
    Serial.print("#");
  }
}
```

### Код для Procesing (Единственное что я делал в данной работе не сам)

```
import processing.serial.*;
import processing.opengl.*;

Serial port;
String serialAngle;
String serialDistance;
String serialData;
float objectDistance;
int radarAngle, radarDistance;
int index = 0;

void setup() {
  size(1280, 720);
  smooth();
  String portName = "COM1";  /// Ставьте здесь тот номер порта к которому подключен Arduino, если у Arduino COM10, то здесь пишите COM10.
  port = new Serial(this, portName, 9600);
  port.bufferUntil('#');
}

void draw() {
  noStroke();
  fill(0, 4);
  rect(0, 0, 1280, 720);
  
  fill(10, 255, 10);
  // Radar Arcs and Lines  
  pushMatrix();
  translate(640, 666);
  noFill();
  strokeWeight(2);
  stroke(10, 255, 10);
  
  // Radar arcs
  arc(0, 0, 1200, 1200, PI, TWO_PI);
  arc(0, 0, 934, 934, PI, TWO_PI);
  arc(0, 0, 666, 666, PI, TWO_PI);
  arc(0, 0, 400, 400, PI, TWO_PI);
  
  // Radar lines
  strokeWeight(4);
  line(-640, 0, 640, 0);
  line(0, 0, -554, -320);
  line(0, 0, -320, -554);
  line(0, 0, 0, -640);
  line(0, 0, 320, -554);
  line(0, 0, 554, -320);
  popMatrix();
  
  pushMatrix();
  strokeWeight(5);
  stroke(10, 255, 10);  
  translate(640, 666);
  
  objectDistance = constrain(radarDistance * 3.5, 0, 500); 
  line(0, 0, objectDistance * cos(radians(radarAngle)), -objectDistance * sin(radians(radarAngle)));  
  popMatrix();
  
  pushMatrix();
  translate(640, 666);
  strokeWeight(5);
  stroke(255, 10, 10); 
  if (radarDistance < 400) {
    line(objectDistance * cos(radians(radarAngle)),
         -objectDistance * sin(radians(radarAngle)),
         633 * cos(radians(radarAngle)),
         -633 * sin(radians(radarAngle)));
  }
  popMatrix();
}

void serialEvent(Serial port) {
  serialData = port.readStringUntil('#');  
  serialData = serialData.substring(0, serialData.length() - 1);
  index = serialData.indexOf("*");
  serialAngle = serialData.substring(0, index);
  serialDistance = serialData.substring(index + 1, serialData.length());
  radarAngle = int(serialAngle);  
  radarDistance = int(serialDistance);
}
```
## Принцип действия.
HC-04 закреплен на сервоприводе и замеряет дистанцию, а Arduino UNO записывает информацию о том,
на какой угол повёрнут HC-04 и какую дальность HC-04 замерил, потом происходит сопоставление
какая дальность соответствует углу на котором установлен HC-04 в данный момент, эти данные
идут в Processing и создаётся графическая визуализация в виде зеленого радара.

## Фото:

![els](https://github.com/user-attachments/assets/55bb87c8-aa27-450e-b0c7-816097f1a80d)

### Заключение.
Надеюсь с помощью данной статьи я смог интересно рассказать о данном проекте и понятно объяснить как
собрать такое же устройство.

Спасибо за прочтение!

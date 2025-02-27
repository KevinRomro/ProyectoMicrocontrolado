# ProyectoMICROCONTROLADO
Sistema Microcontrolado

Lista de Componentes del Brazo: 

Micro Servomotor Tower Pro (X2): Estan para hacer que la pinza del brazo robotico agarre y para controlar el paso de las bolas 
https://tienda.bricogeek.com/servomotores/1880-micro-servo-motor-tower-pro-mg90s.html

Servo estándar Hitec HS-311 (X2): Estan para que el brazo pueda rotar  
https://www.flashrc.com/es/servos/33545-servo-estandar-hitec-hs-311-43g-3-5kg-cm-0-15s-60.html

Sensor de Color RGB TCS34725: Manda una señal en cuanto detecta algo, esta señal siendo un color 
https://electronperdido.com/shop/sensores/luz/tcs34725-sensor-de-color-rgb/

Lampara redonda LED Mokungit-16L: Recibe la señal del sensor de color y se ilumina en base al color que le a mandado el sensor

Placa controladora Arduino modelo MEGA Keyestudio 2560:
https://abcescolar.pt/es/products/placa-controladora-arduino-mega-2560-r3-keyestudio?gad_source=1&gclid=CjwKCAiAmMC6BhA6EiwAdN5iLQ0SSRJHwu8YoRB8UORG6f0nJM6QfniVU8ojshFctLN0tvqdOmw-jxoCp4IQAvD_BwE

Controlador SERVO: Para controlar los motores 
Joystick KY-023


Codigo: [RETO·.txt](https://github.com/user-attachments/files/19014286/RETO.txt)
#include <Arduino.h>
#include <Wire.h>
#include <Adafruit_PWMServoDriver.h>
#include <Adafruit_TCS34725.h>
#include <SPI.h>
#include <ezButton.h>
#include <Adafruit_NeoPixel.h>

// Configuración PCA9685
Adafruit_PWMServoDriver pca9685 = Adafruit_PWMServoDriver(0x40);
#define SERVOMIN 150 // Pulso mínimo
#define SERVOMAX 600 // Pulso máximo

// Pines y configuración
#define BUZZER_PIN 2
#define NEOPIXEL_PIN 4
#define NUM_PIXELS 16
Adafruit_NeoPixel pixels(NUM_PIXELS, NEOPIXEL_PIN, NEO_GRB + NEO_KHZ800);

// Definición de los servos
#define BASE_SERVO 0
#define SHOULDER_SERVO 1
#define ELBOW_SERVO 2
#define GRIPPER_SERVO 3
#define GATE_SERVO 15

// Joystick
#define JOY_X_PIN 34
#define JOY_Y_PIN 35
#define JOY_SW_PIN 32

// Estados
bool gripperOpen = true; // Estado de la pinza
int ballCount = 8; // Número total de pelotas

// Declaración de funciones
void setServoAngle(uint8_t servo, int angle);
void controlJoystick();
void moveServo(uint8_t servo, int delta);
void emitSound();
void illuminateBlue();

void setup() {
  Serial.begin(115200);
  pca9685.begin();
  pca9685.setPWMFreq(50); // Frecuencia de 50 Hz para servomotores

  pinMode(BUZZER_PIN, OUTPUT);
  pinMode(JOY_SW_PIN, INPUT_PULLUP);

  pixels.begin();
  pixels.clear();
  pixels.show();

  // Inicialización de servos en posición neutra
  setServoAngle(BASE_SERVO, 90);
  setServoAngle(SHOULDER_SERVO, 90);
  setServoAngle(ELBOW_SERVO, 90);
  setServoAngle(GRIPPER_SERVO, 90);
  setServoAngle(GATE_SERVO, 90);
}

void loop() {
  controlJoystick();
}

// Definiciones de funciones
void controlJoystick() {
  int joyX = analogRead(JOY_X_PIN);
  int joyY = analogRead(JOY_Y_PIN);
  bool joyPressed = !digitalRead(JOY_SW_PIN);

  const int threshold = 500;

  // Control de dirección
  if (joyX < 2048 - threshold) { // Mover hacia la izquierda
    moveServo(BASE_SERVO, -5);
  } else if (joyX > 2048 + threshold) { // Mover hacia la derecha
    moveServo(BASE_SERVO, 5);
  }

  if (joyY < 2048 - threshold) { // Mover hacia abajo
    moveServo(SHOULDER_SERVO, 5);
    moveServo(ELBOW_SERVO, -5); // Ajustar codo para movimiento fluido
  } else if (joyY > 2048 + threshold) { // Mover hacia arriba
    moveServo(SHOULDER_SERVO, -5);
    moveServo(ELBOW_SERVO, 5); // Ajustar codo para movimiento fluido
  }

  // Control de la pinza (abrir/cerrar)
  if (joyPressed) {
    if (gripperOpen) {
      setServoAngle(GRIPPER_SERVO, 30); // Cerrar pinza
      gripperOpen = false;
      emitSound();
      illuminateBlue();
    } else {
      setServoAngle(GRIPPER_SERVO, 90); // Abrir pinza
      gripperOpen = true;
      emitSound();
      pixels.clear();
      pixels.show();
    }
    delay(500); // Evitar rebotes
  }
}

void moveServo(uint8_t servo, int delta) {
  static int angles[5] = {90, 90, 90, 90, 90}; // Ángulos actuales
  angles[servo] = constrain(angles[servo] + delta, 0, 180);
  setServoAngle(servo, angles[servo]);
}

void setServoAngle(uint8_t servo, int angle) {
  int pulse = map(angle, 0, 180, SERVOMIN, SERVOMAX);
  pca9685.setPWM(servo, 0, pulse);
}

void emitSound() {
  digitalWrite(BUZZER_PIN, HIGH);
  delay(200);
  digitalWrite(BUZZER_PIN, LOW);
}

void illuminateBlue() {
  for (int i = 0; i < NUM_PIXELS; i++) {
    pixels.setPixelColor(i, pixels.Color(0, 0, 255)); // Azul
  }
  pixels.show();
}




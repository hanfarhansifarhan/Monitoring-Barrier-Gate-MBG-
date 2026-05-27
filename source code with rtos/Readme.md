```
#include <Arduino.h>
#include <ESP32Servo.h>

// ==========================
// PIN CONFIGURATION
// ==========================
#define TRIG_PIN 5
#define ECHO_PIN 18
#define SERVO_PIN 19
#define LED_GREEN 2
#define LED_RED 4
#define BUZZER_PIN 15

// ==========================
// GLOBAL VARIABLE
// ==========================
Servo barrierServo;

float distanceCM = 0;
bool vehicleDetected = false;
bool gateOpen = false;

// ==========================
// FUNCTION ULTRASONIC
// ==========================
float readDistance()
{
    digitalWrite(TRIG_PIN, LOW);
    delayMicroseconds(2);

    digitalWrite(TRIG_PIN, HIGH);
    delayMicroseconds(10);

    digitalWrite(TRIG_PIN, LOW);

    long duration = pulseIn(ECHO_PIN, HIGH);

    float distance = duration * 0.034 / 2;

    return distance;
}

// ==========================
// TASK SENSOR
// ==========================
void TaskSensor(void *pvParameters)
{
    while (1)
    {
        distanceCM = readDistance();

        if (distanceCM < 10)
        {
            vehicleDetected = true;
        }
        else
        {
            vehicleDetected = false;
        }

        vTaskDelay(200 / portTICK_PERIOD_MS);
    }
}

// ==========================
// TASK BARRIER GATE
// ==========================
void TaskBarrier(void *pvParameters)
{
    while (1)
    {
        if (vehicleDetected)
        {
            barrierServo.write(90);

            digitalWrite(LED_GREEN, HIGH);
            digitalWrite(LED_RED, LOW);

            gateOpen = true;
        }
        else
        {
            barrierServo.write(0);

            digitalWrite(LED_GREEN, LOW);
            digitalWrite(LED_RED, HIGH);

            gateOpen = false;
        }

        vTaskDelay(300 / portTICK_PERIOD_MS);
    }
}

// ==========================
// TASK MONITORING
// ==========================
void TaskMonitoring(void *pvParameters)
{
    while (1)
    {
        Serial.println("========== MBG STATUS ==========");

        Serial.print("Distance : ");
        Serial.print(distanceCM);
        Serial.println(" cm");

        Serial.print("Vehicle : ");

        if (vehicleDetected)
        {
            Serial.println("DETECTED");
        }
        else
        {
            Serial.println("NOT DETECTED");
        }

        Serial.print("Barrier Gate : ");

        if (gateOpen)
        {
            Serial.println("OPEN");
        }
        else
        {
            Serial.println("CLOSED");
        }

        Serial.println("================================");
        Serial.println();

        vTaskDelay(1000 / portTICK_PERIOD_MS);
    }
}

// ==========================
// TASK ALARM
// ==========================
void TaskAlarm(void *pvParameters)
{
    while (1)
    {
        if (vehicleDetected)
        {
            tone(BUZZER_PIN, 1000);
            vTaskDelay(200 / portTICK_PERIOD_MS);

            noTone(BUZZER_PIN);
            vTaskDelay(200 / portTICK_PERIOD_MS);
        }
        else
        {
            noTone(BUZZER_PIN);
        }

        vTaskDelay(100 / portTICK_PERIOD_MS);
    }
}

// ==========================
// SETUP
// ==========================
void setup()
{
    Serial.begin(115200);

    pinMode(TRIG_PIN, OUTPUT);
    pinMode(ECHO_PIN, INPUT);

    pinMode(LED_GREEN, OUTPUT);
    pinMode(LED_RED, OUTPUT);

    pinMode(BUZZER_PIN, OUTPUT);

    barrierServo.attach(SERVO_PIN);

    barrierServo.write(0);

    // ==========================
    // CREATE TASK
    // ==========================
    xTaskCreate(
        TaskSensor,
        "Sensor Task",
        2048,
        NULL,
        1,
        NULL);

    xTaskCreate(
        TaskBarrier,
        "Barrier Task",
        2048,
        NULL,
        1,
        NULL);

    xTaskCreate(
        TaskMonitoring,
        "Monitoring Task",
        4096,
        NULL,
        1,
        NULL);

    xTaskCreate(
        TaskAlarm,
        "Alarm Task",
        2048,
        NULL,
        1,
        NULL);
}

// ==========================
```
// LOOP
// ==========================
void loop()
{
    // kosong karena menggunakan RTOS
}

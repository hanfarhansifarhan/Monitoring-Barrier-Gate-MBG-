```
#include <Arduino.h>

// ===============================
// PIN CONFIGURATION
// ===============================
#define TRIG_IN 5
#define ECHO_IN 18

#define TRIG_OUT 19
#define ECHO_OUT 21

#define LED_GREEN 2
#define LED_RED 4

#define BUZZER 15

// ===============================
// PARKING CONFIG
// ===============================
#define MAX_PARKING 10

// ===============================
// GLOBAL VARIABLE
// ===============================
int parkingCount = 0;

bool fullParking = false;

float distanceIn = 0;
float distanceOut = 0;

// ===============================
// READ ULTRASONIC FUNCTION
// ===============================
float readDistance(int trigPin, int echoPin)
{
    digitalWrite(trigPin, LOW);
    delayMicroseconds(2);

    digitalWrite(trigPin, HIGH);
    delayMicroseconds(10);

    digitalWrite(trigPin, LOW);

    long duration = pulseIn(echoPin, HIGH);

    float distance = duration * 0.034 / 2;

    return distance;
}

// ===============================
// TASK SENSOR MASUK
// ===============================
void TaskSensorIn(void *pvParameters)
{
    while (1)
    {
        distanceIn = readDistance(TRIG_IN, ECHO_IN);

        if (distanceIn < 8)
        {
            if (parkingCount < MAX_PARKING)
            {
                parkingCount++;

                Serial.println("KENDARAAN MASUK");

                vTaskDelay(2000 / portTICK_PERIOD_MS);
            }
        }

        vTaskDelay(200 / portTICK_PERIOD_MS);
    }
}

// ===============================
// TASK SENSOR KELUAR
// ===============================
void TaskSensorOut(void *pvParameters)
{
    while (1)
    {
        distanceOut = readDistance(TRIG_OUT, ECHO_OUT);

        if (distanceOut < 8)
        {
            if (parkingCount > 0)
            {
                parkingCount--;

                Serial.println("KENDARAAN KELUAR");

                vTaskDelay(2000 / portTICK_PERIOD_MS);
            }
        }

        vTaskDelay(200 / portTICK_PERIOD_MS);
    }
}

// ===============================
// TASK MONITORING
// ===============================
void TaskMonitoring(void *pvParameters)
{
    while (1)
    {
        int availableSlot = MAX_PARKING - parkingCount;

        Serial.println("===== MONITORING PARKIR =====");

        Serial.print("Jumlah Kendaraan : ");
        Serial.println(parkingCount);

        Serial.print("Slot Tersedia : ");
        Serial.println(availableSlot);

        if (parkingCount >= MAX_PARKING)
        {
            fullParking = true;
            Serial.println("STATUS : PARKIR PENUH");
        }
        else
        {
            fullParking = false;
            Serial.println("STATUS : SLOT TERSEDIA");
        }

        Serial.println("=============================");
        Serial.println();

        vTaskDelay(1000 / portTICK_PERIOD_MS);
    }
}

// ===============================
// TASK LED INDICATOR
// ===============================
void TaskIndicator(void *pvParameters)
{
    while (1)
    {
        if (fullParking)
        {
            digitalWrite(LED_RED, HIGH);
            digitalWrite(LED_GREEN, LOW);
        }
        else
        {
            digitalWrite(LED_RED, LOW);
            digitalWrite(LED_GREEN, HIGH);
        }

        vTaskDelay(100 / portTICK_PERIOD_MS);
    }
}

// ===============================
// TASK BUZZER
// ===============================
void TaskBuzzer(void *pvParameters)
{
    while (1)
    {
        if (fullParking)
        {
            tone(BUZZER, 1000);

            vTaskDelay(300 / portTICK_PERIOD_MS);

            noTone(BUZZER);

            vTaskDelay(300 / portTICK_PERIOD_MS);
        }
        else
        {
            noTone(BUZZER);
        }

        vTaskDelay(100 / portTICK_PERIOD_MS);
    }
}

// ===============================
// SETUP
// ===============================
void setup()
{
    Serial.begin(115200);

    pinMode(TRIG_IN, OUTPUT);
    pinMode(ECHO_IN, INPUT);

    pinMode(TRIG_OUT, OUTPUT);
    pinMode(ECHO_OUT, INPUT);

    pinMode(LED_GREEN, OUTPUT);
    pinMode(LED_RED, OUTPUT);

    pinMode(BUZZER, OUTPUT);

    // ===========================
    // CREATE TASK
    // ===========================
    xTaskCreate(
        TaskSensorIn,
        "Sensor In",
        2048,
        NULL,
        1,
        NULL);

    xTaskCreate(
        TaskSensorOut,
        "Sensor Out",
        2048,
        NULL,
        1,
        NULL);

    xTaskCreate(
        TaskMonitoring,
        "Monitoring",
        4096,
        NULL,
        1,
        NULL);

    xTaskCreate(
        TaskIndicator,
        "Indicator",
        2048,
        NULL,
        1,
        NULL);

    xTaskCreate(
        TaskBuzzer,
        "Buzzer",
        2048,
        NULL,
        1,
        NULL);
}

// ===============================
// LOOP
// ===============================
void loop()
{
    // kosong karena menggunakan RTOS
}
```

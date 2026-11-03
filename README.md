# Servo-control-based-on-infrared-distance-sensor
#include <Servo.h>

// Arduino pin assignment
#define PIN_IR    0      // IR sensor at A0
#define PIN_LED   9
#define PIN_SERVO 10

// Servo pulse widths
#define _DUTY_MIN 500    // 0°
#define _DUTY_NEU 1500   // 90°
#define _DUTY_MAX 2500   // 180°

#define _DIST_MIN 80.0
#define _DIST_MAX 330.0  // 250 mm = 25 cm

#define EMA_ALPHA 0.2
#define LOOP_INTERVAL 20 // ms

Servo myservo;

unsigned long last_loop_time = 0;

float dist_ema = _DIST_MIN;

void setup()
{
    pinMode(PIN_LED, OUTPUT);

    myservo.attach(PIN_SERVO);
    myservo.writeMicroseconds(_DUTY_NEU);

    Serial.begin(1000000);
}

void loop()
{
    unsigned long time_curr = millis();

    if (time_curr < last_loop_time + LOOP_INTERVAL)
        return;

    last_loop_time += LOOP_INTERVAL;

    int duty;
    float a_value;
    float dist_raw;

    // Read IR sensor
    a_value = analogRead(PIN_IR);

    // Convert sensor value to distance (mm)
    dist_raw = ((6762.0 / (a_value - 9.0)) - 4.0) * 10.0 - 60.0;
    Serial.print("IR=");
Serial.print(a_value);
Serial.print("  dist=");
Serial.println(dist_raw);

    // Range filter + LED
    if (dist_raw >= _DIST_MIN && dist_raw <= _DIST_MAX)
    {
        digitalWrite(PIN_LED, HIGH);

        // EMA filter
        dist_ema = EMA_ALPHA * dist_raw
                 + (1.0 - EMA_ALPHA) * dist_ema;
    }
    else
    {
        digitalWrite(PIN_LED, LOW);
    }

    // map() equivalent
    duty = _DUTY_MIN +
           (dist_ema - _DIST_MIN) *
           (_DUTY_MAX - _DUTY_MIN) /
           (_DIST_MAX - _DIST_MIN);

    // Limit duty range
    if (duty < _DUTY_MIN)
        duty = _DUTY_MIN;

    if (duty > _DUTY_MAX)
        duty = _DUTY_MAX;

    // Move servo
    myservo.writeMicroseconds(duty);

    // Serial Plotter output
    Serial.print("_DUTY_MIN:");
    Serial.print(_DUTY_MIN);

    Serial.print(",_DIST_MIN:");
    Serial.print(_DIST_MIN);

    Serial.print(",IR:");
    Serial.print(a_value);

    Serial.print(",dist_raw:");
    Serial.print(dist_raw);

    Serial.print(",ema:");
    Serial.print(dist_ema);

    Serial.print(",servo:");
    Serial.print(duty);

    Serial.print(",_DIST_MAX:");
    Serial.print(_DIST_MAX);

    Serial.print(",_DUTY_MAX:");
    Serial.print(_DUTY_MAX);

    Serial.println("");
}

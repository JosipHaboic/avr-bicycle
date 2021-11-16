# AVR Bicycle

## Features

- Speedometer
- Distance traveled
- Battery status
- Back and front direction signalization
- Back red light with multiple patterns to pick from
- Main front lamp control
- Runs on 16.4V (2x serial Li-Ion battery pack) battery managment system (BMS) included

## Requirements

- 1 digital pin to measure velocity and distance, pin will be used as counter
- 1 ADC pin for battery status
- 2 digital pins for front and back direction signalization light control
- 6 digital pins for the back red light control
- 1 digital pin for main front light control
- 1 digital pin for front light control
- 2 (SCL and SDA) pins for display (with Vcc and GND)
- 4 digital pins for buttons

Total: 17 digital pins and 1 ADC pin

## Calculations

### Speedometer

- Distance of sensor and magnet from center of front wheel is 10cm (100mm | 0.1m)
- Perimeter is then 2 π 0.1m =  0.628m
- Minimum velocity thant can be measured is 1km/h (1km/h / 3.6 = 0.277m/s)
- Maximum velocity thant can be measured is 178.5km/h (178.5km/h / 3.6 = 49.583m/s)
- Uses 8bit Timer/Counter2 to count the number of revolutions per second

|Velocity  |Period  |Frequency    |
|----------|--------|-------------|
|1km/h     |2.257s  |0.441Hz      |
|100km/h   |22.677ms|44.1Hz       |
|178.5km/h |8.226ms |121.5Hz      |

``` txt
v = (distance / dt)[m/s] * 3.6 [km/h]
```

we take average of all distances and divide by dt, dt will be 1 second, we multiply by 3.6 to get km/h

### Distance meter

- User is required to set wheel radius
- Uses 8bit Timer/Counter2 to count the number of revolutions per second and multiply it by perimeter of the wheel

#### Example with 16inch (R = 0.4064m) wheel, r = 0.4064m / 2 = 0.2032m

- Perimeter of wheel is 2 \* π * 0.2032m = 1.276m
- Each turn of wheel bicycle has moved 1.276m
- Wheel has turned 5 times in 1 second, so distance traveled that second is; 5 \* 1.276 = 6.38m

### Battery Status

- Maximum battery voltage = 16.8V
- Minimum battery voltage = 12.0V

VREF is ~ 5.0V so to get voltage at ADC to be 5V when battery is at maximum we need voltage divider  
with ratio of 5.0V / 16.8V = 0.297

``` txt
R1 = 23_600 Ω
R2 = 10_000 Ω
R2 / (R1 + R2) = 10kΩ / (10kΩ + 23.600kΩ) = 0.297

Beginners note:
There is no 23_600kΩ available so it will need to be a series of:
22kΩ + 1kΩ + 680Ω = 23.680kΩ
```

And so at minimum battery voltage we read at ADC value of 12.0V \* 0.297 = 3.564V  
Resolution of 8bit ADC is 5 / 255 = 19.607mV, so binary value is 3.564V \* 19.607mV = 181.771 or 181

|Battery Voltage|ADC value|Status                 |
|---------------|---------|-----------------------|
|>16.8V         |>181     |Over Charged           |
|16.8V          |181      |Full                   |
|12.0V          |107      |Discharged             |
|<12.0V         |<107     |Over Discharge         |

### Directional signalization

2 pins control 2 sets of yellow diodes, for left and right side

### Display

Displays velocity,distance traveled,battery status.

#### Menu

- Select velocity unit m/s or km/h
- Select wheel radius (16inch, 18inch, 19.5inch etc)
- Reset distance

### Buttons

- Button for front light on/off, long press shows/hides main menu
- Button for left turn signalization, toggle on press
- Button for right turn signalization, toggle on press
- Button for backlight, long press turns it on/off, short changes effects

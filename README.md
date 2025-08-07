# HomeSense - IoT Environmental Sensor 

ESP32-C3 based IoT Temperature, Humidity, Air Quality and Air Pressure Sensor with logging capabilities. 

![image](docs/HomeSense%20Render.png)

Initially, the goal was simply to monitor temperature and humidity, but I ended up choosing the BME680 sensor from Bosch, which additionally supports air pressure and air quality measurement.

I knew I primarily wanted to use MQTT to collect data from multiple of these sensor boards. However, I also wanted the ability to log data locally in case Wi-Fi wasn't available, so I also added a microSD card slot and a real-time clock.

I started considering the form factor and whether to use one of the pre-made ESP32-C3 modules. While that would have streamlined the design, I wanted the board to be as small as possible, so I decided against it in the end.

The case fits a 250mAh battery and also embeds a small magnet in the back lid to allow it to be mounted on magnetic surfaces. I put in a small light pipe for the RGB-LED, a few ventilation holes/slots for the BME680 and a cutout for the microSD card. Fully assembled it measures 15x25x35mm.

Take a look at the design documents for more info:
- [Schematic](docs/Schematic.pdf)
- [Assembly Drawing](docs/HomeSense%20Assembly%20Drawing.pdf)

# Data Acquisition 📊

Bosch supplies an API for the BME680 which can be used to get raw values from the sensor. This is good enough if you only want temperature and humidity for example. But the sensor is capable of quite a bit more, like providing indoor air quality levels. For this, the additional BSEC (Bosch Sensortec Environmental Cluster) software is needed. This library takes care of all the sensor fusion and provides ambient air temperature, ambient relative humidity, air pressure and indoor air quality (IAQ).

To configure the device, I made a super basic webpage, which is hosted by the ESP32. This allows setting up the Wi-Fi connection and MQTT. There is a lot to be added, but for initial testing this works fine.

Once everything is setup, a JSON string containing all the sensor data can be received every 15min by subscribing to the configured MQTT topic. 

Here is an example of the JSON string:

```json
{
   "timestamp":"2023-07-15T13:03",
   "temperature":{
      "value":27.33,
      "unit":"Celsius"
   },
   "humidity":{
      "value":38.87,
      "unit":"percent"
   },
   "pressure":{
      "value":94787.31,
      "unit":"Pa"
   },
   "gas_resistance":{
      "value":211406.42,
      "unit":"Ohm"
   },
   "iaq":79.60,
   "iaq_accuracy":3,
   "static_iaq":79.18,
   "static_iaq_accuracy":3,
   "co2_equivalent":{
      "value":791.80,
      "unit":"ppm"
   },
   "breath_voc_equivalent":{
      "value":0.97,
      "unit":"ppm"
   },
   "gas_percentage":{
      "value":21.45,
      "unit":"percent"
   },
   "run_in_status":1,
   "stabilization_status":1,
   "battery_voltage":{
      "value":3868,
      "unit":"mV"
   }
}
```

# Issues

Unfortunately the thermal isolation cutouts around the BME680 aren't as effective as I had hoped. When the ESP32 is running continuously, like when the webserver for configuration is running, the temperature reading of the BME680 increases by around 3°C. Ofc the small footprint of the board also doesn't help, since there is very little distance between the ESP and the sensor.

Even though the cutouts seem to help a bit and the ground plane was removed below the sensor area, the bridge for the power and data lines provides a direct path for the heat. A complete U-shape would have probably been a much more effective design choice.

In order to avoid reporting false temperature and humidity data, the ESP has to be in sleep mode for some time to allow the board to cool down before taking another reading from the sensor.

This also applies while charging the battery. Currently the charge current is configured for 50mA, but this is still enough to increase the temperature readings by a small amount.

The workaround for now is to wait around 10min after the webserver is shut off, before taking the first reading and also disable any readings while charging.
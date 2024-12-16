# RaspberryPi-WeatherStation

import board
import digitalio
import adafruit_dht
import time
import matplotlib.pyplot as plt

# Set up GPIO pins
RAIN_SENSOR_PIN = board.D15 # GPIO pin for the rain sensor (water sensor) - Pin 4
TEMP_SENSOR_PIN = board.D17  # GPIO pin for the DHT11 temperature and humidity sensor - Pin 17

# Set up rain sensor (digital input)
rain_sensor = digitalio.DigitalInOut(RAIN_SENSOR_PIN)
rain_sensor.switch_to_input(pull=digitalio.Pull.UP)  # Assuming rain sensor pulls low when it detects rain

# Initialize DHT sensor
dht_device = adafruit_dht.DHT11(TEMP_SENSOR_PIN)

# Initialize lists for plotting
temp_data = []
rain_data = []
time_data = []

# Set up the plot
plt.ion()  # Enable interactive mode for live plotting

while True:
    # Read rain sensor data
    rain_detected = not rain_sensor.value  # Assuming the sensor pulls low when rain is detected
    if rain_detected:
        rain_status = "Rain detected!"
    else:
        rain_status = "No rain detected"

    # Read temperature and humidity data from DHT sensor
    try:
        temperature = dht_device.temperature
        humidity = dht_device.humidity
    except RuntimeError as e:
        # Handle DHT sensor read failure (may need retry)
        print(f"Error reading from DHT sensor: {e}")
        temperature = humidity = None

    # Get current timestamp
    current_time = time.strftime("%H:%M:%S")

    # Print real-time data to the console
    if temperature is not None and humidity is not None:
        print(f"{current_time} | Temperature: {temperature:.1f}\u00b0C | Humidity: {humidity:.1f}% | {rain_status}")
    else:
        print(f"{current_time} | Error: Could not read sensor data | {rain_status}")

    # Append data to lists for plotting
    if temperature is not None and humidity is not None:
        temp_data.append(temperature)
        rain_data.append(rain_detected)
        time_data.append(current_time)

    # Plot data every 10 readings
    if len(temp_data) % 10 == 0:
        # Clear the previous plot
        plt.clf()

        # Plot Temperature data
        plt.subplot(2, 1, 1)
        plt.plot(time_data, temp_data, label="Temperature", color='b')
        plt.title("Temperature and Rain Detection")
        plt.xlabel("Time")
        plt.ylabel(f"Temperature (\u00b0C)")  # Using Unicode for degree symbol
        plt.xticks(rotation=45)
        plt.legend()

        # Plot Rain Detection data
        plt.subplot(2, 1, 2)
        plt.plot(time_data, rain_data, label="Rain Detection", color='r')
        plt.xlabel("Time")
        plt.ylabel("Rain Detected")
        plt.xticks(rotation=45)
        plt.legend()

        # Display the plot
        plt.tight_layout()
        plt.pause(0.01)

    # Delay between readings
    time.sleep(2)

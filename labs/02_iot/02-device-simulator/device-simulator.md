# Connect PI Simulator to IoT Hub

![IoT Hub_](./media/pi_simulator.png)

In this lab you will:

* Learn to create a device using Azure Portal

* Connect the simulator to IoT Hub

* Send telemetry data to Azure

## Task 1: Create a Device

Go To your IoT Hub in the portal and click on **IoT Devices**

![Resource Group_](./media/iot_devices.png)

Click on **+ Add** and enter a **Device ID** and click **Save**.

![Resource Group_](./media/add_device.png)

Click on the device and copy the primary key connection string. 

![Resource Group_](./media/connection-string.png)

## Task 2: Configure the Pi Simulator

Please go to the [PI Simulator_](https://azure-samples.github.io/raspberry-pi-web-simulator/#GetStarted).

Replace the connection string with the primary key connection string copied in the previous steps:

![Resource Group_](./media/pi_connection_string_before.png)

After you copy the connection string should look like below

![Resource Group_](./media/pi_connection_string_after.png)

Click Run and start sending messages. LED will start blinking

![Resource Group_](./media/pi_message.png)

Messages will start flowing into IoT Hub

![Resource Group_](./media/iothub_messages.png)

> You will work with Labs in the Next Module to Visualize the Data flowing into IoT Hub
# Attacking the STM32F030F4P6 microcontroller attached to the ChipWhisperer Nano
I will be using a Jupyter Notebook VM to execute this SPA attack on. The ChipWhisperer Nano provides a helpful utility for this function, as I do not have access to an oscilloscope. Despite its lower sample rate of 20MS/s, it will suffice for this simple attack, as we will underclock the MCU to 8MHz, allowing us to capture 2.5 samples per cycle. 


**Setup**



First, setup the ChipWhisperer and the target, ensure they are connected and configured properly. Then we can move on to building and loading the firmware.

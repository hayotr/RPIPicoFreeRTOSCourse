1. Purpose of the Code

This program demonstrates how FreeRTOS SMP runs multiple tasks in parallel on both cores of the Raspberry Pi Pico.
The code shows:

How to create FreeRTOS tasks

How to bind tasks to a specific CPU core

How tasks interact to control LEDs and a counter

How to monitor task and memory usage at runtime

2. What the Code Does (Step by Step)
2.1 System Startup

Initializes USB serial communication

Detects whether FreeRTOS SMP is enabled

Starts the FreeRTOS scheduler

2.2 Task Creation

The program creates one main FreeRTOS task:

MainTask

Responsible for starting all other agents

Runs continuously after the scheduler starts

2.3 Agents Used

The code uses two helper classes:

BlinkAgent

Blinks an LED on a specific GPIO pin

Runs as its own FreeRTOS task

CounterAgent

Displays a 4-bit value using GPIO pins 2–5

Runs as its own FreeRTOS task

Receives values from the main task

3. Core Assignment (Important Part)

The code explicitly assigns tasks to CPU cores:

Task	                    Core
BlinkAgent (GPIO 0)	      Core 0
BlinkAgent (GPIO 15)	    Core 1
CounterAgent	            Core 1
MainTask	                Core 0


4. Runtime Behavior

Once running:

LED on GPIO 0 blinks on Core 0

LED on GPIO 15 blinks on Core 1

Counter outputs values on GPIO 2–5

Every 3 seconds:

MainTask generates a random 4-bit number

Sends it to the CounterAgent

Task and heap statistics are printed to the serial console

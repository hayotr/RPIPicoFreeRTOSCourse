FreeRTOS LED Control Demo: Binary Random Blinking

This program demonstrates using FreeRTOS on the Raspberry Pi Pico to blink 4 LEDs in a random binary pattern. The program utilizes three agents (BlinkAgent, CounterAgent, and DecoderAgent) to manage the LED blinking, counting, and decoding tasks, respectively. The task runs in a loop and generates random 4-bit values, then prints the values in JSON format.

Features

FreeRTOS-based Task Management: Uses multiple tasks managed by FreeRTOS.

LED Blinking: LEDs blink according to random binary values.

JSON Output: Random values are encoded as JSON strings and printed to the console.

Real-time Task and Heap Statistics: Monitors the system's memory usage and task statuses.

Hardware Requirements

Raspberry Pi Pico.

4 LEDs connected to GPIO pins 0–7.

GPIO 0 for the main LED.

GPIO 1-7 for additional LEDs (used by CounterAgent).

USB cable for serial communication.

Pin Definitions

The following GPIO pins are used for LED control:

#define LED_PAD     0  // Main LED (blinks based on random value)
#define LED1_PAD    1  // LED 1
#define LED2_PAD    2  // LED 2
#define LED3_PAD    3  // LED 3
#define LED4_PAD    4  // LED 4
#define LED5_PAD    5  // LED 5
#define LED6_PAD    6  // LED 6
#define LED7_PAD    7  // LED 7

Code Breakdown
1. runTimeStats() Function

This function collects and prints runtime statistics about the tasks and heap usage, including the current task list, priority, stack usage, and memory statistics.

void runTimeStats() {
    // Print task status and heap usage
    ...
}


It is called every 3 seconds to monitor FreeRTOS tasks and memory usage.

2. mainTask() Function

The mainTask() function is the main FreeRTOS task that:

Controls the LED blinking: Uses the BlinkAgent for a simple LED blink.

Generates random values: A random 4-bit value is generated every 3 seconds.

Encodes values in JSON format: These values are printed as JSON strings and sent to the DecoderAgent.

Starts other agents: The CounterAgent handles controlling multiple LEDs to display a binary count based on the random values.

void mainTask(void *params) {
    char line[80];
    BlinkAgent blink(LED_PAD);
    CounterAgent counter(LED1_PAD, LED2_PAD, LED3_PAD, LED4_PAD, LED5_PAD, LED6_PAD, LED7_PAD);
    DecoderAgent decoder(&counter);

    printf("Main task started\n");

    blink.start("Blink", TASK_PRIORITY);
    counter.start("Counter", TASK_PRIORITY);
    decoder.start("Decode", TASK_PRIORITY);

    while (true) {  // Infinite loop
        runTimeStats();  // Print task and memory stats
        uint8_t r = rand() & 0x0F;  // Generate random 4-bit value

        sprintf(line, "{\"count\": %d}\r\n", r);  // JSON format
        printf("Providing Json %s\n", line);
        decoder.add(line);  // Add JSON string to decoder

        vTaskDelay(3000);  // 3-second delay
    }
}

3. Agents:

BlinkAgent: Controls LED blinking on the specified GPIO pin.

CounterAgent: Controls multiple LEDs to display the binary representation of values.

DecoderAgent: Takes the JSON string and decodes it. In this case, it processes the random values and communicates them to the CounterAgent.

4. Task Launching and Scheduler

The vLaunch() function creates and starts the main task (mainTask) and initializes the FreeRTOS scheduler:

void vLaunch(void) {
    TaskHandle_t task;
    xTaskCreate(mainTask, "MainThread", 500, NULL, TASK_PRIORITY, &task);
    vTaskStartScheduler();  // Start FreeRTOS scheduler
}

5. main() Function

This function sets up serial communication, waits for 2 seconds to allow USB initialization, and starts the FreeRTOS tasks:

int main(void) {
    stdio_init_all();  // Initialize serial communication
    sleep_ms(2000);    // Wait for USB initialization
    printf("GO\n");

    // Start tasks and scheduler
    vLaunch();

    return 0;
}

6. Random Number Generation

The random number (r) is generated using rand() & 0x0F to ensure a 4-bit value (0–15). This value is used to control the counter LEDs.

Real-time Statistics Output

The program periodically prints the task and heap usage stats every 3 seconds:

Number of tasks 3
Task: 1   cPri:1   bPri:1   hw:100   MainThread
Task: 2   cPri:1   bPri:1   hw:90    Blink
Task: 3   cPri:1   bPri:1   hw:80    Counter
HEAP avl: 12345, blocks 3, alloc: 12, free: 6


This provides valuable insights into task behavior, stack usage, and memory health.

JSON Output Example

The main task generates a random value and formats it as a JSON string:

{"count": 5}
{"count": 8}
{"count": 2}


This output is printed to the serial console and sent to the DecoderAgent for further processing.

Customization and Configuration
Changing LED Pins

You can modify the LED_PAD, LED1_PAD, and other LEDx_PAD constants to use different GPIO pins for the LEDs.

Changing Random Value Range

The rand() & 0x0F line generates a 4-bit random value. You can change 0x0F to any other value to modify the range of the random numbers.

Task Priorities

The TASK_PRIORITY constant can be adjusted to change the priority of the tasks in the FreeRTOS scheduler.

How to Run

Upload the code to your Raspberry Pi Pico.

Connect the LEDs to the GPIO pins specified in the code (e.g., GPIO 0 for the main LED, GPIO 1–7 for the counter LEDs).

Monitor the serial output for JSON data and task statistics.

Observe the LEDs: The LEDs will blink according to the random values generated.

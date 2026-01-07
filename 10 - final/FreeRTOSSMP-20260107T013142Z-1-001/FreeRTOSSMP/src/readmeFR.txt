FreeRTOS SMP Demo on Raspberry Pi Pico

This demo program uses FreeRTOS SMP (Symmetric Multi-Processing) to control LED blink behavior on Core 0 and Core 1 of the Raspberry Pi Pico.

Core 0 is responsible for blinking an LED connected to GPIO 0 and controlling the counter task.

Core 1 is responsible for blinking an LED on GPIO 15 and managing a counter that shows values on GPIO 2 to 5.

This demo demonstrates FreeRTOS running on both cores with specific tasks bound to each core.

Features

SMP (Symmetric Multi-Processing): Demonstrates the use of both CPU cores of the Raspberry Pi Pico.

LED Blinking: Core 0 blinks an LED on GPIO 0, and Core 1 blinks another LED on GPIO 15.

Counter: Displays a counter on GPIO 2 to GPIO 5 managed by Core 1.

Task Binding: Tasks are explicitly bound to specific cores using vTaskCoreAffinitySet.

Hardware Requirements

Raspberry Pi Pico.

2 LEDs connected to GPIO 0 (Core 0) and GPIO 15 (Core 1).

4 LEDs connected to GPIO 2 to 5 to show the counter value.

USB cable for serial communication.

Code Explanation
1. LED Pin Definitions

The LEDs are connected to the following GPIO pins on the Raspberry Pi Pico:

#define LED_PAD     0  // LED on Core 0 (GPIO 0)
#define LED1_PAD    1  // LED 1
#define LED2_PAD    2  // LED 2 (used for Counter display)
#define LED3_PAD    3  // LED 3 (used for Counter display)
#define LED4_PAD    4  // LED 4 (used for Counter display)
#define LED5_PAD    5  // LED 5 (GPIO 15, used for Core 1 blinking)
#define LED6_PAD    6  // LED 6
#define LED7_PAD    7  // LED 7

2. runTimeStats() Function

This function displays real-time statistics for the system, including task status, memory usage, and heap statistics. It helps monitor FreeRTOS performance during runtime.

void runTimeStats() {
    // Retrieve task status and print information
    ...
    // Print heap statistics
    HeapStats_t heapStats;
    vPortGetHeapStats(&heapStats);
    printf("HEAP avl: %d, blocks %d, alloc: %d, free: %d\n", ...);
}

3. mainTask() Function

The main task is responsible for:

Blinking LEDs on Core 0 and Core 1.

Sending instructions to the counter task to update the LED display on GPIO 2-5.

Binding tasks to specific CPU cores using vTaskCoreAffinitySet.

void mainTask(void *params) {
    BlinkAgent blink(LED_PAD);     // LED on Core 0
    BlinkAgent blink1(LED5_PAD);   // LED on Core 1
    CounterAgent counter(LED1_PAD, LED2_PAD, LED3_PAD, LED4_PAD, LED6_PAD, LED7_PAD); // Counter on Core 1

    printf("Main task started\n");

    // Bind tasks to specific cores
    UBaseType_t coreMask = 0x2;
    vTaskCoreAffinitySet(blink1.getTask(), coreMask);  // Bind blink1 to Core 1
    vTaskCoreAffinitySet(counter.getTask(), coreMask); // Bind counter to Core 1

    coreMask = 0x1;
    vTaskCoreAffinitySet(blink.getTask(), coreMask);  // Bind blink to Core 0

    while (true) { 
        runTimeStats();  // Print runtime stats
        printf("Main on Core %ld\n", sio_hw->cpuid);

        uint8_t r = rand() & 0xF;  // Generate random value
        counter.on(r);  // Update counter LEDs
        vTaskDelay(3000);  // Wait 3 seconds
    }
}

4. vLaunch() Function

This function creates the main task and starts the FreeRTOS scheduler to begin execution of tasks.

void vLaunch(void) {
    TaskHandle_t task;
    xTaskCreate(mainTask, "MainThread", 500, NULL, TASK_PRIORITY, &task);  // Create main task
    vTaskStartScheduler();  // Start the FreeRTOS scheduler
}

5. main() Function

The main() function sets up the serial communication and initializes FreeRTOS. It then launches the tasks on both cores.

int main(void) {
    stdio_init_all();  // Initialize serial communication
    sleep_ms(2000);    // Wait for 2 seconds for USB initialization
    printf("GO\n");

    // Print RTOS and core information
    printf("Starting FreeRTOS on both cores:\n");

    vLaunch();  // Start tasks and scheduler
    return 0;
}

Real-Time Statistics

Every 3 seconds, the program prints statistics on the system's tasks and memory:

Number of tasks 3
Task: 1   cPri:1   bPri:1   hw:100   MainThread
Task: 2   cPri:1   bPri:1   hw:90    Blink 0
Task: 3   cPri:1   bPri:1   hw:80    Counter
HEAP avl: 12345, blocks 3, alloc: 12, free: 6


This provides information about task priorities, stack usage, and memory usage.

SMP (Symmetric Multi-Processing)

The demo runs FreeRTOS on both Core 0 and Core 1 of the Raspberry Pi Pico:

Core 0: Handles the main task and blinks the LED on GPIO 0.

Core 1: Handles the counter task and blinks the LED on GPIO 15.

The tasks are explicitly bound to their respective cores using vTaskCoreAffinitySet.

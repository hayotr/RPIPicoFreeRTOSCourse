Demo to Control 4 LEDs on Raspberry Pi Pico with FreeRTOS

This demo program uses FreeRTOS to control 4 LEDs on a Raspberry Pi Pico. The LEDs' behavior is simulated using a randomly generated binary value every second. The program makes the LEDs blink based on this binary value and prints task and memory statistics in real time.

Features

Blinking LEDs based on random binary values.

Monitoring tasks and memory usage through real-time statistics.

Using FreeRTOS for task management and task priority.

Hardware Requirements

Raspberry Pi Pico.

4 LEDs connected to the GPIO pins defined in the code (by default: LED_PAD, LED1_PAD, etc.).

Code Explanation
1. LED Pin Definitions

The LEDs are connected to the following pins on the Raspberry Pi Pico:

#define LED_PAD     0  // Main LED (blinks based on random value)
#define LED1_PAD    1  // LED 1
#define LED2_PAD    2  // LED 2
#define LED3_PAD    3  // LED 3
#define LED4_PAD    4  // LED 4
#define LED5_PAD    5  // LED 5
#define LED6_PAD    6  // LED 6
#define LED7_PAD    7  // LED 7

2. runTimeStats() Function

This function prints information about the system state, such as memory usage, task status, and heap memory usage. It helps monitor task execution and memory management during the program's run.

void runTimeStats(){
    // Retrieve task status information
    ...
    // Print task information
    for( x = 0; x < uxArraySize; x++ )
    {
        printf("Task: %d \t cPri:%d \t bPri:%d \t hw:%d \t%s\n", ...);
    }
    // Print heap statistics
    HeapStats_t heapStats;
    vPortGetHeapStats(&heapStats);
    printf("HEAP avl: %d, blocks %d, alloc: %d, free: %d\n", ...);
}


This function is called every 3 seconds to print statistics about running tasks and available memory.

3. mainTask() Function

This main task handles the LED blinking based on random binary values. It runs an infinite loop that:

Prints runtime statistics.

Generates a 4-bit random number (rand() & 0x0F), and uses it to control the LEDs.

Pauses the program for 3 seconds (vTaskDelay(3000)).

void mainTask(void *params){
    BlinkAgent blink(LED_PAD);
    CounterAgent counter(LED1_PAD, LED2_PAD, LED3_PAD, LED4_PAD, LED5_PAD, LED6_PAD, LED7_PAD);

    printf("Main task started\n");

    blink.start("Blink", TASK_PRIORITY);
    counter.start("Counter", TASK_PRIORITY);

    while (true) { 
        runTimeStats();
        uint8_t r = rand() & 0x0F;  // 4-bit random value
        counter.blink(r);  // Blink LEDs based on the random value
        printf("Blinking R=0x%X\n", r);
        vTaskDelay(3000);  // 3-second delay
    }
}

4. vLaunch() Function

This function creates the main task mainTask and starts the FreeRTOS scheduler to execute the tasks.

void vLaunch(void) {
    TaskHandle_t task;
    xTaskCreate(mainTask, "MainThread", 500, NULL, TASK_PRIORITY, &task); // Create the main task
    vTaskStartScheduler(); // Start the FreeRTOS scheduler
}

5. main() Function

The main() function initializes serial communication, waits a few seconds for stabilization, and then starts the system execution.

int main(void) {
    stdio_init_all();  // Initialize serial communication
    sleep_ms(2000);    // Wait for 2 seconds
    printf("GO\n");

    vLaunch();  // Start tasks and the scheduler
    return 0;
}

Program Execution
Steps to Start:

Upload the code: Upload the code to your Raspberry Pi Pico.

Connect the LEDs: Ensure that the LEDs are correctly connected to the defined GPIO pins.

Observe the Behavior: The program will make the LEDs blink based on random binary values and print runtime statistics every 3 seconds.

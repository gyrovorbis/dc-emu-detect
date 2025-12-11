```c
/* emu_detect.c
   Simple Sega Dreamcast Emulator Detection Routine for KOS-based Apps.
   Copyright (C) 2025 Falco Girgis

   This little bastard simply leverages the fact that no emulator that I am
   aware of is emulating the SH4's watchdog timer peripheral, as no commercial
   game or SDK ever made use of it... UNTIL NOW. 
*/

#include <stdbool.h>
#include <stdatomic.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

// Include my KOS WDT driver, which nobody is emulating, lulz.
#include <dc/wdt.h>

/* Callback function that gets called upon WDT timeouts.

   Simply setting an atomic boolean to true, to signify that the WDT
   peripheral fired its timeout interrrupt properly.
*/
static void wdt_timeout(void *user_data) {
    atomic_bool *fired = (atomic_bool *)user_data;
    *fired = true;
}

/* Returns true if the program is running on an emulator,
   returns false if it's running on real HW. */
bool detect_emulator(void) {
    // Initialize our flag for whether the WDT has fired to false.
    atomic_bool fired = false;
    /* Enable the WDT in interval timer mode, with an initial counter valuoe of 0,
       counting up to 1 millisecond before calling our callback routine with a maximum
       IRQ priority value of 15, passing our flag to toggle to denote that it was called. */
    wdt_enable_timer(0, 1, 15, wdt_timeout, &fired); 

    // Wait for 2 ms before checking whether the WDT IRQ has fired (plenty of fuckin' time!)
    usleep(2000);

    // If it never fired, we're on an emulator...
    return !fired;
}

// Program entry-point.
int main(int argc, const char* argv[]) {
    // Give her a shot!
    if(detect_emulator())
        printf("I see you're running on an emulator!\n");
    else
        printf("I see you're running on actual Dreamcast HW!\n");

    return EXIT_SUCCESS;
}
```

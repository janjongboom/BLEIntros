#Event-Driven Programming

mbed programming is event-driven: it responds to interrupts coming from the hardware, generated by changes in electrical signals or system activity (such as radio communication). Interrupts often lead to the execution of event handlers by the OS. In the context of BLE, event handlers may be triggered quite regularly, for example if a sensor sends a measurement every x seconds, or they may be triggered intermittently.

Event handlers are often preemptive, meaning they can interrupt the main program’s execution to force their own execution; the main program will resume when the interrupting event is fully handled. In the case of BLE, we expect the main program to be a sleep loop (``waitForEvent``), which means that the device will sleep unless it receives an interrupt - which is why BLE is a low energy technology.

The relationship between ``main()`` and event handlers - and especially the decision which code to move to an event handler and which to leave in ``main()`` - is all about timing. Handler execution time is often determined not by the size of the code but by how many times it must run - for example, how many iterations of a data-processing loop it performs - or by communication with external components such as sensors (also called polling). Communication delays can range from a few microseconds to milliseconds, depending on the sensor involved. Reading an accelerometer can take around a millisecond, and a temperature sensor can take up to a few hundred microseconds. A barometer, on the other hand, can take up to 10 milliseconds to yield a new sensor value. 

If an event, such as a sensor reading, arrives when the program is in ``main()`` (in our case, then, when the device is sleeping), it can be executed. But if it arrives when another event is being executed, it may have to wait for the first event to be handled in full. In this scenario, the first event is blocking the execution of the second event. Because event handlers can block each other, they are supposed to execute quickly and return control to ``main()``, to allow the system to remain responsive. In the world of microcontrollers, anything longer than a few dozen microseconds is too long and a millisecond is an eternity. Long-running activities - anything longer than 100 microseconds, such as data processing and sensor communication - should be left in ``main()`` rather than an event handler. This is because ``main()`` can then be interrupted by event handlers, so that the long-running process doesn’t affect the system’s responsiveness. 

In these cases, the event handler is used not to perform functions but rather to enqueue work for the main loop. In the heart rate demo, the work of polling for heart rate data is communicated to the main loop through the variable ``triggerSensorPolling``, which gets set periodically from an event handler called ``periodicCallback()``.
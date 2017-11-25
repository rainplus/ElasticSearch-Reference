## Heap size check

If a JVM is started with unequal initial and max heap size, it can be prone to pauses as the JVM heap is resized during system usage. To avoid these resize pauses, it’s best to start the JVM with the initial heap size equal to the maximum heap size. Additionally, if [`bootstrap.memory_lock`](important-settings.html#bootstrap.memory_lock "bootstrap.memory_lockedit") is enabled, the JVM will lock the initial size of the heap on startup. If the initial heap size is not equal to the maximum heap size, after a resize it will not be the case that all of the JVM heap is locked in memory. To pass the heap size check, you must configure the [heap size](heap-size.html "Set JVM heap size via jvm.options").
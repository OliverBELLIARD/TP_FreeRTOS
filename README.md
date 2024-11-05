# TP_FreeRTOS

## Enable FreeRTOS

As a solution for the message:
```
When RTOS is used, it is strongly recommended to use a HAL timebase source other than the Systick. The HAL timebase source can be changed from the Pinout tab under SYS
```
When enabling FreeRTOS, the SYS Time base source has been changed to
use a timer to use the HAL time base. More info in this [st community forum](https://community.st.com/t5/stm32-mcus-products/cubemx-freertos-gt-using-systick/td-p/412607).

As a solution for the message:
```
The USE_NEWLIB_REENTRANT must be set in order to make sure that newlib is fully reentrant. The option will increase the RAM usage. Enable this option under FreeRTOS > Advanced Settings > USE_NEWLIB_REENTRANT
```
When enabling FreeRTOS, we simply follow the detailed instructions.
However this change can arrise sume bugs tied to the software, latest issues were reported in this [st community forum](https://community.st.com/t5/stm32cubemx-mcus/cubeide-1-9-0-freertos-advanced-settings-gt-use-newlib-reentrant/td-p/127513).
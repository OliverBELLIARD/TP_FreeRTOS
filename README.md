# TP_FreeRTOS

## Activation de FreeRTOS

Lors de l'activation de FreeRTOS pour la première fois nous avons un message contenant 2 avertissements. Pour résoudre le premier message ci-dessous :
```
When RTOS is used, it is strongly recommended to use a HAL timebase source other than the Systick. The HAL timebase source can be changed from the Pinout tab under SYS
```
la source de base temporelle SYS doit être modifiée pour utiliser un timer comme base temporelle HAL. Plus d'infos dans ce [forum de la communauté ST](https://community.st.com/t5/stm32-mcus-products/cubemx-freertos-gt-using-systick/td-p/412607).

Ensuite, pour le deuxième message ci-dessous :
```
The USE_NEWLIB_REENTRANT must be set in order to make sure that newlib is fully reentrant. The option will increase the RAM usage. Enable this option under FreeRTOS > Advanced Settings > USE_NEWLIB_REENTRANT
```
When enabling FreeRTOS, Nous suivons simplement les instructions détaillées. Cependant, ce changement peut entraîner certains bugs liés au logiciel, les derniers problèmes ont été signalés dans ce [forum de la communauté ST](https://community.st.com/t5/stm32cubemx-mcus/cubeide-1-9-0-freertos-advanced-settings-gt-use-newlib-reentrant/td-p/127513).

# 1 FreeRTOS, tâches et sémaphores
## 1.1 Tâche simple

### Importance de `TOTAL_HEAP_SIZE`

1. Le paramètre **`TOTAL_HEAP_SIZE`** est crucial dans FreeRTOS car il détermine la quantité de mémoire disponible pour l'allocation dynamique au sein du système. Ce paramètre contrôle la taille totale de la mémoire (en octets) que FreeRTOS peut utiliser pour créer des objets dynamiques comme les tâches, les files d'attente, les sémaphores, et les timers.

1. **Allocation de Mémoire pour les Objets FreeRTOS** : FreeRTOS utilise une mémoire dite « heap » pour allouer dynamiquement l'espace mémoire nécessaire pour les objets du système d'exploitation. Lorsque l’on crée une tâche ou une file d’attente, la mémoire pour cette tâche est extraite de la mémoire définie par `TOTAL_HEAP_SIZE`. Si ce paramètre est trop bas, il peut être impossible de créer de nouveaux objets ou d’allouer la mémoire nécessaire.

2. **Stabilité du Système** : Si `TOTAL_HEAP_SIZE` est configuré trop bas pour les besoins de l’application, cela risque d’entraîner des erreurs de type « out of memory » (OOM), où FreeRTOS ne peut plus allouer de mémoire pour des tâches ou des files d’attente, entraînant potentiellement un comportement imprévisible ou des plantages du système. Un bon dimensionnement est donc essentiel pour garantir la stabilité de l’application.

3. **Mémoire Disponibilité dans l’Embedded System** : Les microcontrôleurs ont souvent des quantités de mémoire RAM limitées. Le paramètre `TOTAL_HEAP_SIZE` doit donc être défini en tenant compte des autres besoins en mémoire de l’application pour ne pas épuiser les ressources de l’appareil.

### Utilité de `portTICK_PERIOD_MS`

Le paramètre **`portTICK_PERIOD_MS`** dans FreeRTOS est une constante qui représente la durée d'un **tick** en millisecondes. Le tick est l'intervalle de temps au bout duquel FreeRTOS effectue des tâches de gestion, telles que le changement de contexte (c'est-à-dire, le passage d'une tâche à une autre, si nécessaire). 

1. **Calcul des Délais et Temporisations** : 
   - `portTICK_PERIOD_MS` permet de convertir des durées en millisecondes vers des **ticks**, l'unité de temps interne de FreeRTOS. Cela est utile pour les fonctions d'attente et de temporisation telles que `vTaskDelay()`, `xQueueReceive()`, etc., qui prennent des durées exprimées en ticks.
   - Par exemple, si on veut qu'une tâche attende 500 ms, on peut convertir cette durée en ticks en divisant par `portTICK_PERIOD_MS` :
     ```c
     vTaskDelay( 500 / portTICK_PERIOD_MS );  // Délai de 500 ms
     ```
   - Ainsi, `portTICK_PERIOD_MS` rend le code plus lisible et portable, car il permet d'exprimer des délais en millisecondes sans se soucier de la fréquence exacte du tick.

2. **Adaptation aux Changements de Fréquence du Tick** :
   - FreeRTOS permet de configurer la fréquence de génération des ticks via le paramètre **`configTICK_RATE_HZ`** dans le fichier de configuration `FreeRTOSConfig.h`.
   - `portTICK_PERIOD_MS` est calculé en fonction de `configTICK_RATE_HZ` :
     ```c
     #define portTICK_PERIOD_MS ( ( TickType_t ) 1000 / configTICK_RATE_HZ )
     ```
   - Si la fréquence du tick est modifiée, `portTICK_PERIOD_MS` sera automatiquement ajusté, ce qui garantit que les délais basés sur `portTICK_PERIOD_MS` restent corrects sans besoin de modifier le code.

3. **Synchronisation et Délais Multiples de la Période de Tick** :
   - Dans les systèmes temps réel, il est souvent nécessaire de synchroniser des tâches à des intervalles réguliers. `portTICK_PERIOD_MS` permet de définir des intervalles facilement en fonction de la période du tick.
   - Par exemple, dans une application qui nécessite une tâche qui s’exécute toutes les 100 ms, il est possible de faire :
     ```c
     vTaskDelay( 100 / portTICK_PERIOD_MS );  // Délai de 100 ms
     ```

## 1.2 Sémaphores pour la synchronisation
3. Après programmation des sémaphores on observe :
```
Avant avoir pris le sémaphore taskTake
Après avoir pris le sémaphore taskTake
Avant avoir donné le sémaphore taskGive
Après avoir donné le sémaphore taskGive
``` 

4. Pour la gestion d'erreur, nous avons procédés de la façon suivante :
```c
if(task_sync != NULL)
{
  printf("Avant avoir pris le sémaphore taskTake\r\n");
  if(xSemaphoreTake(task_sync, (TickType_t) SEMAPHORE_RETRY_TIME / portTICK_PERIOD_MS) == pdTRUE)
  {
    /* We were able to obtain the semaphore and can now access the
          shared resource. */
    printf("Après avoir pris le sémaphore taskTake\r\n");
    HAL_GPIO_TogglePin(LD2_GPIO_Port, LD2_Pin);
    vTaskDelay((TickType_t) duree / portTICK_PERIOD_MS);
  }
  else
  {
    /* We could not obtain the semaphore and can therefore not access
          the shared resource safely. */
    printf("taskTake n'a pas pu prendre le semaphore après %.3f\r\n", (float)SEMAPHORE_RETRY_TIME);
    Error_Handler();
  }
}
```
Le sémaphore n'étant pas emprunté par d'autres tâches pour l'instant, nous observons pas encore l'erreur.

5. Avec taskGive qui a une priorité supérieure à taskTake on a le message suivant :
```
Tâche crée avec succès                                                          
Tâche crée avec succès                                                          
Avant avoir pris le sémaphore taskGive                                          
Avant avoir pris le sémaphore taskTake                                          
taskGive n'a pas pu prendre le semaphore après 1000.000 ms
```
  
6. En inversant les priorités on observe le comportement suivant :
```
Tâche crée avec succès                                                          
Tâche crée avec succès                                                          
Avant avoir pris le sémaphore taskTake                                          
Avant avoir pris le sémaphore taskGive                                          
taskTake n'a pas pu prendre le semaphore après 1000.000 ms
```
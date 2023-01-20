# _HAL-F401-DMA-UART2
This repository was create to follow the content of the course 'ARM Cortex M Microcontroller DMA Programming Demystified'. 

I am using: 
* NUCLEO-F401RE Board 
* STM32Cube FW_F4 V1.27.1 
* STM32CubeIDE version 1.10.1

# STM32CubeMX

We don't change the USART configurations.

![image](https://user-images.githubusercontent.com/58916022/213687190-7272ec3b-3759-4a65-be7f-e1af054cbcc2.png)

Neither the NVIC interrupt for the USART2.

![image](https://user-images.githubusercontent.com/58916022/213687302-b412a1d1-8e1e-4106-8129-0b375258066e.png)

To configure the DMA, we need to set if we will work with RX or TX mode. In our case is RX. Then we need to set the 'Increment Address' option. For peripheral, the DR Address is unique and don't increment. It receiver 1 byter/time. For the memory, since we will be working with more than 1 byte of information, the option must be ticked.

![image](https://user-images.githubusercontent.com/58916022/213688238-2d59ca5f-3c87-483f-be98-df49f3dcb9bd.png)

Now in NVIC section we will have the interrupt for the DMA1. 

![image](https://user-images.githubusercontent.com/58916022/213689187-7188eb15-cf30-40ec-a8c0-8d9554c9276d.png)

And since we will be using interrupt mode (for DMA), we can leave the following options ticked.

![image](https://user-images.githubusercontent.com/58916022/213689666-b3ab5366-6bb5-4325-a8b7-935427f62763.png)

We don't need to change the clock configuration, the code is ready to be generated.

# Code

It is always good to check and understand all the generated code. We can see and review the topic by going to 'Section 7' of the course. 

The propose topic is: toggle led using dma, transfer data from sram1 to sram2 and transfer data from uart do sram1.

Inside USART registers, in CR3, there is a bit field to DMA enable receiver/transmitter (DMAR/DMAT).

* 1 - Data comes from PC to UART2
* 2 - Data is on DR of UART2
* 3 - DMA gets notification (instead of ARM processor) - must be enabled
* 4 - DMA sends data from DR to SRAM1
* 5 - DMA says to ARM that all bytes were received, so ARM can use them
* 6 - ARM can execute DMA IRQ Handler 
* 7 - ARM calls the Callback function

We have 3 options for the received data in UART, that are inside the 'STM32F4xx_HAL_Driver' fold, 'stm32f4xx_hal_uart.c' file. We can see those in the 'Project Explorer'.

![image](https://user-images.githubusercontent.com/58916022/213693769-9488cfde-d1c5-453e-a1bf-cc4592533016.png)

Then we need to study its parameters and return value. After that we can implement this API inside our code. We will need to send the SRAM1 address, so we set some macros to help us.

´´´c
/* Private includes ----------------------------------------------------------*/
/* USER CODE BEGIN Includes */
#define OFFSET 			0x800
#define DEST_ADDRESS 	(SRAM1_BASE+OFFSET)
/* USER CODE END Includes */
´´´

After that we can call the API inside the while loop.

´´´c
  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
	  HAL_UART_Receive_DMA(&huart2, (uint8_t*) DEST_ADDRESS, 10); //To receive 10 bytes
  }
  /* USER CODE END 3 */
´´´c

After 10 bytes, DMA issue an interrupt to the ARM and the IRQ Handler (stm32f4xx_it.c) will be called.
Then, after the IRQ is handled, the Callback function will be called (It is a weak function that need to be
implemente in the user file).
Then we implemented the callback function (keeping the name of the API, HAL_UART_RxCpltCallback).

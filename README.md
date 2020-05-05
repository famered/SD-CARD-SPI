# SD-CARD-SPI
STM32 SD-CARD-SPI COMMUNICATION
/* USER CODE BEGIN Header */
/**
  ******************************************************************************
  * @file           : main.c
  * @brief          : Main program body
  ******************************************************************************
  * @attention
  *
  * <h2><center>&copy; Copyright (c) 2020 STMicroelectronics.
  * All rights reserved.</center></h2>
  *
  * This software component is licensed by ST under Ultimate Liberty license
  * SLA0044, the "License"; You may not use this file except in compliance with
  * the License. You may obtain a copy of the License at:
  *                             www.st.com/SLA0044
  *
  ******************************************************************************
  */
/* USER CODE END Header */

/* Includes ------------------------------------------------------------------*/
#include "main.h"
#include "fatfs.h"

/* Private includes ----------------------------------------------------------*/
/* USER CODE BEGIN Includes */
#include "fatfs_sd.h"
#include "string.h"
#include "stdio.h"

/* USER CODE END Includes */

/* Private typedef -----------------------------------------------------------*/
/* USER CODE BEGIN PTD */

/* USER CODE END PTD */

/* Private define ------------------------------------------------------------*/
/* USER CODE BEGIN PD */

/* USER CODE END PD */

/* Private macro -------------------------------------------------------------*/
/* USER CODE BEGIN PM */

/* USER CODE END PM */

/* Private variables ---------------------------------------------------------*/
RTC_HandleTypeDef hrtc;

SPI_HandleTypeDef hspi1;

UART_HandleTypeDef huart1;
UART_HandleTypeDef huart2;

/* USER CODE BEGIN PV */





/* USER CODE END PV */

/* Private function prototypes -----------------------------------------------*/
void SystemClock_Config(void);
static void MX_GPIO_Init(void);
static void MX_SPI1_Init(void);
static void MX_USART1_UART_Init(void);
static void MX_RTC_Init(void);
static void MX_USART2_UART_Init(void);
/* USER CODE BEGIN PFP */

/* USER CODE END PFP */

/* Private user code ---------------------------------------------------------*/
/* USER CODE BEGIN 0 */
FATFS fs;  // file system
FIL fil;  // file
FRESULT fresult;  // to store the result
char buffer[1024]; // to store data

UINT br, bw;   // file read/write count

/* capacity related variables */
FATFS *pfs;
DWORD fre_clust;
unsigned long total, free_space;
char data[32];
char time[10];
char date[20];
char info[20];
uint8_t veri=0;
/* to send the data to the uart */
void send_uart (char *string)
{
	uint16_t len = strlen (string);
	HAL_UART_Transmit(&huart1, (uint8_t *) string, len, 2000);  // transmit in blocking mode
}

/* to find the size of data in the buffer */
int bufsize (char *buf)
{
	int i=0;
	while (*buf++ != '\0') i++;
	return i;
}

void bufclear (void)  // clear buffer
{
	for (int i=0; i<1024; i++)
	{
		buffer[i] = '\0';
		}
}
 void set_time(void){
	RTC_TimeTypeDef sTime = {0};
  RTC_DateTypeDef DateToUpdate = {0};
  
  sTime.Hours = 0x12;
  sTime.Minutes = 0x55;
  sTime.Seconds = 0x59;

  if (HAL_RTC_SetTime(&hrtc, &sTime, RTC_FORMAT_BCD) != HAL_OK)
  {
    Error_Handler();
  }
  DateToUpdate.WeekDay = RTC_WEEKDAY_MONDAY;
  DateToUpdate.Month = RTC_MONTH_MARCH;
  DateToUpdate.Date = 0x17;
  DateToUpdate.Year = 0x20;

  if (HAL_RTC_SetDate(&hrtc, &DateToUpdate, RTC_FORMAT_BCD) != HAL_OK)
  {
    Error_Handler();
  }
	HAL_RTCEx_BKUPWrite(&hrtc, RTC_BKP_DR1, 0X32F2);
	
  /* USER CODE BEGIN RTC_Init 2 */

	
	
}
void get_time(void) {
	RTC_TimeTypeDef gTime;
  RTC_DateTypeDef gDate;
	 
	HAL_RTC_GetTime(&hrtc, &gTime, RTC_FORMAT_BIN);
	HAL_RTC_GetDate(&hrtc, &gDate, RTC_FORMAT_BIN);
	 
    sprintf((char*)time, " %02d:%02d:%02d", gTime.Hours, gTime.Minutes, gTime.Seconds);
  	sprintf((char*)date, "\f [%02d-%02d-%2d] %02d:%02d:%02d   %d \n\r ", gDate.Date, gDate.Month, 2000+gDate.Year,gTime.Hours, gTime.Minutes, gTime.Seconds,veri);
	  
	
	//HAL_UART_Transmit(&huart1, (uint8_t*)date, sprintf((char*)date, "\f [%02d-%02d-%2d]  %02d:%02d:%02d    \n\r", gDate.Month, gDate.Date, 2000+ gDate.Year,gTime.Hours, gTime.Minutes, gTime.Seconds),200);
		//HAL_Delay(1000);
		
	
	
  /* USER CODE END RTC_Init 2 */


}

/* USER CODE END 0 */

/**
  * @brief  The application entry point.
  * @retval int
  */
int main(void)
{
  /* USER CODE BEGIN 1 */

  /* USER CODE END 1 */
  

  /* MCU Configuration--------------------------------------------------------*/

  /* Reset of all peripherals, Initializes the Flash interface and the Systick. */
  HAL_Init();

  /* USER CODE BEGIN Init */

  /* USER CODE END Init */

  /* Configure the system clock */
  SystemClock_Config();

  /* USER CODE BEGIN SysInit */

  /* USER CODE END SysInit */

  /* Initialize all configured peripherals */
  MX_GPIO_Init();
  MX_SPI1_Init();
  MX_USART1_UART_Init();
  MX_FATFS_Init();
  MX_RTC_Init();
  MX_USART2_UART_Init();
  /* USER CODE BEGIN 2 */
   set_time(); 
   fresult = f_mount(&fs, "", 0);
    if (fresult != FR_OK) send_uart ("error in mounting SD CARD...\n");
    else send_uart("SD CARD mounted successfully...\n");

 f_getfree("", &fre_clust, &pfs);

        total = (unsigned long)((pfs->n_fatent - 2) * pfs->csize * 0.5);
        sprintf (buffer, "SD CARD Total Size: \t%lu\n",total);
        send_uart(buffer);
        bufclear();
        free_space = (unsigned long)(fre_clust * pfs->csize * 0.5);
        sprintf (buffer, "SD CARD Free Space: \t%lu\n",free_space);
        send_uart(buffer);

       

       strcpy (buffer, "GOKTURK-2020 TELEMETRY SYSTEM LOG FILE\n");

          fresult = f_write(&fil, buffer, bufsize(buffer), &bw);
        /************* The following operation is using PUTS and GETS *********************/


        /* Open file to write/ create a file if it doesn't exist */
        //fresult = f_open(&fil, "file1.txt", FA_OPEN_ALWAYS | FA_READ | FA_WRITE);

        /* Writing text */
        //fresult = f_puts("This data is from the First FILE\n\n", &fil);
        //fresult = f_puts(data , &fil);
        /* Close file */
        //fresult = f_close(&fil);

        //send_uart ("File1.txt created and the data is written \n");

        /* Open file to read */
       // fresult = f_open(&fil, "file1.txt", FA_READ);


        /* Read string from the file */
//        f_gets(buffer, fil.fsize, &fil);

  //      send_uart(buffer);

        /* Close file */
    //    f_close(&fil);

       // bufclear();


        /**************** The following operation is using f_write and f_read **************************/

        /* Create second file with read write access and open it */
       fresult = f_open(&fil, "file2.txt", FA_OPEN_ALWAYS | FA_READ | FA_WRITE);
        
        /* Writing text */
        strcpy (buffer, "GOKTURK-2020 TELEMETRY SYSTEM LOG FILE\n");
        
					
		
					
				fresult = f_write(&fil, buffer, bufsize(buffer), &bw);
        //fresult = f_write(&fil, data, sprintf(data, "\f%d\n\r",sayi), &bw);
			        	
				//fresult = f_write(&fil, data, strlen(data), &bw);
				//fresult = f_write(&fil, &sayi, 8, &bw);
        //send_uart ("File2.txt created and data is written\n");
				
				
						
				
        /* Close file */
        //f_close(&fil);
				

				
				
        // clearing buffer to show that result obtained is from the file
       // bufclear();

        /* Open second file to read */
        //fresult = f_open(&fil, "file2.txt", FA_READ);

        /* Read data from the file
         * Please see the function details for the arguments */
        //f_read (&fil, buffer, fil.fsize, &br);
        //send_uart(buffer);

        /* Close file */
        //f_close(&fil);

        //bufclear();


        /*********************UPDATING an existing file ***************************/

        /* Open the file with write access */
        //fresult = f_open(&fil, "file2.txt", FA_OPEN_ALWAYS | FA_WRITE);

        /* Move to offset to the end of the file */
        //fresult = f_lseek(&fil, fil.fsize);

        /* write the string to the file */
        //fresult = f_puts("This is updated data and it should be in the end \n", &fil);

        //f_close (&fil);

        /* Open to read the file */
//        fresult = f_open (&fil, "file2.txt", FA_READ);

        /* Read string from the file */
  //      f_read (&fil, buffer, fil.fsize, &br);
     //   send_uart(buffer);

					/* Close file */
       // f_close(&fil);

        //bufclear();


        /*************************REMOVING FILES FROM THE DIRECTORY ****************************/

        //fresult = f_unlink("/file1.txt");
        //if (fresult == FR_OK) send_uart("file1.txt removed successfully...\n");

        //fresult = f_unlink("/file2.txt");
        //if (fresult == FR_OK) send_uart("file2.txt removed successfully...\n");

        /* Unmount SDCARD */
        //fresult = f_mount(NULL, "", 1);
        //if (fresult == FR_OK) send_uart ("SD CARD UNMOUNTED successfully...\n");
         
				 
				 
			

  /* USER CODE END 2 */

  /* Infinite loop */
  /* USER CODE BEGIN WHILE */
  while (1)
  {
		
		veri++;
//		HAL_UART_Receive(&huart1, &veri, 8, 200);
		
	//	HAL_Delay(500);
		sprintf(data, "\f%d\n\r", veri);
		
		HAL_UART_Transmit(&huart2, (uint8_t*)info, sprintf(info, "n0.val=%dÿÿÿ",veri),200);
	  HAL_Delay(100);
		
      			
			
		
		get_time();
    
		fresult = f_write(&fil, date, strlen(date), &bw);
		
		
		//HAL_UART_Receive(&huart1, &veri, 8, 200);
		fresult = f_write(&fil, data, strlen(data), &bw);
	  
		
   if(veri>50){
		 
		 
		
		
			f_close(&fil);
		
	 }
			
    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
  }
  /* USER CODE END 3 */
}

/**
  * @brief System Clock Configuration
  * @retval None
  */
void SystemClock_Config(void)
{
  RCC_OscInitTypeDef RCC_OscInitStruct = {0};
  RCC_ClkInitTypeDef RCC_ClkInitStruct = {0};
  RCC_PeriphCLKInitTypeDef PeriphClkInit = {0};

  /** Initializes the CPU, AHB and APB busses clocks 
  */
  RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_LSI|RCC_OSCILLATORTYPE_HSE;
  RCC_OscInitStruct.HSEState = RCC_HSE_ON;
  RCC_OscInitStruct.HSEPredivValue = RCC_HSE_PREDIV_DIV1;
  RCC_OscInitStruct.HSIState = RCC_HSI_ON;
  RCC_OscInitStruct.LSIState = RCC_LSI_ON;
  RCC_OscInitStruct.PLL.PLLState = RCC_PLL_ON;
  RCC_OscInitStruct.PLL.PLLSource = RCC_PLLSOURCE_HSE;
  RCC_OscInitStruct.PLL.PLLMUL = RCC_PLL_MUL9;
  if (HAL_RCC_OscConfig(&RCC_OscInitStruct) != HAL_OK)
  {
    Error_Handler();
  }
  /** Initializes the CPU, AHB and APB busses clocks 
  */
  RCC_ClkInitStruct.ClockType = RCC_CLOCKTYPE_HCLK|RCC_CLOCKTYPE_SYSCLK
                              |RCC_CLOCKTYPE_PCLK1|RCC_CLOCKTYPE_PCLK2;
  RCC_ClkInitStruct.SYSCLKSource = RCC_SYSCLKSOURCE_PLLCLK;
  RCC_ClkInitStruct.AHBCLKDivider = RCC_SYSCLK_DIV1;
  RCC_ClkInitStruct.APB1CLKDivider = RCC_HCLK_DIV2;
  RCC_ClkInitStruct.APB2CLKDivider = RCC_HCLK_DIV1;

  if (HAL_RCC_ClockConfig(&RCC_ClkInitStruct, FLASH_LATENCY_2) != HAL_OK)
  {
    Error_Handler();
  }
  PeriphClkInit.PeriphClockSelection = RCC_PERIPHCLK_RTC;
  PeriphClkInit.RTCClockSelection = RCC_RTCCLKSOURCE_LSI;
  if (HAL_RCCEx_PeriphCLKConfig(&PeriphClkInit) != HAL_OK)
  {
    Error_Handler();
  }
}

/**
  * @brief RTC Initialization Function
  * @param None
  * @retval None
  */
static void MX_RTC_Init(void)
{

  /* USER CODE BEGIN RTC_Init 0 */

  /* USER CODE END RTC_Init 0 */

  RTC_TimeTypeDef sTime = {0};
  RTC_DateTypeDef DateToUpdate = {0};

  /* USER CODE BEGIN RTC_Init 1 */

  /* USER CODE END RTC_Init 1 */
  /** Initialize RTC Only 
  */
  hrtc.Instance = RTC;
  hrtc.Init.AsynchPrediv = RTC_AUTO_1_SECOND;
  hrtc.Init.OutPut = RTC_OUTPUTSOURCE_ALARM;
  if (HAL_RTC_Init(&hrtc) != HAL_OK)
  {
    Error_Handler();
  }

  /* USER CODE BEGIN Check_RTC_BKUP */
    
  /* USER CODE END Check_RTC_BKUP */

  /** Initialize RTC and set the Time and Date 
  */
  sTime.Hours = 0x0;
  sTime.Minutes = 0x0;
  sTime.Seconds = 0x0;

  if (HAL_RTC_SetTime(&hrtc, &sTime, RTC_FORMAT_BCD) != HAL_OK)
  {
    Error_Handler();
  }
  DateToUpdate.WeekDay = RTC_WEEKDAY_MONDAY;
  DateToUpdate.Month = RTC_MONTH_JANUARY;
  DateToUpdate.Date = 0x1;
  DateToUpdate.Year = 0x0;

  if (HAL_RTC_SetDate(&hrtc, &DateToUpdate, RTC_FORMAT_BCD) != HAL_OK)
  {
    Error_Handler();
  }
  /* USER CODE BEGIN RTC_Init 2 */

	
	
}
 
	
  /* USER CODE END RTC_Init 2 */



/**
  * @brief SPI1 Initialization Function
  * @param None
  * @retval None
  */
static void MX_SPI1_Init(void)
{

  /* USER CODE BEGIN SPI1_Init 0 */

	
  /* USER CODE END SPI1_Init 0 */

  /* USER CODE BEGIN SPI1_Init 1 */

  /* USER CODE END SPI1_Init 1 */
  /* SPI1 parameter configuration*/
  hspi1.Instance = SPI1;
  hspi1.Init.Mode = SPI_MODE_MASTER;
  hspi1.Init.Direction = SPI_DIRECTION_2LINES;
  hspi1.Init.DataSize = SPI_DATASIZE_8BIT;
  hspi1.Init.CLKPolarity = SPI_POLARITY_LOW;
  hspi1.Init.CLKPhase = SPI_PHASE_1EDGE;
  hspi1.Init.NSS = SPI_NSS_SOFT;
  hspi1.Init.BaudRatePrescaler = SPI_BAUDRATEPRESCALER_32;
  hspi1.Init.FirstBit = SPI_FIRSTBIT_MSB;
  hspi1.Init.TIMode = SPI_TIMODE_DISABLE;
  hspi1.Init.CRCCalculation = SPI_CRCCALCULATION_DISABLE;
  hspi1.Init.CRCPolynomial = 10;
  if (HAL_SPI_Init(&hspi1) != HAL_OK)
  {
    Error_Handler();
  }
  /* USER CODE BEGIN SPI1_Init 2 */

  /* USER CODE END SPI1_Init 2 */

}

/**
  * @brief USART1 Initialization Function
  * @param None
  * @retval None
  */
static void MX_USART1_UART_Init(void)
{

  /* USER CODE BEGIN USART1_Init 0 */

  /* USER CODE END USART1_Init 0 */

  /* USER CODE BEGIN USART1_Init 1 */

  /* USER CODE END USART1_Init 1 */
  huart1.Instance = USART1;
  huart1.Init.BaudRate = 9600;
  huart1.Init.WordLength = UART_WORDLENGTH_8B;
  huart1.Init.StopBits = UART_STOPBITS_1;
  huart1.Init.Parity = UART_PARITY_NONE;
  huart1.Init.Mode = UART_MODE_TX_RX;
  huart1.Init.HwFlowCtl = UART_HWCONTROL_NONE;
  huart1.Init.OverSampling = UART_OVERSAMPLING_16;
  if (HAL_UART_Init(&huart1) != HAL_OK)
  {
    Error_Handler();
  }
  /* USER CODE BEGIN USART1_Init 2 */

  /* USER CODE END USART1_Init 2 */

}

/**
  * @brief USART2 Initialization Function
  * @param None
  * @retval None
  */
static void MX_USART2_UART_Init(void)
{

  /* USER CODE BEGIN USART2_Init 0 */

  /* USER CODE END USART2_Init 0 */

  /* USER CODE BEGIN USART2_Init 1 */

  /* USER CODE END USART2_Init 1 */
  huart2.Instance = USART2;
  huart2.Init.BaudRate = 9600;
  huart2.Init.WordLength = UART_WORDLENGTH_8B;
  huart2.Init.StopBits = UART_STOPBITS_1;
  huart2.Init.Parity = UART_PARITY_NONE;
  huart2.Init.Mode = UART_MODE_TX_RX;
  huart2.Init.HwFlowCtl = UART_HWCONTROL_NONE;
  huart2.Init.OverSampling = UART_OVERSAMPLING_16;
  if (HAL_UART_Init(&huart2) != HAL_OK)
  {
    Error_Handler();
  }
  /* USER CODE BEGIN USART2_Init 2 */

  /* USER CODE END USART2_Init 2 */

}

/**
  * @brief GPIO Initialization Function
  * @param None
  * @retval None
  */
static void MX_GPIO_Init(void)
{
  GPIO_InitTypeDef GPIO_InitStruct = {0};

  /* GPIO Ports Clock Enable */
  __HAL_RCC_GPIOD_CLK_ENABLE();
  __HAL_RCC_GPIOA_CLK_ENABLE();
  __HAL_RCC_GPIOB_CLK_ENABLE();

  /*Configure GPIO pin Output Level */
  HAL_GPIO_WritePin(SD_CS_GPIO_Port, SD_CS_Pin, GPIO_PIN_RESET);

  /*Configure GPIO pin : SD_CS_Pin */
  GPIO_InitStruct.Pin = SD_CS_Pin;
  GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
  HAL_GPIO_Init(SD_CS_GPIO_Port, &GPIO_InitStruct);

}

/* USER CODE BEGIN 4 */

/* USER CODE END 4 */

/**
  * @brief  This function is executed in case of error occurrence.
  * @retval None
  */
void Error_Handler(void)
{
  /* USER CODE BEGIN Error_Handler_Debug */
  /* User can add his own implementation to report the HAL error return state */

  /* USER CODE END Error_Handler_Debug */
}

#ifdef  USE_FULL_ASSERT
/**
  * @brief  Reports the name of the source file and the source line number
  *         where the assert_param error has occurred.
  * @param  file: pointer to the source file name
  * @param  line: assert_param error line source number
  * @retval None
  */
void assert_failed(uint8_t *file, uint32_t line)
{ 
  /* USER CODE BEGIN 6 */
  /* User can add his own implementation to report the file name and line number,
     tex: printf("Wrong parameters value: file %s on line %d\r\n", file, line) */
  /* USER CODE END 6 */
}
#endif /* USE_FULL_ASSERT */

/************************ (C) COPYRIGHT STMicroelectronics *****END OF FILE****/

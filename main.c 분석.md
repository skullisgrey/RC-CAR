# main.c 설명
### 보면 도움이 될 수도 있음

## Buzzer
HAL_GPIO_TogglePin(buzzer_GPIO_Port, buzzer_Pin);
	HAL_Delay(AA);
	HAL_GPIO_TogglePin(buzzer_GPIO_Port, buzzer_Pin);
	HAL_Delay(AA);
  ...

위와 같은 형태는, 내부 금속판을 진동시켜 음을 발생시키는 피에조 버저의 특성에 따름.

gpio toggle을 통하여, 0 1 0 1 0 1 신호를 주고, 이로 인해 내부 금속판이 진동 -> 음 발생

timer를 이용하는 방법이 있지만, 현재 prescale 및 counter peroid값이 buzzer에 사용하기 힘든 값이라 차선책으로 토글 사용.


## DC Motor 회전 방향 제어
void smartcar_forward(void)
{
	__HAL_TIM_SET_COMPARE(&htim1, TIM_CHANNEL_2, del);
	__HAL_TIM_SET_COMPARE(&htim1, TIM_CHANNEL_3, 0);
	__HAL_TIM_SET_COMPARE(&htim2, TIM_CHANNEL_2, del);
	__HAL_TIM_SET_COMPARE(&htim2, TIM_CHANNEL_3, 0);
	__HAL_TIM_SET_COMPARE(&htim3, TIM_CHANNEL_1, del);
	__HAL_TIM_SET_COMPARE(&htim3, TIM_CHANNEL_2, 0);
	__HAL_TIM_SET_COMPARE(&htim4, TIM_CHANNEL_3, del);
	__HAL_TIM_SET_COMPARE(&htim4, TIM_CHANNEL_4, 0);


}

기본적으로, gpio_write 사용 -> 이는 최대 속력으로만 회전함

tim_set_compare -> del값에 따라 duty가 변화 -> 평균 전압 변경 -> 입력되는 전압에 따라 회전 속력이 달라지는 DC Motor의 속력 변경 가능

여기서, del 값은 최대 counter period값까지 가능. (현재 counter period가 25000이므로 del의 최댓값은 25000)

## PWM 제어 및 Buzzer

void gear_shift(void)
{
	if(gear == '1')
		{
		del = 5000;
		gear = 0;
		}
	else if(gear == '2')
	{
		del = 10000;
		gear = 0;
	}
	else if(gear == '3')
	{
		del = 15000;
		gear = 0;
	}
	else if(gear == '4')
		{
			del = 20000;
			gear = 0;
		}
	else if(gear == '5')
			{
				del = 25000;
				gear = 0;
			}
	else if(gear == '0')
	{
		sound();
		gear = 0;
	}
}

gear라는 변수에 들어오는 값에 따라, del 변경 및 Buzzer를 실행.

변수를 초기화 시켜주지 않으면 계속 반복하는 문제가 발생하므로, 초기화를 시켜주는게 좋음.

##  입력 변수 분리
void commander(void)
{
	if(COMMAND == '0' || COMMAND == '1' || COMMAND == '2' || COMMAND == '3' || COMMAND == '4' || COMMAND == '5')
	{
		ch = 0;
		gear = COMMAND;
	}

	else if(COMMAND == 'w' || COMMAND == 's' || COMMAND == 'a' || COMMAND == 'd' || ' ')
	{
		ch = COMMAND;
	}

}

하나의 Bluetooth로 통신을 하기 때문에, 하나의 입력값만 받아야 원활한 동작 가능.

따라서, 모터 회전 방향과 속력 제어에 따른 입력값을 분리하는 함수를 만듦.

여기서, gear라는 변수에 입력이 들어갈 때, 방향 제어와 관련된 변수 ch를 초기화 시키지 않는다면, PWM제어를 하는데 이상 작동을 할 수 있음.


## untrasonic distancement
void timer_start(void)
{
   HAL_TIM_Base_Start(&htim1);
}

void delay_us(uint16_t us)
{
   __HAL_TIM_SET_COUNTER(&htim1, 0); // initislize counter to start from 0
   while((__HAL_TIM_GET_COUNTER(&htim1))<us); // wait count until us
}

void trig(void)
   {
       HAL_GPIO_WritePin(Trig_GPIO_Port, Trig_Pin, 1);
       delay_us(10);
       HAL_GPIO_WritePin(Trig_GPIO_Port, Trig_Pin, 0);
   }

void trig1(void)
   {
       HAL_GPIO_WritePin(Trig1_GPIO_Port, Trig1_Pin, 1);
       delay_us(10);
       HAL_GPIO_WritePin(Trig1_GPIO_Port, Trig1_Pin, 0);
   }

long unsigned int echo(void)
   {
       long unsigned int echo = 0;

       while(HAL_GPIO_ReadPin(Echo_GPIO_Port, Echo_Pin) == 0){}
            __HAL_TIM_SET_COUNTER(&htim1, 0);
            while(HAL_GPIO_ReadPin(Echo_GPIO_Port, Echo_Pin) == 1);
            echo = __HAL_TIM_GET_COUNTER(&htim1);
       if( echo >= 240 && echo <= 23000 ) return echo;
       else return 0;
   }

long unsigned int echo1(void)
   {
       long unsigned int echo1 = 0;

       while(HAL_GPIO_ReadPin(Echo1_GPIO_Port, Echo1_Pin) == 0){}
            __HAL_TIM_SET_COUNTER(&htim1, 0);
            while(HAL_GPIO_ReadPin(Echo1_GPIO_Port, Echo1_Pin) == 1);
            echo1 = __HAL_TIM_GET_COUNTER(&htim1);
       if( echo1 >= 240 && echo1 <= 23000 ) return echo1;
       else return 0;
   }

void distance(void){

	      trig();
	  	  int echo_time = echo();
	  	  if(echo_time != 0)
	  	  {
	  		  dist = (int)(17 * echo_time / 100);
	  		  printf("RD%d\n", dist);
	  	  }

	  	  trig1();
	  	  int echo_time1 = echo1();
	  	  if(echo_time1 != 0)
	  	  {
	  		  dist1 = (int)(17 * echo_time1 / 100);
	  		  printf("LD%d\n", dist1);
	  	  }

}


timer start는 timer 작동을 위한 기본 실행 함수

delay_us는 μs단위의 시간을 재기 위해 만든 함수 // prescale 63 --> 1MHz --> 10^-6 --> 1μs가 됨.

trig는 측정기의 tx에서 초음파 발진을 위한 함수

echo는 tx에서 발진되었을 때부터, 그 초음파를 수신할 때 까지의 시간을 측정. 최소 240μs 최대 23000μs까지 대기.

만약, 저 범위를 초과한다면 측정이 되지 않음. // 기대할 수 있는 함수 대기 시간 최대 23ms~


## DC Motor 회전 방향 제어 조합(RC CAR 방향 제어)
void moving_car(void)
{

	  if (ch == 'w')
	  	  {
	  		  smartcar_forward();
	  		 distance();
	  		  HAL_Delay(70);
	  		 smartcar_stop();
	  	  }
	  	  else if (ch == 's')
	  		  {
	  			  smartcar_backward();
	  			distance();
	  			 HAL_Delay(70);
	  			  	smartcar_stop();
	  		  }
	  		  else if (ch == 'a')
	  			  {
	  					 smartcar_13();
	  					distance();
	  					HAL_Delay(70);
	  					 smartcar_stop();
	  			  }
	  			  else if (ch == 'd')
	  				  {
	  				  smartcar_24();
	  				distance();
	  				HAL_Delay(70);
	  				  	smartcar_stop();
	  				  }
	  			  else if (ch == ' ')
	  				  {
	  				  smartcar_stop();
	  				  distance();
	  				  HAL_Delay(70);
	  				  }
}

Delay -> 커맨드 입력 후, 최소한의 회전 시간 보장

여기서, distance 함수의 경우, 위에서 최대 대기 시간이 23μs이므로, 실제 delay는 70~100ms로 추정 가능.


# 메인 함수

 /* USER CODE BEGIN 2 */

  timer_start();

  HAL_TIM_PWM_Start(&htim1, TIM_CHANNEL_2);
  HAL_TIM_PWM_Start(&htim1, TIM_CHANNEL_3);
  HAL_TIM_PWM_Start(&htim2, TIM_CHANNEL_2);
  HAL_TIM_PWM_Start(&htim2, TIM_CHANNEL_3);
  HAL_TIM_PWM_Start(&htim3, TIM_CHANNEL_1);
  HAL_TIM_PWM_Start(&htim3, TIM_CHANNEL_2);
  HAL_TIM_PWM_Start(&htim4, TIM_CHANNEL_3);
  HAL_TIM_PWM_Start(&htim4, TIM_CHANNEL_4);

  /* USER CODE END 2 */

  /* Infinite loop */
  /* USER CODE BEGIN WHILE */
  while (1)
  {

	  HAL_UART_Receive(&huart3, &COMMAND, 1, HAL_MAX_DELAY);

	  commander();

	  gear_shift();
	  moving_car();

	  fflush(stdout);

    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
  }
  /* USER CODE END 3 */
}

timer start, pwm start를 해야 제어 가능.


receive로 커맨드 입력을 하나 받고, 이에 따른 값을 분리하고 실행함.


moving_car라는 함수 안에 distance 함수가 있는 이유는, 커맨드 - 다음 커맨드 사이의 딜레이가 최소화되어, 결과적으로 1개의 bluetooth 통신으로도 원활하게 동작.

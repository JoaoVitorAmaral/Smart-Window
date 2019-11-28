#include <stm32f4xx_hal.h>
#include <stm32_hal_legacy.h>

#ifdef __cplusplus
extern "C"
#endif
void SysTick_Handler(void)
{
	HAL_IncTick();
	HAL_SYSTICK_IRQHandler();
}

//Constantes para configuração da GPIO dos botões
#define botAcionamento_GPIO	GPIOA
#define botAcionamento_Pin	GPIO_PIN_0

#define BOTs_GPIO GPIOC
#define botMode_Pin GPIO_PIN_4
#define FDCa_Pin GPIO_PIN_7
#define FDCf_Pin GPIO_PIN_9

//Constantes para configuração da GPIO da ponte H
#define	in_GPIO GPIOE
#define	in1_Pin GPIO_PIN_7
#define in2_Pin GPIO_PIN_9
#define in3_Pin GPIO_PIN_11
#define in4_Pin GPIO_PIN_13

//Constantes para configuração da GPIO do sensor de chuva (SC)
#define SC_GPIO GPIOD
#define SC_Pin GPIO_PIN_0

//Variaveis de estado

int botAnt = 0;
int botAtu = 0;
int borda = 0;
int solto = 0;
int pressionado = 1;
int aberta = 0;
int fechada = 1;
int entreaberta = 2;
int statusJanela = 0;

int parar = 0;
int abrir = 1;
int fechar = 2;
int acao = 0;

int parado = 0;
int girando = 1;
int estadoMotor = 0;


int horario = 0;
int antiHorario = 1;
int sentido = 0;

int semChuva = 0;
int chovendo = 1;

#define autonomo 0
#define dependente 1
int modo = 0;

#define acionaPeloSC 0
#define acionaPeloTMP 1
#define acionaPeloBOT 2
uint8_t estadosModoAutonomo = 0;

int motorUsado = 0;
int motorA = 0;
int motorB = 1;

//Dados
int leituraSC = 0;

void configuraGPIO();
int leiturabotMode();
int leiturabotAcionamento();
int leituraFDCa();
int leituraFDCf();
int interpretaJanela();
int lerSC();
int deteccaoDeBorda(GPIO_TypeDef* porta, int pino, int botAntProt, int botAtuProt, int bordaProt);
void acionamento(int motorProt, int acao);

//TMP100 declarations
I2C_HandleTypeDef hI2C;
uint16_t TMP100_resolution12bits = 0x60;
float temperatureRead = 0;
#define temperatureThresholdHigh 27 //oC

//TMP100 functions
void setupLEDs();
void I2Csetup();
void TMP100_configuration(I2C_HandleTypeDef* i2c_instance, uint8_t TMP_resolution);
float TMP100_readTemperature(I2C_HandleTypeDef* i2c_instance);

int main(void)
{
	HAL_Init();
	__GPIOA_CLK_ENABLE();
	__GPIOC_CLK_ENABLE();
	__GPIOD_CLK_ENABLE();
	__GPIOE_CLK_ENABLE();
	
	setupLEDs();
	
	//Configura GPIO dos pinos: botMode, botAcionamento, FDCa e FDCf
	GPIO_InitTypeDef botGPIO;

	botGPIO.Pin = botMode_Pin | botAcionamento_Pin | FDCa_Pin | FDCf_Pin;
	botGPIO.Mode = GPIO_MODE_INPUT;
	botGPIO.Speed = GPIO_SPEED_FREQ_HIGH;
	botGPIO.Pull = GPIO_PULLUP;
	HAL_GPIO_Init(BOTs_GPIO, &botGPIO);
	
	//Configura GPIO do pinos: SC
	GPIO_InitTypeDef scGPIO;

	scGPIO.Pin = SC_Pin;
	scGPIO.Mode = GPIO_MODE_INPUT;
	scGPIO.Speed = GPIO_SPEED_FREQ_HIGH;
	scGPIO.Pull = GPIO_PULLUP;
	HAL_GPIO_Init(SC_GPIO, &scGPIO);
	
	//Configura GPIO do pinos da ponteH: in1, in2, in3 e in4
		
	GPIO_InitTypeDef ponteH;

	ponteH.Pin = in1_Pin | in2_Pin | in3_Pin | in4_Pin;
	ponteH.Mode = GPIO_MODE_OUTPUT_PP;
	ponteH.Speed = GPIO_SPEED_FREQ_HIGH;
	ponteH.Pull = GPIO_NOPULL;
	HAL_GPIO_Init(in_GPIO, &ponteH);

	I2Csetup();
	
	//Configura TMP100 com resolução de 12bits
	TMP100_configuration(&hI2C, TMP100_resolution12bits);
	
	for (;;)
	{
		//Interpreta o estado da janela
		statusJanela = interpretaJanela();
				
		switch (modo)
		{
		case autonomo:
			switch (estadosModoAutonomo)
			{
			case acionaPeloSC:
				//Realiza a leitura do sensor de chuva
				leituraSC = !lerSC();
								
				if (leituraSC == chovendo)
				{
					if (estadoMotor == parado)
					{
						if (statusJanela != fechada || estadoMotor == girando)
						{
							HAL_GPIO_WritePin(GPIOD, GPIO_PIN_13, GPIO_PIN_SET);
					
							//Aciona motor para fechar janela 
							HAL_GPIO_WritePin(in_GPIO, in3_Pin, GPIO_PIN_SET);
							HAL_GPIO_WritePin(in_GPIO, in4_Pin, GPIO_PIN_RESET);
						}
				
						if (statusJanela == fechada)
						{
							HAL_GPIO_WritePin(GPIOD, GPIO_PIN_13, GPIO_PIN_RESET);
						
							//Parar motor
							HAL_GPIO_WritePin(in_GPIO, in3_Pin, GPIO_PIN_SET);
							HAL_GPIO_WritePin(in_GPIO, in4_Pin, GPIO_PIN_SET);
						}
					}
				}
				estadosModoAutonomo = acionaPeloTMP;
				
				break;
			case acionaPeloTMP:
				//Realiza a leitura do sensor de temperatura (TMP100) em 2 bytes
				temperatureRead = TMP100_readTemperature(&hI2C);
				
				if (temperatureRead > temperatureThresholdHigh)
				{
					HAL_GPIO_WritePin(GPIOD, GPIO_PIN_15, GPIO_PIN_SET);
				}
				else
				{
					HAL_GPIO_WritePin(GPIOD, GPIO_PIN_15, GPIO_PIN_RESET);
				}
					
				estadosModoAutonomo = acionaPeloSC;
				
				break;
			}
			break;
		case dependente:

			break;
		}
	}
}

void configuraGPIO() {
	//Habilita RCCs referentes aos periféricos que serão configurados a seguir
	__GPIOA_CLK_ENABLE();
	__GPIOB_CLK_ENABLE();
	__GPIOC_CLK_ENABLE();
	__GPIOD_CLK_ENABLE();
	
	//Configura GPIO dos pinos: botMode, botAcionamento, FDCa e FDCf
	GPIO_InitTypeDef botGPIO;

	botGPIO.Pin = botMode_Pin | botAcionamento_Pin | FDCa_Pin | FDCf_Pin;
	botGPIO.Mode = GPIO_MODE_INPUT;
	botGPIO.Speed = GPIO_SPEED_FREQ_HIGH;
	botGPIO.Pull = GPIO_PULLUP;
	HAL_GPIO_Init(BOTs_GPIO, &botGPIO);

	//Configura GPIO dos pinos: in1, in2, in3, e in4
	
	GPIO_InitTypeDef pontehGPIO;

	pontehGPIO.Pin = in1_Pin | in2_Pin | in3_Pin | in4_Pin;
	;
	pontehGPIO.Mode = GPIO_MODE_OUTPUT_PP;
	pontehGPIO.Speed = GPIO_SPEED_FREQ_HIGH;
	pontehGPIO.Pull = GPIO_NOPULL;
	HAL_GPIO_Init(in_GPIO, &pontehGPIO);

	//Configura GPIO do pinos: SC
	
	GPIO_InitTypeDef scGPIO;

	scGPIO.Pin = SC_Pin;
	scGPIO.Mode = GPIO_MODE_INPUT;
	scGPIO.Speed = GPIO_SPEED_FREQ_HIGH;
	scGPIO.Pull = GPIO_NOPULL;
	HAL_GPIO_Init(SC_GPIO, &scGPIO);
}

int deteccaoDeBorda(GPIO_TypeDef* porta, int pino, int botAntProt, int botAtuProt, int bordaProt) {
	
	//Atualiza os estados da linha do botão
	botAntProt = botAtuProt;
	botAtuProt = HAL_GPIO_ReadPin(porta, pino);
	
	if ((botAntProt == 0 && botAtuProt == 1) || (botAntProt == 1 && botAtuProt == 0))
	{
		bordaProt = 1;
	}
	return bordaProt;
}

int leiturabotMode() {
	return !HAL_GPIO_ReadPin(BOTs_GPIO, botMode_Pin);
}

int leiturabotAcionamento() {
	return !HAL_GPIO_ReadPin(BOTs_GPIO, botAcionamento_Pin);
}

int leituraFDCa() {
	return !HAL_GPIO_ReadPin(BOTs_GPIO, FDCa_Pin);
}

int leituraFDCf() {
	return !HAL_GPIO_ReadPin(BOTs_GPIO, FDCf_Pin);
}

int interpretaJanela() {
	int statusProt = 0;
	
	if (leituraFDCa() == pressionado && leituraFDCf() == solto)
	{
		statusProt = aberta;
	}
	else if (leituraFDCa() == solto && leituraFDCf() == pressionado)
	{
		statusProt = fechada;
	}
	else if (leituraFDCa() == solto && leituraFDCf() == solto)
	{
		statusProt = entreaberta;
	}
	return statusProt;
}

int lerSC() {
	return HAL_GPIO_ReadPin(SC_GPIO, SC_Pin);
}

void acionamento(int motorProt, int acaoProt) {
	if (acaoProt == abrir)
	{
		sentido = horario;
	}
	else if (acaoProt == fechar)
	{
		sentido = antiHorario;
	}
	else if (acaoProt == parar)
	{
		if (motorProt == 0)
		{
			HAL_GPIO_WritePin(in_GPIO, in1_Pin, GPIO_PIN_SET);
			HAL_GPIO_WritePin(in_GPIO, in2_Pin, GPIO_PIN_SET);
			HAL_GPIO_WritePin(in_GPIO, in3_Pin, GPIO_PIN_RESET);
			HAL_GPIO_WritePin(in_GPIO, in4_Pin, GPIO_PIN_RESET);
		}
		else
		{
			HAL_GPIO_WritePin(in_GPIO, in1_Pin, GPIO_PIN_RESET);
			HAL_GPIO_WritePin(in_GPIO, in2_Pin, GPIO_PIN_RESET);
			HAL_GPIO_WritePin(in_GPIO, in3_Pin, GPIO_PIN_SET);
			HAL_GPIO_WritePin(in_GPIO, in4_Pin, GPIO_PIN_SET);
		}
	}
	
	if (motorProt == 0) {
		if (sentido == horario) {
			HAL_GPIO_WritePin(in_GPIO, in1_Pin, GPIO_PIN_SET);
			HAL_GPIO_WritePin(in_GPIO, in2_Pin, GPIO_PIN_RESET);
		}
		else if (sentido == antiHorario) {
			HAL_GPIO_WritePin(in_GPIO, in1_Pin, GPIO_PIN_RESET);
			HAL_GPIO_WritePin(in_GPIO, in2_Pin, GPIO_PIN_SET);
		}
	}
	else if (motorProt == 1) {
		if (sentido == horario) {
			HAL_GPIO_WritePin(in_GPIO, in3_Pin, GPIO_PIN_SET);
			HAL_GPIO_WritePin(in_GPIO, in4_Pin, GPIO_PIN_RESET);
		}
		else if (sentido == antiHorario) {
			HAL_GPIO_WritePin(in_GPIO, in3_Pin, GPIO_PIN_RESET);
			HAL_GPIO_WritePin(in_GPIO, in4_Pin, GPIO_PIN_SET);
		}
	} 
}

//TMP functions

void setupLEDs()
{
	__GPIOD_CLK_ENABLE();
	GPIO_InitTypeDef GPIO_InitStructure;

	GPIO_InitStructure.Pin = GPIO_PIN_12 | GPIO_PIN_13 | GPIO_PIN_14 | GPIO_PIN_15;
	GPIO_InitStructure.Mode = GPIO_MODE_OUTPUT_PP;
	GPIO_InitStructure.Speed = GPIO_SPEED_FREQ_HIGH;
	GPIO_InitStructure.Pull = GPIO_NOPULL;
	HAL_GPIO_Init(GPIOD, &GPIO_InitStructure);
}
void I2Csetup()
{
	__GPIOB_CLK_ENABLE();
	
	GPIO_InitTypeDef GPIO_I2Ccfg;
	GPIO_I2Ccfg.Pin = GPIO_PIN_10 | GPIO_PIN_11;
	GPIO_I2Ccfg.Mode = GPIO_MODE_AF_OD;
	GPIO_I2Ccfg.Speed = GPIO_SPEED_FREQ_VERY_HIGH;
	GPIO_I2Ccfg.Pull = GPIO_NOPULL;
	GPIO_I2Ccfg.Alternate = GPIO_AF4_I2C2;
	HAL_GPIO_Init(GPIOB, &GPIO_I2Ccfg);

	__HAL_RCC_I2C2_CLK_ENABLE();
	
	hI2C.Instance = I2C2;
	hI2C.Init.ClockSpeed = 100000;
	hI2C.Init.DutyCycle = I2C_DUTYCYCLE_16_9;
	hI2C.Init.AddressingMode = I2C_ADDRESSINGMODE_7BIT;
	hI2C.Init.OwnAddress1 = 0;
	hI2C.Init.DualAddressMode = I2C_DUALADDRESS_DISABLE;
	hI2C.Init.OwnAddress2 = 0;
	hI2C.Init.GeneralCallMode = I2C_GENERALCALL_DISABLE; 
	hI2C.Init.NoStretchMode =  I2C_NOSTRETCH_DISABLE; 
	HAL_I2C_Init(&hI2C);
}

void TMP100_configuration(I2C_HandleTypeDef* i2c_instance, uint8_t TMP_resolution) {
	uint32_t TMP100_SA = 0x92;
	uint16_t TMP100_CR = 0X01;
	
	//Configura a resolução do TMP100
	HAL_I2C_Mem_Write(i2c_instance, TMP100_SA, TMP100_CR, 1, &TMP_resolution, 1, HAL_MAX_DELAY);
}

float TMP100_readTemperature(I2C_HandleTypeDef* i2c_instance)
{
	uint32_t TMP100_SA = 0x92;
	uint16_t TMP100_TR = 0X00;
	
	uint8_t temperatureBytes[2] = {0, 0};
	//Lê a temperatura e guarda em 2 bytes
	HAL_I2C_Mem_Read(i2c_instance, TMP100_SA, TMP100_TR, 1, temperatureBytes, 2, HAL_MAX_DELAY);
	
	//Formata os dados do buffer TR do TMP100
	return ((float)temperatureBytes[0] + ((float)temperatureBytes[1]) / 256);
}

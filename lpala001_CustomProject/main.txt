/* Lenze Palaganas - lpala001@ucr.edu
 * Lab Section 023
 * Custom Lab: NES Style 8-bit RPG
 * 
 */ 


#include <avr/io.h>
#include <avr/eeprom.h>
#include <avr/interrupt.h>

#include <stdio.h>
#include "io.c"
//#include "usart.h"

uint16_t  EEMEM SAVE_ADDRESS;

unsigned char snapshot;

unsigned char spot = 1;



unsigned char health = 5;
unsigned char wisdom = 5;
unsigned char strength = 5;
unsigned char speed = 5;

unsigned char eHealth1 = 7;


enum title_screen_states{init1,title,start,wait1,story1,story2,story3,story4,transit1}titleScreenState;
unsigned long titleTime = 0;
const unsigned long timePeriod = 200;
unsigned char arr1[] = {"The village is under a plague"};
unsigned char arr2[] = {"You're one of the few who live"};
unsigned char arr3[] = {"The city is just past the forest"};
unsigned char arr4[] = {"You must hurry and buy medicine!"};


enum custom_character_states{init2,custom1,choto1,custom2,choto2,custom3,choto3,custom4,choto4,custom5,choto5,transit2,transit3}CCState;
unsigned long customTime = 0;
const unsigned long customPeriod = 200;
	
	
enum GA_states{init3,YAD,GA}GAState;
unsigned long yadTime = 0;
const unsigned long yadPeriod = 200;

enum d1{init4,crossRoad1,crossRoad2,home,obs1,o1wait,oc1,oc2,obs2,o2choice,run,fail,o2wait,move}d1State;
unsigned long d1Time = 0;
const unsigned long d1Period = 200;

enum fight1{nfight,fight,strike}fightState;
unsigned char fightOn = 0;

enum save{init5}saveState;
	
unsigned char buttonHeld;
unsigned char button1;
unsigned char button2;

enum b1State{inb1,bnp,bp} b1State;

void fight_tick()
{
	switch(fightState)
	{
		case nfight:
			if(fightOn == 1)
			{
				fightState = fight;
				break;
			}
			else
			{
				fightState = nfight;
				break;
			}
			break;
		case fight:
			if(health <= 2)
			{
				ga_tick();
				break;
			}
			if(eHealth1 <= 2)
			{
				fightOn = 0;
				fightState = nfight;
				d1State = move;
				break;			
			}
			if(snapshot == 0x01 && buttonHeld)
			{
				fightState = strike;
				break;
			}
			else
			{
				fightState = fight;

				break;
			}
			break;
		case strike:
			if(snapshot == 0x01 && buttonHeld)
			{
				if(strength > 7)
				{
					--eHealth1;
					--eHealth1;
					--eHealth1;
					--health;
					--health;
					fightState = fight;
					break;
				}
				else if (strength > 3 && strength <= 7)
				{
					--eHealth1;
					--eHealth1;
					--health;
					--health;
					fightState = fight;
					break;
				}
				else if(strength <= 3)
				{
					--eHealth1;
					--health;
					--health;
					fightState = fight;
					break;
				}
				else if (eHealth1 <= 2)
				{				
					fightOn = 0;
					fightState = nfight;
					d1State = move;
					break;
				}
				
			}
			else
			{
				fightState = strike;
				break;
			}
	}
	switch(fightState)
	{
		case nfight:
			break;
		case fight:
			{
				LCD_DisplayString(1,"Attack(A)");
				break;
			}
		case strike:
		{
				LCD_DisplayString(1,"Both of you attack!");
				break;
		}
	}
}

void d1_tick()
{
	switch(d1State)
	{
		case init4:
		{
			d1State = crossRoad1;
			break;
		}
		case crossRoad1:
		{
			if(d1Time >= 4000)
			{
				d1Time = 0;
				d1State = crossRoad2;
				break;
			}
			else
			{
				d1Time += d1Period;
				d1State = crossRoad1;
				break;
			}
			break;
		}
		case crossRoad2:
		{
			if(buttonHeld && snapshot == 0x10)
			{
				d1State = obs1;
				break;
			}
			else if(buttonHeld && snapshot == 0x20)
			{
				d1State = home;
				break;
			}
			else
			{
				d1State = crossRoad2;
				break;
			}
			break;
		}
		case home:
			ga_tick();
			break;
		case obs1:
			if(d1Time >= 4000)
			{
				d1Time = 0;
				d1State = o1wait;
				break;
			}
			else
			{
				d1Time += d1Period;
				d1State = obs1;
				break;
			}
			break;
		case o1wait:
		if(snapshot == 0x01 && buttonHeld)
		{
			if(strength >= 8)
			{
				d1State = move;
				break;
			}
			else
			{
				d1State = oc1;
				break;
			}
			break;
		}
		else if(snapshot == 0x02 && buttonHeld)
		{
			d1State = obs2;
			break;
		}
		break;
		case oc1:
		{
			if(d1Time >= 4000)
			{
				d1Time = 0;
				d1State = o1wait;
				break;
			}
			else
			{
				d1Time += d1Period;
				d1State = oc1;
				break;
			}
			break;
		}
		case move:
			if(buttonHeld)
			{
				d1State = move;
				break;
			}
			else
			{
				d1State = move;
				break;
			}
		case obs2:
			if(d1Time >= 4000)
			{
				d1Time = 0;
				d1State = o2choice;
				break;
			}
			else
			{
				d1Time += d1Period;
				d1State = obs2;
				break;
			}
			break;
		case o2choice:
		{
			if(snapshot == 0x10 && buttonHeld)
			{
				d1State = o2wait;
				break;
			}
			else if (snapshot == 0x20 && buttonHeld)
			{
				d1State = run;
				break;
			}
		}
		case o2wait:
		
			fightOn =1;
			fight_tick();
			break;
		case run:
			if(speed >= 8 || wisdom >= 8)
			{
				d1State = move;
				break;
			}
			else
			{
				d1State = fail;
				break;
			}
			break;
		case fail:
		{
			if(snapshot == 0x01 && buttonHeld)
			{
				d1State = o2wait;
				break;
			}
			else 
			{
				d1State = fail;
				break;
			}
			break;
		}
		default:
			d1State = init4;
			break;
	}
	switch(d1State)
	{
		case init4:
		{
			break;
		}
		case crossRoad1:
		{
			LCD_DisplayString(1, "You are at a crossroad,ready?");
			break;
		}
		case crossRoad2:
		{
			LCD_DisplayString(1,"Forest(UP) Home(DOWN)");
			break;
		}
		case home:
		{
			LCD_DisplayString(1,"You decided not to go...a shame");
			break;
		}
		case obs1:
		{
			LCD_DisplayString(1,"A log blocks your way");
			break;
		}
		case o1wait:
		{
			LCD_DisplayString(1,"Lift it(A) Turn away(B)");
			break;
		}
		case oc1:
		{
			LCD_DisplayString(1,"You are too weak");
			break;
		}
		case obs2:
		{
			LCD_DisplayString(1,"You have encountered a bandit!");
			break;
		}
		case o2choice:
		{
			LCD_DisplayString(1,"Fight(UP) Run(DOWN)");
			break;
		}
		case fail:
			LCD_DisplayString(1,"You were unable to escape");
			break;
		case move:
		{
			LCD_DisplayString(1,"You went to a new area!");
			break;
		}
	}
}

void ga_tick()
{
	switch(GAState)//Transitions
	{
		case init3:
			GAState = YAD;
			break;
		case YAD:
			if(yadTime >= 4000)
			{
				yadTime = 0;
				GAState = GA;
				break;
			}
			else
			{
				yadTime += yadPeriod;
				GAState = YAD;
				break;
			}
			break;
		case GA:
			if(buttonHeld)
			{
		
				title_tick();
				break;
			}
			else
			{
				GAState = GA;
				break;
			}
			break;
		default:
			GAState = init3;
			break;
	}
	switch(GAState)//Actions	
	{
		case init3:
			break;
		case YAD:
			LCD_DisplayString(1,"You Failed");
			break;
		case GA:
			LCD_DisplayString(1,"Game Over");
			break;
	}
}

void custom_tick()
{
	switch(CCState) //Transitions
	{
		case init2:
			CCState = custom1;
			break;
		case custom1:
			if(buttonHeld && snapshot == 0x01)
			{
				//button1;
				++health;
				++health;
				++speed;
				++strength;
				CCState = choto1;
				//break;
			}
			else if(buttonHeld && snapshot == 0x02)
			{
				//button2;
				--health;
				--health;
				++wisdom;
				++wisdom;
				--speed;
				CCState = choto1;
				//break;
			}
			else
			{
				CCState = custom1;
			}
			break;
		case choto1:
			if(!buttonHeld)
			{
				CCState = custom2;
				//break;
			}
			else
			{
				CCState = choto1;
				//break;
			}
			break;	
		case custom2:
			if(buttonHeld && snapshot == 0x01)
			{
				//!button1;
				--strength;
				++speed;
				++speed;
				CCState = choto2;
			}
		
			else if(buttonHeld && snapshot == 0x02)
			{
				//!button2;
				++strength;
				++strength;
				--speed;
				CCState = choto2;
			}
			else
			{
				CCState = custom2;
			}
			break;// missing break lol			
		case choto2:
			if(!buttonHeld)
			{
				CCState = custom3;
				//break;
			}
			else
			{
				CCState = choto2;
				//break;
			}
			break;
		case custom3:
			if(buttonHeld && snapshot == 0x01)
			{
				//!button1;
				++wisdom;
				++wisdom;
				CCState = choto3;
			}
			else if(buttonHeld && snapshot == 0x02)
			{
				//!button2;
				--wisdom;
				--wisdom;
				CCState = choto3;
			}
			else
			{
				CCState = custom3;
			}
			break;
		case choto3:
			if(!buttonHeld)
			{
				CCState = custom4;
				//break;
			}
			else
			{
				CCState = choto3;
				//break;
			}
			break;
		case custom4:
			if(buttonHeld && snapshot == 0x01)
			{
				//!button1;
				--health;
				--strength;
				CCState = choto4;
			}
			else if(buttonHeld && snapshot == 0x02)
			{
				//!button2;
				++health;
				++strength;
				CCState = choto4;
			}
			else
			{
				CCState = custom4;
			}
			break;
		case choto4:

			if(!buttonHeld)
			{
				CCState = custom5;
				//break;
			}
			else
			{
				CCState = choto4;
				//break;
			}
			break;
		case custom5:
			if(buttonHeld && snapshot == 0x01)
			{
				//!button1;
				health = 0;
				CCState = choto5;
				break;
			}
			else if(buttonHeld && snapshot == 0x02)
			{
				++strength;
				++health;
				CCState = choto5;
				break;
			}
			else
			{
				CCState = custom5;
				break;
			}
			break;
		case choto5:
			if(health == 0)
			{
				CCState = transit2;
				break;
				
			}

			else
			{
				
				CCState = transit3;
				break;
			}
			break;
		case transit2:
			ga_tick();
			break;
		case transit3:
			d1_tick();
			break;
		default:
			CCState = init2;
			break;
	}
	switch(CCState) //Actions
	{
		case init2:
			break;
		case custom1:
			LCD_DisplayString(1,"Are you young(A) or old(B)?");
			break;
		case choto1:
			break;
		case custom2:
			LCD_DisplayString(1,"Are you small(A) or large(B)?");
			break;
		case choto2:
			break;
		case custom3:
			LCD_DisplayString(1,"Are you sharp(A) or dull(B)?");
			break;
		case choto3:
			break;
		case custom4:
			LCD_DisplayString(1,"Are you weak(A) or strong(B)?");
			break;
		case choto4:
			break;
		case custom5:
			LCD_DisplayString(1,"Are you ill(A) or healthy(B)?");
			break;
		case transit2:
		break;
	}
}


void title_tick()
{
	switch(titleScreenState) //Transitions
	{
		case init1:
			titleScreenState = title;
			break;
		case title:
			if(titleTime >= 4000)
			{
				titleTime = 0;
				titleScreenState = start;
				break;
			}
			else
			{
				titleTime += timePeriod;
				titleScreenState = title;
				break;
			}
		case start:
			if(buttonHeld == 1)
			{

				titleScreenState = wait1;
				break;
			}
			else
			{
				titleScreenState = start;
				break;
			}
		case wait1:
			if(buttonHeld == 1)
			{
				titleScreenState = wait1;
			}
			else if(snapshot == 0x00)
			{
				titleScreenState = story1;
			}
		case story1:
			if(titleTime >= 4000)
			{
				titleTime = 0;
				titleScreenState = story2;
			}

			else 
			{
				titleTime += timePeriod;
				titleScreenState = story1;
				break;
			}


		case story2:
			if(titleTime >= 4000)
			{
				titleTime = 0;
				titleScreenState = story3;
			}

			else
			{
				titleTime += timePeriod;
				titleScreenState = story3;
				break;
			}
		case story3:
			if(titleTime >= 4000)
			{
				titleTime = 0;
				titleScreenState = story4;
			}

			else
			{
				titleTime += timePeriod;
				titleScreenState = story3;
				break;
			}
		case story4:
			if(buttonHeld)
			{
	
				//custom_tick();
				//Call tick
				//exit(0);
				titleScreenState = transit1;
				break;
				
			}
			else
			{
				titleScreenState = story4;
				break;
			}
		case transit1:

				custom_tick();
				break;
		default:
		
			titleScreenState = init1;
			break;

	}
	switch(titleScreenState) //Actions
	{
		case init1:
			break;
		case title:
			LCD_DisplayString(1, "Lone Traveller");
			break;
		case start:
			LCD_DisplayString(1, "Play(Press Any Button)");
			break;
		case wait1:
			break;
		case story1:
			LCD_DisplayString(1,arr1);
			break;

		case story2:
			LCD_DisplayString(1,arr2);
			break;

		case story3:
			LCD_DisplayString(1,arr3);
			break;

		case story4:
			LCD_DisplayString(1,arr4);
			break;
			//Write out the theme and script of the story
	}
}

volatile unsigned char TimerFlag = 0; // TimerISR() sets this to 1. C programmer should clear to 0.

// Internal variables for mapping AVR's ISR to our cleaner TimerISR model.
unsigned long _avr_timer_M = 1; // Start count from here, down to 0. Default 1 ms.
unsigned long _avr_timer_cntcurr = 0; // Current internal count of 1ms ticks

void TimerOn() {
	// AVR timer/counter controller register TCCR1
	TCCR1B = 0x0B;// bit3 = 0: CTC mode (clear timer on compare)
	// bit2bit1bit0=011: pre-scaler /64
	// 00001011: 0x0B
	// SO, 8 MHz clock or 8,000,000 /64 = 125,000 ticks/s
	// Thus, TCNT1 register will count at 125,000 ticks/s

	// AVR output compare register OCR1A.
	OCR1A = 125;	// Timer interrupt will be generated when TCNT1==OCR1A
	// We want a 1 ms tick. 0.001 s * 125,000 ticks/s = 125
	// So when TCNT1 register equals 125,
	// 1 ms has passed. Thus, we compare to 125.
	// AVR timer interrupt mask register
	TIMSK1 = 0x02; // bit1: OCIE1A -- enables compare match interrupt

	//Initialize avr counter
	TCNT1=0;

	_avr_timer_cntcurr = _avr_timer_M;
	// TimerISR will be called every _avr_timer_cntcurr milliseconds

	//Enable global interrupts
	SREG |= 0x80; // 0x80: 1000000
}

void TimerOff() {
	TCCR1B = 0x00; // bit3bit1bit0=000: timer off
}

void TimerISR() {
	TimerFlag = 1;
}

// In our approach, the C programmer does not touch this ISR, but rather TimerISR()
ISR(TIMER1_COMPA_vect) {
	// CPU automatically calls when TCNT1 == OCR1 (every 1 ms per TimerOn settings)
	_avr_timer_cntcurr--; // Count down to 0 rather than up to TOP
	if (_avr_timer_cntcurr == 0) { // results in a more efficient compare
		TimerISR(); // Call the ISR that the user uses
		_avr_timer_cntcurr = _avr_timer_M;
	}
}

// Set TimerISR() to tick every M ms
void TimerSet(unsigned long M) {
	_avr_timer_M = M;
	_avr_timer_cntcurr = _avr_timer_M;
}

// 0.954 hz is lowest frequency possible with this function,
// based on settings in PWM_on()
// Passing in 0 as the frequency will stop the speaker from generating sound




// this is where it all happens as far as grabbing the NES control pad data
void GetNESControllerData() {

	PORTA = 0x04; //set NESLatch to 1
	PORTA = 0x00; //set NESLatch to 0
	
	snapshot = 0x00;
	for (unsigned char i = 0; i <= 7; ++i) {
		snapshot = (~PINA & 0x01) << i | snapshot;
		PORTA = 0x02; //set NESClock to 1
		PORTA = 0x00; //set NESClock to 0
	}
}


/////	The NES code was received on GitHub and belongs to Andrew Juarez
/////	I found it easy to understand and edited it heavily to suit the necessary
/////	actions for my RPG

void NESControllerTick() {
	// 	NES_A       B00000001
	// 	NES_B       B00000010
	//  NES_SELECT  B00000100
	// 	NES_START   B00001000
	// 	NES_UP      B00010000
	// 	NES_DOWN    B00100000
	// 	NES_LEFT    B01000000
	// 	NES_RIGHT   B10000000
	GetNESControllerData();
	if (snapshot == 0x00 && buttonHeld) {
		buttonHeld = 0;
	}

	else if(snapshot == 0x01 && !buttonHeld) { //A is pressed
		buttonHeld = 1;
	}
	else if(snapshot == 0x02 && !buttonHeld) { //B is pressed
		buttonHeld = 1;
	}
	else if(snapshot == 0x10 && !buttonHeld) { //up is pressed
		buttonHeld = 1;
	}
	else if(snapshot == 0x20 && !buttonHeld) { //down is pressed
		buttonHeld = 1;
	}
	else if(snapshot == 0x08 && !buttonHeld) { //START is pressed
		buttonHeld = 1;
			//save

			
			eeprom_update_word(&SAVE_ADDRESS,health);
			eeprom_update_word(&SAVE_ADDRESS,wisdom);
			eeprom_update_word(&SAVE_ADDRESS,strength);
			eeprom_update_word(&SAVE_ADDRESS,speed);
			eeprom_update_word(&SAVE_ADDRESS,d1State);

	}
	else if(snapshot == 0x04 && !buttonHeld) { //START is pressed
		buttonHeld = 1;
		//save
	
		eeprom_read_word(&SAVE_ADDRESS);
	
		//eeprom_read_word(&SAVE_ADDRESS,health);
		//eeprom_read_word(&SAVE_ADDRESS,wisdom);
		//eeprom_read_word(&SAVE_ADDRESS,strength);
		//eeprom_read_word(&SAVE_ADDRESS,speed);
		//eeprom_read_word(&SAVE_ADDRESS,d1State);

	}
	else if(snapshot == 0x04 && !buttonHeld) { //Select is pressed
		buttonHeld = 1;
		//softReset = 1;
	}

}

void health_tick()
{
	if(health == 8)
	{
		PORTB = 0xFF;
	}
	else if(health == 7)
	{
		PORTB = 0xFE;
	}
	else if(health == 6)
	{
		PORTB = 0xFC;
	}
	else if(health == 5)
	{
		PORTB = 0xF8;
	}
	else if(health == 4)
	{
		PORTB = 0xF0;
	}
	else if(health == 3)
	{
		PORTB = 0xE0;
	}
	else if(health == 2)
	{
		PORTB = 0xC0;
	}
	else if(health == 1)
	{
		PORTB = 0x80;
	}
	else if(health == 0)
	{
		PORTB = 0x00;
	}
}

int main(void)
{
	DDRC = 0xFF; PORTC = 0x00; // LCD data lines
	DDRD = 0xFF; PORTD = 0x00; // LCD control lines
	DDRB = 0xFF; PORTB = 0x00; // Output: tbd
	DDRA = 0b11110110; PORTA = 0xFF; // Input: buttons
	
	
	titleScreenState = init1;
	CCState = init2;
	GAState = init3;
	d1State = init4;
	
	buttonHeld = 0;
	
	GetNESControllerData();
	
	TimerSet(timePeriod);
	TimerOn();
	//PWM_on();
	
	LCD_init();
	
	// Starting at position 1 on the LCD screen, writes Hello World
			//LCD_DisplayString(1, "Lone Traveller  Press button...");
			//LCD_DisplayString(16,"Click any button to start");
	
	while(1) {

		//button1 = ~PINA & 0x01;
		//button2 = ~PINA & 0x02;
		NESControllerTick();
		health_tick();
		//tick();
		title_tick();


		//custom_tick();
		while (!TimerFlag){};	
		TimerFlag = 0;
		//titleTime += timePeriod;
	}
	
}

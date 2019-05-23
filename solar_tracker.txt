#include "msp430.h"
#define ADC_CH 4
unsigned int samples[ADC_CH];

#define SENS_LEFT BIT0
#define SENS_RIGHT BIT1
#define SENS_UP BIT2
#define SENS_DOWN BIT3

#define MCU_CLOCK           1000000
#define PWM_FREQUENCY       46
#define SERVO_STEPS         45


unsigned int PWM_Period     = (MCU_CLOCK / PWM_FREQUENCY);  // PWM Period
unsigned int PWM_Duty       = 0;                            // %
int i ;
int count  ;
int count2  ;

void configureADC(void){

ADC10CTL1 = INCH_3 | ADC10DIV_0 | CONSEQ_3 | SHS_0;

ADC10CTL0 = SREF_0 | ADC10SHT_2 | MSC | ADC10ON | ADC10IE;

ADC10AE0 =SENS_LEFT + SENS_RIGHT + SENS_UP + SENS_DOWN ;

ADC10DTC1 = ADC_CH;

}
void configureLED(void)
{
    if (samples[3]>550)
    {   P2OUT&=~(0x3B);
        P2OUT|=0x01;
        __delay_cycles(10000);
    }
    if (samples[3]<550)
       {   P2OUT&=~(0x3B);

           __delay_cycles(10000);
       }
    if (samples[3]>600)
    {   P2OUT&=~(0x3B);
        P2OUT|=0x03;
        __delay_cycles(10000);
    }
    if (samples[3]>650)
        {   P2OUT&=~(0x3B);
            P2OUT|=0x0B;
            __delay_cycles(10000);
        }
    if (samples[3]>700)
        {   P2OUT&=~(0x3B);
            P2OUT|=0x1B;
            __delay_cycles(10000);
        }
    if (samples[3]>750)
        {   P2OUT&=~(0x3B);
            P2OUT|=0x3B;
            __delay_cycles(10000);
        }
}

void main(void) {

WDTCTL = WDTPW | WDTHOLD;

        TA1CCTL1    = OUTMOD_7 ;
        TA0CCTL1    = OUTMOD_7  ;

        TA1CTL  = TASSEL_2 + MC_1 ;
        TA0CTL  = TASSEL_2 + MC_1 ;

        TA1CCR0 = PWM_Period-1;        // PWM Period
        TA0CCR0 = PWM_Period-1 ;

        TA1CCR1 = 0 ;            // TACCR1 PWM Duty Cycle
        TA0CCR1 = 0 ;

P1DIR = 0; /* set as inputs */
P1SEL = 0; /* set as digital I/Os */
P1OUT = 0; /* set resistors as pull-downs */

P1REN |= (SENS_LEFT|SENS_RIGHT|SENS_DOWN|SENS_UP); /* enable pull-up on SENSOR */

        P1DIR   |= BIT6  ;
        P2DIR   |= BIT2 ;

        P1SEL   |= BIT6  ;
        P2SEL   |= BIT2  ;
P2DIR|=0x3B;
P2OUT&=~(0x3B);
configureADC();
__enable_interrupt();
while (1) {

    __delay_cycles(1000);
    ADC10CTL0 &= ~ENC;
    while (ADC10CTL1 & BUSY);
    ADC10SA = (unsigned int)samples;
    ADC10CTL0 |= ENC + ADC10SC;

    __bis_SR_register(CPUOFF + GIE);

    for (i = 0; i < SERVO_STEPS; i++) {
        TA1CCR1 = count2 ;
        TA0CCR1 = 2*count ;
        __delay_cycles(20000);
        TA1CCR1 = count2 ;
        TA0CCR1 = 2*count ;
         __delay_cycles(20000);
        }
    }
}

#pragma vector = ADC10_VECTOR
__interrupt void ADC10_ISR (void){



if (samples[0] < samples[1]){
    if(samples[1]> 550){
        count = 850;
        }else {
            count = 1350;
            }} else if ((samples[0] + samples[1])/2 < 500) {
                count = 1350;
                 } else {
    if(samples[0]>550){
        count = 1850;
        }else {
            count = 1350;
            }}
if (samples[2] < samples[3]) {
    if(samples[3]> 550){
        count2 =850;
        }else {
            count2 = 1150;
            }
            } else if ((samples[2] + samples[3])/2 < 500) {
                count2 = 1150;
                } else {
    if(samples[2]>550){
        count2 = 1850;
        }else {
        count2 = 1150;
        }
        }
configureLED();

__bic_SR_register_on_exit(CPUOFF);
    }

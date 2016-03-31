# Embedded


targil3

#include <p32xxxx.h>
#pragma config FPLLMUL = MUL_20, FPLLIDIV = DIV_2, FPLLODIV = DIV_1, FWDTEN = OFF
#pragma config POSCMOD = HS, FNOSC = PRIPLL, FPBDIV = DIV_8

void busy(void);
unsigned int portMap;
unsigned int portMap1;
unsigned int portMap2;
void main(void)
           {
char men[8]={0x04,0x06,0x07,0x04,0x04,0x1F,0x1F,0xE};
char father[1]={0};// סירה
char control[7]={0x38,0x38,0x38,0xe,0x6,0x1,0x40};
// set CGRAM=0x40
	int i, dir, counter;
	int x, s2, s7; 
	int buzz = 1;
    portMap = TRISG;
	portMap &= 0xFFFF7FFF;
	TRISG = portMap;
	PORTGbits.RG15 = 0;//buzzer=0ff
	portMap = TRISB;
	portMap &= 0xFFFF7FFF;
	TRISB = portMap;
 	portMap = TRISF;
	portMap &= 0xFFFFFEF8;
	TRISF = portMap;
	PORTFbits.RF8 = 1;
       
    portMap = TRISE;
	portMap &= 0xFFFFFF00;
	TRISE = portMap;
       
    portMap = TRISD;
	portMap &= 0xFFFFFFCF;
	TRISD = portMap;

	PORTF = 0x00;
	for(i = 0;i < 7;i++)
     {
      PORTDbits.RD5 = 0;//w/r
      PORTBbits.RB15 = 0;//rs 
      PORTE=control[i];
        PORTDbits.RD4=1;//enable=1
        PORTDbits.RD4=0;//enable=0
          busy();
	}
       PORTFbits.RF8 = 1;
       PORTF = 0x00;
        for(i = 0;i < 8;i++)
           {
         PORTDbits.RD5 = 0;//w/r
        PORTBbits.RB15 = 1;//rs 
        PORTE=men[i];
        PORTDbits.RD4=1;//enable=1
        PORTDbits.RD4=0;//enable=0
        busy(); } 

	  PORTDbits.RD5 = 0;//w/r
	  PORTBbits.RB15 = 0;//rs control 
	  PORTE=0x80;//DDRAM
	  PORTDbits.RD4=1;//enable=1
	  PORTDbits.RD4=0;//enable=0
	  busy();

        PORTDbits.RD5 = 0;//w/r
        PORTBbits.RB15 = 1;//rs 
        PORTE=0;
        PORTDbits.RD4=1;//enable=1
        PORTDbits.RD4=0;//enable=0
        busy();
		counter = 0;
		while (1)
		{

			// Get buzzer
			portMap1 = TRISE;
			portMap2 = PORTF;
			TRISE=0xff;
			PORTF=3;
			PORTDbits.RD4=1;
			PORTDbits.RD4=0; 
			x=PORTE;
		
			TRISE = portMap1;
			PORTF = portMap2;	
			TRISE =portMap1;
			PORTF = portMap2;
		
			s2=((x&4)!=0);
			s7=((x&128)!=0);

			// Get direction
			dir = getDir();
			if  (dir == 0 && counter == 16)
			{
				// Move back to 0
				for(i=0;i<16;i++)
				{
					PORTDbits.RD5 = 0;//w/r
				    PORTBbits.RB15 = 0;//rs 
				    PORTE = 0x1B;
				    PORTDbits.RD4=1;//enable=1
				    PORTDbits.RD4=0;//enable=0
				    busy();
				}
					
				counter = 0;
			}
			else if  (dir == 1 && counter == 0)
			{
				// Move back to 16
				for(i=0;i<16;i++)
				{
					PORTDbits.RD5 = 0;//w/r
				    PORTBbits.RB15 = 0;//rs 
				    PORTE = 0x1F;
				    PORTDbits.RD4=1;//enable=1
				    PORTDbits.RD4=0;//enable=0
				    busy();
				}
				counter = 16;
			}

			PORTDbits.RD5 = 0;//w/r
		    PORTBbits.RB15 = 0;//rs 
			if (dir == 0)
			{
		    	PORTE = 0x1F;
				counter++;
			}
			else
			{
				PORTE = 0x1B;
				counter--;
			}
		    PORTDbits.RD4=1;//enable=1
		    PORTDbits.RD4=0;//enable=0
		    busy();

			// Activate buzzer
			/*if (s2 == 0)
			{
				portMap = TRISG;
				portMap1 = PORTGbits.RG15;
				TRISG = 0;
				PORTGbits.RG15 = buzz;
				buzz = !buzz;
				PORTDbits.RD4 = 1;
				PORTDbits.RD4 = 0;
				TRISG = portMap;
				PORTGbits.RG15 = portMap1;

			}
*/
			for (i = 0; i < 128000; i++);
		}
	}

int getDir()
{
	int x, s0;
	unsigned int portMap1;
	unsigned int portMap2;

	// Receive switches input
	portMap1 = TRISE;
	portMap2 = PORTF;
	TRISE=0xff;
	PORTF=3;
	PORTDbits.RD4=1;
	PORTDbits.RD4=0; 
	x=PORTE;

	TRISE = portMap1;
	PORTF = portMap2;	

	// Interpret each switch
	s0=((x&1)!=0);

	// Determine direction
	return(s0==0 ? 0 : 1);
}

void busy(void)
      { char RD,RS;
        int STATUS_TRISE;
    RD=PORTDbits.RD5;
    RS=PORTBbits.RB15;
    STATUS_TRISE=TRISE;
	PORTDbits.RD5 = 1;//w/r
	PORTBbits.RB15 = 0;//rs 
               portMap = TRISE;
	portMap |= 0x80;
	TRISE = portMap;
do
     {
 PORTDbits.RD4=1;//enable=1
 PORTDbits.RD4=0;//enable=0
     }
   while(PORTEbits.RE7); // BF בדיקה דגל
    PORTDbits.RD5=RD; 
    PORTBbits.RB15=RS;
    TRISE=STATUS_TRISE;   
        }
/*
// TODO - handle switches 2 and 7
		TRISE=0xff;
       PORTF=3;
       PORTDbits.RD4=1;
       PORTDbits.RD4=0; 
       x=PORTE;
		s2=((x&4)!=0);
		s7=((x&128)!=0);
	
*/

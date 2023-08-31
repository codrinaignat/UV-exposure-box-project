#include "newxc8_header.h"
#include <xc.h>
#include <stdbool.h>


#define _XTAL_FREQ 10000000
// LCD pins
#define EN PORTAbits.RA4
#define RS PORTAbits.RA5
#define D4 PORTBbits.RB7
#define D5 PORTBbits.RB6
#define D6 PORTBbits.RB5
#define D7 PORTBbits.RB4

#define Relay_Control PORTCbits.RC3

//buttons
#define Increment_Button PORTCbits.RC1
#define Decrement_Button PORTCbits.RC2
#define Start_Button PORTCbits.RC7
#define Stop_Button PORTCbits.RC6

int contor = 0;
char Buffer[3];
int check=false;

void Lcd_SetBit(char data_bit) {
    if (data_bit & 1)
        D4 = 1;
    else
        D4 = 0;

    if (data_bit & 2)
        D5 = 1;
    else
        D5 = 0;

    if (data_bit & 4)
        D6 = 1;
    else
        D6 = 0;

    if (data_bit & 8)
        D7 = 1;
    else
        D7 = 0;
}

void Lcd_Cmd(char a) {
    RS = 0;
    Lcd_SetBit(a); //Incoming Hex value
    EN = 1;
    __delay_ms(5);
    EN = 0;
}

void Lcd_Clear() {
    Lcd_Cmd(0); //Clear the LCD
    Lcd_Cmd(1); //Move the cursor to first position
}

void Lcd_Set_Cursor(char a, char b) {
    char temp;
    char z = 0;
    char y = 0;
    if (a == 1) {
        //(Chapter 1.1)
        temp = 0x80 + b - 1;
        z = temp >> 4; //Lower 8-bits
        y = temp & 0x0F; //Upper 8-bits
        Lcd_Cmd(z); //Set Row
        Lcd_Cmd(y); //Set Column
    } else if (a == 2) {
        //(Chapter 1.2)
        temp = 0xC0 + b - 1;
        z = temp >> 4; //Lower 8-bits
        y = temp & 0x0F; //Upper 8-bits
        Lcd_Cmd(z); //Set Row
        Lcd_Cmd(y); //Set Column
    }
}

void Lcd_Start() {
    //driver controller SPLC780D
    Lcd_SetBit(0x00);
    int i;
    for (i = 32767; i <= 0; i--) NOP();
    Lcd_Cmd(0x03); //1
    __delay_ms(1);
    Lcd_Cmd(0x03); //2
    __delay_ms(1);
    Lcd_Cmd(0x03); //3
    Lcd_Cmd(0x02); //4
    Lcd_Cmd(0x02); //5
    Lcd_Cmd(0x08); //6
    Lcd_Cmd(0x00); //7
    Lcd_Cmd(0x0C); //8
    Lcd_Cmd(0x00); //9
    Lcd_Cmd(0x06); //10

}

void Lcd_Print_Char(char data) //Send 8-bits through 4-bit mode
{
    char Lower_Nibble;
    char Upper_Nibble;
    Lower_Nibble = data & 0x0F; 

    RS = 1; //
    Lcd_SetBit(Upper_Nibble >> 4); // transmit Upper_Nibble dar shiftat cu 4 bit -  0000 1011
    EN = 1;
    int i;
    for (i = 32767; i <= 0; i--) NOP(); 
    EN = 0;
    Lcd_SetBit(Lower_Nibble); 
    EN = 1;
    for (i = 32767; i <= 0; i--) NOP();
    EN = 0;
}

void Lcd_Print_String(char *a) {
    int i;
    for (i = 0; a[i] != '\0'; i++)
        Lcd_Print_Char(a[i]);
}

char* itoa(int i, char b[]) {
    char const digit[10] = "0123456789";
    char* p = b;
    if (i < 0) {
        *p++ = '-';
        i *= -1;
    }
    int shifter = i;
    do { //Move to where representation ends
        ++p;
        shifter = shifter / 10;
    } while (shifter);
    *p = ' '; /
    do {
        *--p = digit[i % 10];
        i = i / 10;
    } while (i);
    return b;
}

void initializing() {
    Lcd_Start();
        Lcd_Clear();
    Lcd_Set_Cursor(1, 1);
    Lcd_Print_String("Booting.");
    __delay_ms(1);
    Lcd_Set_Cursor(2, 1);
    Lcd_Print_String("...");
    __delay_ms(666);
    Lcd_Clear();
    Lcd_Set_Cursor(1, 1);
    Lcd_Print_String("Booting.");
    __delay_ms(1);
    Lcd_Set_Cursor(2, 1);
    Lcd_Print_String("..");
    __delay_ms(666);
    Lcd_Clear();
    Lcd_Set_Cursor(1, 1);
    Lcd_Print_String("Booting.");
    __delay_ms(1);
    Lcd_Set_Cursor(2, 1);
    Lcd_Print_String(".");
    __delay_ms(666);
    Lcd_Clear();
    Lcd_Set_Cursor(1, 1);
    Lcd_Print_String("Booting.");
    __delay_ms(666);
    Lcd_Clear();
    Lcd_Set_Cursor(1, 1);
    Lcd_Print_String("UVBox v.");
    __delay_ms(1);
    Lcd_Set_Cursor(2, 1);
    Lcd_Print_String("088 Beta ");
    __delay_ms(999);
    Lcd_Clear();
    Lcd_Set_Cursor(1, 1);
    Lcd_Print_String("EMC Engi");
    __delay_ms(1);
    Lcd_Set_Cursor(2, 1);
    Lcd_Print_String("neering");
    __delay_ms(999);
    
    
}
void main(void) {
    OSCCON = 0b01110010;
    TRISA = 0;
    TRISB = 0;
    ANSELA = 0;
    ANSELB = 0;
    ANSELC = 0;
    TRISCbits.TRISC1 = 1; //increment
    TRISCbits.TRISC2 = 1; //decrement
    TRISCbits.TRISC7 = 1; //start
    TRISCbits.TRISC6 = 1; //stop
    TRISCbits.TRISC3 = 0; //relay_control
    Relay_Control
    initializing();
    contor=101;
    int i;
    Lcd_Clear();
    itoa(contor, Buffer);
    Lcd_Set_Cursor(1, 1);
        for (i = 0; i < 3; i++) 
        {
            Lcd_Print_Char(Buffer[i]);
        }
    Lcd_Set_Cursor(1, 5);
    Lcd_Print_String("seco");
    Lcd_Set_Cursor(2, 1);
    Lcd_Print_String("nds");
    int j;
    while (contor < 300) {
            if (Increment_Button == 0) {
                Lcd_Clear();
                if (contor == 299)
                {

                    Lcd_Clear();
                    Lcd_Set_Cursor(1, 1);
                    Lcd_Print_String("Maximum!");
                }
                else
                {
                Lcd_Clear();
                contor = contor + 1;
                itoa(contor, Buffer);
                
                Lcd_Clear();
                Lcd_Set_Cursor(1, 1);
                for (j = 0; j < 3; j++) {
                    Lcd_Print_Char(Buffer[j]);
                }
                Lcd_Set_Cursor(1, 5);
                Lcd_Print_String("seco");
                Lcd_Set_Cursor(2, 1);
                Lcd_Print_String("nds");
                __delay_ms(30);

            }
            }
            if (Decrement_Button == 0) {
                Lcd_Clear();
                if (contor < 100)
                {

                    Lcd_Clear();
                    Lcd_Set_Cursor(1, 1);
                    Lcd_Print_String("Min. 100");
                    Lcd_Set_Cursor(2, 1);
                    Lcd_Print_String(" seconds");
                }
                else
                {
                contor = contor - 1;
                itoa(contor, Buffer);

                Lcd_Clear();
                Lcd_Set_Cursor(1, 1);
                for (j = 0; j < 3; j++) {
                    Lcd_Print_Char(Buffer[j]);
                }
                Lcd_Set_Cursor(1, 5);
                Lcd_Print_String("seco");
                Lcd_Set_Cursor(2, 1);
                Lcd_Print_String("nds");
                __delay_ms(30);
            }
            }
            if (Start_Button == 0) {
                Lcd_Clear();
                __delay_ms(500);
                Lcd_Clear();
                if (contor >=0 && contor <=99 )
                {

                    Lcd_Clear();
                    Lcd_Set_Cursor(1, 1);
                    Lcd_Print_String("Min. 100");
                    Lcd_Set_Cursor(2, 1);
                    Lcd_Print_String(" seconds");
                }
                else{

                Lcd_Clear();
                Lcd_Set_Cursor(1, 1);
                Lcd_Print_String("Exposure");
                Lcd_Set_Cursor(2, 1);
                Lcd_Print_String(" started");
                check=true;
                __delay_ms(333);
                while(check==true)
                {
                    Relay_Control=1;
                    Lcd_Clear();
                    if (contor == 0)
                        {
                        
                        Lcd_Clear();
                        Relay_Control=0;
                        Lcd_Set_Cursor(1, 1);
                        Lcd_Print_String("Exposure");
                        Lcd_Set_Cursor(2, 1);
                        Lcd_Print_String(" done!");
                        check=false;
                        __delay_ms(888);
                        Lcd_Clear();
                        Lcd_Set_Cursor(1, 1);
                        for (i = 0; i < 3; i++) 
                            {
                                Lcd_Print_Char(Buffer[i]);
                            }
                        Lcd_Set_Cursor(1, 5);
                        Lcd_Print_String("seco");
                        Lcd_Set_Cursor(2, 1);
                        Lcd_Print_String("nds");
                        break;
                        }
                    contor = contor - 1;
                    itoa(contor, Buffer);
                    //Lcd_Start();
                    Lcd_Clear();
                    Lcd_Set_Cursor(1, 1);
                    for (j = 0; j < 3; j++) {
                    Lcd_Print_Char(Buffer[j]);
                                            }
                    Lcd_Set_Cursor(1, 5);
                    Lcd_Print_String("rema");
                    Lcd_Set_Cursor(2, 1);
                    Lcd_Print_String("ining");
                    while(Stop_Button == 0)
                    {
                        Lcd_Clear();
                        Relay_Control=0;
                        __delay_ms(222);
                        Lcd_Set_Cursor(1, 1);
                        Lcd_Print_String("Exposure");
                        Lcd_Set_Cursor(2, 1);
                        Lcd_Print_String(" stopped");
                        __delay_ms(888);
                        Lcd_Clear();
                        Lcd_Set_Cursor(1, 1);
                        for (i = 0; i < 3; i++) 
                            {
                                Lcd_Print_Char(Buffer[i]);
                            }
                        Lcd_Set_Cursor(1, 5);
                        Lcd_Print_String("seco");
                        Lcd_Set_Cursor(2, 1);
                        Lcd_Print_String("nds");
                        check=false;
                    }
                    __delay_ms(666);
                }
                }
            }

    }
    return;
}

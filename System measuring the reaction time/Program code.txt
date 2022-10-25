#include <inttypes.h>
#include <avr/io.h>
#include <avr/interrupt.h>
#include <avr/sleep.h>
#include <stdlib.h>
#include <util/delay.h>  


//--------------------------------------------------------------------------WYSWIETLANIE LCD
#define LCD_Dir  DDRD		// PRZYPISANIE DDR WYSWIETLACZA
#define LCD_Port PORTD		// PRZYPISANIE PORT WYSWEITLACZA
#define RS PD0				// PRZYPISANIE PINU RS
#define EN PD1 				// PRZYPISANIE PINU EN
										 // PIN RW PODPINAMY DO MASY
										 

void LCD_komenda( unsigned char cmnd )
{
	LCD_Port = (LCD_Port & 0x0F) | (cmnd & 0xF0);				// WYSYŁANIE 4 BARDZIEJ ZNACZACYCH BITOW
	LCD_Port &= ~ (1<<RS);							// RS = 0
	LCD_Port |= (1<<EN);							// EN = 1
		_delay_us(1);	
	LCD_Port &= ~ (1<<EN);							// EN = 0
		_delay_us(200);

	LCD_Port = (LCD_Port & 0x0F) | (cmnd << 4);				// WYSYŁANIE 4 MNIEJ ZNACZACYCH BITOW
	LCD_Port |= (1<<EN);							// EN = 1
		_delay_us(1);
	LCD_Port &= ~ (1<<EN);							// EN = 0
		_delay_ms(2);
}

void LCD_napis( unsigned char data )
{
	LCD_Port = (LCD_Port & 0x0F) | (data & 0xF0);				// WYSYŁANIE 4 BARDZIEJ ZNACZACYCH BITOW
	LCD_Port |= (1<<RS);							// RS = 1
	LCD_Port|= (1<<EN);							// EN = 1
		_delay_us(1);
	LCD_Port &= ~ (1<<EN);							// EN = 0
		_delay_us(200);
	LCD_Port = (LCD_Port & 0x0F) | (data << 4);				// WYSYŁANIE 4 MNIEJ ZNACZACYCH BITOW
	LCD_Port |= (1<<EN);							// EN = 1
		_delay_us(1);
	LCD_Port &= ~ (1<<EN);							// EN = 0
		_delay_ms(2);
}

void LCD_start (void)			
{
	LCD_Dir = 0xFF;								// USTAWIENIE PORTU JAKO WYJSCIA
		_delay_ms(20);
	LCD_komenda(0x02);							// 4 BITOWA INICJALIZACJA
	LCD_komenda(0x28);							// 2 WIERSZE, MACIERZ 5x7 W TRYBIR 4 BITOWYM
	LCD_komenda(0x0c);							// WYŁĄCZENIE KURSORA
	LCD_komenda(0x06);							// PRZESUNIECIE KURSORA W PRAWO
	LCD_komenda(0x01);							// CZYSZCZENIE WYSWIETLANIA
		_delay_ms(2);
}

void LCD_wyswietl (char *str)	
{
	int i;
	for(i=0;str[i]!=0;i++)							// WYSLIJ KAZDY ZNAK LANCUCHA 
	{
		LCD_napis (str[i]);
	}
}

void LCD_wyswietl_pozycja (char wiersz, char kolumna, char *napis)
{
	if (kolumna == 0 && kolumna<16)
		LCD_komenda((kolumna & 0x0F)|0x80);
	else if (wiersz == 1 && kolumna<16)
		LCD_komenda((kolumna & 0x0F)|0xC0);	
		LCD_wyswietl(napis);
}

void LCD_wyczysc()
{
	LCD_komenda (0x01);							// CZYSZCZENIE WYSWIETLACZA
	_delay_ms(2);
	LCD_komenda (0x80);							// KURSOR W DOMYSLNEJ POZYCJI

}



//--------------------------------------------------------------------------------------------MAIN
 

unsigned int  x=0, losowa_liczba, najwiekszy_czas=0, najmniejszy_czas=9999, suma_czasow=0, sredni_czas, liczba_bledow, y=15, z=1;
double czas_ms[] = {0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0}, czas_start;
volatile unsigned int zliczanie;							//uzywamy gdy chcemy korzystac ze zmiennej w przerwaniu 

ISR(TIMER0_COMPB_vect)					//Co ustalony czas przerywa program odswieza kolejny wyswietlacz i wraca do programu
{
zliczanie++;
}





int main()
 { 
      
	LCD_start();

	
//-------------------------------------------------------------	USTAWIENIA WYJSC/WEJSC
      DDRD = 0b11110011; //
      DDRB = 0b00000000; //WEJSCIA
      DDRC = 0b1111111; //WYJSCIA
       
       
//------------------------------------------------------------- USTAWIENIE PRZERWAN

      sei();					//Włączenie przerwań globalnych
      TIMSK0 |= (1<<OCIE0B);			//Włączenie porownania przerwania dla Timera0A	
      TCCR0B = 0b00001001;			//Ustawienie preskalera /16/255 (3 najmlodsze bity), WDM01 (4 bit od prawej, wymusza zliczanie do wartosci OCR0)
      OCR0A = 255;
      
//TIMER WYKONUJE PRZERWANIE 3650 RAZY W CIAGU SEKUNDY 
	 while (1)
	 {	
		liczba_bledow=0;
		x=0;
		
		if (z==1)
		{
		LCD_wyczysc();
		LCD_wyswietl_pozycja(0,4,"WCISNIJ START");
		_delay_ms(200);
		}
		z=0;
		
		
	       if (bit_is_clear(PIND,2)) // Jesli nacisnie sie start
		  { 
		  czas_start = zliczanie;
		  srand(czas_start);
		LCD_wyczysc();
		LCD_wyswietl("PRZYGOTUJ SIE!");
		  _delay_ms(1000);
		  
		LCD_wyczysc();
		LCD_wyswietl_pozycja(1, 8, "3");
		  _delay_ms(1000);
		LCD_wyczysc();
		LCD_wyswietl_pozycja(1, 8, "2");
		  _delay_ms(1000);
		LCD_wyczysc();
		LCD_wyswietl_pozycja(1, 8, "1");
		  _delay_ms(1000);
		LCD_wyczysc();
		
		     while (x<y)
			{

			      losowa_liczba = rand()%6+0;
			      
			      
			      
				       char x_tekst[7];
				       itoa(x, x_tekst, 10);
		
				       LCD_wyczysc();
				       LCD_wyswietl(x_tekst);
				       LCD_wyswietl("  /15");
				       
			      
				    switch(losowa_liczba)
				       {
					     case 0:
						{
						   zliczanie=0;
						   PORTC |= (1<<PC0);			//Podaje stan niski (ON) na wyświetlacz tysięcy
						   while (bit_is_set(PINB,0)) 
						   {
						   if(bit_is_clear(PINB, 1)) {liczba_bledow++;	LCD_wyswietl_pozycja(1, 6, "Blad!");	 while (bit_is_clear(PINB,1)){}; 	_delay_ms(20);	}
						   if(bit_is_clear(PINB, 2)) {liczba_bledow++;	LCD_wyswietl_pozycja(1, 6, "Blad!");	 while (bit_is_clear(PINB,2)){};	_delay_ms(20);	}
						   if(bit_is_clear(PINB, 3)) {liczba_bledow++;	LCD_wyswietl_pozycja(1, 6, "Blad!");	 while (bit_is_clear(PINB,3)){};	_delay_ms(20);	}
						   if(bit_is_clear(PINB, 4)) {liczba_bledow++;	LCD_wyswietl_pozycja(1, 6, "Blad!");	 while (bit_is_clear(PINB,4)){};	_delay_ms(20);	}
						   if(bit_is_clear(PINB, 5)) {liczba_bledow++;	LCD_wyswietl_pozycja(1, 6, "Blad!");	 while (bit_is_clear(PINB,5)){};	_delay_ms(20);	}
						   };
						   czas_ms[x] = zliczanie/3.84;
						   break;				//Wychodzi z funkcji switch i powraca do programu glownego
						}
						
					     case 1:
						  {
						   zliczanie=0;
						   PORTC |= (1<<PC1);			//Podaje stan niski (ON) na wyświetlacz tysięcy
						   while (bit_is_set(PINB,1)) 
						   {
						   if(bit_is_clear(PINB, 0)) {liczba_bledow++;	LCD_wyswietl_pozycja(1, 6, "Blad!");	 while (bit_is_clear(PINB,1)){}; 	_delay_ms(20);	}
						   if(bit_is_clear(PINB, 2)) {liczba_bledow++;	LCD_wyswietl_pozycja(1, 6, "Blad!");	 while (bit_is_clear(PINB,2)){};	_delay_ms(20);	}
						   if(bit_is_clear(PINB, 3)) {liczba_bledow++;	LCD_wyswietl_pozycja(1, 6, "Blad!");	 while (bit_is_clear(PINB,3)){};	_delay_ms(20);	}
						   if(bit_is_clear(PINB, 4)) {liczba_bledow++;	LCD_wyswietl_pozycja(1, 6, "Blad!");	 while (bit_is_clear(PINB,4)){};	_delay_ms(20);	}
						   if(bit_is_clear(PINB, 5)) {liczba_bledow++;	LCD_wyswietl_pozycja(1, 6, "Blad!");	 while (bit_is_clear(PINB,5)){};	_delay_ms(20);	}
						   };
						   czas_ms[x] = zliczanie/3.84;
						   break;				//Wychodzi z funkcji switch i powraca do programu glownego
						   }						
					     case 2:
						{
						   zliczanie=0;
						   PORTC |= (1<<PC2);			//Podaje stan niski (ON) na wyświetlacz tysięcy
						   while (bit_is_set(PINB,2)) 
						   {
						   if(bit_is_clear(PINB, 1)) {liczba_bledow++;	LCD_wyswietl_pozycja(1, 6, "Blad!");	 while (bit_is_clear(PINB,1)){}; 	_delay_ms(20);	}
						   if(bit_is_clear(PINB, 0)) {liczba_bledow++;	LCD_wyswietl_pozycja(1, 6, "Blad!");	 while (bit_is_clear(PINB,2)){};	_delay_ms(20);	}
						   if(bit_is_clear(PINB, 3)) {liczba_bledow++;	LCD_wyswietl_pozycja(1, 6, "Blad!");	 while (bit_is_clear(PINB,3)){};	_delay_ms(20);	}
						   if(bit_is_clear(PINB, 4)) {liczba_bledow++;	LCD_wyswietl_pozycja(1, 6, "Blad!");	 while (bit_is_clear(PINB,4)){};	_delay_ms(20);	}
						   if(bit_is_clear(PINB, 5)) {liczba_bledow++;	LCD_wyswietl_pozycja(1, 6, "Blad!");	 while (bit_is_clear(PINB,5)){};	_delay_ms(20);	}
						   };
						   czas_ms[x] =zliczanie/3.84;
						   break;				//Wychodzi z funkcji switch i powraca do programu glownego
						 }
					     case 3:
						  {
						   zliczanie=0;
						   PORTC |= (1<<PC3);			//Podaje stan niski (ON) na wyświetlacz tysięcy
						   while (bit_is_set(PINB,3)) 
						   {
						   if(bit_is_clear(PINB, 1)) {liczba_bledow++;	LCD_wyswietl_pozycja(1, 6, "Blad!");	 while (bit_is_clear(PINB,1)){}; 	_delay_ms(20);	}
						   if(bit_is_clear(PINB, 2)) {liczba_bledow++;	LCD_wyswietl_pozycja(1, 6, "Blad!");	 while (bit_is_clear(PINB,2)){};	_delay_ms(20);	}
						   if(bit_is_clear(PINB, 0)) {liczba_bledow++;	LCD_wyswietl_pozycja(1, 6, "Blad!");	 while (bit_is_clear(PINB,3)){};	_delay_ms(20);	}
						   if(bit_is_clear(PINB, 4)) {liczba_bledow++;	LCD_wyswietl_pozycja(1, 6, "Blad!");	 while (bit_is_clear(PINB,4)){};	_delay_ms(20);	}
						   if(bit_is_clear(PINB, 5)) {liczba_bledow++;	LCD_wyswietl_pozycja(1, 6, "Blad!");	 while (bit_is_clear(PINB,5)){};	_delay_ms(20);	}
						   };
						   czas_ms[x] = zliczanie/3.84;
						   break;				//Wychodzi z funkcji switch i powraca do programu glownego
						   }
						   
						   
					     case 4:
						{
						   zliczanie=0;
						   PORTC |= (1<<PC4);			//Podaje stan niski (ON) na wyświetlacz tysięcy
						   while (bit_is_set(PINB,4))
						   {
						   if(bit_is_clear(PINB, 1)) {liczba_bledow++;	LCD_wyswietl_pozycja(1, 6, "Blad!");	 while (bit_is_clear(PINB,1)){}; 	_delay_ms(20);	}
						   if(bit_is_clear(PINB, 2)) {liczba_bledow++;	LCD_wyswietl_pozycja(1, 6, "Blad!");	 while (bit_is_clear(PINB,2)){};	_delay_ms(20);	}
						   if(bit_is_clear(PINB, 3)) {liczba_bledow++;	LCD_wyswietl_pozycja(1, 6, "Blad!");	 while (bit_is_clear(PINB,3)){};	_delay_ms(20);	}
						   if(bit_is_clear(PINB, 0)) {liczba_bledow++;	LCD_wyswietl_pozycja(1, 6, "Blad!");	 while (bit_is_clear(PINB,4)){};	_delay_ms(20);	}
						   if(bit_is_clear(PINB, 5)) {liczba_bledow++;	LCD_wyswietl_pozycja(1, 6, "Blad!");	 while (bit_is_clear(PINB,5)){};	_delay_ms(20);	}
						   };
						   czas_ms[x] = zliczanie/3.84;
						   break;				//Wychodzi z funkcji switch i powraca do programu glownego
						}
					     case 5:
						{
						   zliczanie=0;
						   PORTC |= (1<<PC5);			//Podaje stan niski (ON) na wyświetlacz tysięcy
						   while (bit_is_set(PINB,5)) 
						   {
						   if(bit_is_clear(PINB, 1)) {liczba_bledow++;	LCD_wyswietl_pozycja(1, 6, "Blad!");	 while (bit_is_clear(PINB,1)){}; 	_delay_ms(20);	}
						   if(bit_is_clear(PINB, 2)) {liczba_bledow++;	LCD_wyswietl_pozycja(1, 6, "Blad!");	 while (bit_is_clear(PINB,2)){};	_delay_ms(20);	}
						   if(bit_is_clear(PINB, 3)) {liczba_bledow++;	LCD_wyswietl_pozycja(1, 6, "Blad!");	 while (bit_is_clear(PINB,3)){};	_delay_ms(20);	}
						   if(bit_is_clear(PINB, 4)) {liczba_bledow++;	LCD_wyswietl_pozycja(1, 6, "Blad!");	 while (bit_is_clear(PINB,4)){};	_delay_ms(20);	}
						   if(bit_is_clear(PINB, 0)) {liczba_bledow++;	LCD_wyswietl_pozycja(1, 6, "Blad!");	 while (bit_is_clear(PINB,5)){};	_delay_ms(20);	}
						   };
						   czas_ms[x] =zliczanie/3.84;
						   break;				//Wychodzi z funkcji switch i powraca do programu glownego
						 }

	   
					  }	//koniec switch	
					  
					  
		  
			PORTC &= ~(1<<PC0);	PORTC &= ~(1<<PC1);	PORTC &= ~(1<<PC2);	PORTC &= ~(1<<PC3);	PORTC &= ~(1<<PC4);	PORTC &= ~(1<<PC5);		
			_delay_ms(500);		       
			x++;			   
			}	//koniec while(x<20)
			   
			x=0;
			   while (x<y)
			   {

			      
			      if (czas_ms[x] > najwiekszy_czas)
				 {najwiekszy_czas = czas_ms[x];}
			
			      if (czas_ms[x] < najmniejszy_czas)
				 {najmniejszy_czas = czas_ms[x];}
			
			      suma_czasow = suma_czasow + czas_ms[x];	
			      x++;
			   } 
			
	       sredni_czas = suma_czasow/y;	

		
		LCD_wyczysc();
		LCD_wyswietl("KONIEC BADANIA!");
		  _delay_ms(1000);
		
		
		char najwiekszy_czas_tekst[7];
		itoa(najwiekszy_czas, najwiekszy_czas_tekst, 10);		  		  
		  LCD_wyczysc();
		  LCD_wyswietl("MAKS.:");
		  LCD_wyswietl_pozycja(0, 10, najwiekszy_czas_tekst);
		  LCD_wyswietl_pozycja(0, 14, "ms");
		  LCD_wyswietl_pozycja(1, 1, "WCISNIJ START>>");
		  _delay_ms(500);
		  while (bit_is_set(PIND,2)) {};
		 
		char najmniejszy_czas_tekst[7];
		itoa(najmniejszy_czas, najmniejszy_czas_tekst, 10);
		  LCD_wyczysc();
		  LCD_wyswietl("MIN.:");
		  LCD_wyswietl_pozycja(0, 10, najmniejszy_czas_tekst);
		  LCD_wyswietl_pozycja(0, 14, "ms");
		  LCD_wyswietl_pozycja(1, 1, "WCISNIJ START>>");
		  _delay_ms(500);
		  while (bit_is_set(PIND,2)) {};
		 
		 
		char sredni_czas_tekst[7];
		itoa(sredni_czas, sredni_czas_tekst, 10);		 
		  LCD_wyczysc();
		  LCD_wyswietl("SREDN.:");
		  LCD_wyswietl_pozycja(0, 10, sredni_czas_tekst);
		  LCD_wyswietl_pozycja(0, 14, "ms");
		  LCD_wyswietl_pozycja(1, 1, "WCISNIJ START>>");
		  _delay_ms(500);
		  while (bit_is_set(PIND,2)) {};
		
		char liczba_bledow_tekst[7];
		itoa(liczba_bledow, liczba_bledow_tekst, 10);		
		LCD_wyczysc();
		LCD_wyswietl("BLEDY:");
		LCD_wyswietl_pozycja(0, 10, liczba_bledow_tekst);
		LCD_wyswietl_pozycja(1, 1, "WCISNIJ START>>");
		  _delay_ms(500);
		 while (bit_is_set(PIND,2)) {};
		 z=1;

		
	       }	//koniec if
  
	}	//koniec while (1)
      
   return 0;
} //koniec int main
# basic_calculator_8051
Đồ án 1, mất file proteus, còn code only(lib lcd của electroSome)


//LCD Module Connections
extern bit RS;                                                                   
extern bit EN;                           
extern bit D0;
extern bit D1;
extern bit D2;
extern bit D3;
extern bit D4;
extern bit D5;
extern bit D6;
extern bit D7;
//End LCD Module Connections 
void Lcd_Delay(int a)
{
    int j;
    int i;
    for(i=0;i<a;i++)
    {
        for(j=0;j<100;j++)
        {
        }
    }
}

void Lcd8_Port(char a)
{
	if(a & 1)
		D0 = 1;
	else 
		D0 = 0;
	
	if(a & 2)
		D1 = 1;
	else
		D1 = 0;
	
	if(a & 4)
		D2 = 1;
	else
		D2 = 0;
	
	if(a & 8)
		D3 = 1;
	else
		D3 = 0;
	
	if(a & 16)
		D4 = 1;
	else
		D4 = 0;

	if(a & 32)
		D5 = 1;
	else
		D5 = 0;
	
	if(a & 64)
		D6 = 1;
	else 
		D6 = 0;
	
	if(a & 128)
		D7 = 1;
	else
		D7 = 0;
}
void Lcd8_Cmd(char a)
{ 
  RS = 0;             // => RS = 0
  Lcd8_Port(a);             //Data transfer
  EN  = 1;             // => E = 1
  Lcd_Delay(5);
  EN  = 0;             // => E = 0
}

Lcd8_Clear()
{
	  Lcd8_Cmd(1);
}

void Lcd8_Set_Cursor(char a, char b)
{
	if(a == 1)
	  Lcd8_Cmd(0x80 + b);
	else if(a == 2)
		Lcd8_Cmd(0xC0 + b);
}

void Lcd8_Init()
{
	Lcd8_Port(0x00);
	RS = 0;
	Lcd_Delay(200);
	///////////// Reset process from datasheet /////////
Lcd8_Cmd(0x30);
	Lcd_Delay(50);
Lcd8_Cmd(0x30);
	Lcd_Delay(110);
  Lcd8_Cmd(0x30);
  Lcd8_Cmd(0x38);    //function set
  Lcd8_Cmd(0x0C);    //display on,cursor off,blink off
  Lcd8_Cmd(0x01);    //clear display
  Lcd8_Cmd(0x06);    //entry mode, set increment
}

void Lcd8_Wrt_Chr(char a)
{
   RS = 1;             // => RS = 1
   Lcd8_Port(a);             //Data transfer
   EN  = 1;             // => E = 1
   Lcd_Delay(5);
   EN  = 0;             // => E = 0
}

void Lcd8_Wrt_Str(char *a)
{
	int i;
	for(i=0;a[i]!='\0';i++)
	 Lcd8_Wrt_Chr(a[i]);
}

void Lcd8_Shift_Right()
{
	Lcd8_Cmd(0x1C);
}

void Lcd8_Shift_Left()
{
	Lcd8_Cmd(0x18);
}


Code chương trình 

#include <regx51.h>
#include <lcd.h>
#include <stdio.h>
#include <string.h>

sbit RS = P3^7;                      
sbit EN = P3^6;                     
//kết nối LCD với 8051
sbit D0 = P2^0;                    
sbit D1 = P2^1;
sbit D2 = P2^2;
sbit D3 = P2^3;
sbit D4 = P2^4;
sbit D5 = P2^5;
sbit D6 = P2^6;
sbit D7 = P2^7;
//Keypad 
sbit R1 = P1^0;                   
sbit R2 = P1^1;                   
sbit R3 = P1^2;                   
sbit R4 = P1^3;                   
sbit C1 = P1^4;                   
sbit C2 = P1^5;                   
sbit C3 = P1^6;                   
sbit C4 = P1^7;                   
/------------------------------------------------/
void scan();
void Delay (int a);
void Sign();
void Calx();
void Cal();
void Num();
void Dec();
/------------------------------------------------/
int op_key,k,N1,N2,i,j,q,err; // dấu, cờ của dấu, cờ đánh dấu phím số, cờ đánh dấu phím dấu, cờ giá trị số , cờ dấu phẩy , q , báo lỗi
float f_number,s_number,result,num_key; //số thứ nhất, số thứ hai, phím số vừa nhấn
char d=' ';
char t=' ';
char tx[10];
/------------------------------------------------/
void main()
{
	f_number=s_number=result=N1=N2=k=op_key=num_key=i=err=0;
	Lcd8_Clear();
	Lcd8_Init();                           //khởi động LCD
	Lcd8_Cmd(0x95);
	Lcd8_Wrt_Str("MAY TINH DON GIAN ");   
	Lcd8_Cmd(0xd8);
	Lcd8_Wrt_Str("  DO AN 1");            
	Lcd8_Cmd(0x80);
	while(1)
	{
		scan();
	}
}	
//=================End Main===============//
void scan()
{
	if(err==0)
		{
			char d=' ';
			R1=0;R2=R3=R4=1;             // đặt Row 1 =0
			if(C1==0){d='7';num_key=7;Num();}  
			if(C2==0){d='8';num_key=8;Num();}  
			if(C3==0){d='9';num_key=9;Num();}  
			if(C4==0){d='/';op_key=4;Sign();} 
				             
			R2=0;R1=R3=R4=1;            //đặt Row 2= 0  
			if(C1==0){d='4';num_key=4;Num();}  
			if(C2==0){d='5';num_key=5;Num();}  
			if(C3==0){d='6';num_key=6;Num();}  
			if(C4==0){d='x';op_key=3;Sign();} 
				                  
			R3=0;R1=R2=R4=1;             //đặt Row 3 = 0
			if(C1==0){d='1';num_key=1;Num();}  
			if(C2==0){d='2';num_key=2;Num();}  
			if(C3==0){d='3';num_key=3;Num();}  
			if(C4==0){d='-';op_key=2;Sign();} 
				                        
			R4=0;R1=R2=R3=1;             // đặt Row 4 = 0
			if(C1==0){d='.';Dec();}      
			if(C2==0){d='0';num_key=0;Num();}  
			if(C3==0){Cal();}            
			if(C4==0){d='+';op_key=1;Sign();} 
			                               
			if(d!=' ')
			{
				Lcd8_Wrt_Chr(d);
				Delay(300);
			}
	  }
		else{Lcd8_Clear();   
			Lcd8_Wrt_Str("Error");           // hiển thị Error lên LCD
         Delay(3000);}
	
				 
		
}
//=================End Scan===============//
void Dec()            //nhận biết, lưu, hiện thị chuỗi số thập phân
{
	if(i==0&&op_key==0){Lcd8_Clear();Lcd8_Wrt_Chr('0');}   
	if(j==0)
	{
		i=2;
		j=1;
	}
	else{err=1;}
}
//=================End Dec===============//
void Numx()
{
	if     (i==0){f_number=num_key;i=1;}    
	else if(i==1){f_number=f_number*10+num_key;}
	else if(i==2)
	{
		//Lcd8_Wrt_Chr('a');
		for(q=0;q<j;q++){num_key=num_key*0.1;}
		f_number=f_number+num_key;
		j=j+1;
	}
}
//=================End Numx===============//
void Num() //lưu số
{
	if(N2==1)   
	{
		if(k==0)         
		{
			Numx();
		}
		else//k!=0
		{
			if     (i==0){s_number=num_key;i=1;}   
			else if(i==1){s_number=s_number*10+num_key;} 
			else if(i==2)
			{
				for(q=0;q<j;q++){num_key=num_key*0.1;}  
				s_number=s_number+num_key;
				j=j+1;
			}
		}
	}
	if(N2==0)
	{
		Numx(); 
	}
	N1=1; // phím số được bấm
}
//=================End Num===============//
void Sign()       //lưu mã dấu để tính toán, hiện thị lên LCD
{
	i=0;
	j=0;
	if(N1==0)  
	{
		if(f_number!=0&&k!=0) 
		{
			Lcd8_Clear();
			sprintf(tx,"%.2f",f_number);
			Lcd8_Wrt_Str(tx);
		}
		if(f_number==0&&k!=0)
		{
			Lcd8_Clear();
		}
		k=op_key;
	}
	else if(N1==1)    
	{
		if(k==0){k=op_key;}  
		else 								
		{
			Calx();	
			f_number=result;
			k=op_key;
			s_number=op_key=0;	
		}
	}
	N2=1;N1=0;
}
//=================End Sign===============//
void Calx()       //tính toán và in kết quả ra màn hình 
{
	if(k==4)        // phép chia 
	{
		if(s_number==0){err=1;}
else{result=f_number/s_number;sprintf(tx,"%.2f",result);Lcd8_Clear();Lcd8_Wrt_Str(tx);}
	}
	else
	{
		if     (k==0){result=f_number;}        //dấu =     
		else if(k==1){result=f_number+s_number;}       
		else if(k==2){result=f_number-s_number;}      
		else if(k==3){result=f_number*s_number;}     
		Lcd8_Clear();             
		sprintf(tx,"%.2f",result);Lcd8_Wrt_Str(tx);
	}
}
//=================End Calx===============//
void Cal()  			// hàm được gọi khi nhấn dấu =
{
	Calx();
	f_number=result;
	k=op_key=s_number=num_key=N1=i=j=0;
	N2=1;
	Delay(300);
}
//=================End Cal===============//
void Delay (int a)
{
	int j;
	int i;
	for (i=0;i<a;i++)
	{
		for (j=0;j<100;j++)
		{
		}
	}
}

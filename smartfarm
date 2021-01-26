#include <arduino-timer.h>          // timer interrupt.
#include <LiquidCrystal_74HC595.h>  // LCD display: Shift resistor use.
#include <SPI.h>                    // LCD display Shift resistor use.
#include <DHT11.h>                  // Temperature & Humidity sensor.
#include <Servo.h>                  // Servo motor.
#include <Stepper.h>                // Stepper motor.
#include <SoftwareSerial.h>         // Blue_Tooth header file.

#define DS 13                       // Shift resistor pin set ↓
#define SHCP 11                     
#define STCP 12                     
#define RS 1                        
#define E 3                   
#define D4 4
#define D5 5
#define D6 6
#define D7 7                        // Shift resistor pin set end.
#define heater A3                   // heater(led) pin set.
#define pump A4                     // pump motor pin set.
#define fan A5                      // fan motor pin set.
#define door_interrupt 2            // door interrupt switch pin set.

const int stroke = 256;             // Sunshade 1 cycle stroke.
int angle=0;                        // Servo motor 1 cycle.
int motorstate = 3;
float temp=0.0, humi=0.0;
int sunshine=120;                   //sunshine inite
int moisture=550;                   //moisture inite
int h_master, f_master, p_master, s_master=0; // heater, fan, pump, sunshades master declare a variable.
int master=0;
int sum=0;
int temp1=0;

DHT11 dht11(A0);                                              // Temperature and Humidity sensor(DHT11) Pin Set.
LiquidCrystal_74HC595 lcd(13, 11, 12, RS, E, D4, D5, D6, D7); // DS, SHCP, STCP, RS, E, D4, D5, D6, D7
Stepper Sunshades(stroke,10,8,9,7);                           // IN4, IN2, IN3, IN1
Servo door;
SoftwareSerial BTSerial(5,4); // TX, RX

// timer function ↓
auto timer1 = timer_create_default();       // create a timer with default settings.

void setup() {
  BTSerial.begin(9600);
  
  pinMode(heater, OUTPUT);
  pinMode(fan, OUTPUT);
  pinMode(pump, OUTPUT);
  pinMode(door_interrupt, INPUT_PULLUP);
  Sunshades.setSpeed(60);                  // Stepper motor speed.
  door.write(5);                        // Door pos reset.
  delay(5);
  analogWrite(6, 60);                       // LCD 명암, 값이 낮을수록 밝아짐.
  
  lcd.begin(16,2);
  lcd.setCursor(0,0);
  lcd.print("Welcome to");
  lcd.setCursor(0,1);
  lcd.print("Smart farm!");
  
// timer set
// call the function every 2000, 5000 millis (2, 5 second)
  
  timer1.every(2000, All_sensing);              
  timer1.every(5000, lcddisplay_sensing);

// interrupt
  attachInterrupt(digitalPinToInterrupt(door_interrupt),door_control,FALLING); //도어 인터럽트
}

void All_sensing() 
{
  if(master==0||master==1) {
    if(dht11.read(humi, temp)==0)  //Temperature & Humidity sensing.
    {
      lcd.clear();              
      lcd.setCursor(0,0);
      lcd.print("temp: ");
      lcd.print(temp,0);
      lcd.write(223);
      lcd.print("C");
      lcd.setCursor(0,1);
      lcd.print("humi: ");
      lcd.print(humi,0);
      lcd.print(" %");
      
      BTSerial.print(temp);
      BTSerial.print("℃"); 
      BTSerial.print("                                 ");
      BTSerial.print(humi);
      BTSerial.print("％");
    }
    sunshine = analogRead(A1);    //Illuminance sensing.
    sum=sum+sunshine;
    temp1++;
  }
  moisture = analogRead(A2);    //Moisture sensing.
}

void lcddisplay_sensing()
{       
  if(master==0){
    if(digitalRead(heater) == HIGH)
    {
        lcd.clear();                       
        lcd.setCursor(0,0);
        lcd.print("Not comfortable");
        lcd.setCursor(0,1);
        lcd.print("Heater running..");      
        delay(500);
    }
    
    if(digitalRead(pump) == HIGH)
    {    
        lcd.clear();
        lcd.setCursor(0,0);
        lcd.print("Not comfortable");
        lcd.setCursor(0,1);
        lcd.print("pump running..");
        delay(500);     
    }
    
    if(digitalRead(fan) == HIGH)
    {   
        lcd.clear();
        lcd.setCursor(0,0);
        lcd.print("Not comfortable");
        lcd.setCursor(0,1);
        lcd.print("fan running..");
        delay(500);     
    }
  }
}

void heater_control() 
{  
  if(master==1){                    // blue_tooth master mode 
    if(h_master==1){
      digitalWrite(heater, HIGH);
    }else if(h_master==0){
      digitalWrite(heater, LOW);
    }
  }
  else if(master==0){
    if(temp < 25) {
      digitalWrite(heater, HIGH);
      h_master=1;
    }
    else {
      h_master=0;
      digitalWrite(heater, LOW);
    }
  }
}

void fan_control() 
{
  if(master==1){
     if(f_master==1){
      digitalWrite(fan, HIGH);
    }else if(f_master==0){
      digitalWrite(fan, LOW);
    }
  }
  else if(master==0){
    if(humi >60) 
    {
      f_master=1;
      digitalWrite(fan, HIGH);   
    }
    else if(humi <30)
    {
      f_master=0;
      digitalWrite(fan, LOW);
    }
  }
}

void pump_control()
{
    if(master==1){
     if(p_master==1){
      lcd.clear();
      delay(50);
      digitalWrite(pump, HIGH);
      delay(50);
    } else if(p_master==0){
      digitalWrite(pump, LOW);
      delay(100);
      lcd.begin(16,2);
      delay(50);
    }
  }
    else if(master==0||master==3||master==2){
      if(moisture < 500)
    {
       
       master=2;
       lcd.clear();
       delay(50);
       if(master==2&&(digitalRead(pump)==LOW)) {
       p_master=1;   
       digitalWrite(pump, HIGH);
            
        
       delay(50); 
       master=3;
       }
    }
    else
    {
      if(master==2&&(digitalRead(pump)==HIGH)) {
       p_master=0;
       digitalWrite(pump, LOW);
       delay(100);
       lcd.begin(16,2);
       delay(50);
       master=0;
      }
    }
  }
}

void sunshades_control() 
{
  if(master==1){
     if(s_master==1)
     {
        Sunshades.step(-stroke);  // up
        delay(100);
      }
    else if(s_master==2)
    {
        Sunshades.step(stroke);   // down
        delay(100);  
      }
      else if(s_master==0)
      {
        Sunshades.step(0);
      }
     }
  else if(master==0)
  {
  if(sunshine > 360) // Blind UP
    {          
        Sunshades.step(-stroke);
        delay(50);
    }
  
    else if(sunshine < 140) // Blind DOWN
    {          
        Sunshades.step(stroke);
        delay(50);                       
    }

    else if(sunshine > 140 && sunshine < 360)
      {
        Sunshades.step(0);
      }
  }
}

void door_control()  // interrupt function
{
  delayMicroseconds(100);
  if(digitalRead(door_interrupt)==LOW){
      if(motorstate == 3)
      {
        motorstate=0;                                
      }
      else if(motorstate == 4)
      {
        motorstate= 1;         
      }   
  }
}

void blue_tooth()
{
    char ch = BTSerial.read();
   
    switch(ch) {
     case 0:
       h_master = 1;
       f_master = 1;
       p_master = 1; 
       s_master = 1;
       break;
      
     case 1:
      h_master = 0;
      f_master = 0;
      p_master = 0; 
      s_master = 0;
      break;
   
    case 2:
      h_master=1;
      break;
   
    case 3:
      h_master=0;
      break;
   
    case 4:
      f_master=1;
      break;
    
    case 5:
      f_master=0;
      break;

    case 6:
      p_master=1;
      break;

    case 7:
      p_master=0;
      break;
      
    case 8:
      s_master=1;
      break;

    case 9:
      s_master=2;
      break;
      
    case 10:
       s_master=0;
       break;
       
    case 11:
      master=0;
      break;
      
    case 12:
      master=1;
      h_master = 0;
      f_master = 0;
      p_master = 0; 
      s_master = 0;
      break;
    
  }
}

void loop() {
      
    timer1.tick(); //All_sensing. //lcddisplay_sensing.
    
      if (motorstate==0) 
      {
          door.attach(3);
          delay(2);
            
            for(angle=0; angle<140; angle++) 
            {
                door.write(angle);
                delay(20);
            }
          motorstate=4;
          door.detach();
      }
      
      else if (motorstate==1) 
      {
          door.attach(3);
          delay(2);
        
            for(angle=140; angle>0; angle--)
            {
                door.write(angle);
                delay(20);
            }    
          motorstate=3;
          door.detach();
      }
      
if(BTSerial.available()>0) {
  blue_tooth();
}
    heater_control();
    fan_control();
    pump_control();
    sunshades_control();
}

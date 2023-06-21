# c-3
#connect coustamize cultivate
#include <DHT.h>  
#include <LiquidCrystal.h>
 const int rs = 16, en = 5, d4 = 4, d5 = 14, d6 = 12, d7 = 13;
LiquidCrystal lcd(rs, en, d4, d5, d6, d7);
#include <ESP8266WiFi.h>
int x;

 
String apiKey = "OOGTQSYFUO2V3USX";    
 
const char *ssid =  "○";     
const char *pass =  "hi123456";
const char* server = "api.thingspeak.com";
char buf[40];
 byte smiley[8] = {
  0b00000,
  0b00000,
  0b01010,
  0b00000,
  0b00000,
  0b10001,
  0b01110,
  0b00000
};

byte frownie[8] = {
  0b00000,
  0b00000,
  0b01010,
  0b00000,
  0b00000,
  0b00000,
  0b01110,
  0b10001
};

float h;
float t;

#define DHTPIN 0         
 
DHT dht(DHTPIN, DHT11);
 
WiFiClient client;
int dateDiff(int year1, int mon1, int day1, int year2, int mon2, int day2)
{
    int ref,dd1,dd2,i;
    ref = year1;
    if(year2<year1)
    ref = year2;
    dd1=0;
    dd1=dater(mon1);
    for(i=ref;i<year1;i++)
    {
        if(i%4==0)
        dd1+=1;
    }
    dd1=dd1+day1+(year1-ref)*365;
    dd2=0;
    for(i=ref;i<year2;i++)
    {
        if(i%4==0)
        dd2+=1;
    }
    dd2=dater(mon2)+dd2+day2+((year2-ref)*365);
    return dd2-dd1;
}

int dater(int x)
{ const int dr[]= { 0, 31, 59, 90, 120, 151, 181, 212, 243, 273, 304, 334};
  return dr[x-1];
}
void setup() 
{
  
    lcd.begin(16, 2);

 
  lcd.createChar(1, smiley);

  lcd.createChar(2, frownie);


 
  lcd.setCursor(0, 0);
   if( t>35&& h<37){

  lcd.print(" ");
  lcd.write(byte(2)); 
  lcd.print(" bad");
     }
  else{
       lcd.write(byte(1));
       lcd.print("  pleasent");
}
       Serial.begin(115200);
       delay(10);
       dht.begin();
 
       Serial.println("Connecting to ");
       Serial.println(ssid);
 
 
       WiFi.begin(ssid, pass);
 
      while (WiFi.status() != WL_CONNECTED) 
     {
            delay(500);
            Serial.print(".");
     }
      Serial.println("");
      Serial.println("WiFi connected");
  
  int x=dateDiff(2015,12,24, 2017,1,1 );
  Serial.println(x);
  
 
}

void loop() 
{
      float h = dht.readHumidity();
      float t = dht.readTemperature();
    
    int sensorReading = analogRead(A0);
  // map the result to 200 - 1000:
  int delayTime = map(sensorReading, 0, 1023, 200, 1000);
 

      
           if (isnan(h) || isnan(t)) 
                 {
                     Serial.println("Failed to read from DHT sensor!");
                      return;
                 }
 
                         if (client.connect(server,80))  
                      {  
                            
                             String postStr = apiKey;
                             postStr +="&field1=";
                             postStr += String(t);
                             postStr +="&field2=";
                             postStr += String(h);
                             postStr +="&field3=";
                             postStr += String(x);//i just tried to import date  to the browser so to provide accurate date and time of the sensor reading
                             postStr +="&field4=";//so that apart from thing speak if it goes to app (realtime)the controller can even calculte the code and update it.
                             postStr += String(21);
                             postStr += "\r\n\r\n";
 
                             client.print("POST /update HTTP/1.1\n");
                             client.print("Host: api.thingspeak.com\n");
                             client.print("Connection: close\n");
                             client.print("X-THINGSPEAKAPIKEY: "+apiKey+"\n");
                             client.print("Content-Type: application/x-www-form-urlencoded\n");
                             client.print("Content-Length: ");
                             client.print(postStr.length());
                             client.print("\n\n");
                             client.print(postStr);
 
                             Serial.print("Temperature: ");
                             Serial.print(t);
                             Serial.print(" degrees Celcius, Humidity: ");
                             Serial.print(h);
                             Serial.println("%. Send to Thingspeak.");
                        }
          client.stop();
 
          Serial.println("Waiting...");
  
  // thingspeak needs 
    delay(1000);}

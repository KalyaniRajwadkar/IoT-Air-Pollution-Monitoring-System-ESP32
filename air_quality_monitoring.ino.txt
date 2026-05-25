include <DHT.h>
#include <LiquidCrystal.h>
#include <WiFi.h>

//------------------- DHT --------------------
#define DHTPIN 12
#define DHTTYPE DHT11
DHT dht(DHTPIN, DHTTYPE);

//------------------- LCD --------------------
LiquidCrystal lcd(23,22,21,19,18,5);

//------------------- WIFI -------------------
const char *ssid = "IOT";
const char *password = "IOT123456";
String apiKey = "DDHQ4BMDQ3AC3YYW";
const char* server = "api.thingspeak.com";
WiFiClient client;

//------------------- Sensors ----------------
int mq135 = 34;

int mq135Value;


float temp;
float hum;

int mq135ppm;

//--------------------------------------------
long writingTimer = 15; //server response time in sec
long startTime = 0;
long waitTime = 0;
///*********************************
void setup() {


lcd.begin(16,2);

WiFi.begin(ssid,password);

lcd.print("Connecting WiFi");

if(WiFi.status()!=WL_CONNECTED){
delay(500);
Serial.print(".");
}
else{
lcd.clear();
lcd.print("WiFi Connected");
delay(2000);
}
lcd.clear();
lcd.setCursor(0,0);
lcd.print("Air Quality");
lcd.setCursor(0,1);
lcd.print(" Monitoring");
delay(2000);
lcd.clear();
lcd.setCursor(0,0);
lcd.print("Kalyani,Arpita");
lcd.setCursor(0,1);
lcd.print("Kalyani");
delay(2000);
}

//--------------------------------------------

void loop() {


mq135Value = analogRead(mq135);

mq135ppm = map(mq135Value,0,4095,0,1000);

//-------- DHT Sensor ----------
temp = dht.readTemperature();
hum = dht.readHumidity();

//-------- Serial Monitor -------
Serial.println("------ Air Quality Data ------");

Serial.print("Temperature: ");
Serial.print(temp);
Serial.println(" C");

Serial.print("Humidity: ");
Serial.print(hum);
Serial.println(" %");


Serial.print("MQ135 Air Quality: ");
Serial.print(mq135ppm);
Serial.println(" ppm");

Serial.println("------------------------------");

//-------- LCD Display ----------
lcd.clear();
lcd.setCursor(0,0);
lcd.print("T:");
lcd.print(temp);
lcd.print(" H:");
lcd.print(hum);

lcd.setCursor(0,1);

lcd.print("MQ135:");
lcd.print(mq135ppm);
lcd.print(" ppm");

delay(1500);


waitTime = millis() - startTime;

  if (waitTime > (writingTimer * 1000))
  {
    lcd.clear();
    // lcd.setCursor(0, 0);
    lcd.print("Updating.....");
    Serial.println("updating...");
    // sensordata();
    updatedata();

    startTime = millis();

  }

}
void updatedata(){
if (client.connect(server,80)){

String postStr = apiKey;

postStr += "&field1=";
postStr += String(temp);

postStr += "&field2=";
postStr += String(hum);



postStr += "&field3=";
postStr += String(mq135ppm);

postStr += "\r\n";

client.print("POST /update HTTP/1.1\n");
client.print("Host: api.thingspeak.com\n");
client.print("Connection: close\n");
client.print("X-THINGSPEAKAPIKEY: "+apiKey+"\n");
client.print("Content-Type: application/x-www-form-urlencoded\n");
client.print("Content-Length: ");
client.print(postStr.length());
client.print("\n\n");

Serial.println("Data Sent to ThingSpeak");
}
}

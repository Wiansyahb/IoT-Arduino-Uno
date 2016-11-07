#include <SPI.h>
#include <Ethernet.h>

//Setup IP & MAC
byte mac[] = { 0xDE, 0xAD, 0xBE, 0xEF, 0xFE, 0xED };
byte ip[]  = { 192, 168, 1, 5 }; //ip arduino
char server[] = "192.168.137.1"; //ip server/DNS

//Setup Ultrasonic
const int trig = 9;
const int echo = 8;
int distance, save = 0;

//Motor
const int Motor = 7;
const int controlMode = 6;

//To read from web
char inString[32];
int stringPos = 0;
boolean startRead = false;
int nilaiControl;

EthernetClient client;

void setup() {
  Serial.begin(115200);
  Ethernet.begin(mac, ip);
  Serial.print("IP Addr : ");
  Serial.println(Ethernet.localIP());
  pinMode(trig, OUTPUT);
  pinMode(echo, INPUT);
  pinMode(Motor, OUTPUT);
}

void loop() {
  Serial.println("----------------------------------");
  ultrasonic_sensor();
  send_data();
  control();
  Serial.println("----------------------------------");
  Serial.println("");
  delay(1900);
}

void ultrasonic_sensor() {
  digitalWrite(trig, LOW);
  delayMicroseconds(2);
  digitalWrite(trig, HIGH);
  delayMicroseconds(10);
  digitalWrite(trig, LOW);
  long duration = pulseIn(echo, HIGH);
  distance = duration * 0.034 / 2; //0.034 untuk cm
  Serial.println("Tinggi Air      : " + (String)distance + "cm");
}

void send_data() {
  Serial.print("Send Data to DB : ");
  if (client.connect(server, 80)) {
    Serial.print("Connected");
    client.print( "GET /add_data.php?");
    client.print("tingair=");
    client.print(distance);
    client.println(" HTTP/1.1");
    client.print("Host: " );
    client.println(server);
    client.println("Connection: close");
    client.println();
    client.println();
    client.stop();
    Serial.println(" -> Data sent");
  } else {
    Serial.println("Connection failed");
  }
  delay(100);
}

void control() {
  Serial.print("Mode ");
  if (client.connect(server, 80)) {
    client.println("GET /nilai_control2.php HTTP/1.0");
    client.println();
    String readMotor = readControl();
    nilaiControl = readMotor.toInt();
    if (nilaiControl == 1) {
      motor();
    } else {
      Serial.print("Manual     : ");
      if (nilaiControl == 2) {
        digitalWrite(Motor, HIGH);
        Serial.println("Motor ON");
      } else if (nilaiControl == 0 || nilaiControl == 3) {
        digitalWrite(Motor, LOW);
        Serial.println("Motor OFF");
      }
    }
  } else {
    Serial.println("           : Failed");
  }
}

String readControl() {
  stringPos = 0;
  memset( &inString, 0, 32 );
  while (true) {
    if (client.available()) {
      char c = client.read();
      if (c == '<' ) {
        startRead = true;
      } else if (startRead) {
        if (c != '>') {
          inString[stringPos] = c;
          stringPos ++;
        } else {
          startRead = false;
          client.stop();
          client.flush();
          return inString;
        }
      }
    }
  }
}

void motor() {
  Serial.print("Auto       : ");
  if (save == 1 && distance > 20) {
    digitalWrite(Motor, HIGH);
    Serial.println("Motor ON");
    if (distance <= 20) {
      save = 0;
    }
  } else {
    digitalWrite(Motor, LOW);
    Serial.println("Motor OFF");
    if (distance > 100) {
      save = 1;
    }
  }
}

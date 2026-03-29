#include <WiFi.h>
#include <WebServer.h>

// ===== WIFI =====
const char* ssid = "YOURWIFISSID";
const char* password = "YOURWIFIPASSWORD";

// ===== PINS (ESP32-C3 SAFE) =====
#define LED_PIN 8
#define RXD2 3
#define TXD2 4
#define RELAY_UHF 5
#define RELAY_ANT 6

HardwareSerial CAT(1);
WebServer server(80);

// ===== STATE =====
long freq = 7085000;
int mode = 1; // 1=LSB,2=USB,3=CW,4=FM,5=AM
bool uhf = false;
bool ant2 = false;

// ===== LOG =====
#define LOG_SIZE 20
String commandLog[LOG_SIZE];
int logIndex = 0;

void addLog(String msg){
  commandLog[logIndex] = msg;
  logIndex = (logIndex+1) % LOG_SIZE;
  Serial.println(msg);
}

String getLogJson(){
  String json = "[";
  for(int i=0;i<LOG_SIZE;i++){
    int idx = (logIndex+i) % LOG_SIZE;
    if(commandLog[idx].length()>0){
      json += "\"" + commandLog[idx] + "\"";
      if(i<LOG_SIZE-1) json += ",";
    }
  }
  json += "]";
  return json;
}

// ===== CAT COMMANDS =====
void sendFreq(String senderIP="local"){
  String f = String(freq);
  while(f.length()<11) f="0"+f;
  String cmd="FA"+f+";";
  CAT.print(cmd);
  addLog("[" + senderIP + "] Sent Frequency -> " + f);
}

void sendMode(int m,String senderIP="local"){
  mode = m;
  String cmd="MD"+String(m)+";";
  CAT.print(cmd);
  addLog("[" + senderIP + "] Sent Mode -> " + String(m));
}

void setUHF(bool state,String senderIP="local"){
  digitalWrite(RELAY_UHF,state?HIGH:LOW);
  uhf=state;
  addLog("[" + senderIP + "] UHF Relay -> " + String(state?"ON":"OFF"));
}

void setANT(bool state,String senderIP="local"){
  digitalWrite(RELAY_ANT,state?HIGH:LOW);
  ant2=state;
  addLog("[" + senderIP + "] Antenna -> " + String(state?"2":"1"));
}

// ===== WEB PAGE =====
String page=R"rawliteral(
<!DOCTYPE html>
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>REMOTE USDX HF TRANSCEIVER</title>
<style>
body{background:#2b2f2a;color:#c7d0b0;font-family:monospace;text-align:center;}
.panel{background:#3a3f36;padding:10px;max-width:480px;margin:auto;}
.display{background:black;color:#7CFF00;font-size:40px;padding:10px;margin-bottom:5px;border-radius:4px;}
button{padding:8px;margin:2px;min-width:50px;border-radius:4px;font-weight:bold;border:2px solid #222;}
button.active{background:#00FF00;color:#000;}
.row{display:flex;justify-content:center;flex-wrap:wrap;}
#console{background:black;color:#00FF00;font-family:monospace;font-size:14px;padding:8px;height:150px;overflow-y:auto;border:2px solid #00FF00;width:95%;margin:auto;border-radius:4px;}
</style>
</head>
<body>
<h2>USDX REMOTE CONTROL</h2>
<div class="panel">
<div id="freq" class="display">0.000.000</div>

<div class="row">
<button onclick="step(-100)">-0.1</button>
<button onclick="step(-1000)">-1</button>
<button onclick="step(-10000)">-10</button>
<button onclick="step(-100000)">-100</button>
<button onclick="step(-1000000)">-1M</button>
<button onclick="step(1000000)">+1M</button>
<button onclick="step(100000)">+100</button>
<button onclick="step(10000)">+10</button>
<button onclick="step(1000)">+1</button>
<button onclick="step(100)">+0.1</button>
</div>

<div class="row">
<button onclick="setMode(1)" id="btnLSB">LSB</button>
<button onclick="setMode(2)" id="btnUSB">USB</button>
<button onclick="setMode(5)" id="btnAM">AM</button>  <!-- swapped -->
<button onclick="setMode(4)" id="btnFM">FM</button>
<button onclick="setMode(3)" id="btnCW">CW</button>  <!-- swapped -->
</div>

<div class="row">
<button onclick="preset(6993000,1)">6993 LSB</button>
<button onclick="preset(7085000,1)">7085 LSB</button>
<button onclick="preset(7100000,1)">7100 LSB</button>
<button onclick="preset(7148000,1)">7148 LSB</button>
</div>

<div class="row">
<button onclick="preset(27305000,2)">27305 USB</button>
<button onclick="preset(27455000,2)">27455 USB</button>
<button onclick="preset(27555000,2)">27555 USB</button>
<button onclick="preset(8900000,2)">8900 USB</button>
</div>

<div class="row">
<button onclick="toggleUHF()" id="btnUHF">UHF OFF</button>
<button onclick="toggleANT()" id="btnANT">ANT1</button>
</div>

<div id="console"></div>

</div>

<script>
let freq=7085000;
let mode=1;
let uhf=false;
let ant2=false;

function formatFreq(f){
  let mhz=Math.floor(f/1000000);
  let khz=Math.floor((f%1000000)/1000);
  let hz=f%1000;
  return mhz+"."+khz.toString().padStart(3,'0')+"."+hz.toString().padStart(3,'0');
}

function update(){
  document.getElementById("freq").innerText=formatFreq(freq);
  document.getElementById("btnLSB").className=(mode==1?"active":"");
  document.getElementById("btnUSB").className=(mode==2?"active":"");
  document.getElementById("btnAM").className=(mode==5?"active":"");  // AM
  document.getElementById("btnFM").className=(mode==4?"active":"");
  document.getElementById("btnCW").className=(mode==3?"active":"");  // CW
  document.getElementById("btnUHF").innerText=uhf?"UHF ON":"UHF OFF";
  document.getElementById("btnUHF").className=uhf?"active":"";
  document.getElementById("btnANT").innerText=ant2?"ANT2":"ANT1";
  document.getElementById("btnANT").className=ant2?"active":"";
}

function updateLog(){
  fetch("/getLog")
    .then(resp=>resp.json())
    .then(log=>{
      const cons=document.getElementById("console");
      cons.innerHTML="";
      log.forEach(line=>{cons.innerHTML+=line+"<br>";});
      cons.scrollTop=cons.scrollHeight;
    });
}

function step(x){freq+=x;if(freq<100000)freq=100000;if(freq>60000000)freq=60000000;update();fetch("/setf?f="+freq);updateLog();}
function preset(f,m){freq=f;update();fetch("/setf?f="+freq);setMode(m);updateLog();}
function setMode(m){mode=m;fetch("/mode?m="+m);update();updateLog();}
function toggleUHF(){uhf=!uhf;fetch("/uhf?s="+(uhf?1:0));update();updateLog();}
function toggleANT(){ant2=!ant2;fetch("/ant?s="+(ant2?1:0));update();updateLog();}

setInterval(updateLog,1000);
update();
</script>
</body>
</html>
)rawliteral";

// ===== HANDLERS =====
void handleRoot(){server.send(200,"text/html",page);}
void handleFreq(){ freq=server.arg("f").toInt(); sendFreq(server.client().remoteIP().toString()); server.send(200,"text/plain","OK"); }
void handleMode(){ int m=server.arg("m").toInt(); sendMode(m,server.client().remoteIP().toString()); server.send(200,"text/plain","OK"); }
void handleUHF(){ int s=server.arg("s").toInt(); setUHF(s,server.client().remoteIP().toString()); server.send(200,"text/plain","OK"); }
void handleANT(){ int s=server.arg("s").toInt(); setANT(s,server.client().remoteIP().toString()); server.send(200,"text/plain","OK"); }
void handleLog(){server.send(200,"application/json",getLogJson());}

// ===== SETUP =====
void setup(){
  Serial.begin(115200);
  delay(1000);

  pinMode(LED_PIN,OUTPUT);
  digitalWrite(LED_PIN,HIGH);

  pinMode(RELAY_UHF,OUTPUT);
  pinMode(RELAY_ANT,OUTPUT);
  digitalWrite(RELAY_UHF,LOW);
  digitalWrite(RELAY_ANT,LOW);

  CAT.begin(38400,SERIAL_8N1,RXD2,TXD2);

  WiFi.begin(ssid,password);
  Serial.print("Connecting");
  while(WiFi.status()!=WL_CONNECTED){delay(500);Serial.print(".");}
  Serial.println("\nConnected! IP: "+WiFi.localIP().toString());

  server.on("/",handleRoot);
  server.on("/setf",handleFreq);
  server.on("/mode",handleMode);
  server.on("/uhf",handleUHF);
  server.on("/ant",handleANT);
  server.on("/getLog",handleLog);

  server.begin();
  sendFreq();
  sendMode(mode);
}

// ===== LOOP =====
unsigned long lastBlink=0;
bool ledState=false;
void loop(){
  server.handleClient();
  unsigned long now=millis();
  if(now-lastBlink>2000){
    lastBlink=now;
    ledState=!ledState;
    digitalWrite(LED_PIN,ledState?LOW:HIGH);
  }
}

# IOT 5
Samenvatting door Brecht Carlier voor het vak *Internet Of Things* op AP Hogeschool (Elektronica - ICT).

### License
The license for this repo is MIT (check the license file). This means you can do whatever you want to do! 
If you want to use this, or parts of this is that totally legal. This is copyleft work! Feel free to use it!

I would appreciate it if you link to this repo if you use this repo as source! Or give me some credits (leave my name).

I based my question/answers on slides of [Dhr. M. Luyts](https://github.com/luytsm). 
Some of the pictures I used are comming from his slides. I also used some texts of his slides. 
There are also transelations of his text.

If you're an author of something, and think there are violations against copyright in my repo, please contant me. It wasn't intended!

If you think there is something wrong, please use the issues system of GitHub. Ofcourse I also accepts pull requests with optimizations.

### 1. Rond wat draait het IoT?
**Data**. Bij een IoT netwerk is het de bedoeling we verschillende data beschikbaar stellen via een netwerk. 
De data kan afkomstig zijn van verschillende sensoren.

### 2. Hoe kunnen we verschillende nodes in een IoT systeem met elkaar laten communiceren?
![IoT architectuur](http://i.imgur.com/BpK0UDV.png)

*Noot:* IoT device en IoT gateway zijn meestal dezelfde apparaaten (Arduino, Raspberry Pi, ...). Het enigste verschil is hun programmatie.
In kleinere IoT netwerken kunnen we beide functies op 1 apparaat programmeren!

### 3. Wat is het nut (functie) van een IoT gateway?
TODO

### 4. Hoe definieert het IoT het zenden en sturen van data?
Het IoT is niets nieuw!

-  Embedded technologiën bestaan al (UART, SPI, I²C)
-  De interfaces zijn gedefinieerd. (RS232, RS422, Modbus, ..) 
-  De protocols zijn gedefinieerd

**Het IoT is dus een verzameling van protocollen, interfaces en standaarden**. 
Er wordt gebruik gemaakt van 'oude' protocollen (uit de embedded wereld) maar er worden ook vaak nieuwe 'IoT' protocollen verzonnen. 

### 5. Hoe kunnen we sensoren uitlezen (bedraad)
- ADC (uitgangsspanning meten)
- Serieel (communiceren met sensor)
    -  UART
    -  I²C
    -  SPI
    
### 6. Hoe kunnen we actuators sturen (bedraad)
- Spanning (IO)
  - PWM       
  - Direct (DAC)
- Serieel (communiceren met actuator)
  - UART
  - I²C
  - SPI
  
### 7. Wat is een seriele communicatie?
Bij een seriele communicatie, zal er 'gepraat' worden tussen twee apparaten. Er is dus een koppeling tussen zender en ontvanger. 
Dit is niet het geval bij ADC uitlezing. Dit heeft als voordeel dat er two way communicatie is. 
De ontvanger kan dus zeggen ik heb het niet ontvangen stuur het opnieuw. 
Hoe deze communicatie gebeurd noemen we het **protocol**, hierin wordt dus gedefinieerd hoe een communicatie gestart/gestopt wordt en andere regels!

Dit gebeurd allemaal via een seriele interface. Deze interface zal een serie van spanningspulsen versturen. 
Deze pulsen komen dan overeen met een logische 0 of een logische 1.
Welke spanning overeenkomt met een logische 0 / logische 1 wordt bepaald door het achterliggende protocol!

### 8. Wat is de USART?
Universal Synchronous and Asynchronous serial Receiver and Transmitter 

Twee modussen:
- Asynchroon: De klok wordt niet meegestuurd (UART)
- Synchroon: de klok wordt meegestuurd

Vaak gebruikte seriele standaard. De seriele poort op je Arduino is bijvoorbeeld een UART. 
De seriele poorten op je computer (in een ver verleden) zijn ook een vorm van een UART (zie later).

### 9. Leg de UART uit!
De seriële poort is een asychrone communicatie interface die gebruikt maakt van 2 draden (tx / rx) voor de overdracht van data.

UART bestaat uit 3 delen
-  Transmitter
-  Klok generatie
-  Reciever

##### Transmitter
![UART transmit](http://i.imgur.com/5OGjohu.png)

##### Klok generatie
Omdat de klok niet wordt doorgestuurd, moeten we op ontvangen en zender dezelfde klok genereren. De klok wordt uitgedrukt als baudrate.
De baudrate van een systeem is het aantal symbool veranderingen in een seconde. In een bit gebaseerd systeem is dit maar 1 symbool
verandering per keer. Hierdoor is de baudrate gelijk aan het het aantal bits per second (bps). 

Uiteraard moet deze op beide systemen op dezelfde waarde worden ingesteld!

De baudrate wordt gegenereerd door een downcounter die aftelt. Wanneer deze counter op 0 is dan wordt er een puls opgewekt/verwerkt.

#### Reciever
Indien zijn downcounter nul bereikt zal hij één puls (0 of 1) binnenshiften in het geheugen van de UART. We hebben de data nu ontvangen.

### 10. Hoe gebeurd UART communicatie praktisch in de ATmega328p?
In de µController zitten registers. Dit is intern geheugen dat gebruikt wordt om zijn taak te voltooien. 
Zo zullen erin de register configuratie waarden zitten. Maar ook de data die we ontvangen met onze seriële communicatie. 
De µController zal data shiften in zijn register. 
Hierna kan dit opnieuw gebruikt worden, zoals opslaan in een variable (andere geheugenplaats).

![UART registers](http://i.imgur.com/9cYiged.png?1)

##### Klok generatie
In **UBRRn** slaan we onze baudrate op de we hebben ingesteld. Dit wordt niet rechtstreeks opgeslagen. 
De waarde die zich in dat register bevind is gelijk aan de (oscillatie frequentie / 16 keer de baudrate) - 1.

Nu wordt d.m.v. *de baudrate generator* een correct klok puls signaal gemaakt dat wordt gebruikt voor de rest van de registers (zie Digitale Systemen 2).

##### Transmit
Je data die je wilt verzenden wordt in **UDRn (Transmit)** opgeslagen. Deze wordt dan gekopieerd in het schuif register. 
Elke keer er een klokpuls van *de baudrate generator* komt zal er een bit (de LSB) uit het register geshift worden (de rest schuift ook mee uiteraard). 
Deze bit wordt nu verzonden via de tx lijn.

Herhaal dit tot schuif register leeg is. Is er nieuwe data in **UDRN (Transmit)** start opnieuw.

TX Buffer leeg & **TXCIEn** set?
- True & True → Gooi TX Complete Interrupt (callback)
- True & False → Gedaan (er wordt geen interrupt afgevuurd, zelf pollen)

##### Recieve
Omgekeerd aan Transmit, elke keer er een klokpuls van *de baudrate generator* zal de bit die aanwezig is op de rx lijn in het schijfregister geshift worden. 
Zit het schijfregister vol dan wordt de inhoud gekopieerd naar **UDRn(recieve)**. 

Indien dit het geval is:
- Wordt **RXCn** 1
- Indien **RXCIEn** 1 is (geset):
  - True → Gooi RX Complete Interrupt 
    - Via callback (ISR) kun je de data verwerken, bijvoorbeeld opslagen in een variabele)
  - False → NOP 
    - Poll Mechanisme. Elke keer kijken is het register vol. Ja dan kunnen we de data opslagen bijvoorbeeld.

### 11. Wat is bitbanging
Dit is het manueel hoog en laag zetten van pinnen om bepaalde dedicted functies te emuleren op je µController. 
Indien je µController niet beschikt over i²c of spi in hardware kun je het dus software matig oplossen.

Zo kun je toch communiceren met een andere IC via dat protocol. 
Zoals altijd is een software matige oplossing altijd slechter dan een hardware matige oplossing. 
Zo kan het minder snel werken en kan het meer energie gebruiken.

### 12. Wat is SPI?
SPI staat voor **Serial Peripheral Interface** is een simpel, synchroon en point-to-point serieële interface.
Het is gebaseerd op een master-slave princiepe. Het zorgt voor full-duplex communicatie tussen een master en één of meerdere slaves.
De interface bestaat uit 4 aparte lijnen

- CLK (SCLK, SCK): Clock wordt geleverd door de master
- MOSI: Master Out Slave In, op deze lijn wordt er data verzonden van master naar slave
- MISO: Master In Salve Out, op deze lijn wordt er data verzonden van slave naar master
- SS': Slave Select (actief laag), de master stuurt deze lijn, het bepaald voor welke slave de data is! 


### 13. Teken het diagram van SPI
![SPI diagram](http://i.imgur.com/nessE3A.png)

### 14. Welk data modes heeft SPI + leg uit.
![SPI modes](http://i.imgur.com/xujvPCs.png)

CPOL => Clock
- 0 => Actief hoog
- 1 => Actief laag

CPHA => Data
- 0 => Samplen op eerste transitie (rising edge bij actief hoge clock, falling edge bij actief lage clock) </br>
  ![CPHA 0](http://i.imgur.com/Pttc8BP.png)
- 1 => Samplen op tweede transitie (falling edge bij actieg hoge clock, rising edge bij actief lage clock) </br>
  ![CPHA 1](http://i.imgur.com/cH1JqV8.png)

### 15. Werking SPI (Labo)
TODO

### 16. Wat is I²C
I²C staat voor Inter - Integrated Circuit. Het wordt ook al wel eens TWI genoemd. I²C is een multi-master protocol.
Het gebruikt maar 2 lijnen! Deze twee I²C lijnen zijn 'serial data' (**SDA**) en 'serial clock' (**SCL**). 
Het aantal slaves / masters dat op dezelfde bus communiceren kan theoretisch oneindig zijn. 

![I²C schema](https://upload.wikimedia.org/wikipedia/commons/thumb/3/3e/I2C.svg/425px-I2C.svg.png)

### 17. Leg I²C timing diagram voor (elke stap)

![I²C timing](http://i.imgur.com/TEn18kX.png)

#####  1. Idle state
I²C start met een actief hoge idle state. Als de twee I²C lijnen hoog zijn, dan is de I²C bus in een idle state. 
De klokt pulst niet tijdens de idle state.

##### 2. Start signal
Normaal gezien als de klok lijn op een logisch hoog signaal staat dan moet de data stabiel zijn (niet veranderen). 
De data lijn veranderd pas van state als de klok op een logische laag signaal staat.
Er zijn twee uitzondering dit is voor het start signaal en het stop signaal.

Het start signaal gaat altijd uit van de master.
Het start signaal is een HOOG naar LAAG transitie op de SDA lijn terwijl het klok signaal op een logisch HOOG signaal staat.

##### 3. Clock Signal
Het klok signaal wordt pas actief zodra het start signaal wordt gegeven.

##### 4. Send address
Zodra het start signaal wordt gegeven, wordt het adres van de target slave op de lijn gezet. 
Doordat het start signaal is gegeven luisteren alle slaves op de bus naar dit adres. 

Het adres is 7 bits groot.

##### 5. Read or Write
Daarna wordt er een bit doorgestuurd die aangeeft of er een read of write actie gaat gebeuren.Dit is de 8ste bit dat wordt doorgestuurd.

- 0 = Write
- 1 = Read

##### 6. Acknowledge
Als er een slave op de I²C zijn adres herkent dan trekt de slave het signaal laag. Dit is de 9 bit dat
wordt doorgestuurd. Nu weet de master dat die IC actief is en kan het zenden beginnen!

##### 7. Data Transfer
Nadat de master de acknowledge van de slave heeft ontvangen dan begin de data transfer. 
Een data transfer is een 8 bit data segment gevolgd door een 1 bit acknowledgment van de ontvanger.
Dit kan dus zowel de master als de slave zijn. Afhankelijk of dat het een read or write actie is. 
Er kunnen zoveel data transfers zijn zolang er nodig zijn.

##### 8. Stop signal
Analoog aan het start signaal maar andersom. Samen met start sequentie, enigste keer wanneer SDA veranderd terwijl de clock hoog is.

Het stop signaal gaat ook altijd uit van de master.
Het stop signaal is een LAAG naar HOOG transitie op de SDA lijn terwijl het klok signaal op een logisch HOOG signaal staat.

### 18. Welke bedraade protocollen kennen we om tussen IoT devices te communiceren? (langere afstand)
- Ethernet (Dhr Masset)
- RS-232
- RS-422
- RS-485
- CAN-BUS
- 4-20 mA
- ...

### 19. Wat is RS-232?
Uitbreiding op UART. Het is de seriële poort die vaak beschikbaar was op (oudere) computers.

Verschillen:
- Logische levels verschillen (spanning)
  - Logische 1 => −15 to −3 V
  - Logische 0 => +3 to +15 V
- Beschikt over flow control
  - Flow control wordt gebruikt om geen buffer overrun te creeren. De juiste signalen moeten gezet zijn voor er communicatie kan plaats vinden.
- 2 mogelijke connectors
  - DB9 (Standaard)
  - DB25
 
### 20. Wat is RS-422 / RS-485?
RS-232 met een differentieel signaal. Differentieel signaal gebruikt signaal lijnen om ruis te onderdrukken op het signaal. Zie afbeelding

![Differentieel signaal](http://i.imgur.com/ahTz4GK.png)

Hierdoor was de maximale afstand en maximum bandbreedte aanzienlijk groter geworden!

Het verschil tusen RS-422 en RS-485 zit in het aantal receivers en transmitters. RS485 kan er meer hebben. 
Ook is de snelheid nog wat gestegen bij RS-485

### 21. Wat is CAN-bus?
CAN (Controller Area Network) is een serieel bus systeem dat gebruikt wordt om verschillende 8 microcontrollers met elkaar te laten communiceren. 

Het heeft als voordeel dat het *Error Detection* ingebouwd heeft. 
- Message level (CRC, frame check, ACK errors)
- Bit level (monitoring, bit stuffing)

Het wordt vaak gebruikt in de auto industrie!

### 22. Wat is Current Loop?
Analoog systeem waarbij we gegevens versturen aan de hand van de stroomsterkte. Stroom verzwakt niet over afstand. 
Spanning doet dit wel door de ongewenste weerstand (draadweerstand). 
Hierdoor wordt het gebruikt voor communicatie over een lange afstand. Er zijn een paar digitale protocolen die werken over *Current Loop*.

- 4 mA => Laagste waarde
- 20 mA => Hoogste waarde

### 23. Voordelen en nadelen van draadloze protocolen

|Voordelen|Nadelen|
|---------|-------|
|Geen connectors|Minder bandbreedte|
|Goedkoper|Security|
|Veilige flexible connectiviteit|Beschikbaarheid|
|Beter resources verdeling|Beschikbare banden|
|Makkelijkere installatie|Antennas|
|Mobiliteit|

### 24. Geef de kenmerken van WiFi (WLAN)
- 802.11.X
- Draadloos internet access (direct internet verbinding, geen nood meer aan tussen apparaat)
- 2450 MHz and 5800 MHz bands
- 1Mb/s .. 1.3Gb/s
- Niet energie kritieke applicaties
- Kleine dekking

Communicatie over WiFi gebeurt over TCP / IP stack (ook gekend als de Internet Protocol Suite) </br>
TCP / IP Programming → Socket Programming

### 25. Geef enkele WiFi solutions in de IoT wereld

##### Arduino WiFi
Er zijn shields beschikbaar om bestaande Arduino uit te breiden, er is ook een Arduino met WiFi ingebouwd.

Abstractie door gebruikt te maken van libraries → Geen manuele socket creatie  </br>
Sockets kunnen bezien worden als een stream → Weinig verschil in gebruik zoals met een serieële communicatie (read en write)

##### Electrical Imp
Formfactor: SD card

##### ESP8266
Maakt gebruik van AT commando's (serieel doorsturen via UART)

###### NodeMCU
Prototype board gebaseerd op ESP8266. Kun je programmeren via Lua. Dit is een makkelijke scripting taal. 
Wordt gebruikt in games, wireshark, ... Weinig OO support, prototype taal (snel iets gemaakt)

##### Particle Photon
Arduino like development
- Web IDE + Atom gebaseerde IDE
- Interfacing
  - Mobile SDKs
  - Javascript
  - CLI
- Over The Air Programming 
 
### 26. Wat is MQTT?
Is een protocol om data tussen twee "machines" te verzenden. Vergelijk het met REST en SOAP. 

MQTT leunt niet op het client / server concept zoals REST (HTTP) en SOAP doet. *Maar gebruikt het publish/subscribe messaging concept*.
Dit houdt in dat de client zich zal subscriben op een onderwerp. Vergelijk dit met een 1 op veel relatie. 
Een ander bekend voorbeeld volgens dit concept is het observer pattern bij OO software ontwikkiling (zie vorig jaar).

De subscriber, is diegene die data 'kwijt' wilt en zal aan zijn data een onderwerp geven. 
De subscriber zend zijn data niet rechtstreeks naar alle clients. 
Hij zend zijn data naar een **Message Broker**, deze stap noemt *pusblish*.

Dit is een apparaat waar de logica gebeurd. 
Deze zorgt ervoor dat iedereen die gesubscript is op het onderwerp dat de subscriber meegeeft aan zijn data die data ontvangt. 

![MQTT netwerk](http://www.hivemq.com/wp-content/uploads/pub-sub-mqtt-1024x588.png)

Er is hierdoor een splitsing tussen zender en ontvanger. De zender en ontvanger weten niets van elkaar (IP-adres, ...)
Hierdoor kan de zender een klein sensor zijn. Het heeft immers niet veel processing power nodig. 
De **message broker** daar in tegen moet genoeg connecties kunnen verwerken. 
Ook moet het genoeg rekenkracht hebben om alle topics (onderwerpen) te filteren en uit te zoeken wie de data allemaal moet ontvangen!
Ook zal deze zorgen voor authenticatie. Niet iedereen mag zich subscriben op bepaalde onderwerpen.
Een **message broker** kan zeer schaalbaar uitgevoerd worden. Het leunt zich daarom perfect voor de cloud!

MQTT werkt op het TCP protocol. Maar is gebouwd met een lage footprint. Dit wilt zegen weinig overhead.
Hierdoor zijn de MQTT pakketjes zeer klein t.o.v. HTTP. Wat zorgt dat het sneller gaat en dat het minder energie gebruikt. 
Beide belangrijke kenmerken in een IOT netwerk.

NOOT: De foto komt van een volledige uit één zetting van MQTT door HiveMQ. [Hier is de link voor deel 1 van hun uitleg](http://www.hivemq.com/blog/mqtt-essentials-part-1-introducing-mqtt).
De rest van de uitleg vind je telkens bij Pingback bij de reacties!

### 27. Geef de 5 eigenschappen van MQTT
##### TCP/IP Based
MQTT bevindt zich op de *application layer*. Voor de rest van de stack rust het op TCP als transport layer en IP als internet / network layer.
Alle apparaten in je MQTT netwerk moeten dus beschiken over de TCP/IP stack!</br>
Het chat systeem van Facebook gebruikt MQTT!

##### Quality of Service
MQTT heeft een vorm van QoS ingebouwd. TCP zelf is al vrij betrouwbaar. 
Maar danzij QoS kan MQTT ook *application-to-application reliability* toevoegen. Dit kan belangrijk zijn op slechte netwerken.
Draadloze connecties met veel 'ruis', interferentie zijn hier een voorbeeld van (worden vaak gebruikt in een IoT netwerk).

MQTT lost dit op door te werken met 3 niveaus
- **Level 0: At most once** </br>
  Voegt geen extra QoS zekerheden toe. Gebruikt dus de mechanismen van het onderliggend TCP protocol (3 - way handshake, ...). 
  Er zullen geen ACK's gestuurd worden op applicatie niveau. 
- **Level 1: At least once** </br>
  Zender stuurt zijn data repetetatief naar Message Broker tot hij een ACK krijgt (applicatie niveau). 
  Vanaf hij deze ontvangen heeft stopt hij met die data te verzenden. Nu zijn we minstens zeker dat de broker de data ontvangen heeft.
  Het kan zijn dat de zender de dezelfde data meer dan één keer gezonden heeft!
- **Level 2: Exactly once**
  Hoogste level! Het is het veiligste maar traagste mechanisme voor QoS in het MQTT protocol.
  Het gebruikt heen en weer communicatie tussen zender en ontvanger om zeker te zijn dat de data is aangekomen.

##### Lightweight
MQTT is gebouwd om zo weinig mogelijk overhead te hebben. Hierdoor zijn de headers van de applicatie laag zeer klein. Hierdoor kan MQTT efficienter werken. Dit betekend dat het niet veel energie en bandbreedte nodig heeft.

|Header|Size| 
|------|----|
|MQTT 'PUBLISH'| 2-4 bytes|
|MQTT 'CONNECT'| 14 bytes|
|HTTP| 0.1 - 1 **k**bytes|

##### Retain
TODO

##### Last Will en Testament
TODO

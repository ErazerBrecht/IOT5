# IOT 5
Samenvatting door Brecht Carlier voor het vak *Internet Of Things* op AP Hogeschool (Elektronica - ICT).

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
Zoals altijd is een software matige oplossing altijd slechter dan een hardware matige oplossing. Zo kan het minder snel werken en kan het meer energie gebruiken.

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

#### 8. Stop signal
Analoog aan het start signaal maar andersom. Samen met start sequentie, enigste keer wanneer SDA veranderd terwijl de clock hoog is.

Het stop signaal gaat ook altijd uit van de master.
Het stop signaal is een LAAG naar HOOG transitie op de SDA lijn terwijl het klok signaal op een logisch HOOG signaal staat.

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
TODO


# 2 misja projektu Antares - czerwiec 2019
Celem misji jest wysłanie komórek ludzkich i zwierzęcych różnego typu na pokładzie balonu stratosferycznego, aby wystawić je na działanie promieniowania obecnego w stratosferze, które jest wielokrotnie wyższe niż na powierzchni Ziemi. 

**Główne założenia:**
* na pokładzie balonu znajdować się będzie 12 pojemików (T25 cell culture flask) zawierających specjalny żel z odżywką i tlenem, w którym umieszczone zostaną różne linie komórek,
* temperatura żelu z komórkami pozostanie w zakresie 35-37°C w czasie całego lotu
* na pokładzie balonu znajdować się będą czujniki umożliwiające pomiar dawki promieniowania

## Opis systemu
![#uml](https://github.com/simle-stardust/docs/blob/master/general-diagram.jpg)

### Gondola sensorowa
Zawierać będzie trzy płytki PCB. Pierwsza z nich zawierać będzie:
* mikrokontroler STM32L151RDT6
* karta microSD do zapisu danych z czujników
* moduł HC-08 do komunikacji BLE z systemem odcięcia
* moduł ESP8266 do komunikacji przez WiFi
* 7 czujników DS18B20PAR do pomiaru temperatury: 4 umieszczone bezpośrednio na płytce, 1 na zewnątrz gondoli, 2 przy bateriach,
* czujniki korzystające z magistrali I2C: Pololu AltIMU 10 v3, MAX30205, DS3231, HSCDANN001BA2A3, HDC1080
* tranzystory do kontrolowania zasilania brzęczyka i rezystora grzejnego (do baterii)
* wyprowadzenia do podłączenia płytki PCB z licznikiem geigera
* wyprowadzenia do podłączenia zewnętrznej płytki PCB
> schemat: https://trello.com/c/jY1YlqFJ/54-pcb-elektronika  
> oprogramowanie: https://github.com/simle-stardust/Antares2018  

Druga płytka PCB zawierać będzie:
* mikrokontroler z rodziny STM32
* wyprowadzenia do trzech tub Geigera-Mullera - SBM20
* przetwornicę na 400V do zasilenia tub
> schemat: w przygotowaniu  
> oprogramowanie: w przygotowaniu  

Trzecia płytka PCB znajdować się będzie na zewnątrz balonu i zawierać będzie:
* sygnalizacyjne diody LED
* moduł RF-LORA zawierający chip SX1272 do komunikacji dalekobieżnej za pomocą protokołu LoRa
* GPS - uBlox NEO 6M
* Brzęczyk
> schemat: w przygotowaniu  

Poza płytkami, w gondoli sensorowej znajdą się trzy niezależne źródła zasilania: 
* 6V dla ogólnie pojętej elektroniki
* 6V dla rezystora grzejnego
* 9V dla komunikacji przez LoRa

### Gondola eksperymentalna
Zawierać będzie płytkę PCB odpowiedzialną za pomiar temperatury próbek biologicznych i jej kontrolę poprzez sterowanie zasilaniem rezystorów grzewczych.
> schemat: https://trello.com/c/RVC6CGu9/56-pcb-medycyna  
> oprogramowanie: w przygotowaniu  

### Interfejs stacja naziemna <-> PC [WIP]
Stacja naziemna komunikuje się z komputerem poprzez port szeregowy, wykorzystując interfejs UART mikrokontrolera STM32 będącego centralnym elementem stacji naziemnej. W chwili obecnej komunikaty wysyłane na komputer wyglądają następująco:   
![#interface](https://github.com/simle-stardust/docs/blob/master/interface.png)  

Powyższe ramki zbudowane są następująco:  
`[ALT3][ALT2][ALT1][ALT0][LAT3][LAT2][LAT1][LAT0][LON3][LON2][LON1][LON0][TEMP1][TEMP0][TEMPINFO2][TEMPINFO1][TEMPINFO0][LF1][LF0][ST1][ST0][STKOM1][STKOM0]`  
> każdy nawias kwadratowy to jeden bajt  
> przedrostek `0x` oznacza zapis heksadecymalny  
* `[ALT3][ALT2][ALT1][ALT0]` - 4 bajty zawierające wysokość, np. `0x00001234` oznacza wysokość `4660m`
* `[LAT3][LAT2][LAT1][LAT0]` - 4 bajty zawierające szerokość geograficzną odczytaną z GPS, pomnożoną przez 10000 np. `0x20527EC6` oznacza `5422.77318 N` (litera domyślnie bo przecież półkuli nie zmienimy w trakcie lotu),
* `[LON3][LON2][LON1][LON0]` - 4 bajty zawierające długość geograficzną odczytaną z GPS, pomnożoną przez 10000 np. `0xAF21BF0` oznacza `1836.39024 E` (litera jw.)
* `[TEMP1][TEMP0]` - starszy i młodszy bajt zawierające średnią temperaturę próbek przemnożoną przez 100, np. `0x1234` oznacza tempraturę `46.60°C`
* `[TEMPINFO2][TEMPINFO1][TEMPINFO0]` - 3 bajty (24 bity) zawierające informacje o teperaturach każdej próbki (jest 12 próbek, numerowanie od 11 do 0), każde kolejne dwa bity to info o próbce, zakodowane w następujący sposób:
    * `00` - temperatura ok (między 35 a 37 stopni)
    * `01` - temperatura za wysoka (powyżej 37)
    * `10` - teperatura za niska (poniżej 35)
    * `11` - jakiś error code, np. awaria czujnika    
    * Czyli jeśli nasze 3 bajty `TEMPINFO` wyglądają tak: `0x1500A0`, czyli `000101010000000010100000`, czyli możemy to podzielić na
  `|00|01|01|01|00|00|00|00|10|10|00|00`, to wiemy że:  
      * próbki nr 11, 7, 6, 5, 4, 1, 0 mają emperatury OK,
      * próbki nr 10, 9, 8 mają za wysoką temperaturę,
      * próbki nr 3, 2 mają za niską temperaturę.
* `[LF1][LF0]` - starszy i młodszy bajt ostatnio odebranej ramki ze stacji, np. `0x0101` oznacza że odebrano ramkę `0x0101`
* `[ST1][ST0]` - starszy i młodszy bajt statusu, łącznie 16 bitów, każdy bit odpowiada za inny error:  
     * `I2C_PRESSURE_ERR` - błąd czujnika ciśnienia HSC, gdy zerowy bit `Status` = `1`  
     * `I2C_RTC_ERR` - błąd zegara czasu rzeczywistego, gdy 1 bit `Status` = `1`  
     * `I2C_GEIGER_ERR` - błąd licznika geigera, gdy 2 bit `Status` = `1`   
     * `I2C_MAX30205_ERR` - błąd czujnika temperatury MAX30205, gdy 3 bit `Status` = `1`  
     * `I2C_HDC1080_ERR` - błąd czujnika wilgotności, gdy 4 bit `Status` = `1`  
     * `I2C_INA3221_ERR` - błąd czujnika prądu, gdy 5 bit `Status` = `1`  
     * `I2C_GYRO_ERR` - błąd żyroskopu, gdy 6 bit `Status` = `1`  
     * `I2C_ACC_ERR` - błąd akcelerometru, gdy 7 bit `Status` = `1`  
     * `I2C_BARO_ERR` - błąd czujnika ciśnienia z IMU, gdy 8 bit `Status` = `1`  
     * `DS18_ERR` - błąd jednego z czujników temperatury DS18B20, gdy 9 bit `Status` = `1`  
     * `SD_ERR` - błąd karty SD, gdy 10 bit `Status` = `1`  
     * `GPS_ERR` - błąd GPSu, gdy 11 bit `Status` = `1`  
     * `WIFI_ERR` - błąd komunikacji za pomocą WiFi, gdy 12 bit `Status` = `1`  
     * `BLE_ERR` - błąd komunikacji za pomocą Bluetooth, gdy 13 bit `Status` = `1`  
     * `UNDEF_ERR1` - **do uzgodnienia**, gdy 14 bit `Status` = `1`  
     * `UNDEF_ERR2` - **do uzgodnienia**, gdy 15 bit `Status` = `1`  
* `[STKOM1][STKOM0]` - starszy i młodszy bajt statusu gondoli z komórkami, na razie nie wiadomo jakie dokładnie tam będą wartości (musiałoby ruszyć programowanie gondoli z komórkami)

### Protokół komunikacji przez WiFi [WIP]
Schemat blokowy komunikacji przez WiFi przedstawiony jest poniżej:  
![#wifi-proto](https://github.com/simle-stardust/docs/blob/master/wifi-proto.png)  
Główną rolę pełni ESP8266 obecne w gondoli głównej. Udostępnia ono sieć lokalną po podłączeniu do której możliwe jest wyświetlenie aktualnych danych pomiarowych np. przez przeglądarkę. Dane które powinny być udostępnione na serwerze to:  
  
**Dane z gondoli głównej**
* dane z zegara czasu rzeczywistego - godzina, minuta, sekunda \[3 bajty/24 bitów\],  
* 7 temperatur z czujników DS18B20 \[każdy ma 2 bajty/16 bitów\],  
* dane z czujnika wilgotności  \[2 bajty/16 bitów\],   
* dane z czujnika ciśnienia  \[2 bajty/16 bitów\],  
* szerokość geograficzna \[4 bajty/32 bitów\],   
* długość geograficzna \[4 bajty/32 bitów\],   
* wysokość \[4 bajty/32 bitów\],  
* status flagi \[2 bajty/16 bitów\],  
  
**Dane z gondoli komórkowej**
* 30 temperatur z czujników RTD z gondoli komórkowej \[każdy ma 2 bajty/16 bitów\],  
* 12 statusów sygnału sterującego mosfetami \[każdy ma 2 bajty/16 bitów\],   
* status flagi gondoli z komórkami  \[2 bajty/16 bitów\].  
  
**Dane z odcinacza**
* status odcinacza - czy sygnał odcinający został uruchomiony  \[1 bajt/8 bitów\]. (1 bit by wystarczył ale pewnie łatwiej będzie parsować dane bajtami).
  
Aby aktualizować i pobierać dane z serwera, poszczególne moduły ESP powinny obsługiwać następujące komendy przez interfejs UART:
>Uwaga! Wartości w kwadratowych nawiasach powinny być traktowane jako surowe bajty a nie jako znaki kodu ASCII!!!  
**ESP8266 w gondoli głównej**  
* `@MarcinOdcinaj!` - powoduje wysłanie komendy do odcinacza skutkującej włączeniem przepływu prądu przez drut oporowy (odcięcie balonu). Całkowita długość ramki, łącznie z headerem i znakiem końca to 15 bajtów.  
* `@MarcinSetValues:[HH],[MM],[SS],[DS18B20_1_1][DS18B20_1_0],[DS18B20_2_1][DS18B20_2_0],[DS18B20_3_1][DS18B20_3_0],[DS18B20_4_1][DS18B20_4_0],[DS18B20_5_1][DS18B20_5_0],[DS18B20_6_1][DS18B20_6_0],[DS18B20_7_1][DS18B20_7_0],[HUM1][HUM0],[PRES1][PRES0],[LAT3][LAT2][LAT1][LAT0],[LON3][LON2][LON1][LON0],[ALT3][ALT2][ALT1][ALT0],[SF1][SF0]!` - powoduje ustawienie wysłanych wartości jako aktualne wartości pomiarowe na serwerze. Całkowia długość komendy, łącznie z headerem i znakiem końca to 68 bajtów. Poszczególne zmienne oznaczają:
  * `[HH]` - godzina,  
  * `[MM]` - minuta,  
  * `[SS]` - sekunda,  
  * `[DS18B20_x_1][DS18B20_x_0]` - starszy i młodszy bajt temperatury z czujnika DS18B20 o numerze `x`,  
  * `[HUM1][HUM0]` - starszy i młodszy bajt wilgotności,  
  * `[PRES1][PRES0]` - starszy i młodszy bajt ciśnienia,  
  * `[LAT3][LAT2][LAT1][LAT0]` - 4 bajty zawierające szerokość geograficzną,  
  * `[LON3][LON2][LON1][LON0]` - 4 bajty zawierające długość geograficzną,  
  * `[ALT3][ALT2][ALT1][ALT0]` - 4 bajty zawierające wysokość,  
  * `[SF1][SF0]` - 2 bajty zawierające status flagi.  
* `@MarcinGetValues!` powoduje zwrócenie aktualnych wartości pomiarowych z serwera. Całkowita długość komendy to 17 bajtów. Odpowiedź na tą komendę powinna mieć postać  
`@MarcinOK:[RTD1_1][RTD1_0],[RTD2_1][RTD2_0],[RTD3_1][RTD3_0],[RTD4_1][RTD4_0],[RTD5_1][RTD5_0],[RTD6_1][RTD6_0],[RTD7_1][RTD7_0],[RTD8_1][RTD8_0],[RTD9_1][RTD9_0],[RTD10_1][RTD10_0],[RTD11_1][RTD11_0],[RTD12_1][RTD12_0]!` (łączna długość = 46 bajtów), gdzie:
  * `[RTDx_1]` - starszy i młodszy bajt temperatury z RTD o numerze `x`,  
  
**ESP8266 w gondoli z komórkami**
* `@MarcinSetValuesKom:[RTD1_1][RTD1_0],[RTD2_1][RTD2_0],[RTD3_1][RTD3_0],(...),[RTD30_1][RTD30_0],[MOSFET1_1][MOSFET1_0],[MOSFET2_1][MOSFET2_0],(...),[MOSFET12_1][MOSFET12_0],[SFKOM1][SFKOM0]!` - powoduje utawienie wysłanych wartości jako aktualne wartości pomiarowe na serwerze. Łączna długość ramki to 150 bajtów. Poszczególne zmienne oznaczają:
  * `[RTDx_1][RTDx_0]` - starszy i młodszy bajt temperatury z RTD o numerze `x`,  
  * `[MOSFET12_1][MOSFET12_0]` - starszy i młodszy bajt sygnału sterującego MOSFETAMI,
  * `[SFKOM1][SFKOM0]` - status flagi gondoli z komórkami.

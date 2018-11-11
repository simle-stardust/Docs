# 2 misja projektu Antares - styczeń 2019
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

* 1 - komunikat o braku odebrania pakietu w ciągu ostatnich ~20 sekund
* 2 - surowy, nieprzetworzony pakiet odebrany przez stację naziemną, zgodny z protokołem LoRaWAN ([dokumentacja protokołu](https://lora-alliance.org/sites/default/files/2018-04/lorawantm_specification_-v1.1.pdf) )  
 `[MHDR][DevAddr][Fctrl][Fcnt][Fport][Payload][MIC]`
     * `[MHDR]` - 1 bajtowy nagłówek LoRaWAN, w naszym przypadku zawsze `0x40`,
     * `[DevAddr]` - 4 bajtowy adres urządzenia, little endian, w naszym przypadku zawsze `0xC5130126`,
     * `[Fctrl]` - 1 bajt różnych opcje kontrolne, u nas nieużywane i zawsze `0x00`,
     * `[Fcnt]` - 2 bajtowy licznik wysłanych ramek, little endian,
     * `[Fport]` - 1 bajtowy port, u nas zawsze `0x01`,
     * `[Payload]` - 8 bajtów właściwej wiadomości (w przyszłości pewnie więcej), zaszyfrowane za pomocą algorytmu AES128 przy użyciu supertajnych kluczy urządzenia, na obrazku jest to `DEE18036B75476EE`
     * `[MIC]` - 4 bajty Message Integrity Code, coś w stylu CRC, obliczane na podstawie kluczy urządzenia i licznika wysłanych ramek, dzięki temu mamy pewność że wiadomość faktycznie pochodzi z naszego urządzenia
* 3 - wartości uzyskane po odszyfrowaniu i odpowiednim przetworzeniu pola `Payload` ramki LoRaWAN, jest to
     * `Alt` - wysokość
     * `Temp` - temperatura
     * `Status` - specjalna zmienna zawierająca zakodowany stan balonu - tzn. stan czujników, co działa, co nie działa itd.
     * `Last received frame` - ostatnia wiadomość którą odebrał balon ze stacji nadawczej, `0xFFFF` oznacza że nie odebrano żadnej. Komunikacja z balonem jest obustronna, po odebraniu każdego pakietu z balonu stacja nadawcza automatycznie wysyła 2 bajty wiadomości potwierdzającej. Domyślnie jest to `0x0001`, użytkownik może zmodyfikować wysyłaną wiadomość poprzez nadanie przez UART odpowiedniego znaku, **jeszcze do uzgodnienia**
* 4 - komunikaty o błędach czujników wyznaczone na podstawie zmiennej `Status`
* 5 - SNR i RSSI odebranego pakietu

**Payload**   
Właściwa zawartość ramki LoRaWAN wysłanej z balonu zbudowana jest w następujący sposób:  
`[ALT1][ALT0][TEMP1][TEMP0][LF1][LF0][ST1][ST0]`
* `[ALT1][ALT0]` - starszy i młodszy bajt zawierające wysokość, np. `0x1234` oznacza wysokość `4660m`
* `[TEMP1][TEMP0]` - starszy i młodszy bajt zawierające temperaturę przemnożoną przez 100, np. `0x1234` oznacza tempraturę `46.60°C`
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

# 2 misja projektu Antares - styczeń 2019
Celem misji jest wysłanie komórek ludzkich i zwierzęcych różnego typu na pokładzie balonu stratosferycznego, aby wystawić je na działanie promieniowania obecnego w stratosferze, które jest wielokrotnie wyższe niż na powierzchni Ziemi. 

**Główne założenia:**
* na pokładzie balonu znajdować się będzie 12 pojemików (T25 cell culture flask) zawierających specjalny żel z odżywką i tlenem, w którym umieszczone zostaną różne linie komórek,
* temperatura żelu z komórkami pozostanie w zakresie 35-37°C w czasie całego lotu
* na pokładzie balonu znajdować się będą czujniki umożliwiające pomiar dawki promieniowania

## Opis systemu
![#uml](http://www.plantuml.com/plantuml/png/SoWkIImgAStDuOfspyzBoSz9L4XAhCelJunLqDEpKuWEBabCpafKo4kioapDpKkCvK8JWQh3q6obu9CVb8ZTKBXWbK9gTd51Qb5bRcfUIMekI5Tufbic5su5c7Q1dLHP39HLo4z9pZmoCpaJQ8sDdXw6EYi59nzNBbmU272ELU02abZz3VPGGNvHYK9nLMfHQdf-UIMNGsfU2Z3S0000)

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

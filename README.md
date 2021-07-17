# STM32_LCD


Podłączenie SPI
Schemat podstawowego podłączenia jest następujący:

![alt text](https://cdn.forbot.pl/blog/wp-content/uploads/2015/11/381px-SPI_single_slave.svg_.png)


Jak widzimy do podłączenia potrzebujemy 4 linii. Najczęściej master-em jest mikrokontroler i takim przypadkiem będziemy się dalej zajmować. Można również wykorzystywać STM32 jako slave, ale wbrew pozorom jest to trudniejszy przypadek, niż wersja master.

Pierwsza linia jest oznaczana SS (Slave Select), która służy do wyboru urządzenia – pozostałe linie mogą być podpięte do wielu układów, ale tylko jeden może być w danej chwili aktywny. Linia ta często jest nazywana CS, czyli Chip Select - i tej nazwy będziemy używać podczas kursu.

Pozostałe linie to:

MOSI (Master Output Slave Input) – dane wysyłane z mastera do slave
MISO (Master Input Slave Output) – dane wysyłane z slave do mastera
SCLK (Serial Clock) – linia zegara
Dzięki temu, że sygnał zegarowy jest przesyłany na liniach interfejsu, komunikacja jest znacznie łatwiejsza. Możliwe jest również uzyskanie znaczenie większych prędkości transmisji niż w przypadku interfejsu UART.

Wybór aktywnego układu dokonywany jest przez wystawienie stanu niskiego na odpowiedniej linii CS. Tylko jeden układ może być aktywny w danej chwili (pozostałych liniach CS stan wysoki).

![alt_text](https://cdn.forbot.pl/blog/wp-content/uploads/2015/11/363px-SPI_three_slaves.svg_.png)


Przeczytac:
Komunikacja SPI na STM

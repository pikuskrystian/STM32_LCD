Wyświetlacz graficzny

RST - linia resetująca rejestry wyświetlacza. Przed rozpoczęciem pracy z wyświetlaczem, należy wygenerować na niej stan zero przez co najmniej 100 ns. Podczas normalnej pracy wyświetlacza, na tej linii powinien być ciągle stan wysoki.
CE - jest to linia CS interfejsu SPI, nazwa pochodzi od Chip Enable.
DC - linia ustalająca, czy przesyłamy dane (stan wysoki), czy komendy (stan niski).
Din - linia danych, czyli MOSI interfejsu SPI.
CLK - linia zegarowa SPI, odpowiada SCLK.
VCC - napięcie zasilające moduł (3.3V).
BL - podświetlanie wyświetlacza.
GND - masa.

Jak widzimy znajdziemy tutaj jednokierunkowy interfejs SPI, czyli nie możemy odczytywać danych z wyświetlacza. Producent zastosował inne oznaczenia linii (CE zamiast CS, Din zamiast MOSI i CLK zamiast SCLK), jednak sam interfejs działa bez zmian.

Dodatkowe linie sterujące to: RST, która resetuje zawartość pamięci i rejestrów sterownika wyświetlacza oraz DC, która pozwala na wybór typu przesyłanych danych. Można przesyłać dane przeznaczone do wyświetlania lub komendy nim sterujące. Poza tym mamy trzy linie zasilające: masę (GND), zasilanie 3.3V (VCC) oraz zasilanie dla podświetlania (BL).


Komunikacja z wyświetlaczem
Ponieważ nowych linii sterujących jest nieco więcej, zdefiniujemy stałe, które będą określały, do których pinów podłączyliśmy wyświetlacz (wszystkie dotyczą portu C):
```
#define LCD_DC			GPIO_PIN_1
#define LCD_CE			GPIO_PIN_2
#define LCD_RST			GPIO_PIN_3
```

Konfiguracja SPI została przedstawiona w poprzednik repozytorium, teraz kod wygląda dokładnie tak samo. Dodajemy tylko konfigurację nowych linii:
```
   gpio.Mode = GPIO_MODE_OUTPUT_PP;
	 gpio.Pin = LCD_DC|LCD_CE|LCD_RST;
	 HAL_GPIO_Init(GPIOC, &gpio);
	 HAL_GPIO_WritePin(GPIOC, LCD_CE|LCD_RST, GPIO_PIN_SET);
```
Linia CE oraz RST jest aktywowana stanem niskim, więc wystawiamy na nich logiczną jedynkę. Pierwszym krokiem podczas komunikacji z wyświetlaczem jest zresetowanie jego kontrolera. W tym celu, na wyjściu RST musimy wygenerować stan 0 przez co najmniej 100 ns.

Napiszmy więc prostą procedurę:
```
void lcd_reset()
{
	HAL_GPIO_WritePin(GPIOC, LCD_RST, GPIO_PIN_RESET);
	HAL_GPIO_WritePin(GPIOC, LCD_RST, GPIO_PIN_SET);
}
```
Opóźnienie nie było konieczne, ale w razie problemów zawsze można dodać małe opóźnienie do procedury resetowania.

Następny etap to wysyłanie komend sterujących pracą wyświetlacza. Do rozróżnienia między danymi, a komendami służy linia DC, stan niski informuje o wysyłaniu komendy sterującej. Teraz możemy napisać prostą procedurę, która wyśle komendę do wyświetlacza:
```
void lcd_cmd(uint8_t cmd)
{
	HAL_GPIO_WritePin(GPIOC, LCD_CE|LCD_DC, GPIO_PIN_RESET);
	HAL_SPI_Transmit(&spi, &cmd, 1, HAL_MAX_DELAY);
	HAL_GPIO_WritePin(GPIOC, LCD_CE|LCD_DC, GPIO_PIN_SET);
}
```
Ostatni etap, to wysłanie danych do wyświetlania. Przygotujemy procedurę, która za jednym wywołaniem prześle cały bufor z pamięci RAM lub Flash do wyświetlacza. Jako parametry będziemy podawali adres bufora oraz liczbę bajtów do przesłania:
```
void lcd_data(const uint8_t* data, int size)
{
	HAL_GPIO_WritePin(GPIOC, LCD_DC, GPIO_PIN_SET);
	HAL_GPIO_WritePin(GPIOC, LCD_CE, GPIO_PIN_RESET);
	HAL_SPI_Transmit(&spi, (uint8_t*)data, size, HAL_MAX_DELAY);
	HAL_GPIO_WritePin(GPIOC, LCD_CE, GPIO_PIN_SET);
}
```

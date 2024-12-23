# Выполнили Погребняк Степан Сиваселвамович и Кирилин Геннадий Дмитривевич

# Цель работы

![image](https://github.com/user-attachments/assets/8ca4a071-42d4-4af1-9fd4-adedc3c6d6d6)




```
#include <stdint.h>
#include <stdbool.h>

// Определения регистров и пинов
#define GPIOA_BASE  0x48000000
#define RCC_BASE    0x40021000

#define GPIOA_MODER     (*(volatile uint32_t*)(GPIOA_BASE + 0x00))
#define GPIOA_ODR       (*(volatile uint32_t*)(GPIOA_BASE + 0x14))
#define RCC_AHBENR      (*(volatile uint32_t*)(RCC_BASE + 0x14))

#define LCD_RS_PIN  (1 << 0) // PA0
#define LCD_EN_PIN  (1 << 1) // PA1
#define LCD_D4_PIN  (1 << 2) // PA2
#define LCD_D5_PIN  (1 << 3) // PA3
#define LCD_D6_PIN  (1 << 4) // PA4
#define LCD_D7_PIN  (1 << 5) // PA5

// Задержка
void simpleDelay(volatile uint32_t count) {
    while (count--);
}

// Отправка команды на ЖКИ
void lcdSendCommand(uint8_t cmd) {
    GPIOA_ODR &= ~LCD_RS_PIN; // RS = 0 (команда)
    GPIOA_ODR |= (cmd & 0xF0); // Старшие 4 бита
    GPIOA_ODR |= LCD_EN_PIN;  // EN = 1
    simpleDelay(5000);
    GPIOA_ODR &= ~LCD_EN_PIN; // EN = 0

    GPIOA_ODR &= 0xFFFFFFF0; // Очистить старшие биты
    GPIOA_ODR |= ((cmd << 4) & 0xF0); // Младшие 4 бита
    GPIOA_ODR |= LCD_EN_PIN;  // EN = 1
    simpleDelay(5000);
    GPIOA_ODR &= ~LCD_EN_PIN; // EN = 0
}

// Отправка данных на ЖКИ
void lcdSendData(uint8_t data) {
    GPIOA_ODR |= LCD_RS_PIN; // RS = 1 (данные)
    GPIOA_ODR |= (data & 0xF0); // Старшие 4 бита
    GPIOA_ODR |= LCD_EN_PIN;  // EN = 1
    simpleDelay(5000);
    GPIOA_ODR &= ~LCD_EN_PIN; // EN = 0

    GPIOA_ODR &= 0xFFFFFFF0; // Очистить старшие биты
    GPIOA_ODR |= ((data << 4) & 0xF0); // Младшие 4 бита
    GPIOA_ODR |= LCD_EN_PIN;  // EN = 1
    simpleDelay(5000);
    GPIOA_ODR &= ~LCD_EN_PIN; // EN = 0
}

// Инициализация ЖКИ
void lcdInit() {
    // Инициализация GPIO
    RCC_AHBENR |= (1 << 17); // Включение GPIOA
    GPIOA_MODER &= ~0x3FF;   // Очистка режимов для PA0-PA5
    GPIOA_MODER |= 0x155;    // Установка режимов Output для PA0-PA5

    simpleDelay(50000);

    lcdSendCommand(0x02); // Переход в 4-битный режим
    lcdSendCommand(0x28); // 2 строки, 5x7 символов
    lcdSendCommand(0x0C); // Включить дисплей, выключить курсор
    lcdSendCommand(0x06); // Автосмещение курсора вправо
    lcdSendCommand(0x01); // Очистка дисплея
    simpleDelay(50000);
}

// Вывод строки на ЖКИ
void lcdPrint(const char *str) {
    while (*str) {
        lcdSendData(*str++);
    }
}

int main() {
    lcdInit();

    // Отображение текста
    lcdPrint("Группа: 123");
    lcdSendCommand(0xC0); // Переход на вторую строку
    lcdPrint("Факультет: IT");

    while (1) {
        // Бесконечный цикл
    }

    return 0;
}

```

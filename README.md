# STM32F407 Non-Blocking UART & Circular Buffer Driver

## 📌 Overview
This repository contains a modular, interrupt-driven UART driver implemented on the STM32F407 microcontroller. The architecture utilizes **Circular Buffers** for both RX and TX lines, ensuring zero data loss and preventing main-loop blocking during asynchronous communication. 

Instead of relying on standard blocking HAL functions, this driver uses a custom firmware layer that handles data routing entirely inside hardware interrupts, making it highly efficient for industrial and real-time embedded applications.

## ⚙️ Key Features
*   **Interrupt-Driven Architecture:** Utilizes `RXNE` (Receive Not Empty) and `TXE` (Transmit Data Register Empty) hardware interrupts.
*   **Circular Buffer Implementation:** Dynamic buffers (128-byte) for safe data queuing (FIFO logic).
*   **Command Parsing:** Real-time string parsing (`strstr`, `strchr`) for hardware control (e.g., `LED1=ON`, `LED2=1`).
*   **ADC/DAC Integration:** Seamlessly handles DMA-based ADC conversions and continuous DAC output without interrupting UART communication.
*   **Modular Design:** Separated driver files (`uart_ex.c`, `uart_ex.h`) for easy porting to other STM32 families.

## 🧠 System Architecture & Data Flow
<img width="2816" height="1536" alt="Gemini_Generated_Image_nfh1bdnfh1bdnfh1" src="https://github.com/user-attachments/assets/e035d0f8-4326-46cf-b116-f173a4c7001d" />

*(Buraya resmi ekleyeceğiz, nasıl yapılacağını aşağıda anlattım)*

## 🛠️ Hardware Setup
*   **MCU:** STM32F407VGT6 (Discovery Kit)
*   **UART Interface:** USART3 (PB10 = TX, PB11 = RX)
*   **Baud Rate:** 9600 bps
*   **Converter:** USB-TTL (CH340/CP2102) - *Note: TX/RX lines must be cross-connected.*

## 🚀 Usage Example
The driver pulls complete lines from the circular buffer without halting the CPU:
```c
// Non-blocking read from the Circular Buffer
if(UARTx_ReadLine(&uart3, rxBuffer, sizeof(rxBuffer))) 
{
    // Parse the command safely
    if(strstr(rxBuffer, "LED2=") != NULL) 
    {
        char *equalPtr = strchr(rxBuffer, '=');
        if(equalPtr != NULL) 
        {
            uint8_t value = (uint8_t)atoi(equalPtr + 1);
            if(value == 1) ioInfo.outputInfo.ledOrange.pinState = GPIO_PIN_SET;
            else if(value == 0) ioInfo.outputInfo.ledOrange.pinState = GPIO_PIN_RESET;
        }
    }
    // Prevent buffer overflow and infinite loops
    memset(rxBuffer, 0, sizeof(rxBuffer)); 
}

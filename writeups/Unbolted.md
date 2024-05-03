![alt text](<../images/writeups/Unbolted/Pasted image 20240114151101.png>)

**MCU**: ARMX - STM32F103 - RBT6 - GH28593 - CHNGH712 (LQFP64)
Datasheet: https://www.mouser.fr/datasheet/2/389/stm32f103c8-1851025.pdf

**FLASH**: VP235 - 67M - A85J

**I2C serial EEPROM**: 24LC64 - J/SM - 1637WHK
Datasheet: https://ww1.microchip.com/downloads/aemDocuments/documents/MPD/ProductDocuments/DataSheets/24AA64-24FC64-24LC64-64-Kbit-I2C-Serial-EEPROM-20001189U.pdf
**SPI serial EEPROM**: 25LC080
Datasheet: https://ww1.microchip.com/downloads/aemDocuments/documents/MPD/ProductDocuments/DataSheets/25LCXXX-8K-256K-SPI-Serial-EEPROM-High-Temp-Family-Data-Sheet-20002131E.pdf

![alt text](<../images/writeups/Unbolted/Pasted image 20240114203933.png>)

# CHALL 1 - I2C

Frequency information:
![alt text](<../images/writeups/Unbolted/Pasted image 20240301191406.png>)

Read sequence at current address:
![alt text](<../images/writeups/Unbolted/Pasted image 20240301191552.png>)
![alt text](<../images/writeups/Unbolted/Pasted image 20240301191959.png>)
We understand that in order to read at curr addr we need to send the value:
```
0b10100111 == 0xa7
```


![alt text](<../images/writeups/Unbolted/Pasted image 20240114163533.png>)

![alt text](<../images/writeups/Unbolted/Pasted image 20240114163453.png>)

# CHALL 2 - UART

On fait le montage pour s'interfacer en série en UART sur la board et on branche en parallèle l'analyseur logique pour déterminer le baudrate. Le montage donne ça:

![alt text](<../images/writeups/Unbolted/Pasted image 20240310134249.png>)

Ensuite avec Pulseview on calcule le baudrate avec un signal d'appui sur une touche du touchpad de la board:
![alt text](<../images/writeups/Unbolted/Pasted image 20240310134030.png>)

baudrate = 1 / 17us = 1 / 0.000017 =~ 57600

Ensuite on décode ça en UART pour vérif sur Pulseview et on obtient:
![alt text](<../images/writeups/Unbolted/Pasted image 20240310134139.png>)
Bingo :)

Ensuite on s'interface en série avec miniterm:
![alt text](<../images/writeups/Unbolted/Pasted image 20240310134411.png>)
On se rend compte qu'il y a un menu avec une option FLAG qui permet de tester si le flag est bon, en testant plusieurs fois on se dit qu'il serait possible d'effectuer une timing attack sur cet input et donc on écrit ce script python3:
```python
import serial
import time
from string import printable

ser = serial.Serial('/dev/ttyUSB0', 57600)

# ser.write(b"1")
# ser.write(b"\x0A\x0D")
ser.write(b"FLAG")
# time.sleep(0.000017)
ser.write(b"\x0D\x0A")
print("Message envoyé avec succès")

flag = ""
chars = printable[:-6]

while True:
    best_time = 0
    best_c = ''
    for c in chars:
        buff = flag + c
        start = time.time()
        ser.write(buff.encode()+b"\x0A\x0D")
        ser.readlines(2)
        end = time.time()
        timer = end-start
        if timer > best_time:
            best_time = timer
            best_c = c

    print(best_time)
    flag = flag + best_c
    print(flag)
    if flag[-1] == '}':
        break


print(f"Flagged: {flag}")
ser.close()
```

Et on obtient bien le flag :)
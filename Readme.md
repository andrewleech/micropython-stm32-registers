# Micropython stm32 registers

This tool is a code generator to directly expose STM32 registers
as pure-micropython classes for easy use in applications.

It does this by parsing requested register definitions and their 
dependencies from the stmlib c headers and rewriting them as python 
consts and classes. Behind the scenes uctypes is used with the class
definitions acting like a wrapper around the uctypes definitions.

## Tool Usage
The register builder tool is run on desktop with cpython, directly 
from this repo, eg.
``` bash
$ python3 stm_register_builder.py stm32wb55 DMA2* FLASH_*
```

Available registers can be listed like so:
``` bash    
$ python3 stm_register_builder.py stm32wb55 --list
Listing register definitions from stm32lib/CMSIS/STM32WBxx/Include/stm32wb55xx.h
TIM2
LCD
RTC
WWDG
...
```

The generated file can become _quite_ large, so it's best to only 
include needed registers - other dependant registers and settings should
be pulled in automatically.  
The generated file should be inspected manually to see if there's any missing
or out of order definitions, sometimes a few manual cleanups can be needed.
There may be some commented out lines with content that can't be automatically
parsed, often ones with more complex type casting. These may need manual fixups.

## Autogenerated code usage
The output is a single file generally named `_stm_registers.py`

It can be used on board / in an application like so:
``` bash
mpremote mount .
```

``` python
>>> from _stm_registers import *
>>> FLASH.__inspect__()
FLASH_TypeDef
ACR        (0x0000):  0x0603
KEYR       (0x0008):  0x0000
OPTKEYR    (0x000c):  0x0000
SR         (0x0010):  0x8000
CR         (0x0014):  0xc0000000
ECCR       (0x0018):  0x0000
OPTR       (0x0020):  0x3dfff1aa
PCROP1ASR  (0x0024):  0x01ff
PCROP1AER  (0x0028):  0x80000000
WRP1AR     (0x002c):  0x00ff
WRP1BR     (0x0030):  0x00ff
PCROP1BSR  (0x0034):  0x01ff
PCROP1BER  (0x0038):  0x0000
IPCCBR     (0x003c):  0xffffc000
C2ACR      (0x005c):  0x0600
C2SR       (0x0060):  0x0000
C2CR       (0x0064):  0x0000
SFR        (0x0080):  0x10f4
SRRVR      (0x0084):  0xa083d000

>>> FLASH.KEYR = FLASH_KEY1
>>> FLASH.KEYR = FLASH_KEY2
>>> hex(FLASH.CR)
'0x40000000'
>>> FLASH.OPTKEYR = FLASH_OPTKEY1
>>> FLASH.OPTKEYR = FLASH_OPTKEY2
>>> hex(FLASH.CR)
'0x0'

```

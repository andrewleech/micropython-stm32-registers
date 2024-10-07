# Micropython stm32 registers

This tool is a code generator to directly expose STM32 registers
as pure-micropython classes for easy use in applications.

It does this by parsing requested register definitions and their 
dependencies from the stmlib c headers and rewriting them as python 
consts and classes. Behind the scenes uctypes is used with the class
definitions acting like a wrapper around the uctypes definitions.

## Tool Usage
The register builder tool is run on desktop with cpython, directly 
from this repo.

The generated register classes are subclasses of :
``` python
class Register:

    def __reg_addr__(self, register: str):
        return self.__addr__ + self.__regs__[register]

    def __repr__(self):
        return f"<{self.__class__.__name__} 0x{self.__addr__:x}>"

    def __dir__(self):
        return [k for k in self.__regs__]

    def __inspect__(self):
        print(self.__class__.__name__)
        struct = self.__struct__
        for k, v in sorted(self.__regs__.items(), key=lambda t: t[1]):
            val = getattr(struct, k)
            print(f"{k : 10} (0x{v:04x}):  0x{val:04x}")

    def __getattr__(self, __name: str):
        if __name.startswith("_"):
            raise AttributeError(__name)

        v = getattr(self.__struct__, __name)
        return v

    def __setattr__(self, __name: str, __value) -> None:
        if __name.startswith("_"):
            object.__setattr__(self, __name, __value)
        else:
            setattr(self.__struct__, __name, __value)

```

eg. for 
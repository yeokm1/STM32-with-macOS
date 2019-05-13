Using GCC and Makefiles on macOS to build STM32CubeMX projects
================================================================

As of v4.21.0, [STM32CubeMX] is now capable of generating Makefiles that can be used to build projects using the [GNU ARM Embedded Toolchain]. Makefiles allow you to be IDE independent and use you favorite text editor. For some people, IDEs are slow and take up a lot of resources. With a Makefile, building your project is as simple as typing `make` in your Terminal be you in Linux, Mac, or Windows. No more restrictions.

Although this tutorial has been written with macOS in mind, similar steps can be applied to Linux or Windows machines.


0 - Installing the toolchain
----------------------------
### Requirements:
- [STM32CubeMX] to generate project templates
- [STM32CubeProgrammer] to easily program STM32 products using a GUI
- A hardware development board (*e.g.* a [NUCLEO-L476RG] board)
- macOS Command Line Tools (CLT)
- [Homebrew] package manager (recommended to install gcc-arm-embedded, openOCD and stlink)
- [GNU ARM Embedded Toolchain] (arm-none-eabi) for compiler and other tools
- [OpenOCD] (>= 0.10.0) or [texane/stlink] for programming and running a GDB server


1. Install Xcode Command Line Tools (CLT). This will install *Make* and other UNIX goodies:
```
$ xcode-select --install
```
After the *Command Line Tools* were successfully installed, the remaining toolchain requirements can be installed using *Homebrew*.

2. Install *Homebrew*. Follow instructions available on [brew.sh][Homebrew]
3. Install GCC ARM Embedded Toolchain:
```
$ brew install armmbed/formulae/arm-none-eabi-gcc
$ arm-none-eabi-gcc --version
arm-none-eabi-gcc (GNU Tools for Arm Embedded Processors 8-2018-q4-major) 8.2.1 20181213 (release) [gcc-8-branch revision 267074]
Copyright (C) 2018 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

4. Install OpenOCD:
```
$ brew install openocd
$ openOCD --version
Open On-Chip Debugger 0.10.0
Licensed under GNU GPL v2
For bug reports, read
    http://openocd.org/doc/doxygen/bugs.html
```

5. Install open source [texane/stlink]:
```
$ brew install stlink
$ st-info --version
v1.4.0
```

6. Install [STM32CubeMX]. After Downloading the installer, extract the archieve and try to run the macOS installer. If the macOS installer doesn't work, use the following command to manually launch the install. The procedure is described in the **STM32CubeMX User Manual** [UM1718].
```
$ cd ~/Downloads/
$ unzip en.stm32cubemx.zip -d en.stm32cubemx
$ cd en.stm32cubemx
$ java -jar SetupSTM32CubeMX-4.26.1.exe
```

7. Install [STM32CubeProgrammer]. Similarly to CubeMX, if the installer doesn't work, use:

```
$ unzip en.stm32cubeprog.zip -d en.stm32cubeprog
$ cd en.stm32cubeprog
$ java -jar SetupSTM32CubeProgrammer-1.1.0.exe
```
If the above command does not work, you could try installing `java8`:
```
$ brew tap caskroom/versions
$ brew cask install java8
```
And Java 8 will be installed at `/Library/Java/JavaVirtualMachines/jdk1.8.xxx.jdk/`. You can then use the full java path to use version 1.8 to launch the  STM32CubeProgrammer setup.
```
$ /Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/bin/java -jar SetupSTM32CubeProgrammer-1.0.0.exe
```
**Pro Tip**: Create a symbolic link to one of the binary directory searched by your `$PATH` variable:
```
$ ln -sv /Applications/STMicroelectronics/STM32Cube/STM32CubeProgrammer/STM32CubeProgrammer.app/Contents/MacOs/bin/STM32_Programmer_CLI /usr/local/bin/
```
Then, `STM32_Programmer_CLI` can be invoked diectly without having to specify the full path:
```
$ STM32_Programmer_CLI
      -------------------------------------------------------------------
                        STM32CubeProgrammer v1.1.0
      -------------------------------------------------------------------


Usage : 
STM32_Programmer_CLI.exe [command_1] [Arguments_1][[command_2] [Arguments_2]...]
```

1 - Create a Project using CubeMX
---------------------------------
If you are also using an NUCLEO-L476RG, you can use the example *"blinky"* project by cloning the following repo:
```
$ git clone https://github.com/glegrain/STM32-with-macOS.git
$ cd STM32-with-macOS/Example_Project
```


Alternatively, you can generate you own project:
1. Create a **New Project**, and select your part number or development board
2. Configure your **Pins**, **Clock Settings** and **Peripherals**
3. When you click **Project->Generate Code**, the **Project Settings** window will show up. Under **Toolchain / IDE**, select **Makefile**.

![Project Settings](images/Project_Settings.png)

For more information, refer to the **STM32CubeMX User Manual** available on [st.com](http://www.st.com/resource/en/user_manual/dm00104712.pdf). Usefull sections include:
- Tutorial 1: From pinout to project C code generation using an STM32F4 MCU
- Tutorial 4: Example of UART communications with a STM32L053xx Nucleo board

2 - Configure your Makefile
-----------------------------
Unfortunately, Makefiles generated by CubeMX do not work *out-of-the-box*. You need to edit the file and set your compiler path. Luckily, this step only has to be done once. Later on, if you want to add source, header files or simply change your compiler options, refer to the [Editing your Makefile] section for more details.

1. Locate the **ARM Embedded GCC** compiler binary location:
```
$ which arm-none-eabi-gcc
/usr/local/bin/arm-none-eabi-gcc
```

2. Open up the **Makefile** with your favorite text editor to set the `BINPATH` variable to the location of your compiler returned above:
```Makefile
#######################################
# binaries
#######################################
BINPATH = /usr/local/bin
PREFIX = arm-none-eabi-
CC = $(BINPATH)/$(PREFIX)gcc
AS = $(BINPATH)/$(PREFIX)gcc -x assembler-with-cpp
CP = $(BINPATH)/$(PREFIX)objcopy
AR = $(BINPATH)/$(PREFIX)ar
SZ = $(BINPATH)/$(PREFIX)size
HEX = $(CP) -O ihex
BIN = $(CP) -O binary -S
```
**Pro Tip**: To make the Makefile more portable between different users and environment, you can remove the `BINPATH` variable and edit the `CC`, `AS`, `CP`, `AR`, `SZ` as shown bellow. This way, *make* will look for binaries in your environment (*i.e.* executables located in your `$PATH` setting):
```Makefile
#######################################
# binaries
#######################################
PREFIX = arm-none-eabi-
CC = $(PREFIX)gcc
AS = $(PREFIX)gcc -x assembler-with-cpp
CP = $(PREFIX)objcopy
AR = $(PREFIX)ar
SZ = $(PREFIX)size
HEX = $(CP) -O ihex
BIN = $(CP) -O binary -S
```

3 - Building your project
-------------------------
In a Terminal, navigate to your project's root directory (or Makefile location). Then use the `make` command to invoke the Makefile to compile your project:
```
$ cd ~/path/to/Example_Project
$ make
```
**Pro Tip**: for faster build time, `make` can be invoked using [parallel build](https://www.gnu.org/software/make/manual/make.html#Parallel) with the `-j` option:
```
$ make -j 4
```

If all goes well, you should see a result without any errors or warning:
```
$ make
...
arm-none-eabi-size build/Example_Project.elf
   text    data     bss     dec     hex filename
   8880      24    1688   10592    2960 build/Example_Project.elf
arm-none-eabi-objcopy -O ihex build/Example_Project.elf build/Example_Project.hex
arm-none-eabi-objcopy -O binary -S build/Example_Project.elf build/Example_Project.bin
```

**Pro Tip**: The Makefile generated by CubeMX comes with a predefined rule called `clean` to delete all generated files during the build process (object files, binaries, ... in the `build/` directory).
This rule is very useful to force rebuild all or to cleanup the project directory before packaging your project for archiving.
```
$ make clean
rm -fR .dep build
```


4 - Programming the board
-------------------------

### Option 1 - Using STM32CubeProgrammer GUI:
1. Open **STM32CubeProgrammer**
2. Connect a USB cable from the board to your computer
3. Click "**Connect**"
4. Go to the "Erasing & Programming" window
5. **Browse** to load the binary file (`*.hex`, `*.bin` or `*.elf` located in the `build/` directory)
    1. In case of a `*.bin` binary, the Start Address needs to be specified (typically `0x08000000` for STM32)
6. Click **Start Programming**
7. By default, CubeProgrammer does not run the application after programming. **Press the black reset button** to run the firmware. You should see LD2 blinking.

![STM32CubeProgrammer programming](images/CubeProg_program.png)

**Note**: Because STM32CubeProgrammer is still relatively new, chances are you will have to upgrade your ST-Link firmware.

For more information, you can refer to the [STM32CubeProgrammer User Manual](http://www.st.com/resource/en/user_manual/dm00403500.pdf)

### Option 1.1 - Using STM32CubeProgrammer CLI:
Below are some example commands to erase and program the target using STM32CubeProgrammer CLI:
```
/Applications/STM32CubeProgrammer/STM32CubeProgrammer.app/Contents/MacOs/bin/STM32_Programmer_CLI -c port=SWD -e all
/Applications/STM32CubeProgrammer/STM32CubeProgrammer.app/Contents/MacOs/bin/STM32_Programmer_CLI -c port=SWD mode=UR reset=HWrst -e all # hold reset button then release when connecting
/Applications/STM32CubeProgrammer/STM32CubeProgrammer.app/Contents/MacOs/bin/STM32_Programmer_CLI -c port=SWD -w build/Example_Project.elf
```

### Option 2 - Using texane stlink:
If all you want to do is program the board, then run any of the following commands:
```
$ st-flash write ./build/*.bin 0x08000000
$ st-flash --format ihex write ./build/*.hex
```


Otherwise, to program and debug run the gdb server with:
```
$ st-util
```

### Option 3 - Using OpenOCD:
OpenOCD requires a a configuration file. If you installed openOCD using Homebrew, list of provided configuration (`*.cfg`) files can be found using the following command:
```
$ ls /usr/local/Cellar/open-ocd/0.10.0/share/openocd/scripts/board/
$ ls /usr/local/Cellar/open-ocd/0.10.0/share/openocd/scripts/interface/
$ ls /usr/local/Cellar/open-ocd/0.10.0/share/openocd/scripts/target/
```

For example, you could the following command to program and verify using elf/hex/s19 files. Verify, reset and exit are optional parameters. Binary files need the flash address passing.
```
$ openocd -f board/st_nucleo_l476rg.cfg -c "program build/Example_Project.hex verify reset exit"
$ openocd -f interface/stlink-v2-1.cfg -f target/stm32l4x.cfg -c "program build/Example_Project.elf verify reset exit"
$ openocd -f interface/stlink-v2-1.cfg -f target/stm32l4x.cfg -c "program build/Example_Project.bin 0x08000000 verify exit"
```


More examples and documentation available [here](http://openocd.org/doc/html/Flash-Programming.html#Flash-Programming)

5 - Debugging
-------------
Because you will be debugging a remote target device, GDB needs to connect to a *gdbserver* compliant debugger. Before launching GDB, you need to start a GDB server using your debugger to act as an interface between GDB and your device.

### Step 1 - Start a GDB server

#### Option 1.1 - Using texane stlink
```
$ st-util
$ st-util --no-reset # to attach while running
$ st-util -p 3333    # for OpenOCD version of .gdbinit compatibility
```

#### Option 1.2 - Using OpenOCD
```
$ openocd -f interface/stlink-v2-1.cfg -f target/stm32l4x.cfg
$ openocd -f interface/stlink-v2-1.cfg -f target/stm32l4x.cfg -c "gdb_port 4242" # You can also specify to use the same port as st-util
```

### Step 2 - Launch GDB:

1. To program and debug in a single command, I recommend to creating a `.gdbinit` script

Here is my `.gdbinit`. Place this file in your project's root directory, next to your Makefile:
```GDB
file "./build/Example_Project.elf"

# Connect to texane stlink gdb server
target extended-remote :4242
# Or, connect to openOCD instead
# target remote localhost:3333

# monitor reset init
# monitor halt

# Uncomment to enable semihosting
# monitor arm semihosting enable

# Flash program and load symbols
load
break main

# Run to main (first breakpoint)
continue
```

2. Launch GDB. GDB will execute commands from the`.gdbinit` script when launched
```
$ arm-none-eabi-gdb
...
Loading section .isr_vector, size 0x188 lma 0x8000000
Loading section .text, size 0x1708 lma 0x8000188
Loading section .rodata, size 0x50 lma 0x8001890
Loading section .init_array, size 0x8 lma 0x80018e0
Loading section .fini_array, size 0x8 lma 0x80018e8
Loading section .data, size 0x8 lma 0x80018f0
Start address 0x80017d8, load size 6392
Transfer rate: 9 KB/sec, 1065 bytes/write.
Breakpoint 1 at 0x80016d6: file ./Src/main.c, line 83.
Note: automatically using hardware breakpoints for read-only addresses.

Breakpoint 1, main () at ./Src/main.c:83
83    HAL_Init();
(gdb)
```
**Pro Tip**: Launch GDB in GDB in Text User Interface (TUI) mode to show the source file and GDB commands in separate windows:
```
$ arm-none-eabi-gdb -tui
```

### Step 3 - Using GDB

### Program stepping/execution:
https://sourceware.org/gdb/onlinedocs/gdb/Continuing-and-Stepping.html

Step over (step to next line of C code without going into functions):
```GDB
(gdb) next
90    SystemClock_Config();
(gdb) n
97    MX_GPIO_Init();
(gdb) 
```

Step into (step to next line of C, goes into functions):
```GDB
(gdb) step
SystemClock_Config () at ./Src/main.c:124
124 {
```
**Note**: Use `stepi` for assembly instruction stepping.

Return from a function:
```GDB
(gdb) finish
Run till exit from #0  SystemClock_Config () at ./Src/main.c:124
main () at ./Src/main.c:97
97    MX_GPIO_Init();
```

Run until next breakpoint:
```GDB
(gdb) continue
Continuing.
```
**Pro Tip**: for most command, you can simply type in the first letter. *e.g.*`n` for `next`.

**Pro Tip**: Press enter to repeat the previous command. Very usefull to quickly step trough a program.

**Pro Tip**: gdb also supports TAB completion. *e.g* `cont` + TAB will result in `continue`.

**Pro Tip**: Use `control` + `c` to stop execution

### Setting a breakpoint:
https://sourceware.org/gdb/onlinedocs/gdb/Breakpoints.html#Breakpoints
http://www.unknownroad.com/rtfm/gdbtut/gdbbreak.html

From there, you can add breakpoints using any of the following methods in the GDB command:

Break on line number and run until breakpoint:
```GDB
(gdb) break main.c:107
Breakpoint 2 at 0x800208e: file ./Src/main.c, line 107.
(gdb) continue
Continuing.

Breakpoint 2, main () at ./Src/main.c:107
107     HAL_GPIO_TogglePin(LD2_GPIO_Port, LD2_Pin);
```

Break on function:
```GDB
(gdb) break SystemClock_Config
Breakpoint 2 at 0x8001fd0: file ./Src/main.c, line 132.
(gdb) continue
Continuing.

Breakpoint 2, SystemClock_Config () at ./Src/main.c:132
132   RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_HSI;
(gdb)
```

List breakpoints:
```GDB
(gdb) info break
Num     Type           Disp Enb Address    What
1       breakpoint     keep y   0x0800207e in main at ./Src/main.c:83
    breakpoint already hit 1 time
2       breakpoint     keep y   0x0800208e in main at ./Src/main.c:107
3       breakpoint     keep y   0x08001fd0 in SystemClock_Config at ./Src/main.c:132
```

Remove a breakpoint:
```GDB
(gdb) delete 2
(gdb) disable 3
(gdb) i b
Num     Type           Disp Enb Address    What
1       breakpoint     keep y   0x0800207e in main at ./Src/main.c:83
    breakpoint already hit 1 time
3       breakpoint     keep n   0x08001fd0 in SystemClock_Config at ./Src/main.c:132
```

### Inspecting and setting variables and memory:
```GDB
(gdb) print uwTick 
$1 = 1206
(gdb) set uwTick=0
(gdb) p uwTick 
$2 = 0
```

```GDB
(gdb) x /32x 0x08000000
0x8000000:  0x20018000  0x080017d9  0x08001829  0x08001829
0x8000010:  0x08001829  0x08001829  0x08001829  0x00000000
0x8000020:  0x00000000  0x00000000  0x00000000  0x08001829
0x8000030:  0x08001829  0x00000000  0x08001829  0x08001785
0x8000040:  0x08001829  0x08001829  0x08001829  0x08001829
0x8000050:  0x08001829  0x08001829  0x08001829  0x08001829
0x8000060:  0x08001829  0x08001829  0x08001829  0x08001829
0x8000070:  0x08001829  0x08001829  0x08001829  0x08001829
```

### Manipulating registers:
https://community.st.com/s/question/0D50X00009XkeAmSAJ/reading-io-register-values-with-command-line-gdb
https://gcc.gnu.org/onlinedocs/gcc-7.3.0/gcc/Debugging-Options.html

Include additional debug information, such as all the macro definitions that can be used to inspect I/O registers:
1. Edit you Makefile to add the `-g3` option to the compiler flags
```
CFLAGS += -g3
```

Level 3 includes extra information, such as all the macro definitions present in the program. Some debuggers support macro expansion when you use -g3.

2. In GDB, type:
```GDB
(gdb) p /x *GPIOA
$4 = {MODER = 0xabfff7ff, OTYPER = 0x0, OSPEEDR = 0xc000000, PUPDR = 0x64000000,
  IDR = 0xc020, ODR = 0x20, BSRR = 0x0, LCKR = 0x0, AFR = {0x0, 0x0}, BRR = 0x0,
  ASCR = 0x0}
(gdb) set GPIOA->ODR ^= 0x20
(gdb) p /x TIM3->CCMR1
```

### Viewing the call stack:
https://sourceware.org/gdb/onlinedocs/gdb/Backtrace.html
```
(gdb) backtrace
#0  HAL_RCC_OscConfig (RCC_OscInitStruct=RCC_OscInitStruct@entry=0x20017fac)
    at ./Drivers/STM32L4xx_HAL_Driver/Src/stm32l4xx_hal_rcc.c:409
#1  0x08001ff4 in SystemClock_Config () at ./Src/main.c:141
#2  0x08002086 in main () at ./Src/main.c:90
```

### Restarting/reset:
```
(gdb) monitor reset init
Unable to match requested speed 500 kHz, using 480 kHz
Unable to match requested speed 500 kHz, using 480 kHz
adapter speed: 480 kHz
target halted due to debug-request, current mode: Thread 
xPSR: 0x01000000 pc: 0x080021d0 msp: 0x20018000
adapter speed: 4000 kHz
(gdb) c
Continuing.

Breakpoint 1, main () at ./Src/main.c:83
83    HAL_Init();
(gdb)
```

### Looking at the code (useful when TUI mode is disabled):
```GDB
(gdb) n
107     HAL_GPIO_TogglePin(LD2_GPIO_Port, LD2_Pin);
(gdb) list
102 
103   /* Infinite loop */
104   /* USER CODE BEGIN WHILE */
105   while (1)
106   {
107     HAL_GPIO_TogglePin(LD2_GPIO_Port, LD2_Pin);
108     HAL_Delay(200);
109 
110   /* USER CODE END WHILE */
111
(gdb) n
108     HAL_Delay(200);
(gdb) tui enable
```


### Getting help:
```GDB
(gdb) help step
Step program until it reaches a different source line.
Usage: step [N]
Argument N means step N times (or till program stops for another reason).
```

### Other/Advanced GDB:

Using [gdb-dashboard]  (do not use TUI mode) to get more complex window configuration (similarly to an IDE):
```
$ arm-none-eabi-gdb-py
```


Editing your Makefile
---------------------
### Adding source files:
Any additional source file (`*.c` or `*.s`) that needs to be compiled should be added to the `C_SOURCES` or `ASM_SOURCES` variables inside the Makefile.
```Makefile
######################################
# source
######################################
# C sources
C_SOURCES = \
./Src/main.c \
./Src/new_file.c \
./Src/stm32l4xx_it.c \
./Src/syscalls.c \
./Src/system_stm32l4xx.c \

# ASM sources
ASM_SOURCES =  \
./startup_stm32l476xx.s
```
**Pro Tip**: I prefixed my paths with `./` to help vim trigger file path completion.

### Adding Includes:

```Makefile
# C includes
C_INCLUDES =  \
-IInc \
-IDrivers/STM32L4xx_HAL_Driver/Inc \
-IDrivers/STM32L4xx_HAL_Driver/Inc/Legacy \
-IDrivers/CMSIS/Device/ST/STM32L4xx/Include \
-IDrivers/CMSIS/Include
```
**Pro Tip**: if the compiler complains about a missing header file for example:
```
./Src/usbd_conf.c:53:10: fatal error: usbd_msc.h: No such file or directory
 #include "usbd_msc.h"
          ^~~~~~~~~~~~
```
use `find . -name "usbd_msc.h"` to get the path and add it to Makefile:
```
$ find . -name "usbd_msc.h"
./Middlewares/ST/STM32_USB_Device_Library/Class/MSC/Inc/usbd_msc.h
```

### Adding Preprocessor defines:
https://gcc.gnu.org/onlinedocs/gcc-7.3.0/gcc/Preprocessor-Options.html
```Makefile
# C defines
C_DEFS =  \
-DUSE_HAL_DRIVER \
-DSTM32L476xx \
-D__DEBUG__
```


### Changing optimization options:
https://gcc.gnu.org/onlinedocs/gcc-7.3.0/gcc/Optimize-Options.html

```Makefile
# optimization
OPT = -Os
```

### Understanding compiler options and flags:
- https://gcc.gnu.org/onlinedocs/gcc/ARM-Options.html
- https://gcc.gnu.org/onlinedocs/gcc/Option-Summary.html#Option-Summary

Serial console:
---------------
On embedded devices, `printf` statements can become very useful for debugging. Retargeting the C `printf` function is generally done is one of three ways:
- through **semihosting** (no extra hardware required but slow)
- through **UART interface** (most support, speed depends on baud rate)
- through **ITM messages over SWO** (fastest but not always supported)

### Using semihosting:
Semihosting is relatively easy to setup but it is one of the slowest methods for printing debug messages.
Enable semihosting (see example `.gdbinit`)

http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.dui0471m/pge1358787046598.html

### Using UART:

**Note**: With GCC, add `syscalls.c`

See [my GitHub gist][uart.c] to retarget printf to UART


Check what USB devices are connected to the serial port:
```Shell
$ ls /dev/tty.usbmodem*
/dev/tty.usbmodem413
```

#### Option 1 - GNU Screen
8 bit, no parity, one stop bit, translate input new line carriage return, newline performs a carriage return, local echo

Optional `-` before SETTING indicates negation.
http://man7.org/linux/man-pages/man1/stty.1.html
```Shell
$ screen /dev/tty.usbmodem413 115200,cs8,-parenb,-cstop,inlcr,onlret,echo
```
**Note**: To exit `screen`, press `control-A` then `control-k` and `y`.

#### Option 2 - Minicom
```Shell
$ brew install minicom
$ minicom -D /dev/tty.usbmodem413 -b 115200
```



### Using ITM messages over SWO:
With OpenOCD, ITM can be configured using the `tpiu` command.
- https://mcuoneclipse.com/2016/10/17/tutorial-using-single-wire-output-swo-with-arm-cortex-m-and-eclipse/
- http://blog.japaric.io/itm/


Other Resources:
----------------
https://developer.arm.com/open-source/gnu-toolchain/gnu-rm
http://www.bravegnu.org/gnu-eprog/
http://blog.japaric.io/quickstart/#hello-world
https://www.gnu.org/software/make/manual/make.html


Issues:
-------
When creating a brand new project (example NUCLEO-L476RG), C source files under `Src/` are defined multiple times.

Debugging tips (Part 2?)
------------------------
Please comment to request topics
- mdw
- flash program
- flash mass_erase
- viewing registers and SFRs
- reg
- mdw
- semihosting
- Linker file


[STM32CubeMX]:http://www.st.com/en/development-tools/stm32cubemx.html
[UM1718]:https://www.st.com/resource/en/user_manual/dm00104712.pdf
[GNU ARM Embedded Toolchain]:https://developer.arm.com/open-source/gnu-toolchain/gnu-rm
[STM32CubeProgrammer]:http://www.st.com/en/development-tools/stm32cubeprog.html
[Homebrew]:https://brew.sh/
[OpenOCD]:http://openocd.org/
[texane/stlink]:https://github.com/texane/stlink
[NUCLEO-L476RG]:http://www.st.com/en/evaluation-tools/nucleo-l476rg.html
[gdb-dashboard]:https://github.com/cyrus-and/gdb-dashboard

[uart.c]:https://gist.github.com/glegrain/ca92f631e578450a933c67ac3497b4df

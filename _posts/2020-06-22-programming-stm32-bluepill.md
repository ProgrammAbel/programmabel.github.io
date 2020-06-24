---
layout:     post
title:      Programming the STM32 Blue Pill
date:       2020-06-22
author:     Abel Binoop
summary:    A quick guide on how to program the STM32 Blue Pill.
categories: microcontrollers
thumbnail:  microchip
tags:
- stm32
- microcontroller
- programming
---

A while back, I got my hands on the [STM32 Blue Pill](https://stm32-base.org/boards/STM32F051C8T6-Blue-Pill). It's a nice little board that looks quite like the Arduino Nano, but with much more power. But I, as a nieve and inexperienced 13-year-old, had no prior experience with microcontrollers at all, so I figured it wouldn't hurt to take a stab at it.

But I quickly realised that most people start out with Arduinos. And because of this, most tutorials explaining how to program this board were constantly trying to make it appealing to existing Arduino users by showing how you can program it with the Arduino IDE. While this isn't exactly a *bad* thing, I found out that I actually highly dislike the Arduino IDE - it's made in Java, theme formatting is weird in Linux and it's sort of geared towards beginners. I'm definitely a beginner in microcontrollers, but not really a beginner in programming. It seemed the only good thing the IDE could do that my trusty Vim couldn't by default was compile for my Blue Pill. Furthermore, the program to flash the compiled binary, *STM32CubeProgrammer*, was incredibly buggy and had it's own set of problems for Linux. Undeterred, I sought to find another way, and that's the purpose of this blog post.

For this guide, all you need is:
- An STM32F103 (Blue Pill)
- ST-Link V2 Debugger (you can find 'unofficial' ones for a little over £2)
- A basic understanding of the C/C++ programming language

I'll be using Linux in this guide, and I expect there will be quite a few platform-specific things, so these instructions may differ for Windows. macOS should be fine, though.

By the end of this guide, you should be able to program the STM32 Blue Pill so that the PC13 LED blinks. Let's go!

Open up your terminal and make a project directory for your code. You'll need to install a few packages to start programming the board. You'll need `git`, `make`, and the GNU ARM Embedded Toolchain (`arm-none-eabi-gcc`). Since on most distros the package in the repositories is outdated, you might want to [download the toolchain straight from ARM](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm/downloads), but as I'm using Arch (btw) this won't be a problem for me. Note that if you *are* on Arch, you will also need to install `arm-none-eabi-newlib`. You'll also need the [ST-Link Tools](https://github.com/stlink-org/stlink#installation) to be able to flash your code onto the board.

Once you've completed all these prerequisites, we can get started! In the project directory, clone the [STM32-base git repos](https://github.com/STM32-base/STM32-base), along with the [STM32-base-F1-template](https://github.com/STM32-base/STM32-base-F1-template) repo:

```
git clone https://github.com/STM32-base/STM32-base.git
git clone https://github.com/STM32-base/STM32-base-STM32Cube.git
git clone https://github.com/STM32-base/STM32-base-F1-template.git
```

This is going to contain the makefiles and basically everything else you need to program the STM32.

Now, link the `STM32-base` and `STM32-base-STM32Cube` directories to the `STM32-base-F1-template` directory:

```
cd STM32-base-F1-template
ln -s ../STM32-base
ln -s ../STM32-base-STM32Cube
```

We need to change the path to the toolchain in the Makefile. Edit `STM32-base/make/common.mk` and edit the `TOOLCHAIN_PATH` variable to the path of your toolchain. For me, it's:

```
...
TOOLCHAIN_PATH ?= /usr/bin/
...
```

This will be different if you downloaded the archive from the ARM website, so set it accordingly.

Now, taking a look at the code for our blinking program:

`STM32-base-F1-template/src/main.c`
```
#include "stm32f1xx.h"

// Quick and dirty delay
static void delay (unsigned int time) {
    for (unsigned int i = 0; i < time; i++)
        for (volatile unsigned int j = 0; j < 2000; j++);
}

int main (void) {
    // Turn on the GPIOC peripheral
    RCC->APB2ENR |= RCC_APB2ENR_IOPCEN;

    // Put pin 13 in general purpose push-pull mode
    GPIOC->CRH &= ~(GPIO_CRH_CNF13);
    // Set the output mode to max. 2MHz
    GPIOC->CRH |= GPIO_CRH_MODE13_1;

    while (1) {
        // Reset the state of pin 13 to output low
        GPIOC->BSRR = GPIO_BSRR_BR13;

        delay(500);

        // Set the state of pin 13 to output high
        GPIOC->BSRR = GPIO_BSRR_BS13;

        delay(500);
    }

    // Return 0 to satisfy compiler
    return 0;
}
```

To flash the program, ensure the ST-Link is connected, and run within the template directory:
```
make
make flash
```

And there you go! You should be able to see the LED on the board flashing at this point. From here, virtually anything is possible with the STM32!

Special thanks go to:
- [Russell Smith](https://www.rasmithuk.org.uk/entry/stm32f103-expansion) for giving this board to my dad, as well as an amazing expansion board for it
- [The STM32-base project](https://stm32-base.org/) for the great work on making an easy-to-use project template to get started

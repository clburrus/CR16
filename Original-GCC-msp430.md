# msp430-elf

The following are instructions for building a GCC cross-compiler for the MSP430.
They are based in part on [Peter Bigot's post to mspgcc-users](https://www.mail-archive.com/mspgcc-users@lists.sourceforge.net/msg12028.html).


## Setup

```
export PREFIX=/usr/local/msp430
export PATH="${PREFIX}/bin:${PATH}"
```

Make sure `$PREFIX` exists, is empty, and is writable by you.


## Software

### binutils

Get the latest [binutils](https://www.gnu.org/software/binutils/) (currently 2.25) and extract it.

```
cd binutils-2.25
./configure --prefix="${PREFIX}" --target=msp430-elf
make
make install
cd ..
```


### gcc (host)

The latest [gcc](https://www.gnu.org/software/gcc/) release as of this writing is 4.9.2, which doesn't check for a hardware multiplier (see [patch](https://gcc.gnu.org/ml/gcc-patches/2014-09/msg02545.html)).
We're cheap (our chip doesn't have a multiplier) and lazy (we don't want to have to say `-mhwmult=none` all the time), so we build the latest gcc from trunk instead.

```
svn checkout svn://gcc.gnu.org/svn/gcc/trunk gcc-trunk

mkdir gcc-build

cd gcc-build
../gcc-trunk/configure --prefix="${PREFIX}" --target=msp430-elf \
  --with-newlib --enable-languages=c,c++
make all-host
make install-host
cd ..
```


### newlib

Get the latest [newlib](https://sourceware.org/newlib/) (currently 2.2.0-1) and extract it.

```
cd newlib-2.2.0-1
./configure --prefix="${PREFIX}" --target=msp430-elf
make
make install
cd ..
```


### gcc (target)

```
cd gcc-build
make all-target
make install-target
cd ..
```


## Support files

TI provides the "support files" necessary for building code for the MSP430: headers and linker scripts.
These are distributed as part of [MSP430-GCC-OPENSOURCE](http://www.ti.com/tool/msp430-gcc-opensource) and can be found on the [download page](http://software-dl.ti.com/msp430/msp430_public_sw/mcu/msp430/MSPGCC/latest/index_FDS.html) (at least for version `3_02_03_00`) as `msp430-gcc-support-files.zip`.

```
cd msp430-gcc-support-files
cp *.h "${PREFIX}/msp430-elf/include"
cp *.ld "${PREFIX}/msp430-elf/lib"
cd ..
```


## Test

Now let's make sure it works with the customary blinkenlights.
We assume that [mspdebug](http://mspdebug.sourceforge.net/) is already installed.
You may need to tweak the details for your specific chip model.

Put the following in `test.c`:

```
#include <msp430.h>

void __attribute__((interrupt(TIMERA1_VECTOR))) blink(void) {
	TACCTL1 &= ~CCIFG;

	P1OUT ^= BIT0 | BIT6;
}

void main(void) {
	WDTCTL = WDTPW | WDTHOLD;

	P1DIR |= BIT0 | BIT6;
	P1OUT ^= BIT0;

	BCSCTL3 |= LFXT1S_2;
	BCSCTL1 |= DIVA_1;

	TACCR0 = 1200;
	TACTL = TASSEL_1 | MC_1;
	TACCTL1 = CCIE | OUTMOD_3;

	__bis_SR_register(LPM3_bits | GIE);
}
```

Build and run it with:

```
msp430-elf-gcc -mmcu=msp430g2211 -specs=nosys.specs -o test.elf test.c
mspdebug rf2500 'prog test.elf'
```


## Extras

### gdb

Get the latest [gdb](https://www.gnu.org/software/gdb/) (currently 7.9) and extract it.

```
cd gdb-7.9
./configure --prefix="${PREFIX}" --target=msp430-elf
make
make install
cd ..
```

In `mspdebug`, run `gdb 1234`.
In `msp430-elf-gdb`, run `target remote :1234`.

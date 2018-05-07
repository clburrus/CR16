# cr16-elf

## Prerequisites

* Ubuntu 18.4
* Install g++   
* Install make  
* Install m4    

```
> sudo apt install g++
> sudo apt install make
> sudo apt install m4
```

```
> mkdir cr16-build
> cd cr16-build
```

Copy or download sources
* binutils-2.29
* gcc-7.3.0
* gmp-6.1.2
* mpc-1.1.0
* mpfr-4.0.1
* newlib-2.5.0.20171222

## Environment Variables

```
export PREFIX=${HOME}/cr16  
export PATH="${PREFIX}/bin:${PATH}"
```
Note : (in this example /home/dev/cr16)



## binutils
```
> cd binutils-2.29/
> ./configure --prefix="${PREFIX}" --target=cr16-elf
> make
> make install
> cd ..
```

## gmp
```
> cd gmp-6.1.2/
> ./configure
> make
> sudo make install
> cd ..
```

## mpfr
```
> cd mpfr-4.0.1/
> ./configure
> make
> make check
> sudo make install
> cd ..
```

## mpc
```
> cd mpc-1.1.0/
> ./configure
> make
> sudo make install
> cd ..
```

## GCC
```
> mkdir gcc-build
> cd gcc-build
> /home/dev/cr16-build/gcc-7.3.0/configure --prefix="${PREFIX}" --target=cr16-elf --with-newlib --enable-languages=c
> make all-host
> make install-host
> cd ..
```

## NewLib
```
> ./configure --prefix="${PREFIX}" --target=cr16-elf
> make
> make install
> cd ..
```

## GCC
```
> cd gcc-build
> make all-target
> make install-target
```

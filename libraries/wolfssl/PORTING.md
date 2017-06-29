# Porting notes 

This port is based on release version 3.11.0

# Unpacking

Run the script in IDE/ARDUINO to setup the code base (wolfssl-arduino.sh).

Populate `src/` tree:

cp *.c -> *.cpp
cp -a wolfssl `src/`

I created a helper script to do these parts.  That script is in the 
`bin/` directory.  Edit variables for correct paths in `mklibwolfssl`
before running.

# Setup

## src/wolfssl/wolfcrypt/settings.h

Define a few things.

My suggestion is use one of the ARM settings.  The
`WOLFSSL_IAR_ARM` correctly does not pull in the sys/socket.h
header file.  `NO_OLD_RNGNAME` avoids a redefinition conflict
with the Particle firmware that uses `RNG`.

`TIME_OVERRIDE` gets around this nasty symbol problem.

`/usr/local/gcc-arm-embedded/bin/../lib/gcc/arm-none-eabi/4.8.4/../../../../arm-none-eabi/lib/armv7-m/libg_s.a(lib_a-gettimeofdayr.o): In function `_gettimeofday_r':
gettimeofdayr.c:(.text._gettimeofday_r+0xe): undefined reference to `_gettimeofday'`

Final defines:

```
#define WOLFSSL_IAR_ARM
#define WOLFSSL_PARTICLE_ARM
#define NO_OLD_RNGNAME
#define TIME_OVERRIDE
```

# Code changes

## example/wolfssl_client

### wolfssl_client.ino

Comment out `#include <Ethernet.h>` and replace `EthernetClient client;`
with `TCPClient client;`

## Base code

### internal.h / internal.cpp

Use `WOLFSSL_PARTICLE_ARM` to mark Particle specific items.

TODO: Need to define XGMTIME()

### src/misc.cpp

Create a symbolic link between src/misc.cpp and
wolfcrypt/src/misc.c

### ssl.h

`src/wolfssl/ssl.h:2328:13: error: 'size_t' does not name a type`

Add "application.h" to ssl.h.  
[REF](https://community.particle.io/t/digole-uart-i2c-spi-display-library/2392/281?u=cermak)

### ssl.cpp

One alloc to cast using given defines.

`rng = (WC_RNG*) XMALLOC(sizeof(WC_RNG), ctx->heap, DYNAMIC_TYPE_RNG);`


# noseglasses' layout

This is my personal layout for the [ErgoDox EZ](https://ergodox-ez.com/) 
and the [Planck](https://olkb.com/planck) keyboard.

*Please note:* This layout uses [Papageno](https://github.com/noseglasses/papageno) to process special key combinations. Papageno is currently not officially integrated with the [QMK firmware](https://github.com/qmk/qmk_firmware/) therefore to build it, you need a patched version of QMK you can find [here](https://github.com/noseglasses/qmk_firmware). The most simple way to try and test the layout is to follow the [build instructions](#how-to-build) below.

## Special Features   

At first glance, my layout is quite ordinary. It uses two layers for normal
characters and special characters and some more layers for other stuff. However,
some of the other layers `(2 - ..)` are currently inactive.

What's special about this keymap is the use of the thumb keys. I want to do as much
work as possible with my thumbs and specially on the four large ErgoDox thumb keys as they are most convenient to reach. 

Therefore, I assign key combinations to them that trigger 

* space,
* backspace,
* del,
* tab,
* shift-tab (untab),
* double tab (e.g. for auto completion),
* enter,
* shift (one-shot) and
* layer toggle to symbol layer (one-shot).

A similar assignment could of course also be achieved by using tap dances. 
But to me it seems more healthy to tap two keys with two 
different thumbs in a row than with the same thumb. This is where [Papageno](https://github.com/noseglasses/papageno), a pattern-matching libray comes in.
It enables the definition of key sequences, chords and clusters. Even combinations
of all of those are possible.
It is possible to use matrix key-positions but also keycodes to define
sequences, chords and clusters.

In this way we can easily handle eight different actions, i.d. keys being send, with just four different keys. And it would be possible to add even more functionality to the same four thumb keys.
And all this without any keys permanently held. Neat?

The following provides a description of the exact assignments between keys pressed and keys send.
Here left/right, inner/outer refers to ErgoDox's four large thumb keys or the four center keys in the plancks bottom row.

| Key Send    | Thumb Keys Involved                                         |
|-------------|-------------------------------------------------------------|
| space       | right outer                                                 |
| backspace   | left outer                                                  |
| delete      | sequence of right inner and left outer                      |
| tab         | sequence of left inner and right outer                      |
| shift-tab   | sequence of right outer and left inner                      |
| double tab  | tripple tap on left inner                                   |
| enter       | left inner and right inner in arbitray order                |
| shift       | left inner (tab for one shot, hold for permanent)           |
| layer toggle| right inner (tab for one shot, hold for permanent)          |

Check out the file `ng_papageno_settings.c` to see how to define all this in a clearly arranged manner. 

## How to build

To build it, you need
1) All requirements that are needed to build classical QMK firmware
2) A current version of Papageno (already integrated as a submodule in the patched version of QMK)
3) The noseglasses' layout keymap files

The build process works as follows for a bash shell on GNU/Linux.

```sh
# Define for which keyboard to build (currently supported are ergodox_ez and planck)
#
KEYBOARD=ergodox_ez

# Change to your favorite build directory
# ...

# Clone QMK
#
git clone https://github.com/noseglasses/qmk_firmware.git
cd qmk_firmware
git submodule update --init lib/papageno

# Build Papageno for atmega32u4
#
cd lib/papageno
git pull origin master
mkdir -p build/atmega32u4
cd build/atmega32u4
cmake -DCMAKE_TOOLCHAIN_FILE=$PWD/../../cmake/toolchains/Toolchain-avr-gcc.cmake \
      -DPAPAGENO_PLATFORM=atmega32u4 \
      -DPAPAGENO_ADDITIONAL_INCLUDE_PATHS=$PWD/../../../../tmk_core/common \
      ../..
make

cd ../../../..

# Clone noseglasses' layout
#
git clone https://github.com/noseglasses/noseglasses_qmk_layout.git keyboards/${KEYBOARD}/keymaps/noseglasses

# Build the layout
#
make keyboard=${KEYBOARD} keymap=noseglasses
```

## Compressed builds

Compression is a two stage process. It involves the generation of an intermediate hex-file and a simulated execution of the latter. The intermediate file is executed in an avr-emulator on the build host which generates a compressed version of the specified Papageno data structures. The simulated/emulated execution is necessary as compressed data structures are highly architecture dependent. Thus the compression process can not run e.g. on a x86 architecture. Another reason for the use of an emulator is that the compression mechanism requires more memory than the atmega32u4 (2kb) provides that is used by many programmable keyboards. To cope with this memory restriction, we use a binary compatible atmega1280 that comes with significantly more memory (8kb).

During the simulation the generated data structures are exported as C-code which is included during the second compilation stage which finally generates the actual keyboard-firmware. 

It is obvious that [simavr](https://github.com/buserror/simavr/), the avr emulator must be installed or build and installed on the firmware build host.

The two-stage compilation process above can then be triggered using the script `compress_keymap.sh` that comes with the keymap and that must be called from the QMK root directory.

```sh
# Change to QMK's root directory
#
cd <whatever your qmk root directory is>

# Call build automation script with full relative path
#
./keyboards/<your keyboard, e.g. ergodox_ez>/keymap/noseglasses/compress_keymap.sh
```

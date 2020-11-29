# Snark Barker MCA - A Sound Blaster Compatible for Micro Channel Computers!

Do you desperately want a sound card for playing games on your Micro Channel-based IBM PS/2 computer--a system designed for serious business purposes? Do you not want to pay the ludicrous auction prices for the rare Sound Blaster MCV cards? Well, despair no longer: the Snark Barker MCA has arrived!

![Snark Barker MCA Photo](https://github.com/schlae/snark-barker-mca/blob/master/images/SnarkBarkerMCA.png)

The Snark Barker MCA is a Sound Blaster-compatible sound card designed specifically for computers that use the Micro Channel bus. It supports Ad Lib synthesis, digital sound playback and recording, a standard PC joystick, and SB MIDI.

All of the components are readily available. In the bill of materials, Mouser part numbers are listed where available. The Yamaha synthesis chips may be obtained from a variety of sources in China, but they are not guaranteed to work, so I recommend getting 1 or 2 extras just in case. The same goes for the NE558.

[Schematic](https://github.com/schlae/snark-barker-mca/blob/master/SnarkBarkerMCA.pdf)

[Bill of Materials](https://github.com/schlae/snark-barker-mca/blob/master/SnarkBarkerMCA.csv)

Please note that the 0.1" header pins are *not* listed on the BOM. They are standard breakaway headers. Headers J2, J4, J5, and J6 are optional and only useful for people who want to connect a logic analyzer to the bus for CPLD debugging.

The CMS chips U14 and U15 should not be installed since CMS functionality is not implemented at this time. Other devices listed as "DNI" are to be left empty with no component soldered in place.

Logic chips are all specified as HCT, but ALS may also be used. Do not use LS devices, particularly for U2, since Micro Channel is a much faster bus than ISA and LS logic is not quite fast enough to meet the timing margins.

[Fab Files](https://github.com/schlae/snark-barker-mca/blob/master/fab/SnarkBarkerMCA-Rev2.zip)

## Fabrication Notes
This is a 4-layer board. The order of the two inner layers does not matter, but typically it goes F\_Cu, In1\_Cu, In2\_Cu, and B\_Cu. The board dimensions are 11.5 inches (292.1mm) by 3.475 inches (88.265mm).

When ordering the board, you may want to specify a card edge bevel of 20 degrees (details of the bevel are included in the Dwgs.User layer). This is optional but makes the board much easier to insert in the bus slot. If you want to save money, you could order without the bevel and add a bevel yourself using a file.

Ideally you should use selective gold plating (hard gold) for the edge fingers, but this can get expensive for small orders. ENIG will work but the gold will rub off fairly quickly.

For the soldermask color, pick whatever you want!

## Assembly Notes
I recommend soldering the CPLD (U31), the voltage regulator (U32), and the associated surface mount capacitors before you solder in the remaining components. You can do this by hand with a pencil iron or you can use a reflow process, but be aware that the board is too long to fit in the small hobbyist reflow ovens.

![Photo showing pin 1 alignment of CPLD](https://github.com/schlae/snark-barker-mca/blob/master/images/OrientCPLD.png)

***Be sure the CPLD is soldered in the right way around! Compare it with my photo of the card to be sure you have it right!***

Inspect the CPLD's solder joints carefully with a microscope to make sure you don't have any bad connections or solder bridges. This can save you a lot of grief later on.

Install sockets for the Yamaha chips U9 and U10 in case you get one that doesn't work and need to replace it.

## Bracket
MCA brackets are no longer available. You have a few options:
- Remove the brackets from an existing card, like a token ring card or something common
- 3D print the bracket
- Use the card without a bracket at all

If you want to print your own brackets, here are STL files of the front panel bracket and the rear bracket.

[Front bracket](https://github.com/schlae/snark-barker-mca/blob/master/mech/SnarkBarkerMCABracket3D.STL)

[Rear bracket](https://github.com/schlae/snark-barker-mca/blob/master/mech/MCARearBracket.STL)

For best results, use high quality filament and a layer height of 0.16mm or better. If you print using blue filament, the color will match the original IBM brackets.

The front bracket should be oriented so that the front face of the bracket is facing towards the bed of the printer. The rear bracket should be oriented so its large flat spot (parallel to the card) faces the bed.

## The Firmware
There are two ways to get a programmed 80C51 chip for the Snark Barker MCA. One is to purchase a SB 2.0 DSP chip from China and put it in a 44-PLCC to 40-DIP adapter. This works fine and provides the largest feature set.

Another option is to buy a blank Atmel 89S51 (as listed in the BOM) and program it with [this HEX file](https://github.com/schlae/snark-barker/blob/master/firmware/sb.hex).

## CPLD Notes
The Snark Barker MCA uses a CPLD (Complex Programmable Logic Device) to interface between the Micro Channel bus and the rest of the card's digital logic. This CPLD implements port IO, interrupts, DMA, and the card setup and POS (Programmable Option Select) systems.

The CPLD must be programmed with the bitstream before you can use the card.
[You can find the bitstream here.](https://github.com/schlae/snark-barker-mca/blob/master/verilog/mcsb.jed)

I've had good luck using the Linux command line program 'xc3sprog' with a FTDI FT2232H Mini Module [datasheet here](https://www.ftdichip.com/Support/Documents/DataSheets/Modules/DS_FT2232H_Mini_Module.pdf).

Wiring for the FT2232H mini module:

| Point 1    | Point 2          | Description           |
| ---------- | ---------------- | --------------------- |
| CN2 pin 1  | CN2 pin 11       | V3V3 to VIO strap     |
| CN3 pin 1  | CN3 pin 3        | VBUS to VCC strap     |
| CN2 pin 3  | CN3 pin 12       | V3V3 to VIO strap (2) |
| CN2 pin 5  | JTAG cable pin 6 | V3V3 to Snark Barker (optional) |
| CN2 pin 2  | JTAG cable pin 5 | GND                   |
| CN2 pin 7  | JTAG cable pin 4 | AD0, aka TCK          |
| CN2 pin 9  | JTAG cable pin 3 | AD2, aka TDO          |
| CN2 pin 10 | JTAG cable pin 2 | AD1, aka TDI          |
| CN2 pin 12 | JTAG cable pin 1 | AD3, aka TMS          |

You need to provide external power to the Snark Barker through the 3.3V wire on the programming header. The optional wire does this, or you can use a bench supply.

I obtained xc3sprog [here](https://github.com/matrix-io/xc3sprog). You can compile and run this program on a Raspberry Pi to use the Pi GPIO lines instead of the FTDI mini module. In my case, I chose to patch it (TBD) to remove references to the Pi GPIO library and ran it on a regular Linux PC.

Using the FTDI mini module, programming is quite simple. Connect the cable to
the assembled Snark Barker board, then run the following command. You can do this with just the CPLD, programming header, and surface mount capacitors--there's no need to have any of the other components soldered in place for this step.

`xc3sprog -c ftdi -v mcadlib.jed`

If you run into issues with the cable not being detected, check your udev rules.
In theory you could run xc3sprog as root, but that's bad practice. ;)

### Building the CPLD Project
If you're feeling particularly masochistic, you can try building the CPLD project using the Xilinx ISE Webpack tools. Because the ISE Webpack only supports Ubuntu 14.7, it's simplest to run the program from a Docker container. I use [docker-xilinx](https://github.com/jimmo/docker-xilinx).

### How the CPLD Works
This isn't meant to be an exhaustive tutorial on how Micro Channel works ([you can find a tutorial here](https://github.com/schlae/mca-tutorial)). Functions provided by the CPLD include:
- Card setup and POS
- Address decoding logic
- Bus logic interface to the YM3812
- Wait state logic for the YM3812 interface
- Interrupt logic
- DMA state machine including arbitration logic

Each card implements several 8-bit registers (POS registers). Two of these registers are fixed and implement the 16-bit card ID value, which is how the BIOS and the reference disk software identify the card. The remaining POS registers can be used for any configuration-related function and are designed to replace configuration jumpers. The meanings of each register bit are up to the card designer, with the exception of a reserved bit that is used by the BIOS to enable or disable a card; this way, the BIOS can resolve conflicts automatically. Typically POS register bits control IO and memory address assignments, IRQ lines, and so forth.

The CPLD implements the two required ID registers as well as two additional POS registers. The first contains only a single bit -- the card enable signal. The second register controls the IO port setting, the IRQ and DMA channel settings, and the joystick port enable bit.

When you add a new card to a MCA bus system, the BIOS detects the new card and requires you to run the reference disk configuration program. This program searches for an ADF file associated with the new card. The ADF file is human-readable text that tells the BIOS which I/O address, IRQ, etc match up with each POS register bit. The reference disk program uses this information to find a configuration for all the cards in the computer so that none of them conflict with each other. When it's done, it writes the raw POS register values along with the card ID values to the battery-backed SRAM on the motherboard.

## Compatibility
The Snark Barker MCA has been tested on some models of PS/2. More to be added as
people provide test reports.

| Computer           | Model    | CPU          | Compatible ? |
| ------------------ | -------- | ------------ | ------------ |
| IBM PS/2 Model 50Z | 8550-031 | 80286-10     | Yes          |
| IBM PS/2 Model 95  | 8595     | 486DX2-50    | Yes          |
| NCR System 3400    | 3433     | 486DX2-66    | Yes          |

Note: If you are using a PS/2 Model 80 machine with the type 1 planar (16MHz 386) and you have installed a MCA memory card, it is quite possible that the Snark Barker MCA will behave erratically in this machine. The Plaid Bib CPLD, the ancestor of this card, has issues in this model PS/2 and the root cause has not been identified.

If you build the card and try it out, please let me know how it went. Be sure to give me the full part number of your computer along with any installed options (3rd-party planar, memory cards, etc).

## Installation
You will need the ADF (adapter description file) in order to set up the Snark Barker MCA on your Micro Channel computer.

[Snark Barker MCA ADF](https://github.com/schlae/snark-barker-mca/blob/master/@5085.ADF)

Place the file on a 3 1/2" floppy disk. After you install the card and boot
the computer, it will detect that a new card has been installed and prompt
you to insert the reference disk. Do this and follow the prompts. When the
setup utility asks you to insert the option disk, use the one that contains
the ADF file. After the setup utility configures the card, it will prompt
you to reboot the machine.

Plug your line-level audio amplifier input into the card's 3mm audio jack nearest the joystick port. The other audio jack above that one is for the microphone input.

Once that's finished, try out the Snark Barker MCA with your favorite game!

## Troubleshooting
Hoo boy, this one is a doozie. Someone once told me
> At Creative Labs tech support we would always get a sinking feeling whenever someone said they had a SoundBlaster MCA. Guaranteed at least a 45 minute call. Weekly average call time blown straight to hell.

Keep in mind that Micro Channel systems can be difficult to troubleshoot due to the large number of possible configurations and the varied implementations of the bus. Not all games will even work on some PS/2 systems even without a sound card!

Here are a few things to try out first before you starting really digging in:
- Is the CPLD soldered in correctly, with no solder bridges? Did you program it?
- Did you program and install the 8051 (DSP)?
- If the BIOS detects and installs the card, but you have issues with the audio, try removing all the expansion cards except for the Snark Barker
- Try running [SBDIAG](https://github.com/schlae/snark-barker/tree/master/sbdiag)

### Symptom-by-Symptom

**Computer will not boot at all, Computer boots but does not detect the card**

Suspect the CPLD. Check for bad solder joints. Is pin 1 oriented correctly? Compare it with the photo.

Did you use something other than a 74HCT245 or 74ALS245 for U2? Slower logic families can "crash the bus" and prevent the system from booting.

**Joystick doesn't function**

Is the joystick enabled? Run the setup program and navigate to "Change Configuration" and see if the joystick is on or off. The setup program may have turned it off due to a conflict with another card (Ethernet cards often like to use the joystick port location).

**No digitized sound. Digitized sounds play but are cut off**

It is possible that you have an IRQ conflict. Micro Channel allows for IRQ sharing and not all games support this correctly. Try changing the IRQ using the setup program's "Change Configuration" menu and see if that helps.

Since games implement Sound Blaster support in different ways, some programs may work and others may not!

### Troubleshooting for Engineers
If you have an HP logic analyzer, like a 1670 or 16700 series, you can connect three cables to the Snark Barker MCA using standard HP 01650-63203 termination adapters. The pin assignments are as follows:

| Pin |  J2       |  J4  |  J5      |
|-----|-----------|------|----------|
| CLK | CMD       | ADL  | PREEMPT  |
| D0  | BUSRESET  | A0   | DS16RTN  |
| D1  | M\_IO     | A1   | CHRDYRTN |
| D2  | S0\_WRITE | A2   | MADE24   |
| D3  | S1\_READ  | A3   | CD\_DS16 |
| D4  | CD\_SETUP | A4   | TC       |
| D5  | REFRESH   | A5   | ARB/GNT  |
| D6  | CD\_SFDBK | A6   | CHCK     |
| D7  | CD\_CHRDY | A7   | IRQ7     |
| D8  | DB0       | A8   | IRQ5     |
| D9  | DB1       | A9   | IRQ3     |
| D10 | DB2       | A10  | IRQ2     |
| D11 | DB3       | A11  | ARB3     |
| D12 | DB4       | A12  | ARB2     |
| D13 | DB5       | A13  | ARB1     |
| D14 | DB6       | A14  | ARB0     |
| D15 | DB7       | A15  | BURST    |

An additional debug header, J6, brings out a few additional signals:

| Pin | J6        |
|-----|-----------|
| 1   | DBG1 (CPLD unassigned) |
| 2   | CLK\_14M318 (MCA bus clock) |
| 3   | DBG3 (CPLD unassigned) |
| 4   | DBG4 (CPLD unassigned) |
| 5   | DBG5 (CPLD unassigned) |
| 6   | DBG6 (CPLD unassigned) |
| 7   | DBG7 (CPLD unassigned) |
| 8   | SBHE (MCA bus signal) |
| 9   | GND |

## License
This work is licensed under a Creative Commons Attribution-ShareAlike 4.0
International License. See [https://creativecommons.org/licenses/by-sa/4.0/](https://creativecommons.org/licenses/by-sa/4.0/).


# TODO
* [ ] Hardware
    * [ ] find out difference between master and rev3-wip branch
    * [x] find out if PCBs differ or only schematic -> build rev3-wip (you probably can't build the master branch because 
        main PCB:\
        U7 + Beschaltung rundherum (MCP1661) wurdeauf TLV61046A getauscht 812V für OLED)\
        U5 PAM2301 was changed to TPS62291, new power net (+3.3V_SW) added (not sure where it goes)\
        R26, R27 changed from 10R to 22R, R26 is DNP now
        R25 changed to 22R\
        LED PCBs:\
            no relevant changed noticed\
        SWD PCB:\
            schematic is the same, some changes to the board file, could not find any tough
    [ ] check which LiIon-cell to be used with corresponding AP9211

* [ ] BOM
    * [x] write a complete BOM with all parts
    * [x] find other missing parts (e.g. OLED, Liion-cell)
     * [ ] is the Retimer-IC needed or can it be omitted? (pricey part)


* [x] Mechanical
    * [x] add STL model
    * [ ] add nuts and screws to BOM

* [ ] Software
    [ ] Overview about software architecture and folders
    [ ] Build instructions
    [x] flashing instructions
    [ ] find out if any bugs exist or the software can be used as-is

* [ ] Build cost overview


# Build cost estimate


| Item | Cost | Note |
|---|---|---|
| 3D printed enclosure |  | 3 parts (top, bottom, shim) |
| Screws & nuts |  |  |
| PCB |  | JLCPCB, Main, LED and SWD PCB, all 2 Layer |
| Components | 33€ | Estimation from Mouser (without retimer, A9211) |
| Display | 4.30€ | Sourced from Aliexpress. Must support SPI and have matching connector! Try to get one without PCB (bare FFC) |
| Li-Ion cell | 6€ |  |
| SFP Transceiver |  | probably 5-10€ if you need to buy one |
| SUM |  |  |



# master branch vs rev3-wip branch
Upon start it was unknown if there is a difference between the branches. 
Which version should be assembed. 
And if there are any bugs in one of the versions.

Schematics were manually checked, layout was checked afterwards

**Main PCB**
changes are marked as master -> rev3-wip
* MCU sheet: no changes fround

* Power sheet: U7 MCP1661 changed to TLV61046A (D2, R20, R23 removed, C22 changed from 10uF to 4.7uF)
    Not really sure why this was changed, MCP1661 is available.
    TLV61046A has a potentially better fitting input voltage range (1.8-5.5V), MCP1661 has 2.4-5.5V
    (which might still work, not sure how it behaves when battery charge level is rather low)

* Power sheet: U5 PAM2301 chanted to TPS62291
    Another net (+3.3V_SW) was added, seems to go nowhere
    Reason might be that PAM2301CAAB330 is EOL, not available on normal distributors (and export restricted on Mouser)
    
* Retimer sheet: R25, R26, R27 changed from 10R to 22R, R26 is DNP now


master branch: Gerber ZIP contain PCB with "rev2" in top silkscreen
rev3-wip branch: Gerber ZIP contain PCB with "rev3" in top silkscreen

**LED PCB**
* board.json was modified, but could not find any visual differences
* some changes to the schematic, seems only to be metadata

**SWD PCB**
* boad.json was modified, but could not find any visual differences
* top_block.json was modified, but this only includes metadata

**Gerber.zip**
rev2->rev3-wip (probably needs to be checked)

**Software**
* There are some software changes. rev3-wip seems to have some fixes included.

**TLDR: seems the rev3-wip should be assembled and flashed with the rev3-wip software**


# 3D Prints & Mechanical

printed at 100.6% size to accomodate the PCB and Display better

Mechanical:\
m2 nuts\
m2 countersunk screw\
-> don't use screws too long or they will push into the Li-Ion cell!

# swd debug interface
The STM32 must be programmed over SWD as USB-DFU is not accessible.

Connector on main PCB:\
FH33-6S-0.5SH(10)\
6 pin, FFC, 0.5mm pitch, 0.3mm thickness

main pcb pinout:\
1 SWDCLK\
2 SWDIO\
3 NRST\
4 GND\
5 SWO\
6 +3.3V\

swd pcb pinout:\
1 VDD\
2 TRACESWO\
3 GND\
4 NRST\
5 SWDIO\
6 SWDCLK\

-> needs a FFC with contacts on the same side (not opposite side) as the connectors are already crossed out\
e.g. GCT 05-06-A-0101-A-4-06-4-T



# Hardware topics

## AP9211
I'm not entirely sure why this IC is present and if it is actually used. 
It could serve as redundant protection to the Battery management IC (BQ...).

Multiple versions are available with different protection parameters.
R12 can be used to bridge this IC.

auto-wakeup might make sense to reduce power consumption
Depending on Model, OVP triggers between 4.2V and 4.5V. This should be selected matching to a given Liion cell

## Liion 14500 cell options

https://www.reichelt.at/at/de/shop/produkt/industriezelle_li-ion_14500_3_6_v_800_mah_button_top-253362
2.75-4.2V
https://www.reichelt.at/at/de/shop/produkt/industriezelle_li-ion_14500_3_7_v_750_mah_button_top-232284
3-4.2V
https://www.akkushop-austria.at/at/inr-14500-mit-950mah-36v-bis-37v-li-ion-zelle-ungeschuetzt-485x141mm
2.75-4.2V
https://www.akkushop-austria.at/at/efest-imr-14500-v2-akku-mit-700mah-37v-li-ion-akku-pluspol-erhoeht-abmessungen-ca.-505x142mm-beachten
2.5-4.2V


some cells had a 4.2v +0.05V spec
max 4.25V

BQ24040DSQT is specified with 4.16-4.12 (4.2 nominal) output voltage for charging

R12 bridges the functinality of AP9221???
whatt's the purpose, overcharge, overdischarge, both?
is if bridged in the final product?



## Display
1.3" OLED\
128x64px\
12V used for backlight\
Controlled via SPI (not I2C, most only offer SPI)\
Be areful, the majority has the wrong FFC interface (is too wide)

Following might be compatible:
https://de.aliexpress.com/item/1005003801387081.html (30 pin variant)\
But it seems the image is shifted 2px to the left. Might need a software fix.


# Software 

## STM32 DFU Bootloader
Can the hubble bootloader be programmed via STM32 bootloader?
BOOT0 -> pin 44, PH3

According to https://www.st.com/resource/en/application_note/an2606-stm32-microcontroller-system-memory-boot-mode-stmicroelectronics.pdf
BOOT0 pin needs to be pulled high to activate the bootloader. 
BOOT0 pin is hardwired to GND on the PCB. 

-> can't be used (in uC factory default state, means USB-DFU can also not be used), thus the device has to be progammed via the debug header / SWD

## compile & flash

1. compile bootloader
2. compile firmware image
3. generate signature(checksum)
4. flash bootloader (needs to be done via SWD, see STM32 DFU Bootloader note above)
5. flash firmware (app, how to trigger bootloader?) 


It seems openocd might have been used for flashing (references to localhost port 4444)
The programming adresses are included in the makefiles and are different for dfu bootloader and app.


I'm using a STLink from an STM32 nucleo for programming
Connecg GND, SWDIO, SWDCLK, NRST supply hubble with power (e.g. 3.6V from a lab power supply to the battery terminals)

Check if the target is properly detected
        st-info --probe
This should should show: in the last line: "dev-type:   STM32L41x_L42x"


I've build the software on a debian VM and attached the binaries in the bin folder.

        $ python3 ./fw/target/sign.py ./fw/target/hubble.bin ./fw/target/hubble_signed.bin
        0x2dfafb1e
        st-flash write ./fw/dfu/build/hubble-dfu.bin 0x8000000
        st-flash write ./fw/target/build/hubble_signed.bin 0x8005000

Flashing the unsigned app will result in the hubble-dfu bootloader complaining.\
Flashing the app alone (without hubble-dfu) will result in the app directly jumping into the app (update only possible via debug connector)







# Ultra96-V2_MIPI_Board
This is a project for a camera board & Pmod expansion board for FPGA board Ultra96-V2.

![20230414_1681473993591IMG_4198 1](https://user-images.githubusercontent.com/25518367/232450995-39b33f66-ded9-4b85-ba6f-3fc20dd512af.jpg)

## Background

Ultra96-V2 is a useful board with MPSoC. However, unlike Zybo, it does not have Pmod and so on and lacks expandability. For my own use, I wanted to connect a MIPI CSI-2 like the Raspi camera, but unfortunately, no IO is available. This time, I will design a board that produces CSI-2 and extra IO.

Please note that this board was developed with reference to [Ryuzi's ultra96_multi_io](https://github.com/ryuz/ultra96v2_multi_io). We thank him here.

## Goal

Ultra96 has differential signal system and other GPIOs from HS Header and LS Header respectively. MIPI seems to come from HS.

We aim to design a board with the following performance.

- CSI-2 4Lane
- CSI-2 2Lane
- Power button and reset button must be out
- PL system LED 4ch
- PL system button 2ch
- PS LED 2ch
- PS button 2ch
- MIO SPI 2ch
- To externalize the rest as Pmod.

As a back-up goal, I am going to use [LCSC](https://www.lcsc.com/). LCSC is a Chinese version of Digikey, with cheap parts, and [JLCPCB](https://jlcpcb.com/), a board manufacturer I often use, is also compatible with LCSC. JLCPCB] (), a board manufacturer I often use, is also supported. I have never used it for some reason, so I will use it as the basis for parts selection this time.

I have written a separate article about this, so please take a look at it.



[https://fumimaker.net/entry/2023/03/03/211528]



## Design

Design using [Eagle](https://www.autodesk.co.jp/products/eagle/overview). I really wanted to learn Kicad, but I found myself starting to build with Eagle out of habit. Next time I will **absolutely** do it with [Kicad](https://www.kicad.org/). (I don't think Eagle is easy to use, so if you are going to do it now, Kicad is the way to go if you have no reason to do it.)

### Power Supply

The power supply is 5V and 1.8V from LS. 3.3V is not available, so we will make it with an appropriate LDO. You should [probably use ADP3338 or something like that](http://nemuisan.blog.bai.ne.jp/?eid=220875), but it is super expensive, so I'll use [any 3 letters] 1117 this time. I really prefer ADP3338 or LT1117, which has higher transient response performance. (For the second time) It's less noisy. It doesn't break. It's good. Expensive, though.


[http://nemuisan.blog.bai.ne.jp/?eid=220875]



<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">[any 3 letters]1117 is too much</p>- fumi (@fumi_maker) <a href="[https://twitter. com/fumi_maker/status/1625060053668745218?ref_src=twsrc^tfw](https://twitter.com/fumi_maker/status/1625060053668745218?ref_src= twsrc%5Etfw)">February 13, 2023</a></blockquote> <script async src="[https://platform.twitter.com/widgets.js](https://platform.twitter. com/widgets.js)" charset="utf-8"></script>


### LS Header

The PL pin coming out of the LS header is Bank26 and is 1.8V. On the other hand, Raspi cameras and other devices communicate with 3.3V GPIOs via I2C. Therefore, it is necessary to convert the level from 1.8V to 3.3V. Also, when using Pmod, 3.3V is easier to handle than 1.8V, so it is converted to 3.3V.
https://www.avnet.com/wps/wcm/connect/onesite/b85b9556-0b2a-42b3-ad6a-8dcf3eac1ff9/Ultra96-V2-HW-User-Guide-v1_3.pdf?MOD=AJPERES&%20CACHEID=ROOTWORKSPACE.Z18_NA5A1I41L0ICD0ABNDMDDG0000-b85b9556-0b2a-42b3-ad6a-8dcf3eac1ff9-nDNP5R3

<img width="730" alt="20230414_1681473980584Untitled" src="https://user-images.githubusercontent.com/25518367/232451812-53f356e1-24f2-4873-be11-41d3eb7b54e8.png">


### HS Header

The high speed signal coming out of the HS header is 1.2V for Bank65. If you want to use this, for example if you want to use DSI as a normal IO, this also needs to be level converted. However, the HS and LS headers of the Ultra96 do not output 1.2V. So, if you want to do it, you will need to make and convert 1.2V by yourself.

The FET for LED drive is selected to be able to switch even at 1.2V.
<img width="780" alt="20230414_1681473974839Untitled 1" src="https://user-images.githubusercontent.com/25518367/232451927-08ebd81b-4972-4159-842a-4b1220a28d37.png">


### Level conversion IC

Texas Instruments [TXS0108EPWR](https://www.lcsc.com/product-detail/_Texas-Instruments-_C17206.html) is selected as a level conversion IC. 1.2V is supported, It supports open-drain and push-pull. I chose this one because it is also used for I2C conversion. I have all of them in this package.

I really wanted to use a smaller package, but the price was about 6 times higher, so I compromised with a larger one.

### HS connector

I use [this](https://jlcpcb.com/partdetail/STWXE-BA41_60CT_1NHB/C508686). I don't know what this one is called, but is it a mezzanine connector? This connector is probably interchangeable, but from the datasheet it looks fine, so I'll use this one. Note that I misread the datasheet for the LS pin headers and got the height wrong (conclusion). If you design like I did, use BA41-60CT-1-NHB. I used BA41-60BT-1-NHB and was 2mm short in height.

[BA41-60CT-1-NHB | STWXE | Mezzanine Connectors (Board to Board) | JLCPCB](https://jlcpcb.com/partdetail/STWXE-BA41_60CT_1NHB/C508686)

### LS Connectors

This is a surface-mount type L-shaped double-row 2mm pin header. It is quite a strange one, and I could not find it at all when I looked for it. I managed to find it at LCSC and use [this](https://www.lcsc.com/product-detail/_XKB-Connectivity-_C917615.html).

### LED related

Some LEDs are driven by 1.2V, so I selected FET onsemi [NTZD3154NT1G](https://www.lcsc.com/product-detail/_onsemi-_C145189.html) which can be driven by 1.2V.

### SW related

It is an ordinary PSW. 200 pieces or so were very cheap at LCSC, so I bought them. It seems to be a proper one from Alps Electric, so it is quite reasonable. (I've used it and it looks like a proper one...)

Also, a DIPSW can be attached, but I forgot to buy it, so it is not mounted.

 Note: The DSI pin is assumed to have an internal pull-up.

# Development

<img width="1301" alt="20230414_1681473965693Untitled 2" src="https://user-images.githubusercontent.com/25518367/232452038-ba6597f8-99a6-4201-a74e-a33d91bb2167.png">


Eagle's feature is that it is built into Fusion, so it can handle 3D models seamlessly. (In addition, I haven't been able to use it properly because I haven't properly registered 3D data in the library.)

Here it is.

## Schematic.

[https://github.com/fumimaker/Ultra96-V2_MIPI_Board](https://github.com/fumimaker/Ultra96-V2_MIPI_Board)

Schematics and designs are available [here](https://github.com/fumimaker/Ultra96-V2_MIPI_Board).

![20230414_1681473934050ultra96_mipi_v69](https://user-images.githubusercontent.com/25518367/232452348-172633ed-5339-4e43-9bf3-6814af99d8a6.jpg)

![20230414_1681473925876ultra96_mipi_v130](https://user-images.githubusercontent.com/25518367/232452430-fe6ca34c-15a4-47f2-bc0b-c24cb1c9d331.jpg)


There are two MIPI CSI-2s, CSI0 is 4 Lanes and CSI1 is 2 Lanes. If you want to use a common Raspi Camera, you will use CSI1, and CSI0 was made in case you want to use a 4Lane camera in the future to make me tueeeeeeeeee. This is important because everyone in the world has gone to the Raspi camera standard and there is nothing out there with 4Lane's 22 pin 0.5mm pitch connector. I added the 4Lane connector because I thought it would be necessary in the future if I wanted higher resolution camera images.

I noticed later that the level conversion IC (U3) is slightly out of alignment, which is weird. I will fix it next time. Well, Ultra96 is no longer available, so I'm not sure if there will be a next time...

## BOM

BOM. (Pretty messy, sorry.)

https://github.com/fumimaker/Ultra96-V2_MIPI_Board/blob/main/Ultra96-V2_MIPI_CSI-2_Board_BOM.csv

https://docs.google.com/spreadsheets/d/e/2PACX-1vRyfV2nkdrRBBEGrKsxRo6SP4x0MH9KUVkbMfD3b8DZblvdecDmEF19Au3DwG-mPY6oq2ViDohp0APl/pubhtml?gid=1667383814&single=true

# Ordering

I am ordering a board from [JLCPCB](https://jlcpcb.com/). They have always been good to me and I recommend them for good quality. I'd like to try PCBA one of these days.


https://jlcpcb.com/



This setup is 1.6mm thick, EING (gold flats) finish, and black.

# Received.

The board arrived. The usual blue box.

![20230414_1681473914107IMG_4197](https://user-images.githubusercontent.com/25518367/232452542-9bcc302a-fb38-4fd1-a527-3b74d8f98054.jpg)


I have a lot of these at home so I wrote my name on them to make it easier to find them.

They are very clean like this. I knew the black matte and EING would be...awesome! Super cool and highly recommended.

The dachs is still sitting on it and it is good.

![20230414_1681473902637IMG_4198 1](https://user-images.githubusercontent.com/25518367/232452622-5a7faae1-35c9-4b4a-96cd-d33bfe9910f1.jpg)

![20230414_1681473886720IMG_4199](https://user-images.githubusercontent.com/25518367/232452656-c71c645c-fccf-4403-b97c-1cc777f491f5.jpg)



## Aside.

By the way, I wonder if they make Ultra96 anymore.... It was nice and small.... The successor looks like KV260, but I think it's a bit big. I would like MPSoC Zybo if possible.... I wonder if someone could make one for me (other power).

Next, I will make an expansion board for the KV260 (or rather a carrier board for the SOM K26). (I hope) Some people want to use it that way.

# Next time

I'll try to implement it and make it actually work.


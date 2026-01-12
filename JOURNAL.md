---
title: FPGA Accelerator
author: Kai Pereira
description: " A low-cost, PCIe FPGA accelerator M.2 M-Key form factor devboard for hobbyists"
created_at: 2025-12-16
---
## The idea - 2 Hours

I've worked on a ton of different boards ranging from keyboards, to motherboards to hackathon badges to even carriers, but now I want to work on something next level.

I've always been fascinated by hybrid computing in a consumer setting and building complicated stuff, so with that in mind, I've come up with my next idea.

FPGA's are becoming much more mainstream for parallelization and computing so I want to explore how hobbyists can leverage this technology in an affordable and cool way.

The thing about FPGA's, is while they're really useful, it's hard to make practical applications of them as a hobbyist, because it's not like you're going to innovate the next big chip or something, but that's where I want to leverage the fact that FPGA's aren't just useful for mainstream chips, but also more niche applications like very specific hardware acceleration, that consumers can actually make cool use of and practice.

![Pasted image 20251216201028.png](images/Pasted%20image%2020251216201028.png)

So the idea is simple, I want a low-cost way, hobbyists can make practical applications of FPGA's in a bunch of different settings.

To do this, I want to provide an FPGA devboard of sorts, that can interface with your computer hardware so you can practice on real-world applications in more niche settings. But I want to go a step further and also make it function as a standard devboard in case you don't have access to an M.2 slot, so I need to break out like standard USB-C and whatnot.

This also means that I want to add other functions like HDMI, maybe even GbE and just generally make it insanely useful to hobbyists!

There's a couple key goals I have for this project:
- Learn to route BGA and high-density impedance controlled traces
- Learn GbE, HDMI and PCIe routing
- Understand at a high level, the physics behind the board
- Focus on routing best practices, specifically minimizing loops

This project has already been similarly done by Phil's lab, but his project was more-so focused on acceleration instead of as a hobbyist development board, but this video will still be really useful to reference during development of this project! https://www.youtube.com/watch?v=8bw80LiCl7g

## Connectors!! - 4 Hours

Now the first thing I have to decided, is what type of edge connector I want to use. Now NVMe SSD's are standard M.2 size, but have different type of "keys" which is essentially where the different holes are on it:

![Pasted image 20251219113929.png](images/Pasted%20image%2020251219113929.png)

NVMe SSD's mostly just use M-Key edge connectors, so that's what I'm going to use.

But now, there's also more than just one type of size, there's actually 5 of them (probably more than that, but the 5 most common)

![Pasted image 20251219114039.png](images/Pasted%20image%2020251219114039.png)

Now 2280 is by far the most common, but I don't actually think I'll use all that space, so for now, I think I'm going to go with 2242 which is decently common still but for other applications most of the time.

The numbering system is really simple. 2242 means 22mm x 42mm, 2280 means 22mm x 80mm.

So now i have to create my footprint and symbol for these edge connectors. I can also easily change my form factor after making the footprint because it's legitimately just the height that changes.

Lucky for me, someone's already made some M.2 footprints and symbols, so I'm going to use these ones from https://github.com/timonsku/M.2-Card-Footprints

Now this person didn't actually make an M-Key, so I had to modify one of the existing ones to make it. M.2 footprints are really interesting, because they literally just remove the pins where the key is and don't really shift anything, so it's easy to make footprints for:

![Pasted image 20251219114720.png](images/Pasted%20image%2020251219114720.png)

Now I was thinking I might be able to use a B+M-Key to be more compatible, but the ECP5 only has a soft SATA PHY or SERDES which would be really complicated and annoying to implement, so I'm just keeping it symbol.

And then this library I downloaded included the symbols already, so I'm all set to get working:

![Pasted image 20251219115014.png](images/Pasted%20image%2020251219115014.png)

## Edge connector and choosing an FPGA - 4 Hours

Now I want to get working on actually wiring the edge connector! The first thing I'm going to do is wire the power and ground.

I decided to use one 100nF cap per pin, and then one 1uF + 10uF per group of them like so:

![Pasted image 20251219121205.png](images/Pasted%20image%2020251219121205.png)

I'm referencing the M.2 PCI express electromechanical datasheet for pins https://picture.iczhiku.com/resource/eetop/sHksKPigIJigRbbx.pdf

![Pasted image 20251219121236.png](images/Pasted%20image%2020251219121236.png)

I've decided to 0402 for 100nF, and 1uF, and then 0603 for 10uF to save as much space as possible, but still be pretty efficient at decoupling.

Next, I need to choose what FPGA I actually want to use. The first thing that comes to mind is the ECP5. It's a small and cheap FPGA with SERDES so it works with PCIe which is really cool. 

It has a faster (5G) and slower version, so I can have gen 2 PCIe speeds which is pretty cool. The minimum 5G FPGA costs around $18.50 https://jlcpcb.com/partdetail/Lattice-LFE5UM5G_25F8MG285C/C1551932 so it's a pretty good option.

But there's also the Xilinx Artix FPGA which is significantly more powerful, the XC7A100T has nearly 4x the LUTS and is way more powerful, but is also significantly larger, having 4x lanes. The price is actually pretty good though, coming in at just $29.

So the Artix FPGA is probably a better option, because the price and accessibility of the ECP5 for the specs I need is just kind of out of scope. It'll also give me really good practice with a really popular AMD FPGA which could be really helpful for the future! 

Now the Xilinx Aritix has MANY different packages, ranging from 12K to 215K LUTs and 720 to 13000 Kb of memory. I also need a minimum of 4 GTP transceivers which are essentially just high speed pins so 15K LUTs minimum.

But let's stay within the price range, so I want under $20 for the FPGA and preferably over 25K LUTs which gives me many good options!

This means I should also increase the size of my board from 2242 to 2280 which I actually wanted an excuse to do, and I can also add all those other features. 

So in the end, I think I'm going to go with the XC7A50T-1FGG484C, the Xilinx Artix 7, with 50K LUTs, coming in at just $17 at minimum quantities, and having 484 pins!!! https://jlcpcb.com/partdetail/AMDXILINX-XC7A50T1FGG484C/C1521780

I think this is honestly the perfect option, and I know Phil's Lab uses the same one, but it's the best for the job!

## Working on the FPGA - 3 Hours

Now that I know I want to use the Artix 7 FPGA, I need to add it into KiCad. I'm really lucky because KiCad actually has this symbol built in, but the footprint doesn't seem to come up, so I'll need to look into that:

![Pasted image 20251220104108.png](images/Pasted%20image%2020251220104108.png)

![Pasted image 20251220104157.png](images/Pasted%20image%2020251220104157.png)

And lucky me, a footprint actually does exist!!

![Pasted image 20251220104406.png](images/Pasted%20image%2020251220104406.png)

Next, I want to figure out what voltage lines I need for my project. Based off of the datasheet, I'll need:
- VCCINT 1V, FPGA fabric logic, lowest voltage for CMOS is 1V
- VCCBRAM 1V. internal block RAM, needs to be really stable
- VMGTAVCC 1V, analog GTP transceivers voltage, might need to be seperate from VCCINT
- VCCAUX 1V8, powers configuration and clocking stuff, 1V8 for noise margin
- VCCO 3V3, powers I/O banks 3V3 because connector is 3V3 and I use 3V3 logic
- VMGTAVTT 1V2, very interesting termination voltage, it basically controls the impedance of the GTP transceiver/PCIe lines, and makes sure they don't see reflections and too much nosie!
- DDR3 Voltage, 1V5 for DDR3 I/O bank pins which are 1V5 for standard DDR3 (1V35 for DDR3L)

![Pasted image 20251220165608.png](images/Pasted%20image%2020251220165608.png)

So now that I have all the theory worked out, I can get working on making the actual thing! 

This was just a bunch of datasheet reading today and a couple video's, but I have a really good idea on how to do this now!

## Decoupling time! - 5 Hours

Decoupling is fairly simple with the Artix 7, the datasheets tell you almost exactly how to do it:

![Pasted image 20251221082541.png](images/Pasted%20image%2020251221082541.png)
![Pasted image 20251221082558.png](images/Pasted%20image%2020251221082558.png)

So just like that, I organized everything and added in the decoupling: 

![Pasted image 20251221082621.png](images/Pasted%20image%2020251221082621.png)

And I'll also need ferrite beads on these lines, but before that, I want to figure out what PMIC I'm going to use. I have 4 rails, and I need 3V3, so a quad switching buck converter would be a really good option! 

I want to do this, because the characteristics of the bucks will help me determine the values of the ferrite beads!

I spent nearly 6 hours straight, fueled from Yerba and classical music trying to find the best buck, and I've found 2 possible options.

The [MAX20029](https://www.analog.com/en/products/max20029.html) is the lower cost option, with up to 1.5A channels, and is fixed/adjustable, though I would only use the fixed part. It's high efficiency, but the efficiency kind of drops off at lower current, but it has a smaller package and is designed for automotive. You can use the same value inductor on all the rails and it has power sequencing. This is the same one that Phil's lab uses, and it's really good!

The [LTC3370](https://www.digikey.ca/en/products/detail/analog-devices-inc/LTC3370IUH-TRPBF/5155587) is a bit more overkill, with strong 2A channels, and really good efficiency characteristics. It's a bit larger and a bit more expensive, and it's a bit overkill, but you can have the same value inductors, and is also a fantastic option! 

I've legitimately looked at 100's of buck converters, and these are probably the 2 best options! 

In the end, I think I'm going to go with the MAX20029 because it has smaller inductors, is cheaper and perfectly fits the specs. Even though it's what phil uses, it honestly just is the superior option and the only downfall is I can't use any of the fixed variants, so I have to use quite a few voltage dividers. 

I've done a lot of research today, so I think I'm going to hop off for now! 

## Power management! - 8 Hours

Now that I've found what PMIC I want to use, we need to add it into KiCad.

But after some second thought, I've decided that the LTC3370 is actually probably a better choice because of it's higher current demands which means better power tolerance, and I want support for all the extra peripherals I might add too. 

So let's add that in! 

The first thing I need to do is create the symbol for my buck converter. After referencing the datatsheet and tuning things up, I've come up with this pretty clean symbol that will probably change to my likings, but it's good as a template, and has all the necessary pins and footprint! 

![Pasted image 20251225000450.png](images/Pasted%20image%2020251225000450.png)

![Pasted image 20251225000521.png](images/Pasted%20image%2020251225000521.png)

Next, I'm going to wire all the important stuff for the buck converter. Per the datasheet, each input should have a 22uF cap, VCC should have 10uF, and each 2A channel output should have 47uF! 

I calculated the voltage dividers using VOUT = VFB(1 + R2/R1), and used 100K as my bottom resistor to minimize my BOM. This gives me a really nice and convenient voltage divider setup:

I also used 2.2uH inductors which are recommended by the datasheet for 2A, and they have a really low ESR to minimize power loss:

![Pasted image 20251225000820.png](images/Pasted%20image%2020251225000820.png)

BUTTTT, this is kind of when I had a brain blast and decided to just outright switch to the MAX20029.

I realized that I would be saving over $7 with the MAX20029 and it has way simpler power sequencing and smaller package, so it's just significantly better.

So let's make the symbol for the MAX20029! I came up with this tall little symbol, I might make it larger but it kind of fits my needs:

![Pasted image 20251227142741.png](images/Pasted%20image%2020251227142741.png)

And then it's really simple wiring, you just need pullups on PG to have a stable state, and also calculate the voltage dividers and frequency matching capacitors:

![Pasted image 20251227142844.png](images/Pasted%20image%2020251227142844.png)

PG1 will go high when it's active, and the pullup just helps to ensure it gets there. There's no voltage divider on 1V0, because the VOUT is already 1V, and then the rest have voltage dividers to get there and frequency matching rounded down to the nearest E6 capacitor!

## More decoupling and filtering - 5 Hours

Now that I've finished with my power supply, I need to figure out how to properly filter it for my analog rails! 

This actually took me a really long time to figure out because I've never fully understood ferrite beads, but after a bit of research and talking with the KiCad guys, I've done a simple but effective implementation in my opinion:

![Pasted image 20251229001159.png](images/Pasted%20image%2020251229001159.png)

I chose 600R@100MHz to filter out the high frequency noise moderately and it's low DCR so it has a minimal drop and is up to 2A. I made a little low pass filter LC filter by adding some shunts to ground, and added a bulk before the cap on MGTAVCC because it's higher current!

I decided to also set up an LTSpice simulation for these ferrite beads, just to make sure everything will be fine, and I can use it in the future for more accurate testing and adjusting once I've finished routing the board!

This took me a really long time to figure out, but I think it turned out really well! 

![Pasted image 20251231173655.png](images/Pasted%20image%2020251231173655.png)

This isn't too relevant right now, but will be really useful once I've finished routing! 

I also fixed up some of the decoupling on my m.2 edge connector because it wasn't using the standard values in my schematic.

![Pasted image 20260106130817.png](images/Pasted%20image%2020260106130817.png)
It actually took me so long to figure out LTSpice, but that was essentially how my day went! 

## Configurations and JTAG, oh no - 15 hours

So now I have the majority of my fundamental power system in place, but I actually need a way to program this thing. The first thing I have to do is configure the artix 7 to work over master SPI. I referenced the datasheet for this part and it pretty much tells you what to do:

![Pasted image 20260106131135.png](images/Pasted%20image%2020260106131135.png)

A 0 means tie it low, and a 1 means tie it high. I also added this DNP resistor in case you want to use the artix 7 built in flash for debugging, so it's just kind of handy:

![Pasted image 20260106131305.png](images/Pasted%20image%2020260106131305.png)

I also learned how buses work, and created a JTAG bus which connects to a hierarchial label which will connect to my USB to JTAG/TC2030.

The next thing I need to figure out is my JTAG. This is how the board will be programmed, the JTAG interface will feed into the artix 7, and the artix 7 will feed it into SPI.

Now the first thing I want to do is add on a TC2030 interface which is a compact way of programming JTAG using this little debugger:

![Pasted image 20260106131603.png](images/Pasted%20image%2020260106131603.png)

I also wanted to have USB to JTAG because it's way easier and actually cheaper to debug with it because you don't need an external probe. Now I probably spent over 10 hours thinking about how I should do my USB to JTAG, I put so much thought into this.

But at the end of the day, Vivado, the tool I'm probably going to use to program my board, and all the other tools only really support FT2232H, and because I'm probably going to be debugging a bunch, I didn't want to bitbang, so I decided to go with the large and expensive IC for programming!

The thing is, I don't have to place the FT2232H in my production runs, only in my prototypes runs so it's perfectly fine to have for prototyping, it'll just cost more for those boards though....

I also wanted to have USB to power and program the board, so I came up with this monstrosity of a schematic:

![Pasted image 20260106131956.png](images/Pasted%20image%2020260106131956.png)

USB-C comes in, get's bucked down and has some ESD protection. The data lines go into the FT2232H USB to JTAG, and then JTAG get's pulled up so nothing's floating, and goes into my bus for use in the configurations schematic. The TC2030 will go underneath the USB-C receptacle and if you DNP the USB to JTAG logic, it'll still be programmable via the probe. Pretty cool huh :D

I did some really good work for these couple days, but I'm still missing some fundamental logic for this circuit that's really important, but we'll see this in the next section!

## Power architecture - 6 Hours

Now there's a bit of a problem that arises with having both USB-C and an m.2 edge connector supplying our board. What if the current just flows right onto the motherboard, completely frying and destroying it.

This is why we need to implement a power mux on our board! I usually try to shy away from power mux's because there's usually a smarter way of solving power issues, but in this case, it was pretty much necessary, and the slight added cost is important so that our board functions smoothly.

I found this nearly perfect IC, which let's met control the ramp up of the inrush, the max current, switchover and so much more, all in a small package! This is THE TPS2121RUXR.

This beauty took me a whole to wrap my head around, but it does everything for me:

![Pasted image 20260106132555.png](images/Pasted%20image%2020260106132555.png)

I prioritized the M.2 connector source because the motherboard is always going to supply smooth power because you need to be able to have a reliable computer right!

And then of course I added some smooth decoupling, and we got that finished after some time! 

I'm going to end off this section here, because now we're going to get into some really technical territory, figuring out the pinouts for my flash, ram, etc. It's going to be a wild ride! 

## Flash storage and configurations - 4 Hours

Now that I've finished with the power architecture of my board, I need to implement all the configurations for my board and also add the flash storage.

The flash storage is fairly simple, there's some fixed pins on the artix 7 for it, so I can just wire it to those with some pullups according to the artix datasheet. I then added some decoupling:

![Pasted image 20260111151005.png](images/Pasted%20image%2020260111151005.png)![Pasted image 20260111151023.png](images/Pasted%20image%2020260111151023.png)

Next, I added a basic configuration to the board to make it work according to UG470:

![Pasted image 20260111151114.png](images/Pasted%20image%2020260111151114.png)

- DONE is wired so that the LED will turn OFF when the configuration is finished and ON when it's actively configuring. 
- INIT is just a configuration pin that tells you if there's any configuration errors and needs to be pulled high <= 4k7R
- PROGRAM is another configuration pin that's used for configuration reset, and I just needed to pull this high. After reading this a second time, I decided to actually add a button to manually reset it if I wanted to, because this board is technically a devboard ya know (shown below)
- CFGBVS is a bank voltage configuration, it's needed because there's configuration pins on multiple banks so to maintain a near the same voltage of all of the them, I pulled this high to tell the artix 7, they're all 3V3 essentially
- And then, VP/VN, VREFP/VREFN and DXP_0/DXN_0 are essentially just part of the XADC to get internal reads of the FPGA and whatnot, and I just tied them low for now to disable all of that, but I want to revisit it, but not necessarily change any of it!


![Pasted image 20260111153814.png](images/Pasted%20image%2020260111153814.png)

## Fun DDR3 wiring time now :D - 6 Hours

Now that I've wired up the flash storage, I need to add in the DDR3 memory! Flash storage is non-volatile which means it stores information long term, retaining all of it without power, while RAM is volatile and is used for active tasks (but you probably knew that ;). 

DDR3 works pretty fast, so you have to pay attention to how you wire it! I chose my RAM from an artix 7 datasheet which specifies the compatible ones you can use, and I chose MT41J128M16JT.

This is a 2 Gb (gigabit) memory with 16 bit data width which means it's 256 MB (2Gb / 8) of memory that can fit 128M words 16 bits wide, because 2Gb / 16 = 128! 

![Pasted image 20260111211326.png](images/Pasted%20image%2020260111211326.png)

I created this custom DDR3 symbol because I like to have my decoupling pins visible so I don't make any dumb mistakes. I did a 100nF cap per VDD pin and then a bulk cap near both sides to keep it stable.

Next, I wired all of the address, command, data and clock buses to hierarchical sheets to keep it clean.

CS# is then tied low, because I'm only using one memory IC, and I have pulldowns on ODT, CKE and RESET as suggested from most reference boards. I put a 100R shunt resistor (an AC resistor) to match the differential impedance of 100R, and then ZQ is tied to 240R which is like a calibration pin and needs to be tied with this exact value! 

You'll notice a perfect split voltage divider on Vref, which is essentially just half of VCC, and it needs to be really precise to distinguish cleanly between high and low data signals for fast DDR3 signals. I put a bunch of decoupling on it to make sure it's really precise with very low noise! 

And that's our DDR3 wired, it was a lot more complicated to do than I described, but that's the gist of it! 

## PCIe and the GTP WHAA transceivers ;) - 5 Hours

Now that I've added DDR3 in, I can power on my board, fully program it and execute commands on it too! Now I need to wire the PCIe lanes to the GTP transceivers which lets me board connect to your computer via the m.2 connector! 

Remember earlier how we added in the m.2 connector for power, well now I just have to wire all the lanes up and came up with this:

![Pasted image 20260111213647.png](images/Pasted%20image%2020260111213647.png)

I changed up the decoupling to instead be a mid range cap per group of pins, because all I really care about is dampening the inrush when plugging the board in/out and powering it on. 

Now let's list out what all of these signals are for! All except for SUSCLK are SATA only pins, and SUSCLK is just for low power states, when you sleep your computer and want to still maintain basic functions which I don't want.

Next, I actually modified my old symbol to create this edge connector symbol which basically just inverses the PCIe text to be more accurate of what's happening on our board. So the pins that were originally receiving PCIe in the form of a bus, are SENDING them now on our board. So all of these transmits, are what's transmitting to our card.

It's mutually known that the side receiving the signals should handle it's own AC coupling, so my FPGA TX lanes, which are sending from the edge connector, and receiving into my board, are going to be AC coupled! 

Because PCIe is close to a fundamental square waveform, it's got sharp edges. So these capacitors in parallel filter out the DC noise, and only allow the switching waveform noise to pass through, pretty cool huh! I used 220nF because it's suggested for if you want multiple generation backwards compatibility because you can use like gen 3 in gen 2 systems, gen 4 in gen 2 and so on and so forth.

CLK also has AC coupling because the direction isn't fixed so they should both be handling it regardless! 

Next, you'll notice 22R dampening resistors on some of the command lines. These are the longer and slower command lines which just travel really far, so I just want to make sure nothing got messed up on the way. PERST is a bit faster so I don't want to cause anti-resonance by adding in dampening resistors on it.

Now we need to just wire these PCIe signals into our FPGA, and BOOM, we're done!

![Pasted image 20260111214553.png](images/Pasted%20image%2020260111214553.png)

We don't use the other clock and it's probably reserved for some other uses, so I'm just going to no connect it for now! I want to double check I did this right later, but it should be fine! 

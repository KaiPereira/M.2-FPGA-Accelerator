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

## Time to HDMI - 6 Hours

Because I want my FPGA hardware accelerator devboard to function as a standalone devboard, I really want to add HDMI so that I can use it as a linux capable device, and some other very fun shenanigans. 

Because you could plug it into both your computer and HDMI, you could do fun things like low latency video output or video processing or even data visualization for the whole acceleration!

So I came up with this little schematic:

![Pasted image 20260120061538.png](images/Pasted%20image%2020260120061538.png)

First of all, I don't want ethernet powering my board, so I added a diode to prevent that, and it's connected to VBUS so if USB is plugged in, you get the extra features from HDMI.

I want to actually add a really small boost onto the board so that I can get these features, so I'll think about the best way of adding that! 

My board is pretty noisy, so I added a 1uF cap just because I can, and then of course 100nF for basic decoupling.

The only signals that actually need level shifting is DDC because it's I2C so it's decently fast and important signals. I decided to do some fancy current limiting on HPD so it's a really weak 5V signal that can't damage my FPGA but the FPGA can clamp the voltage to still be able to read it! 

And then CEC is just the electronics control and it's a 3V3 signal that I just need a weak pullup on! 

I added ESD protection to all of the lines, just because I want to create a pretty professional board, and I also added minor AC coupling because it's really fast edge-rate signals.

I think I had a really strong fundamental understanding of this wiring which I'm really proud of myself for, but I really need to get that 5V sorted out, and a $1 boost might be the best way to do that. 

## Ethernet shenanigans - 7 Hours

The second peripheral I wanted to add onto the board is ethernet! 

Ethernet is really cool to add to an FPGA board like this, because it allows you to use the FPGA as like a firewall of sort, or even send signals, and it just has some really neat use cases.

I decided to use a magjack with integrated magnetics and other useful stuff because I don't want to have to worry about all of that working with ethernet for the first time. 

![Pasted image 20260120062520.png](images/Pasted%20image%2020260120062520.png)

First, you'll notice really strong pullups on MDI lines. These aren't actually pullups, and are instead parallel termination. Essentially when signal reflections hit these lines, they'll sink into the power/ground instead of just continuously reflecting, and this is why it's on the analog rail, so the power can get filtered! 

Next, you'll notice my basic decoupling and termination on TX/RX lanes. The RX termination will be close to the PHY, and the TX termination will be closer to the MAC.

RMII needs a 50 MHz crystal, so I've added that in, alongside a feedback resistor which just makes the crystal function smoothly, but you don't really need that so it's just DNP.

I added a button onto the ethernet PHY to force reset, because it's a devboard, I just want a bit more customization, so I decided to just add this!

Finally, we have the LED's, one is pulled up internally, and one needs an external pullup, but once they have pullups they act as current sinks so I can just wire those as sinks to the PHY LED's.

And BOOM, we have ethernet fully wired on our board! I'm really excited to work more with ethernet and understand the PHY and magnetics some more, because they seem really cool!

## Revisions and starting to route everything - 8 Hours

Now that I've finished wiring my ethernet, I'm actually done my schematic!

Now we can work on laying out all the components onto the board so we can make sure that it fits nicely! This is a really iterative process, but you can get a nice rough layout which you'll modify greatly as you route, but it'll form the basis of your board!

**I actually decided to remove my ethernet in this step, because the PHY was too large for such a small card, it wasn't worth it...**

First I layed out all my large components onto the frontside of the PCB so that I could get a grasp of how the board would look.

Next I added all the passives for the top layer to have short loop area's, and really just going for optimal practices. There's a lot of thought that goes into doing this, but it's not too difficult!

![Pasted image 20260308170221.png](images/Pasted%20image%2020260308170221.png)

Next I took a look at the back side. Essentially where I couldn't fit all of the other passives is where I would place these ones. This is especially true for stuff like my decoupling which won't have space on the front because of my BGA DDR3, artix 7, etc.

![Pasted image 20260308170313.png](images/Pasted%20image%2020260308170313.png)

There was a LOT of thought that went into this layout. I first modelled it off of existing boards, so fpga in the center, power management at the bottom, connectors on one side, and then USB to JTAG at the top alongside the flash and DDR3!

There's a couple components I left off the board for now, because those aren't too important, and they're in spots where traces might go, so I don't quite know exactly where I want to put them!

This was actually after I started routing, so most of these components are in their final spots. Lots more components were actually off the board, and it's a constantly iterative process!

## Decoupling scheme and some basic routing - 7 Hours

Decoupling an FPGA and all the other peripherals is extremely difficult. They're power intensive devices and you really need to know the theory behind what you're doing.

The main things I was thinking about, was keeping a short loop area, minimizing trace inductance and properly laying out components for future routing.

Ignore the DDR3 traces, I took this screenshot after doing some intensive routing:

![Pasted image 20260308170716.png](images/Pasted%20image%2020260308170716.png)
![Pasted image 20260308171023.png](images/Pasted%20image%2020260308171023.png)

You can see I used the largest possible trace and via which maximized the surface area of the copper, minimizing the density of the magnetic flux, decreasing inductance which give higher efficient decoupling.

The higher frequency decoupling (470nF) is priority, it should be directly next to the pins it's decoupling. The mid frequency decoupling can be a bit farther away, but you still want to keep it close to the group of pins. And then finally, the low frequency decoupling can be quite far away, but you want to make the path extremely low inductance nonetheless. 

You'll notice that there's no via-in-pad, this is an extra cost that I really don't need, and I want to create an extremely professional board, so it's something that was really important to me.

Now there's a lot of decoupling on this board, but I really just want to go over the FPGA and DDR3 decoupling because those are definitely the most intensive.

Next, let's take a look at our DDR3 decoupling. This needs to be extremely efficient for a hungry device, so we keep the paths short and tight.

You need to keep VREF extremely quiet, so you'll notice a bunch of extra decoupling on that too. 

Ignore these other traces, this is after I routed my DDR3 which we'll go over soon:

![Pasted image 20260308171254.png](images/Pasted%20image%2020260308171254.png)
![Pasted image 20260308171307.png](images/Pasted%20image%2020260308171307.png)

And just like that, we've efficiently decoupled our intensive components! Next let's take a lookat the most complicated bit of routing, which I've already done a lot of, our DDR3!

## Intensive DDR3 routing - 50 Hours

Now, let's take a look at arguably the most complicated part of our board, the DDR3 routing. 

There's a LOT of considerations when routing DDR3:
- Efficient power distribution and VREF
- Proper configurations
- Proper layout/dogbone/breakout
- DDR3 to FPGA routing
- Length/delay mathching
- Differential pairs

And a whole lot more.

Let's start off with our power distribution. I already went over this a lot, but all of the DDR3 pins should have 100nF decoupling, and medium/bulk decoupling along the sides of the chip. You can see exactly how we decoupled this chip, high frequency decoupling RIGHT by the pins, and then mid/low frequency at the top.

![Pasted image 20260308171730.png](images/Pasted%20image%2020260308171730.png)

Next we can take a look at VREF. VREF is a little pesky brat that needs to be well filtered, so we have 100nF to 10nF caps right by the pins so we can efficiently run our DDR3. 

![Pasted image 20260308171847.png](images/Pasted%20image%2020260308171847.png)

The layout considerations of these caps is extremely important because you need to image how you're going to get all of your traces through still, so it's a big puzzle game. 

Next we have the actual routing and dogbone. Lots of your traces will need a via to get to other layers, so we put them in the center where there's no pins, perfectly placed so we don't get solder taken away from the pads by the via's.

Routing these traces is definitely what took the longest, because there's so many considerations. 

I routed the traces probably about 10 different times, because you need to pass what's called MIG, which essentially tells you if you used the right pins on the FPGA for your DDR3. The FPGA doesn't have specific pins, but instead byte lanes, where say pins DQ0 - 7 could go, and not other signals, so it's not crazy specific but still tedious.

After passing MIG, you need to length/delay match all of your signals. This is an EXTREMELY dense layout, so doing this is absurdly difficult and tedious, which took a LOT of time.

I'm not going to explain all of the specifics or else I would basically create my own DDR3 datasheet, the important stuff is all already created and I'm not going to repeat it. 

But this is what our final layout looks like, it's absolutely beautiful and took wayyyyy too long:

![Pasted image 20260308172343.png](images/Pasted%20image%2020260308172343.png)

![Pasted image 20260308172401.png](images/Pasted%20image%2020260308172401.png)

![Pasted image 20260308172414.png](images/Pasted%20image%2020260308172414.png)

This isn't actually the end of my DDR3 shenanigans though. I haven't done it yet, but I'll need to simulate the signal integrity of my traces using a field solver because of the extremely dense layout, there's a lot of local crosstalk which could kill my design. 

## More layout and basic routing - 5 Hours

Now it's really hard to say specifically what I'm up to in these parts, but I'm essentially just trying to lay out all the components, and get ready for a massive spree of routing. Because all the connections are made to be as short as possible, routing everything else is actually fairly simple.

All of these small things is just like configurations, passives, all the small details ya know.

So let's take a look at the current status of the board:

![Pasted image 20260308172955.png](images/Pasted%20image%2020260308172955.png)

![Pasted image 20260308173008.png](images/Pasted%20image%2020260308173008.png)

![Pasted image 20260308173018.png](images/Pasted%20image%2020260308173018.png)

The only components left are this button with it's resistor and an LED, because I want to figure out if I want to add some more LED's and I don't quite know exactly where I want to put these yet, so it's not crazy serious.

I was also running the DRC during this time to make sure that I didn't have any problems that would annoy me AFTER I finished doing all of this routing, so it was just like a lot of small stuff I was up to.

Let's just take a minute to admire this beautiful, near 100% density layout :D 

![Pasted image 20260308173205.png](images/Pasted%20image%2020260308173205.png)

Now that I've actually put everything on the board, it's time to actually route it!!!! 

This honestly won't take too long, because all the connections are so optimally placed already, it's just a quick trace ;) 

## Top half routing - 6 Hours

Now initially I thought that the rest of my layout would've been simple to route on 6 layers, but I was actually kind of wrong. 

The first thing I need to do, was route the top half, to the bottom half. This means USB to JTAG, QSPI flash, crystal, etc. I decided to keep the QSPI and JTAG signals on the middle layer and then leave all the simple direct connections to the top and bottom so they could be low inductance. 

![[Pasted image 20260329142145.png]]

![[Pasted image 20260329142208.png]]
![[Pasted image 20260329142235.png]]

You can see it's getting extremely dense which makes it significantly harder to route. I'm currently still in the process of routing the JTAG lines, but it's almost done. 

## Power architecture - 7 Hours

Now that I've routed pretty much the entirety of the top half, I need to route most of the bottom half. This half is a bit more important because it consists of the high speed PCIe gen 2.0 and also the power architecture/rails.

Now this part will only consist of the power architecture just barbecue it took a really long time and was a major part of this board. 

So the first thing I had to do was layout my buck converter so that I could efficiently branch out the power fills on the same layer. 

I drew it out and then matched it onto the cube shaped quad buck:

![[Pasted image 20260329142723.png]]
![[Pasted image 20260329142733.png]]

This gave me a good visualization of how to do the fills, so this is how I layed out my buck:

![[Pasted image 20260329142828.png]]

I also added a small 5V buck boost so that I could always power the HDMI, which I thought was worth it so I didn't have any unpredictable behavior on this board. It'll also let me break out another 5V debug pin which is pretty useful to people. 

Now that I've finished the buck schematic and routing, I can do my power fills and the rest of the bottom routing. Do note that a lot of my bottom stuff has already been routed, and that this is just an old screenshot, but you'll see more in the next log.

And this is how I did my power fills, pretty clean and messy at the same time:

![[Pasted image 20260329143046.png]]
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

![[Pasted image 20251216201028.png]]

So the idea is simple, I want a low-cost way, hobbyists can make practical applications of FPGA's in a bunch of different settings.

To do this, I want to provide an FPGA devboard of sorts, that can interface with your computer hardware so you can practice on real-world applications in more niche settings. But I want to go a step further and also make it function as a standard devboard in case you don't have access to an M.2 slot, so I need to break out like standard USB-C and whatnot.

This also means that I want to add other functions like HDMI, maybe even GbE and just generally make it insanely useful to hobbyists!

There's a couple key goals I have for this project:
- Learn to route BGA and high-density impedance controlled traces
- Learn GbE, HDMI and PCIe routing
- Understand at a high level, the physics behind the board
- Focus on routing best practices, specifically minimizing loops

This project has already been similarly done by Phil's lab, but his project was more-so focused on acceleration instead of as a hobbyist development board, but this video will still be really useful to reference during development of this project! https://www.youtube.com/watch?v=8bw80LiCl7g

## Connectors!!

Now the first thing I have to decided, is what type of edge connector I want to use. Now NVMe SSD's are standard M.2 size, but have different type of "keys" which is essentially where the different holes are on it:

![[Pasted image 20251219113929.png]]

NVMe SSD's mostly just use M-Key edge connectors, so that's what I'm going to use.

But now, there's also more than just one type of size, there's actually 5 of them (probably more than that, but the 5 most common)

![[Pasted image 20251219114039.png]]

Now 2280 is by far the most common, but I don't actually think I'll use all that space, so for now, I think I'm going to go with 2242 which is decently common still but for other applications most of the time.

The numbering system is really simple. 2242 means 22mm x 42mm, 2280 means 22mm x 80mm.

So now i have to create my footprint and symbol for these edge connectors
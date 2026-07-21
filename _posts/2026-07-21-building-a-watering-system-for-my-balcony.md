---
layout: post
title: "Building a watering system for my balcony"
author: "Kuba"
comments: true
description: A weekend electronics project to keep my Berlin balcony's plants alive through the summer heat — an ESP32-controlled drip system, the components, the wiring, and what I'd do differently next time.
image: "/_imgs/hydration-station/board-and-water-tank.png"
---
My balcony faces south on the 4th floor in Berlin. In summer that means it absolutely bakes. Several days of direct heat in a row, and if you forget to water your plants for even 2 to 3 days, they’re gone.

We had been through this once already. It’s painful to watch your plants die like that, and it’s also a waste of plants, money, and time.

So I decided to fix it properly. I finished a masters in control engineering, and over a decade ago I spent a lot of time with microcontrollers and electronics. I hadn’t had a good excuse to get back into it since, and this felt like the perfect one. **What if I could build a small, controlled watering system that adjusts to the moisture of the soil and the weather outside?**

![The prototyping board wired up on the balcony floor, next to the water carboy tucked into the corner]({{ site.url }}/_imgs/hydration-station/board-and-water-tank.png)

### The scope, and where I was wrong

My initial scope was ambitious: a wifi-enabled controller so I could adjust things remotely, the ability to water a few plants on the balcony, and moisture sensors driving electronic valves to regulate the flow of water.

**The moisture sensors turned out to be very low quality and never gave reliable readings, so I dropped the idea, along with the solenoid valves, and replaced them with micro-drippers from a Gardena garden watering system.**

A classic lesson to start from the simplest thing and add scope only if you prove initial value :)

### No taps on Berlin balconies

The biggest constraint had nothing to do with electronics. **In Berlin apartments you don’t get running water on the balcony, so the whole system has to carry its own water and generate its own pressure.**

I solved it with a carboy — the kind used for home wine production, bought at a local construction market — and a 12V pump that pushes water from the carboy through the drip lines. Refillable, self-contained, and it sits happily in the corner of the balcony.

### The components

The core build stayed small:

- **MOSFET** to switch the water pump from the ESP32 — [eBay](https://www.ebay.de/itm/233000146342)
- **Schottky diode** for the pump’s flyback spike — [eBay](https://www.ebay.de/itm/233998850597)
- **Boost converter** stepping 5V up to 12V to power the pump — [eBay](https://www.ebay.de/itm/233627491422)
- **12V water pump** — [eBay](https://www.ebay.de/itm/232597865686)
- **ESP32 starter kit** as the controller — [Amazon](https://www.amazon.de/dp/B0CTMQQ1W4)

The extended scope I ended up shelving:

- **Moisture sensors** — [eBay](https://www.ebay.de/itm/234556768072)
- **Solenoid valves** — [eBay](https://www.ebay.de/itm/315185626029)

And the Gardena-compatible micro-drip parts that replaced the valves:

- **Pipe (4/7mm)** — [Amazon](https://www.amazon.de/-/en/20-meter-garden-irrigation-diameter-watering/dp/B0G48GQMPC/)
- **Drippers and holders** — [Amazon](https://www.amazon.de/-/en/dp/B08K2ZJD1K)
- **T-connectors** — [Amazon](https://www.amazon.de/-/en/Irrigation-T-Connector-Universal-Connector-Fittings/dp/B0FBGX5BSL/)
- **Carboy** for home wine production, from the local construction market

![A Gardena micro-dripper feeding one of the balcony's petunia pots]({{ site.url }}/_imgs/hydration-station/gardena-drip.png)

### The electrical diagram

The wiring is quite simple and I entirely built it on a prototyping board. A single USB-C 5V input feeds everything, and one MOSFET does all the switching. I used Claude to remind me all the nuances of how to keep the MOSFET switching safe.

USB-C 5V powers both the ESP32 and a boost converter that steps 5V up to an adjustable 12V. The ESP32’s GPIO drives the gate of an N-MOSFET with PWM.

The MOSFET low-side switches the 12V pump: pump positive to +12V, pump negative to the drain, source to ground. A 10kΩ resistor pulls the gate to ground, holding the FET off during ESP32 boot and reset.

A Schottky flyback diode clamps the pump’s inductive turn-off spike, cathode to +12V and anode to the drain. Three rails run through it all: +5V from the USB/boost input, +12V from the boost output to the pump, and a common ground tying the USB-C, ESP32, boost, MOSFET source, and diode return together.

![The electrical diagram: USB-C 5V in, a boost converter to 12V, the ESP32 driving an N-MOSFET that switches the pump, and a Schottky flyback diode across it]({{ site.url }}/_imgs/hydration-station/hydration-station-electric-diagram.png)

### The code

The firmware lives on GitHub and was generated in 2025, mainly with the help of GitHub Copilot back then: [github.com/jniechcial/hydration-station](https://github.com/jniechcial/hydration-station).

### The control plane

I didn’t want to depend on the cloud for something this small, so the ESP32 serves a simple website on my local network. **Everything I need to run the system — trigger a watering, set a schedule, check the state — is a page on my LAN, no app and no external service.**

![The hydration station's control website, showing manual pump control, scheduling, and system status]({{ site.url }}/_imgs/hydration-station/control-plane.png)

### Lessons, and what I’d do next

It’s a very reliable basic watering system, but it still needs a bit of manual oversight for the best outcome, especially during extremely hot spells. That said, it did keep the flowers alive during 2 weeks we were away on holiday, with a neighbour just topping up the water tank twice in that time.

**The main limitation is that the watering is uniform, but the plants are not.** You can tune each dripper, but what you really want is per-plant logic: water this one once a day, that one twice. Leave the system with a lot of active time to keep the thirsty pots happy, and the smaller pots quietly get over-watered.

I’d love to get back to the valves-and-sensors idea. If I could find moisture sensors I actually trust, there’s still the visual problem of routing meters and meters of individual power and control cables around the balcony without it looking like a bomb.

Moving everything off the prototype board onto a proper custom PCB, with a 3D-printed enclosure, would be the satisfying finish. Maybe at some point, if I find the time!

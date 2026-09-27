# MNIT T02 notes: plan (scratch)

Based on the slides `RMICT.2_WLAN.pdf` (Topic 2 – WLAN IEEE 802.11). Note: the deck's own header says "SCM" (Sistemas de Comunicação Móvel) rather than "RMIC" like the previous file, but it's the same course under a different/older name used on this particular slide set; keeping MNIT for the file naming, consistent with the folder and with T01.

## Note on non-extractable content

Several slides are screenshotted tables/diagrams (the 802.11 physical standards table, the 2.4 GHz channel/spatial-distribution diagrams, the 5 GHz channels diagram) that `pdftotext` can't pull text from. Where this content is well-established, stable, public knowledge (e.g. the 802.11a/b/g/n/ac/ax standards table), it'll be filled in from general knowledge and flagged as such; anything less certain will be left out or noted as "see slide".

## Planned structure

Same layout as T01: Key Concepts and Terminology first, then one section per part of the lecture.

1. **Key Concepts**
   - Wi-Fi = WLAN based on IEEE 802.11, links devices wirelessly within a limited local area
   - Two main frequency bands: 2.4 GHz (range/penetration) vs 5 GHz (throughput, less crowded)
   - Derived from Ethernet (802.3), but wireless medium forces different MAC approach (CSMA/CA, not CSMA/CD)
2. **Terminology**
   - WLAN, Wi-Fi, ISM band, BSS/ESS/IBSS, AP, DCF/PCF, NAV, hidden/exposed terminal, RTS/CTS, beacon
3. **802.11 Topologies (Service Sets)**
   - BSS (infrastructure, single AP), ESS (multiple APs, joint mgmt), IBSS (ad-hoc), mention MBSS/QBSS
   - ESS roaming: beacon signal, strongest AP selection (not always optimal), overlap needed for seamless roaming, different channels if overlapping
4. **MAC Sub-layer: DCF**
   - CSMA/CA + binary exponential backoff, DIFS, no central coordination
   - Hidden terminal problem (A-B-C scenario, RTS/CTS partial fix)
   - Exposed terminal problem (different scenario, causes delay not collision)
   - RTS/CTS + NAV (virtual carrier sensing)
5. **MAC Sub-layer: PCF**
   - Optional, contention-free polling, implemented by AP, higher priority than DCF, PIFS/SIFS, not for ad hoc
6. **Frame Format and Frame Types**
   - Frame Control, Duration/ID, Sequence number, FCS
   - Management / Control / Data frames
   - 4-address scheme (next/previous/final dest/original source, DS)
7. **Physical Layer / Standards Evolution**
   - a/b/g/n/ac/ax overview (bands, rough max rates) - from general knowledge, slide table itself is an image
8. **Wi-Fi in Practice**
   - ISM band sharing/interference (Zigbee, Bluetooth, microwave ovens, cordless phones, baby monitors...)
   - Dual band devices
   - 2.4 GHz channels (14 total, 11 in US / 13 in Europe, only 3-4 non-overlapping)
   - 5 GHz channels (more complex, briefly mention)
   - Tethering/hotspot
9. **Bluetooth (intro)**
   - WPAN, Bluetooth SIG, replaces cables, no LoS needed
   - (Deck ends here - likely continues in next lecture's slides)

## Things to double check / clarify

- Confirm whether "SCM" vs "RMIC" naming matters for the evaluation/course identity - for now assuming same course, different header used on this slide deck.
- The 802.11 physical standards table (slide ~20-21) is image-only; the section drafted from general knowledge should be checked against current cited source (internet-access-guide.com) if precision matters.
- 2.4 GHz/5 GHz channel diagrams are image-only; only the countable facts explicitly in the text (14 channels, 11 US/13 EU, 3-4 non-overlapping) will be kept, not exact channel numbers/spacing unless recalled reliably.

## Draft: 1. Key Concepts

A **WLAN** (Wireless Local Area Network) is a wireless data network that links two or more computing devices within a limited local area, without the cabling a traditional LAN would need. It's used anywhere a wired connection would be inconvenient or impractical: offices, factories, storage areas, homes, and public areas like schools, shopping centres and shops. **Wi-Fi** is the common name for the technology built on the IEEE 802.11 family of standards, and it's by far the most widespread way of actually deploying a WLAN.

Wi-Fi mainly operates in two frequency bands, each with a different trade-off between range and speed (see section 3 of T01 for why lower frequencies travel further and higher frequencies carry more bandwidth). The **2.4 GHz band** (roughly 2.4 to 2.4835 GHz) offers increased coverage and better penetration through solid objects, is part of the license-free ISM band, and is capped at a maximum transmitted power of 100 mW. The **5 GHz band** (roughly 5.18 to 5.7 GHz in Europe) was introduced as a way of escaping the increasingly overcrowded 2.4 GHz spectrum, trading some range for higher throughput at shorter distances; it's also license-free, but allows a much higher maximum transmitted power (1000 mW, with special power control). More recent equipment also uses **beamforming** and multiple-antenna technologies like **MIMO** (Multiple Input, Multiple Output) to further increase achievable data rates and quality of service.

IEEE 802.11 was designed for simplicity and efficiency, and is deliberately derived from the wired Ethernet standard (IEEE 802.3) so that it could reuse as much of the existing LAN model (framing, addressing, upper-layer compatibility) as possible, rather than starting from scratch. It supports a range of different bit rates and different radio solutions, which is why so many variants of the standard exist (802.11a/b/g/n/ac/ax, covered in section 7). One key difference from wired Ethernet, though, is unavoidable given the medium: Ethernet's collision detection (CSMA/CD) doesn't work over radio, since a transmitting station can't reliably listen for collisions while it's transmitting. Instead, 802.11 is based on **CSMA/CA** (Carrier Sense Multiple Access with Collision Avoidance), which tries to avoid collisions in the first place rather than detect them after the fact (covered in more detail in section 4).

## Draft: 2. Terminology

The **ISM band** (Industrial, Scientific and Medical band) is a set of radio frequency bands reserved internationally for industrial, scientific and medical use, without requiring a license. Wi-Fi's 2.4 GHz band sits inside an ISM band, which is convenient (no licensing needed) but also means devices using it have to share the spectrum with, and tolerate interference from, anything else operating in the same range — Zigbee (IEEE 802.15.4) devices, microwave ovens, Bluetooth, baby monitors, cordless telephones, amateur radio equipment, and more (see section 8).

An **Access Point (AP)** is the device that manages a WLAN and bridges it to a wired LAN or to other access points. 802.11 networks are organized into different **topologies** (also called service sets), depending on how access points are used: a single AP managing its own network (**BSS**), multiple APs under joint management (**ESS**), or no AP at all, with stations talking directly to each other (**IBSS**) — these are covered in full in section 3.

At the MAC layer, 802.11 stations coordinate access to the shared medium using **CSMA/CA** (Carrier Sense Multiple Access with Collision Avoidance), most commonly through the **DCF** (Distributed Coordination Function), a fully decentralized access method with no central coordinator. An optional, centrally-coordinated alternative exists too: the **PCF** (Point Coordination Function), implemented by the AP, which polls stations instead of leaving them to contend for the medium (section 5). Because a wireless station generally can't listen for collisions while transmitting, 802.11 also defines the **NAV** (Network Allocation Vector), a virtual carrier-sensing mechanism: a station that overhears another station reserving the medium can tell in advance that it will be busy, without having to keep physically sensing the channel itself. This ties into two classic wireless-specific problems: the **hidden terminal problem**, where a station causes a collision because it can't hear another station that's already transmitting to the same destination, and the **exposed terminal problem**, where a station needlessly delays its own transmission because it can hear a transmission that wouldn't actually have interfered with it (both detailed in section 4).

A **beacon** is a signal periodically broadcast by an access point to advertise its presence; mobile stations use it to detect and choose which AP to connect to, including while roaming between APs in the same ESS (section 3).

## Draft: 3. 802.11 Topologies (Service Sets)

802.11 networks can be organized in a few different ways, called **topologies** or **service sets**. In **Infrastructure Mode**, a **Basic Service Set (BSS)** consists of a single **Access Point (AP)** managing the WLAN and connecting it to a wired LAN; every station in the BSS communicates through that AP, even when talking to another station in the same BSS. An **Extended Service Set (ESS)** extends this to two or more BSSs (i.e. two or more APs) placed under joint management, so that stations can move around a larger area while staying connected to the same overall network. In **Ad-hoc mode**, an **Independent Basic Service Set (IBSS)** has no AP at all: stations communicate directly with each other, forming the network purely out of themselves. A couple of other, more specialized service sets also exist: the **MBSS** (Mesh Basic Service Set) and the **QBSS** (QoS Basic Service Set).

Within an ESS, a mobile station can **roam** from one BSS to another as it moves. Each AP periodically broadcasts a beacon signal, and a station picks whichever AP's beacon it hears the strongest, connecting to that one — though this isn't always the actual best choice, since "strongest signal" doesn't necessarily mean "best AP to be on" (e.g. it says nothing about how loaded that AP already is). If neighbouring BSSs' coverage areas overlap, a station can hand off from one AP to the other without any interruption to its connection; if they don't overlap, there will be a gap in coverage and the connection is interrupted while the station is in it. Two BSSs can also be deliberately set up with heavily overlapping coverage areas, purely to increase capacity in that area — but if so, their two APs need to use different channels, or they would just interfere with each other instead of adding capacity. In the original 802.11 design, it was the mobile station itself that decided which BSS/AP to connect to, which made it hard to finely manage a lot of overlapping BSSs from the network's side; later refinements to the standard improved on this.

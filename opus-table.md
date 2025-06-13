# Beat Link & Beat Link Trigger + OPUS-QUAD Support

These tables represent the absolute latest state of the project, including the latest findings of interfacing with the OPUS-QUAD.

## "Modes" Envision / High Level Methods

I envision beat-link (or maybe just Beat Link Trigger) can operate in two modes with the OPUS-QUAD. This can be thought of sort of the two high level "modes" of interfacing with the OPUS-QUAD.

| MODE                  | Pre-requisites                                                                                                                           | Stable Metadata Matching | Number of Metadata Archives Supported Simultaneously†† | Precise time sync, beat expressions, & phase-synced Ableton Link | Reliability with multiple metadata archives‡ | Notes                                                                                                                                                                                                                                                                                                                                                                                |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- | ------------------------ | ------------------------------------------------------ | ---------------------------------------------------------------- | -------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1. rekordbox lighting | OPUS-QUAD is connected to the same network, rekordbox songs have been analyzed with phrase infomration, and metadata archive is attached | ✅                       | 3                                                      | ❌                                                               | Reliable                                     | Most stable mode at the moment, phrase triggers work well in my experience, and timecode interpolation is okay-ish. This mode _may_ work for the AZ as well. However note, beat expressions nor Ableton Link phase-syncing is not supported, since beat packets are not sent back (although we have seen the AZ send back beat packets in four-deck mode? But not the OPUS forsure). |
| 2. VirtualCDJ         | OPUS-QUAD is connected to the same network, **database passphrase is set**, then metadata archive is attached                            | ✅                       | 1\*                                                    | ✅                                                               | Most reliable                                | Newest mode due to the recent findings of absolute position packets being sent back, by sending VirtualCDJ keep alive packets. However, this needs work.                                                                                                                                                                                                                             |

\* at the moment. If matching via ID + track duration works out, we can support all 3 archives simultaneously. However, depending on any findings during the implementation, it may be drastically less reliable for metadata matching.

†† It does not matter where the USB is attached on the OPUS-QUAD, its just the amount of metadata archives supported at once, or in other words, the amount of USB slots connected to the OPUS-QUAD that can be supported simultaneously.

‡ We will probably need to point out in the documentation, or even the UI of Beat Link Trigger, that **one attached metadata archive is the most reliable, in all of the modes**. This is because USB slot number detection (see below table) is esentially a workaround, and is not a 100% reliable way to detect the USB slot number. _(In a nutshell, the workarounds we use depend on matching against pieces of track metadata given back from the OPUS-QUAD--specifically phrase information (in MODE 1) and track duration (in MODE 2, planned). In the rare case that two different songs happen to have the same rekordbox ID, as well as phrase information, or track duration, it is highly probable that an incorrect match will be made, and the wrong metadata archive will be used.)_

## Lower Level Methods

Next, here is a table showing the lower level methods of interfacing with the OPUS-QUAD, with the current beat-link code.

Each method assumes there is a metadata archive attached. Note that every method requires a metadata archive to be attached, as the OPUS-QUAD has no way to interface with the rekordbox databases connected to the USB slots. Additionally for context, the USB slot number (the USB number the USB stick is plugged into) is not sent back from the OPUS-QUAD, or has not been found yet. So we have to use workarounds to detect the USB slot number.

Legend:

- 🟢 = Full support
- 🟣 = Planned for full support
- 🔴 = No support
- "VirtualRekordbox" = rekordbox lighting keep alive packet is being sent
- "VirtualCDJ" = VirtualCDJ(?)--the one dysentery sends--keep alive packet is being sent

The table below is sorted by the best to the worst method in terms of beat-link and Beat Link Trigger support. **Note that methods 3 and 4 are there for reference, as they are pretty much useless without PSSI matching or database passphrase, since we can't match metadata reliably without it.** (In a nutshell, we can't match the rekordbox IDs sent back from the OPUS-QUAD with the DeviceSQL database, since we found out that the IDs being sent back are Device Library Plus IDs, and the Device Library Plus database is encrypted. See https://github.com/Deep-Symmetry/beat-link/pull/86 for more details on how METHOD 2 was implemented.)

| METHOD                                  | Precise time sync | Phase-synced Ableton Link/beat expressions support | Metadata matching with metadata archive | USB Slot Number Detection (which makes support for multiple metadata archives possible) | Mode this method is used in | Will this work with AZ in internal four-deck mode, when Pro DJ Link setting is disabled? |
| --------------------------------------- | ----------------- | -------------------------------------------------- | --------------------------------------- | --------------------------------------------------------------------------------------- | --------------------------- | ---------------------------------------------------------------------------------------- |
| **1.** VirtualCDJ + Database Passphrase | 🟢                | 🟣†                                                | 🟢                                      | 🟣\*                                                                                    | MODE 2                      | No? AZ doesn't send absolute packets back when using internal four-deck?                 |
| **2.** VirtualRekordbox + PSSI Matching | 🔴                | 🔴§                                                | 🟢                                      | 🟢                                                                                      | MODE 1                      | Maybe? I think rekordbox lighting is supported even when using internal four-deck?       |
| **3.** VirtualRekordbox                 | 🔴                | 🔴                                                 | 🔴                                      | 🔴                                                                                      | None                        | Same reason as METHOD 2                                                                  |
| **4.** VirtualCDJ                       | 🟢                | 🔴                                                 | 🔴                                      | 🔴                                                                                      | None                        | Same reason as METHOD 1                                                                  |

\* Same reason as in the "Modes Envision" section.

† James will implement the virtual beat packets, since the OPUS-QUAD doesn't send any beat packets at all in any methods. The virtual beat packets will be simulated by using the precise time sync.

§ May work with the AZ in internal four-deck mode. It has been reported that beat packets are sent back in four-deck mode, even when the Pro DJ Link setting is disabled.

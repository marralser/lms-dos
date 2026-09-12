# LMSDOS

**A text-mode Lyrion Music Server controller for DOS PCs**

LMSDOS brings control of a modern [Lyrion Music Server](https://lyrion.org/) to vintage DOS computers.

It provides a compact 80x25 text-mode interface inspired by the LMS web interface and mobile controllers, while being designed to run on old 16-bit DOS hardware.

Networking is provided by **Michael B. Brutman's mTCP TCP/IP library for DOS**.

## Features

LMSDOS communicates directly with the Lyrion Music Server CLI interface over TCP port 9090.

Current features include:

- Automatic connection to a configured Lyrion Music Server
- Multiple LMS player support
- Remembers the last selected player
- Play / pause / stop
- Previous and next track
- Volume control
- Current track and artist display
- Playback progress bar
- Current queue display
- Random / shuffle mode
- Repeat current song
- Repeat playlist
- Favorites browser
- LMS playlists
- Music library browser
  - Songs
  - Artists
  - Genres
- Alphabetic jumping in Artists and Songs
- Music library search
- Add tracks, playlists and favorites to the current queue
- Persistent configuration in `LMSDOS.CFG`
- 80x25 text-mode user interface

The goal is to provide useful access to a modern LMS installation from machines ranging from early IBM PC compatibles upwards, without requiring HTTPS, a graphical environment, or a modern operating system.

## Screenshot

A screenshot will be added here.

## Requirements

### DOS computer

You need:

- DOS or a compatible DOS environment
- An Ethernet adapter with a DOS packet driver
- mTCP-compatible networking
- A Lyrion Music Server reachable over the local network

LMSDOS uses the LMS command-line interface, normally available on TCP port:

```text
9090
```

No HTTPS support is required.

### Lyrion Music Server

LMSDOS is designed for **Lyrion Music Server**, formerly known as Logitech Media Server / Squeezebox Server.

The LMS CLI interface must be accessible from the DOS machine.

## mTCP

Networking in LMSDOS is based on the excellent **mTCP** project by **Michael B. Brutman**.

mTCP provides a lightweight TCP/IP stack and networking applications specifically designed for DOS and old 16-bit x86 computers. LMSDOS uses the mTCP TCP/IP library directly rather than requiring a resident TCP/IP stack.

Official mTCP project:

**https://github.com/mbbrutman/mTCP**

mTCP home page:

**http://www.brutman.com/mTCP/**

Please refer to the official mTCP documentation for packet-driver setup, TCP/IP configuration, DHCP, and information about using the mTCP programming library.

**mTCP is a separate project and is not part of LMSDOS. Credit for mTCP and its TCP/IP implementation belongs to Michael B. Brutman and the mTCP project.**

## Network setup

LMSDOS expects the normal mTCP environment to be configured.

For example:

```dos
SET MTCPCFG=C:\MTCP\MTCP.CFG
```

Your Ethernet packet driver must already be loaded.

A typical startup sequence might therefore look like:

```dos
C:\NET\PKTDRV.COM 0x60
SET MTCPCFG=C:\MTCP\MTCP.CFG
C:\MTCP\DHCP.EXE
LMSDOS.EXE
```

The exact packet-driver command depends on your network adapter.

Consult the mTCP documentation for details.

## First run

On its first start, LMSDOS asks for the Lyrion Music Server address:

```text
LMS server IP/hostname:
LMS CLI port [9090]:
```

The settings are stored in:

```text
LMSDOS.CFG
```

For example:

```ini
SERVER=192.168.1.20
PORT=9090
PLAYER=00:04:20:12:34:56
```

On subsequent starts simply run:

```dos
LMSDOS
```

LMSDOS connects to the saved server automatically.

If the previously selected player is available, it is selected automatically. Otherwise LMSDOS uses the first player reported by LMS.

## Controls

The current interface displays its keyboard commands at the bottom of the screen.

### Browsing

```text
Up / Down       Select item
Page Up/Down    Previous/next page
Enter           Open or play selected item
A               Add selected item to queue
Left/Backspace  Go back
/               Search
```

In the **Artists** and **Songs** views, alphabetic keys can also be used to jump through the library.

For example:

```text
M
```

jumps to the first matching entry around the letter M without sequentially loading the entire library.

### Player

```text
Space           Play / Pause
S               Stop
B               Previous track
N               Next track
+ / -           Volume
P               Select LMS player
```

### Playback modes

```text
X               Toggle random/shuffle
0               Repeat off
1               Repeat current song
2               Repeat playlist
```

### Other

```text
Q / Esc         Quit
R               Refresh
```

## Building LMSDOS

LMSDOS is currently built using **Open Watcom C/C++ 1.9** and the mTCP source tree.

The program is cross-compiled as a 16-bit DOS executable.

A Linux machine can be used as the development/build host.

Example Open Watcom environment:

```bash
export WATCOM=$HOME/watcom19
export PATH=$WATCOM/binl:$PATH
export EDPATH=$WATCOM/eddat
export INCLUDE=$WATCOM/h
```

The project expects the mTCP source tree to be available alongside the LMSDOS directory, for example:

```text
projects/
├── LMS-DOS/
│   ├── LMSDOS.CPP
│   ├── MAKEFILE
│   ├── MTCP_WPP.OPT
│   └── LMSAPP.CFG
│
└── mtcp/
    ├── TCPINC/
    ├── INCLUDE/
    └── TCPLIB/
```

Build with:

```bash
rm -f *.OBJ *.obj *.EXE *.exe *.MAP *.map *.ERR *.err
wmake -f MAKEFILE
```

The resulting executable is:

```text
LMSDOS.EXE
```

### Building on Linux

The official mTCP source tree historically assumes a case-insensitive development environment in several places. This matters when compiling it from Linux because filenames such as `Utils.h` and `UTILS.H` are different files to Linux.

This project therefore includes:

```text
PREP_MTCP_LINUX.SH
```

Run it against the mTCP source directory if required:

```bash
sh PREP_MTCP_LINUX.SH ../mtcp
```

This creates the filename aliases required for the Open Watcom/mTCP build without modifying the LMSDOS source.

## How it works

LMSDOS does not attempt to implement the LMS web interface.

Instead it talks directly to the **Lyrion Music Server CLI**, using TCP port 9090.

Commands include operations such as:

```text
players
songs
artists
genres
playlists
favorites items
```

and player-specific commands for playback, queue management, shuffle, repeat, volume and status.

LMS extended-query responses are parsed directly by LMSDOS. Percent-encoded LMS fields are decoded locally on the DOS machine.

Large libraries are intentionally handled in small pages rather than being loaded into conventional DOS memory all at once.

Alphabetic library jumping uses indexed queries so that even libraries containing many thousands of tracks remain practical on old machines.

## Design goals

LMSDOS deliberately aims to remain simple.

The priorities are:

- compatibility with real DOS hardware
- low conventional-memory usage
- usable performance on slow CPUs
- no HTTPS requirement
- no graphical user interface
- minimal dependencies
- keyboard-driven operation
- direct communication with LMS
- an interface suitable for an 80x25 DOS display

It is not intended to reproduce every feature of the modern LMS web interface.

## Spotify / Spotty

LMS installations using the Spotty plugin may expose imported Spotify content through the normal LMS music database.

Such content may therefore appear in LMSDOS under Artists, Genres, Songs or Playlists.

Direct browsing of the complete:

```text
My Apps -> Spotty
```

menu hierarchy is not currently implemented.

This may be added in a future release.

## Status

LMSDOS is an experimental retrocomputing project and is still under development.

It has been developed specifically with real vintage DOS systems in mind, rather than DOS support being an afterthought.

Testing on different combinations of:

- CPUs
- DOS versions
- Ethernet adapters
- packet drivers
- LMS versions
- Squeezebox-compatible players

is very welcome.

## Credits

### mTCP

LMSDOS would not be practical on early DOS machines without **mTCP**.

mTCP was created by **Michael B. Brutman** and provides a highly efficient TCP/IP implementation and suite of networking applications for DOS.

Official project:

**https://github.com/mbbrutman/mTCP**

Home page:

**http://www.brutman.com/mTCP/**

Many thanks to Michael B. Brutman for developing and maintaining mTCP and for making its TCP/IP library available to other DOS networking projects.

### Lyrion Music Server

LMSDOS is an independent client for **Lyrion Music Server** and uses its CLI interface.

Lyrion Music Server is the community-maintained continuation of the server software originally developed for the Squeezebox ecosystem.

## License

License information for LMSDOS should be added here before publishing the project.

mTCP is a separate project and remains subject to its own licensing terms. Consult the mTCP distribution and official repository for the applicable mTCP license and copyright information.

## Contributing

Bug reports, testing results, documentation improvements and patches are welcome.

Testing on real vintage hardware is particularly useful. When reporting a problem, please include:

- CPU / computer model
- DOS version
- network adapter
- packet driver
- mTCP version
- Lyrion Music Server version
- player type
- description of the problem

---

**LMSDOS — because a 40-year-old PC should still be able to choose the music.**
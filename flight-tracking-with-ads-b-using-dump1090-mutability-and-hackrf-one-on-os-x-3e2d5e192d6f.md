
# Flight tracking with ADS-B using dump1090-mutability and HackRF One on OS X

dump1090 is a popular tool for receiving ADS-B (a secondary surveillance radar technology used in air traffic control) using a RTL-SDR, forks include:

* [antirez/dump1090](https://github.com/antirez/dump1090) original by Salvatore Sanfilippo, updated 2015

* [MalcolmRobb/dump1090](https://github.com/MalcolmRobb/dump1090) update Oct 31, 2014, but— 330 commits ahead

* [**mutability/dump1090](https://github.com/mutability/dump1090)** a day ago, 534 commits ahead MalcomRobb

* [SDRplay/dump1090](https://github.com/SDRplay/dump1090) ([PR](https://github.com/mutability/dump1090/issues/108)) for SDRplay dongle

* [flightaware/dump1090](https://github.com/flightaware/dump1090) [FlightAware.com](https://flightaware.com), based on mutability/dump1090

* [itemir/dump1090_sdrplus](https://github.com/itemir/dump1090_sdrplus) adds HackRF, Airspy, SDRplay support

![[A screenshot of dump1090, from @smallbone](http://lee.smallbone.com/2014/03/ads-b_via_dvb-t/)](https://cdn-images-1.medium.com/max/2598/1*8uO7wSuVvYMbm_zFtCPfPA.jpeg)*[A screenshot of dump1090, from @smallbone](http://lee.smallbone.com/2014/03/ads-b_via_dvb-t/)*

The most comprehensive version appears to be dump1090-mutability. Had no RTL-SDR dongle readily available, but I thought I’d try it anyway, so I built it from source (required a [small fix to resolve errored warnings](https://github.com/mutability/dump1090/pull/125)) and executed it with a [HackRF One](http://greatscottgadgets.com/hackrf/) attached instead:

*No supported RTLSDR devices found.*

At this time, dump1090-mutability only natively supports RTL-SDR, the $20 USB stick originally for digital TV reception, but since co-oped as a general software-defined radio peripheral, you can find cheaply on Amazon (example: [NooElec NESDR Mini USB RTL-SDR](http://www.amazon.com/NooElec-NESDR-Mini-Compatible-Packages/dp/B009U7WZCA)). There is a request for [SDRplay support](https://github.com/mutability/dump1090/issues/108) in dump1090, but I lack one of those as well; only HackRF.

Tested with FlightAware’s prebuilt dump1090 for [PiAware](https://flightaware.com/adsb/piaware/install) on a Raspberry Pi 3, enticed by the possibility of getting a free enterprise account for setting up a receiver, but same result: RTL-SDR only. No HackRF support.

### dump1090_sdrplus

**Update 2016/06/07: updated improved instructions for installing dump1090_sdrplus on OS X for HackRF — supersedes the rest of this article which is now obsolete, use this instead it works better:**

***brew tap rxseger/homebrew-hackrf
brew install dump1090-hackrf***

Searching for a solution, came across [Ilker Temir adding HackRF support to dump1090](http://hackrf-dev.greatscottgadgets.narkive.com/4mLdz4BJ/adding-hackrf-support-to-dump1090), since published as [dump1090_sdrplus](https://github.com/itemir/dump1090_sdrplus). Supporting [RTL-SDR](http://www.rtl-sdr.com), [HackRF One](https://greatscottgadgets.com/hackrf/), [Airspy](http://airspy.com), and [SDRplay](http://www.sdrplay.com), looks promising! Try to build:

*git clone [https://github.com/itemir/dump1090_sdrplus](https://github.com/itemir/dump1090_sdrplus)
cd dump1090_sdrplus
make*

I had already installed the following via [Homebrew](http://brew.sh):

*brew install librtlsdr
brew install hackrf*

so no problems there, but dump1090_sdrplus failed: *dump1090.c:52:10: fatal error: ‘libairspy/airspy.h’ file not found*, install airspy too:

*brew install airspy*

Recompile, progress, yet fails with a different error now:

*dump1090.c:53:10: fatal error: ‘mirsdrapi-rsp.h’ file not found*

This header file is for SDRplay. *brew search sdrplay*, *brew search mirsdr*, nothing. Conceivably one could install the mirsdrapi/SDRplay dependency manually, or craft Homebrew formula, or build a librtlsdr-compatible API for HackRF since so many SDR-related tools seem to use it (unlike, say, the [OsmoSDR](http://sdr.osmocom.org/trac/) abstraction layer used in GNU Radio, [gr-osmosdr](http://sdr.osmocom.org/trac/wiki/GrOsmoSDR)), or just bite the bullet and buy the $20 RTL-SDR despite having the $300 HackRF, but I took a different approach:

### dump1090-mutability + sox + hackrf_transfer

The [dump1090_sdrplus](https://github.com/itemir/dump1090_sdrplus/commit/f02a8cf544928931c2b94da57c1be67eac52908f) example instructions provide a hint on how to solve this problem. dump1090, including the latest and greatest dump1090-mutability, includes support for reading captured samples from a file, offline. RTL-SDR emits unsigned bytes, HackRF signed bytes, but these are easily converted. First install [sox](http://sox.sourceforge.net):

*brew install sox*

Then capture to a file, convert said file, and feed to dump1090:

*hackrf_transfer -r output.sb -f 1090000000 -s 2000000 -p 0 -a 0 -l 40 -g 62
brew install sox
sox -r 2000000 -c 1 output.sb output.ub
./dump1090 --file output.ub*

At last, this works! After capturing enough samples (3.1 GB in my case), for long enough, I received an ADS-B transmission and decoded it successfully.

Can we pipe the data live? hackrf_transfer requires -r filename, doesn’t accept “-” as filename convention for stdout; can specify /dev/stdout, but then interleaves with logging output. Fortunately there is a [patch to hackrf_transfer to allow piping from stdin/stdout](https://github.com/mossmann/hackrf/pull/261), I applied it (may no longer be necessary by the time you read this) and rebuilt from homebrew.

When piping, sox cannot infer the file type from the filename extension, so it has to be specified with -t sb (for signed byte) and -t ub (unsigned byte). The full command, using [dump1090-mutability](https://github.com/mutability/dump1090) (enabling networking for the map on [http://localhost:8080/](http://localhost:8080/.)), becomes:

*hackrf_transfer -r --f 1090000000 -s 2000000 -p 0 -a 0 -l 40 -g 62 | sox --rate 2000000 --channels 1 --type sb - --type ub -| ./dump1090 --ifile - --net*

After letting it run for a while, we receive live signals:

*CRC: 000000
RSSI: -5.1 dBFS
Score: 750
DF 11: All Call Reply.
 Capability : 0 (Level 1)
 ICAO Address: xxxxxx
 IID : II-00*

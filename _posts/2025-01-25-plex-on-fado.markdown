<div style="border: 1px solid black; margin: 12px; padding: 12px; background-color: #fff333;">
This is a copy of a document maintained in my fork of the **pms-docker** repo,
found @ https://github.com/kennethd/pms-docker/blob/fado/fado.md
</div>

*fado* is a headless trigkey minipc running a few services from my apartment,
this document contains notes about the setup & maintenance of the plex server
running there.

# docker image
The Plex server is running via the `plexinc/pms-docker` image, configured via
`docker-compose.fado.yml` in this directory, which is pointed to via a symlink
(note I am on branch **fado**):
```
kenneth@fado ~/git/plex-pms-docker (fado) $ ls -lh docker-compose.yml
lrwxrwxrwx 1 kenneth kenneth 23 Feb  4  2024 docker-compose.yml -> docker-compose.fado.yml
```
Allowing one to manage the server process via the usual commands:
```
kenneth@fado ~/git/plex-pms-docker (fado) $ docker-compose up
kenneth@fado ~/git/plex-pms-docker (fado) $ docker ps
CONTAINER ID   IMAGE                COMMAND          CREATED        STATUS                  PORTS     NAMES
1e0f25d31684   plexinc/pms-docker   "/init"          16 hours ago   Up 16 hours (healthy)             plex
```
etc...

# useful commands
A reference of mostly CLI tools useful for maintinaing the digital library

## ripping cds
Lots of options for ripping CDs and converting audio files, `abcde` looks
nice for being configurable and outputting directly to ogg/flac/mp3, but it
wants to install a boatload of stuff, including an MTA, which turns me off
quite a bit

### cdparanoia
My old favorite, particularly for old scratchy CDs, it only ouputs `.wav` or `.aiff`
files, which then need to be converted.  It has a batch mode which will output
one file per track, and is good at detecting your CD drive, so is usually as
simple as:
```
$ cdparanoia -B
```

### lame
`lame` is the classic mp3 encoder I favored back in the oughts, pretty
straightforward, converts the `.wav`s outputted by `cdparanoia` into `.mp3`,
allowing the setting of id3 tags, though it requires yet another step for renaming the files:
```
$ for NUM in $(seq 18); do lame --ta Migala --tl "diciembre, 3 a.m." --ty 1997 --tc "Acuarela NOIS 1002" --tn $NUM/18 track$(printf "%02d" $NUM).cdda.wav track$(printf "%02d" $NUM).mp3 ; done
```
We still have to go back and add the song titles however...

### id3v2
id3 tagger utility with support for version 2 tags

Skipping ahead to incorporate the next section into a pipeline, iterate over the
song titles and finish off the id3 tags and rename with a series of commands like this:
```
$ id3v2 -t "Isabella Afterhours" track13.mp3  && id3v2 -l track13.mp3 && lltag-rename --yes track13.mp3  && ls
```
In the old days there were services like MusicBrainz, etc that would save you
the trouble of ever typing the id3 tag values, I don't know if they still
exist, I never minded it myself.

The output from that second command in the pipeline will look something like this:
```
kenneth@fado /mnt/seagate/music/migala/diciembre,_3_a.m. $ id3v2 -l track01.mp3
id3v1 tag info for track01.mp3:
Title  : Dead Moon, Cactus & 10% Blue    Artist: Migala
Album  : diciembre, 3 a.m.               Year: 1997, Genre: Unknown (255)
Comment: Acuarela NOIS 1002              Track: 1

id3v2 tag info for track01.mp3:
TSSE (Software/Hardware and settings used for encoding): LAME 64bits version 3.100 (http://lame.sf.net)
TPE1 (Lead performer(s)/Soloist(s)): Migala
TALB (Album/Movie/Show title): diciembre, 3 a.m.
TYER (Year): 1997
COMM (Comments): ()[eng]: Acuarela NOIS 1002
TRCK (Track number/Position in set): 1/18
TLEN (Length): 81786
COMM (Comments): (ID3v1 Comment)[XXX]: Acuarela NOIS 1002
TIT2 (Title/songname/content description): Dead Moon, Cactus & 10% Blue
```

### lltag
For renaming the files based on id3 tag values, I am currently using this alias, or the various artists alt:
```
alias lltag-rename="lltag --rename %a--%A--%n--%t --rename-min --rename-sep _ --rename-regexp \"s/'//\""
alias lltag-rename-va="lltag --rename %A--%n--%a--%t --rename-min --rename-sep _ --rename-regexp \"s/'//\""
```

`lltag` does also support setting tags, id3v2 support is still marked as experimental, and default id3v1 limits fields to 30 chars:
```
$ lltag -a "Maths Balance Volumes" \
        -A "Cattle Skulls and Railroad Tracks" \
        -n4 -t "Midwestern Mountain Rumble/The Rolling Muddist Rumble"  \
        --rename %a--%A--%n--%t --rename-min \
        track_4.mp3

maths balance volumes--cattle skulls and railroad tracks--04--midwestern mountain rumble-the rolling muddist rumble.mp3:
  ARTIST=Maths Balance Volumes
  TITLE=Midwestern Mountain Rumble/The
  ALBUM=Cattle Skulls and Railroad Tra
  NUMBER=4
  COMMENT=Chocolate Monk
```

## ripping dvds
If the DVD is not encrypted, it is as simple as `cat`-ting the `.VOB` files
together and converting to another format, start by mounting the DVD as a filesystem:
```
kenneth@fado /mnt/seagate/dvd/Home Movies $ sudo mkdir -p /mnt/cddvdw
kenneth@fado /mnt/seagate/dvd/Home Movies $ sudo mount -o ro /dev/sr0 /mnt/cddvdw/
kenneth@fado /mnt/seagate/dvd/Home Movies $ ls /mnt/cddvdw/
AUDIO_TS  VIDEO_TS
```
Your drive may be something different than `/dev/sr0`, look at `dmesg`.

Examine the `VIDEO_TS` directory.  Look at the `VTS_*.VOB` files, the format
has a file size limit of 1gb, so for a DVD containing a single movie, you will
probably see a series of 1gb files called `VTS_01_0.VOB`, `VTS_01_1.VOB`, etc,
until the last one `VTS_01_n.VOB` which is less than 1gb.

If there are extras, etc on the disc they will show up as `VTS_02_*.VOB`, and so on.

Note I skip `VTS_01_0.VOB`, a much smaller file that I think is a menu screen:
```
$ cat /mnt/cddvdw/VIDEO_TS/VTS_01_{1,2,3,4}.VOB > Home\ Movies.vob
```

To convert to `.mp4`, you can now do something like:
```
$ ffmpeg -i Home\ Movies.vob -vcodec h264 -acodec mp2 Home\ Movies.mp4
```
There are a ton of options, see some terse intro examples [here](https://shyamjos.com/easily-rip-dvds-in-linux-with-ffmpeg/).

### software for encrypted dvds
For most commercial DVDs, you'll need VLC's `libdvdcss` to be installed on your system.

Example installation on debian, with a couple of related things:
```
$ sudo apt install software-properties-common
$ sudo apt-add-repository contrib
$ sudo apt update && sudo apt upgrade
$ sudo apt install libdvd-pkg
$ sudo dpkg-reconfigure libdvd-pkg
$ sudo apt install regionset libavcodec-extra
```
See this [HOWTO](https://www.cyberciti.biz/faq/installing-plugins-codecs-libdvdcss-in-debian-ubuntu-linux/) for a bit more description.

For ripping encrypted DVDs on the command line, `dvdbackup` is preferred.
```
$ sudo apt install dvdbackup libdvdread8 libdvdread-dev
```

`handbrake` is a popular package that can be used to rip & transcode dvds, I
think it has some Blu-Ray features that `ffmpeg` doesn't, but I don't own any
Blu-Ray discs, so I've never tried it.  There is a separate CLI package if you
want to try it:
```
$ sudo apt install handbrake-cli
```
It has a very windows-style interface

### create backup of encrypted dvd
Begin by examining the DVD with `dvdbackup -I` (again, your device may be
something other than `/dev/sr0`):
```
kenneth@fado /mnt/seagate/dvd/Karamazov Brothers $ dvdbackup -I -i /dev/sr0
libdvdread: Attempting to retrieve all CSS keys
libdvdread: This can take a _long_ time, please be patient
libdvdread: Get key for /VIDEO_TS/VIDEO_TS.VOB at 0x00000122
libdvdread: Elapsed time 0
libdvdread: Get key for /VIDEO_TS/VTS_01_0.VOB at 0x00000195
libdvdread: Elapsed time 0
libdvdread: Get key for /VIDEO_TS/VTS_01_1.VOB at 0x00002f8a
libdvdread: Elapsed time 0
libdvdread: Found 1 VTS's
libdvdread: Elapsed time 0
DVD-Video information of the DVD with title "Karamazov22"

File Structure DVD
VIDEO_TS/
        VIDEO_TS.IFO         12288        12.00 KiB
        VIDEO_TS.VOB         40960        40.00 KiB
        VTS_00_0.IFO         12288        12.00 KiB
        VTS_00_0.VOB         40960        40.00 KiB
        VTS_01_0.IFO        182272       178.00 KiB
        VTS_01_0.VOB      24094720        22.98 MiB
        VTS_01_1.VOB    1073739776      1024.00 MiB
        VTS_01_2.VOB    1073739776      1024.00 MiB
        VTS_01_3.VOB    1073739776      1024.00 MiB
        VTS_01_4.VOB    1073739776      1024.00 MiB
        VTS_01_5.VOB    1073739776      1024.00 MiB
        VTS_01_6.VOB    1073739776      1024.00 MiB
        VTS_01_7.VOB    1073739776      1024.00 MiB
        VTS_01_8.VOB     835135488       796.45 MiB

Main feature:
        Title set containing the main feature is 1
        The aspect ratio of the main feature is 4:3
        The main feature has 1 angle
        The main feature has 1 audio track
        The main feature has 1 subpicture channel
        The main feature has a maximum of 8 chapters in one of its titles
        The main feature has a maximum of 2 audio channels in one of its titles

Title Sets:

        Title set 1
                The aspect ratio of title set 1 is 4:3
                Title set 1 has 1 angle
                Title set 1 has 1 audio track
                Title set 1 has 1 subpicture channel

                Titles included in title set 1 are
                        Title 1:
                                Title 1 has 1 chapter
                                Title 1 has 2 audio channels
                        Title 2:
                                Title 2 has 1 chapter
                                Title 2 has 2 audio channels
                        Title 3:
                                Title 3 has 1 chapter
                                Title 3 has 2 audio channels
                        Title 4:
                                Title 4 has 8 chapters
                                Title 4 has 2 audio channels
                        Title 5:
                                Title 5 has 8 chapters
                                Title 5 has 2 audio channels
                        Title 6:
                                Title 6 has 8 chapters
                                Title 6 has 2 audio channels
                        Title 7:
                                Title 7 has 8 chapters
                                Title 7 has 2 audio channels
                        Title 8:
                                Title 8 has 8 chapters
                                Title 8 has 2 audio channels
                        Title 9:
                                Title 9 has 8 chapters
                                Title 9 has 2 audio channels
```
There are two options for creating the backup, `--feature` or `--mirror`,
`--feature` will only create backup files for the main program, `--mirror`
will also back up any extras that are included on the DVD.
```
$ dvdbackup -i /dev/sr0 --feature --progress
```
For discs with multiple films/episodes/streams there is also an option to
backup one title set at a time `--title 2` will back up the `VTS_02_*.VOB`
files, `--title 3` will do `VTS_03_*.VOB`, etc..

### rip dvd with subtitle track
First, concat all of the `.VOB` files into one big one:
```
$ cat VTS_03_{1,2,3,4,5}.VOB > Heimat\ Chapter\ 1\ -\ Fernweh.vob
```
Next, examine the file to identify what's what:
```
$ ffmpeg -analyzeduration 100M -probesize 100M -i Heimat\ Chapter\ 1\ -\ Fernweh.vob
```
This example asks `ffmpeg` to scan the first 100mb of the file, `.VOB` files
don't include a header describing the streams contained within, and if we only
look at the first few seconds (default is 5 seconds) the subtitle info will
probably not yet be present.

This reports a bunch of stuff about your current version version of `ffmpeg`,
options it was compiled with and so on, at the end there is a list of streams
found in the `.VOB` file:
```
Input #0, mpeg, from 'Heimat Chapter 1 - Fernweh.vob':
  Duration: 01:59:22.18, start: 0.280000, bitrate: 5168 kb/s
  Stream #0:0[0x1bf]: Data: dvd_nav_packet
  Stream #0:1[0x1e0]: Video: mpeg2video (Main), yuv420p(tv, top first), 720x576 [SAR 16:15 DAR 4:3], 25 fps, 25 tbr, 90k tbn
    Side data:
      cpb: bitrate max/min/avg: 9000000/0/0 buffer size: 1835008 vbv_delay: N/A
  Stream #0:2[0x80]: Audio: ac3, 48000 Hz, stereo, fltp, 224 kb/s
  Stream #0:3[0x20]: Subtitle: dvd_subtitle
```
Stream `0:0` is for dvd navigation, we don't need that, `0:1` is the movie
itself, `0:2` is audio, and `0:3` the english subtitles, we want to keep all
three of those.

For a Plex-compatible output capable of combining these three streams into a
single container, we will use a `.mkv` file format, encoding the video with
`H.264` codec and audio as `mp3`.
```
$ ffmpeg -analyzeduration 100M -probesize 100M  \
    -i Heimat\ Chapter\ 1\ -\ Fernweh.vob \
    -map 0:1 -map 0:2 -map 0:3 \
    -metadata:s:s:0 language=eng -metadata:s:s:0 title="English" \
    -codec:v libx264 -crf 21 -codec:a libmp3lame -qscale:a 2 -codec:s copy \
    -threads 16 \
    Heimat\ Chapter\ 1\ -\ Fernweh.mkv
```
Once again, we need the `-analyzeduration 100M -probesize 100M` args (and they
must come first) to instruct `ffmpeg` to scan for all the streams we want to
extract.

Alter the `threads` count to match the number of cores you have.

The other options are copied from, and summarized at, [this page](https://www.internalpointers.com/post/convert-vob-files-mkv-ffmpeg)


# files
The files are served from an external usb3 drive formatted with an ext4 fs.

For maintenance, each of the directories being served is owned by me, group `media`:
```
drwxrwsr-x 175 kenneth media 4.0K Jan 12 19:35 /mnt/seagate/lossless
drwxrwsr-x  46 kenneth media 8.0K Feb  6  2024 /mnt/seagate/movies
drwxr-sr-x 609 kenneth media  16K Jan 13 01:14 /mnt/seagate/music
drwxr-xr-x  56 kenneth media 4.0K Jan 12 23:51 /mnt/seagate/plex-audioblogs
drwxr-xr-x 255 kenneth media  12K Jan 13 19:10 /mnt/seagate/plex-live-shows
drwxr-sr-x 207 kenneth media  12K Feb  6  2024 /mnt/seagate/plex-movies
drwxr-xr-x   2 kenneth media 4.0K Jan 13 19:51 /mnt/seagate/plex-mst3k
drwxrwsr-x  32 kenneth media 4.0K Jan 24  2022 /mnt/seagate/torrents
drwxrwsr-x  45 kenneth media 4.0K Jan 13 03:12 /mnt/seagate/tv
```

## TV Shows
I don't know exactly what the requirements are to make Plex recognize newly
added files, but I finally got my `.mkv` files to show up in my collection &
have verified that the subtitles work by putting them in a directory of the
same name, and including the `s01e01` style episode numbering scheme
(previously I just had "Chapter 1", etc..):
```
kenneth@fado /mnt/seagate $ find  tv/Heimat\ \(1984\)/  -name '*.mkv' | sort
tv/Heimat (1984)/Disc 1/Heimat (1984) - s01e01 - Fernweh.mkv
tv/Heimat (1984)/Disc 2/Heimat (1984) - s01e02 - Die Mitte der Welt.mkv
tv/Heimat (1984)/Disc 2/Heimat (1984) - s01e03 - Weinacht wie noch nie.mkv
tv/Heimat (1984)/Disc 3/Heimat (1984) - s01e04 - Reichshöhenstraße.mkv
tv/Heimat (1984)/Disc 3/Heimat (1984) - s01e05 - Auf und davon und zurück.mkv
tv/Heimat (1984)/Disc 3/Heimat (1984) - s01e06 - Heimatfront.mkv
tv/Heimat (1984)/Disc 4/Heimat (1984) - s01e07 - Die Liebe der Soldaten.mkv
tv/Heimat (1984)/Disc 4/Heimat (1984) - s01e08 - Der Amerikaner.mkv
tv/Heimat (1984)/Disc 5/Heimat (1984) - s01e09 - Hermännchen.mkv
tv/Heimat (1984)/Disc 6/Heimat (1984) - s01e10 - Die stolzen Jahre.mkv
tv/Heimat (1984)/Disc 6/Heimat (1984) - s01e11 - Das Fest der Lebenden und der Toten - Herbst 1982.mkv
```

## Movies
Plex serves the `tv` files directly, but because of the naming convention for
movies, I created a separate directory dedicated to plex:
```
kenneth@fado /mnt/seagate $ ls -1 movies/  | head -n6
Andrei Tarkovsky - 1966 - Andrei Rublev
Andrei Tarkovsky - 1972 - Solaris
Andrei Tarkovsky - 1979 - Stalker
by-title
by-year
Dario Argento - 1977 - Suspira.mkv

kenneth@fado /mnt/seagate $ ls -1 plex-movies/  | head -n6
Across 110th Street (1972) {imdb-tt0068168}
A Girl in Every Port (1928) {imdb-tt0018937}
American Movie (1999) {imdb-tt0181288}
And God Said To Cain (1970) {imdb-tt0064273}
Andrei Rublev (1966) {imdb-tt0060107}
Arsenal (1929) {imdb-tt0019649}
```
Again, it seems to work best if files within the per-movie directory are named
the same as the directory itself
```
kenneth@fado /mnt/seagate $ ls -l ./plex-movies/Andrei\ Rublev\ \(1966\)\ \{imdb-tt0060107\}/
total 1434724
-rw-r--r-- 1 kenneth media 734439424 Feb  4  2024 'Andrei Rublev (1966) {imdb-tt0060107} - Part 1.avi'
-rw-r--r-- 1 kenneth media     55641 Feb  4  2024 'Andrei Rublev (1966) {imdb-tt0060107} - Part 1.srt'
-rw-r--r-- 1 kenneth media 734609408 Feb  4  2024 'Andrei Rublev (1966) {imdb-tt0060107} - Part 2.avi'
-rw-r--r-- 1 kenneth media     39751 Feb  4  2024 'Andrei Rublev (1966) {imdb-tt0060107} - Part 2.srt'
```
One problem I haven't found a solution for is the behavior of the web player
when files are split, as above.  `Part 1` plays and transitions to `Part 2`
just fine, but then it seems impossible to ever get back to `Part 1`, "Start
from the beginning" only goes back to the beginning of `Part 2`.

This is not really a supported use case, and the docs recommend joining the files:
[https://support.plex.tv/articles/naming-and-organizing-your-movie-media-files/](https://support.plex.tv/articles/naming-and-organizing-your-movie-media-files/)
It seems like it would be a tedious task to combine the subtitle files

# system

## uname -a
```
Linux fado 6.1.0-28-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.119-1 (2024-11-22) x86_64 GNU/Linux
```

## external cd/dvd rw usb drive
```
[265287.500593] usb 1-2: new high-speed USB device number 2 using xhci_hcd
[265287.653076] usb 1-2: New USB device found, idVendor=13fd, idProduct=2040, bcdDevice= 9.00
[265287.653081] usb 1-2: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[265287.653083] usb 1-2: Product: USB Mass Storage Device
[265287.653084] usb 1-2: Manufacturer: TSSTcorp
[265287.653086] usb 1-2: SerialNumber: SATAHH0000000043bd7
[265287.653552] usb-storage 1-2:1.0: USB Mass Storage device detected
[265287.653804] scsi host1: usb-storage 1-2:1.0
[265288.678687] scsi 1:0:0:0: CD-ROM            TSSTcorp CDDVDW SE-S204S  TS01 PQ: 0 ANSI: 0
[265288.680071] sr 1:0:0:0: Power-on or device reset occurred
[265288.682614] sr 1:0:0:0: [sr0] scsi3-mmc drive: 125x/125x writer dvd-ram cd/rw xa/form2 cdda tray
[265288.682622] cdrom: Uniform CD-ROM driver Revision: 3.20
[265288.695066] sr 1:0:0:0: Attached scsi CD-ROM sr0
[265288.695268] sr 1:0:0:0: Attached scsi generic sg1 type 5
```

## lsusb
```
Bus 004 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 003 Device 003: ID 8087:0029 Intel Corp. AX200 Bluetooth
Bus 003 Device 002: ID 25a7:fa23 Areson Technology Corp 2.4G Receiver
Bus 003 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 002 Device 002: ID 0bc2:2038 Seagate RSS LLC Expansion HDD
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

## lscpu
```
Architecture:                         x86_64
CPU op-mode(s):                       32-bit, 64-bit
Address sizes:                        48 bits physical, 48 bits virtual
Byte Order:                           Little Endian
CPU(s):                               16
On-line CPU(s) list:                  0-15
Vendor ID:                            AuthenticAMD
Model name:                           AMD Ryzen 7 5800H with Radeon Graphics
CPU family:                           25
Model:                                80
Thread(s) per core:                   2
Core(s) per socket:                   8
Socket(s):                            1
Stepping:                             0
Frequency boost:                      enabled
CPU(s) scaling MHz:                   35%
CPU max MHz:                          4462.5000
CPU min MHz:                          1200.0000
BogoMIPS:                             6388.09
Flags:                                fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx mmxext fxsr_opt pdpe1gb rdtscp lm constant_tsc rep_good nopl nonstop_tsc cpuid extd_apicid aperfmperf rapl pni pclmulqdq monitor ssse3 fma cx16 sse4_1 sse4_2 movbe popcnt aes xsave avx f16c rdrand lahf_lm cmp_legacy svm extapic cr8_legacy abm sse4a misalignsse 3dnowprefetch osvw ibs skinit wdt tce topoext perfctr_core perfctr_nb bpext perfctr_llc mwaitx cpb cat_l3 cdp_l3 hw_pstate ssbd mba ibrs ibpb stibp vmmcall fsgsbase bmi1 avx2 smep bmi2 erms invpcid cqm rdt_a rdseed adx smap clflushopt clwb sha_ni xsaveopt xsavec xgetbv1 xsaves cqm_llc cqm_occup_llc cqm_mbm_total cqm_mbm_local clzero irperf xsaveerptr rdpru wbnoinvd cppc arat npt lbrv svm_lock nrip_save tsc_scale vmcb_clean flushbyasid decodeassists pausefilter pfthreshold avic v_vmsave_vmload vgif v_spec_ctrl umip pku ospke vaes vpclmulqdq rdpid overflow_recov succor smca fsrm
Virtualization:                       AMD-V
L1d cache:                            256 KiB (8 instances)
L1i cache:                            256 KiB (8 instances)
L2 cache:                             4 MiB (8 instances)
L3 cache:                             16 MiB (1 instance)
NUMA node(s):                         1
NUMA node0 CPU(s):                    0-15
Vulnerability Gather data sampling:   Not affected
Vulnerability Itlb multihit:          Not affected
Vulnerability L1tf:                   Not affected
Vulnerability Mds:                    Not affected
Vulnerability Meltdown:               Not affected
Vulnerability Mmio stale data:        Not affected
Vulnerability Reg file data sampling: Not affected
Vulnerability Retbleed:               Not affected
Vulnerability Spec rstack overflow:   Mitigation; safe RET, no microcode
Vulnerability Spec store bypass:      Mitigation; Speculative Store Bypass disabled via prctl
Vulnerability Spectre v1:             Mitigation; usercopy/swapgs barriers and __user pointer sanitization
Vulnerability Spectre v2:             Mitigation; Retpolines; IBPB conditional; IBRS_FW; STIBP always-on; RSB filling; PBRSB-eIBRS Not affected; BHI Not affected
Vulnerability Srbds:                  Not affected
Vulnerability Tsx async abort:        Not affected
```

## /proc/meminfo
```
MemTotal:       13196896 kB
MemFree:          312928 kB
MemAvailable:   11481100 kB
Buffers:          771628 kB
Cached:         10298508 kB
SwapCached:         2096 kB
Active:          4666400 kB
Inactive:        6914020 kB
Active(anon):      67412 kB
Inactive(anon):   440916 kB
Active(file):    4598988 kB
Inactive(file):  6473104 kB
Unevictable:           0 kB
Mlocked:               0 kB
SwapTotal:        999420 kB
SwapFree:         957436 kB
Zswap:                 0 kB
Zswapped:              0 kB
Dirty:                84 kB
Writeback:             0 kB
AnonPages:        489476 kB
Mapped:           176592 kB
Shmem:              1148 kB
KReclaimable:     424372 kB
Slab:            1022560 kB
SReclaimable:     424372 kB
SUnreclaim:       598188 kB
KernelStack:        8000 kB
PageTables:         5572 kB
SecPageTables:         0 kB
NFS_Unstable:          0 kB
Bounce:                0 kB
WritebackTmp:          0 kB
CommitLimit:     7597868 kB
Committed_AS:    1786812 kB
VmallocTotal:   34359738367 kB
VmallocUsed:      121552 kB
VmallocChunk:          0 kB
Percpu:            18176 kB
HardwareCorrupted:     0 kB
AnonHugePages:    202752 kB
ShmemHugePages:        0 kB
ShmemPmdMapped:        0 kB
FileHugePages:         0 kB
FilePmdMapped:         0 kB
HugePages_Total:       0
HugePages_Free:        0
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
Hugetlb:               0 kB
DirectMap4k:      740432 kB
DirectMap2M:    12812288 kB
DirectMap1G:           0 kB
```


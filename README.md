# ryzen-test
Script to reproduce randomly crashing processes under load on AMD Ryzen processors on distributions using apt-get [Debian], dnf [Fedora] or pacman [ArchLinux]. This fork uses GCC 7.4.0 instead of the original 7.1.0 because the latter was causing problem (read "failing") against glib 2.26 and newer.

Please note that I don't have a faulty unit at my disposal to test this script. If you think or know that you have a faulty unit, could you get in touch with me so we can make sure GCC 7.4.0 is able to reproduce the segfaults when the script is running on distributions using apt-get or dnf.

# News
January, 5th, 2018:
* Updating the code and documentation to clearly state the differences between this fork and the original code.
* Cherry-picked some commits from Tobbez's fork to nicely play with ISO-8601, clean some log noise and to log temperature.

July, 12nd, 2019:
* Upgraded GCC to 7.4, throws no error on my `ASUS-ROG-Strix-GL702ZC`
* It works on Ubuntu 18.04 (Linux Mint 19.1)
* For my `ASUS-ROG-Strix-GL702ZC`, I had to use the https://github.com/hirschmann/nbfc program
* If you have an `ASUS-ROG-Strix-GL702ZC`, you can use this config for high load `Asus-ROG-GL702ZC-High-Load.xml.xml` for `nbfc`
* Building OpenWrt all packages on Ryzen 1700 using 17 jobs it takes 2 hours (on an Intel 7700k it is 4.5 hours with 9 jobs)
* Without having to replace my CPU! (The cooling was needed for me that fixed it.)


# Try it
Run

> ./kill-ryzen.sh

and watch the output.

# Method
This script will download GCC sources (version 7.4.0) and build GCC in parallel loops on a compressed ramdisk.
Each loop requires several Gb of disk space (and here, RAM in this case).
Building other software packages might work just as well; the script could be easily adapted.

# Effect
Processes related to the build will eventually crash (e.g., segfault).
Processes unrelated to the build process might crash as well.
An example output of the script where the first process crashed after 153 seconds can be found in "example-output.txt".
There, the "last words" from the build process were (logged to "/mnt/ramdisk/workdir/buildloop.d/loop-*/build.log"):
> Makefile:761: recipe for target 'get_patches.lo' failed
> make[5]: *** [get_patches.lo] Segmentation fault (core dumped)

# Requirements
To successfully finish a round of 16 builds more than 16Gb of RAM are required because GCC is a large software package.
However, very often, one of the parallel build fails long before 16Gb of RAM are actually used (in the example, e.g, at about 2.2Gb total RAM usage).
You can try it, but it will fail if you run out of memory (should it if you have sufficient swap space?).
The use of a compressed ramdisk can also be switched off by replacing "USE_RAMDISK=true" with "USE_RAMDISK=false", but seems to increase the "TIME TO FAIL".
Alternatively, smaller software packages could be tried (suggestions welcome).

You can also try to run fewer loops but allow them to use more threads, e.g., 8 loops with 2 threads each.

> ./kill-ryzen.sh 8 2

However, with only 16Gb RAM, this configuration might still run out of memory.
4 loops with 4 threads each is a safe choice on machines with 16Gb RAM.

> ./kill-ryzen.sh 4 4

# Background from original developer (Suaefar)
Suaefar wrote this script to reproducibly show that there was a severe problem with his early (adopted) Ryzen processor.
First, he experienced segfaults (a few per day) while performing simulations with complex, partly undocumented code that probably only few care about [1,2].
_Compiling_ the GNU _compiler_ collection (GCC) with the GCC seemed much more suitable to demonstrate the problem (ruling out bad code) and was already proposed in an AMD community thread [3] and elsewhere.
Parallel builds with -j 1 seemed to even increase the load and probability to hit the problem, sometimes after only a few seconds.

He got a replacement from AMD for my affected Ryzen R7 1700.
It survived more than 24 accumulated hours (4*8h) of parallel GCC compilation without a single crashed process. No segfault were observed after the CPU had been replaced.

[1] https://github.com/m-r-s/fade
[2] https://github.com/HoerTech-gGmbH/openMHA
[3] https://community.amd.com/thread/215773

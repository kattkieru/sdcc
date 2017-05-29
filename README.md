GBDK3

Small Device C Compiler 3.6.6 (Revision 9917)
=============================================

This is a direct clone from the main repo:
https://sourceforge.net/p/sdcc/code/HEAD/tree/trunk/sdcc/

The CMakeLists.txt file included will bootstrap a build that *only* allows for making Gameboy games. All other targets are turned off for now.  I'm not doing anything special; I just want to make sure that I have a locked version for the Toolchain that I can include using a git submodule and a cmake subdirectory.

Please use cmake to build from this folder for the Toolchain binaries and libs to end up in the right
spot.  Because doing it out-of-source was fighting me, remember that a make clean will require going into
sdcc/ and running "make clean" from there. Also the configure step generates a lot of garbage, so even
cleaner is deleting the sdcc sub-folder and checking it out again.

There doesn't seem to be an easy way of removing the Boost dependency without replacing boost::graph and boost::tie.  Luckily they're both header-only and don't require a build. If you check out the Toolchain repo with all submodules (or check out the boost gbdk3 repo into a folder next to this one), it'll be found at build time.

Remaining issues:
* At least on macOS, the build fails midway through complaining about missing one of a few boost headers. If you run make a second time, it'll go through.  No idea why this happens.
* Do all this again for Windows, somehow. (It'd be nice to use VC2015 but I'm leaning towards one of the unix toolchains.)

~ k



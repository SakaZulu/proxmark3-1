# Standalone Modes

This contains functionality for different StandAlone modes. The fullimage will be built given the correct compiler flags used. Build targets for these files are contained in `Makefile.inc` and `Makefile.hal`

If you want to implement a new standalone mode, you need to implement the methods provided in `standalone.h`.
Have a look at the skeleton standalone mode called IceRun, in the files `lf_icerun.c lf_icerun.h`.

As it is now, you can only have one standalone mode installed at the time.  

## Implementing a standalone mode

We suggest you keep your standalone code inside the `armsrc/Standalone` folder. And that you name your files according to your standalone mode name.

The `standalone.h` states that you must have two functions implemented. 

The ModInfo function, which is your identification of your standalone mode.  This string will show when running the command `hw status` on the client.

The RunMod function, which is your "main" function when running.  You need to check for Usb commands, in order to let the pm3 client break the standalone mode.  See this basic skeleton of main function RunMod() and Modinfo() below.

````
void ModInfo(void) {
    DbpString("  LF good description of your mode - aka FooRun (your name)");
}

void RunMod(void) {
    // led show
    StandAloneMode();

    // Do you target LF or HF?
    FpgaDownloadAndGo(FPGA_BITSTREAM_LF);

    // main loop
    for (;;) {
        WDT_HIT();

        // exit from standalone mode, just send a usbcommand
        if (data_available()) break;

        // do your standalone stuff..
    }
````

## Naming your standalone mode

We suggest that you follow these guidelines:
- Use HF/LF to denote which frequency your mode is targeting.  
- Use you own github name/similar for perpetual honour to denote your mode.

sample:
 `LF_FOO`

Which indicates your mode targets LF and is called FOO.

This leads to your next step, your DEFINE name needed in Makefile.

`WITH_STANDALONE_LF_FOO`


## Update MAKEFILE.HAL

Add your mode to the `Makefile.hal` help and modes list:
```
+==========================================================+
| STANDALONE      | DESCRIPTION                            |
+==========================================================+
...
+----------------------------------------------------------+
| LF_FOO          | My foobar mode will make you coffee    |
+----------------------------------------------------------+

STANDALONE_MODES := LF_SAMYRUN LF_ICERUN LF_PROXBRUTE LF_HIDBRUTE LF_FOO
STANDALONE_MODES += HF_YOUNG HF_MATTYRUN HF_COLIN HF_BOG
```

## Update MAKEFILE.INC
Add your source code files like the following sample in the `Makefile.inc`

```
# WITH_STANDALONE_LF_ICERUN
ifneq (,$(findstring WITH_STANDALONE_LF_ICERUN,$(APP_CFLAGS)))
    SRC_STANDALONE = lf_icerun.c
endif

# WITH_STANDALONE_LF_FOO
ifneq (,$(findstring WITH_STANDALONE_LF_FOO,$(APP_CFLAGS)))
    SRC_STANDALONE = lf_foo.c
endif
```

## Adding identification string of your mode
Do please add a identification string in a function called `ModInfo` inside your source code file.
This will enable an easy way to detect on client side which standalone mode has been installed on the device.

````
void ModInfo(void) {
    DbpString("  LF good description of your mode - aka FooRun (your name)");
}
````

## Compiling your standalone mode
Once all this is done, you and others can now easily compile different standalone modes by just selecting one of the standalone modes (list in `Makefile.hal` or ) , e.g.:

- rename  Makefile.platform.sample -> Makefile.platform
- edit the "STANDALONE" row inside Makefile.platform.  You need to uncomment it and add your standalone mode name

Makefile.platform.sample
```
# If you want to use it, copy this file as Makefile.platform and adjust it to your needs
PLATFORM=PM3RDV4
#PLATFORM_EXTRAS=BTADDON
#STANDALONE=LF_SAMYRUN
```
 becomes
 
 Makefile.platform
 ```
# If you want to use it, copy this file as Makefile.platform and adjust it to your needs
PLATFORM=PM3RDV4
#PLATFORM_EXTRAS=BTADDON
STANDALONE=LF_FOO
```

Remember only one can be selected at a time for now.

The final steps is to 
- force recompilation of all code.  ```make clean```
- compile ```make -j8```
- flash your device
- connect to your device
- press button long time to trigger ledshow and enter your new standalone mode
- if connected with usb / fpc ,  you can also see debug statements from your device in standalone mode. Useful for debugging :)

When compiling you will see a header showing what configurations your project compiled with.
Make sure it says your standalone mode name.  

Happy hacking!

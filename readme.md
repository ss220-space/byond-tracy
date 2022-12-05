# Para-byond-tracy

Para-byond-tracy glues together a byond server the tracy profiler allowing you to analyze and visualize proc calls. This differs from the standard byond-tracy by the nature of it writing to flatfiles inside `data/profiler/`. While these files can be up to 30 gigabytes in size, they allow the RAM usage of DreamDaemon to remain very low at runtime, which is useful for profiling entire rounds (A profile from a full round needs >48GB of RAM just to view, let alone save at runtime).

Note that the files generated cannot be loaded straight into tracy. You must use `replay.py` to load the `.utracy` file and stream "it over the network" (localhost) into `capture.exe` as part of Tracy. You can stream straight into `Tracy.exe`, but this is not advised due to performance overhead.

The above script requires the `lz4` library. This can be installed with `python -m pip install lz4`.

A massive thanks to `mafemergency` for even making this possible. The below readme is adapted from the original repo (branch: `stream-to-file`) [https://github.com/mafemergency/byond-tracy/](https://github.com/mafemergency/byond-tracy/)

## supported byond versions

| windows  | linux    |
| -------- | -------- |
| 515.1592 | 515.1592 |
| 515.1591 | 515.1591 |
| 515.1590 | 515.1590 |
| 514.*    | 514.*    |

## supported tracy versions

`0.8.1` `0.8.2`

## usage

simply call `init` from `prof.dll` to begin collecting profile data and connect using [tracy-server](https://github.com/wolfpld/tracy/releases) `Tracy.exe`

```dm
/proc/prof_init()
    var/lib

    switch(world.system_type)
        if(MS_WINDOWS) lib = "prof.dll"
        if(UNIX) lib = "libprof.so"
        else CRASH("unsupported platform")

    var/init = call_ext(lib, "init")()
    if("0" != init) CRASH("[lib] init error: [init]")

/world/New()
    prof_init()
    . = ..()
```

## building

no build system included, simply invoke your preferred c11 compiler.
examples:

(AA recommended: If you have the MSVC++ buildchain, open `x86 Native Tools Command Prompt for VS 2022` and then cd to this repo. `cl` should be on your path inside of that CLI environment)

```console
cl.exe /nologo /std:c11 /O2 /LD /DNDEBUG prof.c ws2_32.lib /Fe:prof.dll
```

```console
clang.exe -std=c11 -m32 -shared -Ofast3 -DNDEBUG -fuse-ld=lld-link prof.c -lws2_32 -o prof.dll
```

```console
gcc -std=c11 -m32 -shared -fPIC -Ofast -s -DNDEBUG prof.c -pthread -o libprof.so
```

## remarks

byond-tracy is in its infancy and is not production ready for live servers.

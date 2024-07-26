# Para-byond-tracy

Para-byond-tracy glues together a byond server the tracy profiler allowing you to analyze and visualize proc calls. This differs from the standard byond-tracy by the nature of it writing to flatfiles inside `data/profiler/`. While these files can be up to 30 gigabytes in size, they allow the RAM usage of DreamDaemon to remain very low at runtime, which is useful for profiling entire rounds (A profile from a full round needs >48GB of RAM just to view, let alone save at runtime).

Note that the files generated cannot be loaded straight into tracy. You must use `replay.py` to load the `.utracy` file and stream "it over the network" (localhost) into `capture.exe` as part of Tracy. You can stream straight into `Tracy.exe`, but this is not advised due to performance overhead.

> The above script requires the `lz4` library with the stream addon. 
> ```bash
> PYLZ4_EXPERIMENTAL=TRUE python3 -m pip install --no-cache-dir --no-binary lz4 lz4
> ```

> **Update 2023-12-29**: You can now use [https://github.com/AffectedArc07/ParaTracyReplay](https://github.com/AffectedArc07/ParaTracyReplay) to stream the files much faster than the python script.

> **Update 2024-04-18**: You can now use [https://github.com/Dimach/rtracy](https://github.com/Dimach/rtracy) to stream the files even faster than the C# code, as well as split traces and other cool things.

A massive thanks to `mafemergency` for even making this possible. The below readme is adapted from the original repo (branch: `stream-to-file`) [https://github.com/mafemergency/byond-tracy/](https://github.com/mafemergency/byond-tracy/)

## Supported byond versions

| windows  | linux    |
| -------- | -------- |
| 515.1642 | 515.1642 |
| 515.1637 | 515.1637 |
| 515.1636 | 515.1636 |
| 515.1635 | 515.1635 |
| 515.1634 | 515.1634 |
| 515.1633 | 515.1633 |
| 515.1632 | 515.1632 |
| 515.1631 | 515.1631 |
| 515.1630 | 515.1630 |
| 515.1627 | 515.1627 |
| 515.1626 | 515.1626 |
| 515.1625 | 515.1625 |
| 515.1624 | 515.1624 |
| 515.1623 | 515.1623 |
| 515.1622 | 515.1622 |
| 515.1621 | 515.1621 |
| 515.1620 | 515.1620 |
| 515.1619 | 515.1619 |
| 515.1618 | 515.1618 |
| 515.1617 | 515.1617 |
| 515.1616 | 515.1616 |
| 515.1615 | 515.1615 |
| 515.1614 | 515.1614 |
| 515.1613 | 515.1613 |
| 515.1612 |          |
| 515.1611 | 515.1611 |
| 515.1610 | 515.1610 |
| 515.1609 | 515.1609 |
| 515.1608 | 515.1608 |
| 515.1607 | 515.1607 |
| 515.1606 | 515.1606 |
| 515.1605 | 515.1605 |
| 515.1604 | 515.1604 |
| 515.1603 | 515.1603 |
| 515.1602 | 515.1602 |
| 515.1601 | 515.1601 |
| 515.1600 | 515.1600 |
| 515.1599 | 515.1599 |
| 515.1598 | 515.1598 |
| 515.1597 | 515.1597 |
| 515.1596 | 515.1596 |
| 515.1595 | 515.1595 |
| 515.1594 | 515.1594 |
| 515.1593 | 515.1593 |
| 515.1592 | 515.1592 |
| 515.1591 | 515.1591 |
| 515.1590 | 515.1590 |
| 514.*    | 514.*    |

## Supported tracy versions

None built in, you need to reply these.

## Usage

simply call `init` from `prof.dll` to begin writing to a file inside `data/profiler`

init will return the filename it writes to upon success - the filename will always start with `.`

```dm
// In case you need to start the capture as soon as the server boots, uncomment the following lines and recompile:

// /world/New()
// 	prof_init()
// 	. = ..()

#ifndef PROF
// Default automatic PROF detection.
// On Windows, looks in the standard places for `prof.dll`.
// On Linux, looks in `.`, `$LD_LIBRARY_PATH`, and `~/.byond/bin` for either of
// `libprof.so` (preferred) or `prof` (old).

/* This comment bypasses grep checks */ /var/__prof

/proc/__detect_prof()
	if (world.system_type == UNIX)
		if (fexists("./libprof.so"))
			// No need for LD_LIBRARY_PATH badness.
			return __prof = "./libprof.so"
		else if (fexists("./prof"))
			// Old dumb filename.
			return __prof = "./prof"
		else if (fexists("[world.GetConfig("env", "HOME")]/.byond/bin/prof"))
			// Old dumb filename in `~/.byond/bin`.
			return __prof = "prof"
		else
			// It's not in the current directory, so try others
			return __prof = "libprof.so"
	else
		return __prof = "prof"

#define PROF (__prof || __detect_prof())
#endif

// Handle 515 call() -> call_ext() changes
#if DM_VERSION >= 515
#define PROF_CALL call_ext
#else
#define PROF_CALL call
#endif

GLOBAL_VAR_INIT(profiler_enabled, FALSE)

/**
 * Starts Tracy and returns a string filename whenever
 */
/proc/prof_init()
	var/init = PROF_CALL(PROF, "init")()
	if("0" != init) CRASH("[PROF] init error: [init]")
	GLOB.profiler_enabled = TRUE
	if(length(init) != 0 && init[1] == ".") // if first character is ., then it returned the output filename
		return init

/**
 * Stops Tracy
 */
/proc/prof_stop()
	if(!GLOB.profiler_enabled)
		return

	var/destroy = PROF_CALL(PROF, "destroy")()
	if("0" != destroy) CRASH("[PROF] destroy error: [destroy]")
	GLOB.profiler_enabled = FALSE

/world/New()
    var/utracy_filename = prof_init()
    . = ..()

/world/Del()
    prof_stop()
    . = ..()
```

## Building

cmake build system included, or simply invoke your preferred c11 compiler.
examples:

> AA recommended: If you have the MSVC++ buildchain, open `x86 Native Tools Command Prompt for VS 2022` and then cd to this repo. `cl` should be on your path inside of that CLI environment

> azizonkg recommended: use cmake on linux

```console
cmake --build . --config Release
```


```console
cl.exe /nologo /std:c11 /O2 /LD /DNDEBUG prof.c ws2_32.lib /Fe:prof.dll
```

```console
clang.exe -std=c11 -m32 -shared -Ofast3 -DNDEBUG -fuse-ld=lld-link prof.c -lws2_32 -o prof.dll
```

```console
gcc -D_FILE_OFFSET_BITS=64 -std=c11 -m32 -shared -fPIC -Ofast -s -DNDEBUG prof.c -pthread -o libprof.so
```

## Remarks

byond-tracy is in its infancy and is not production ready for live servers.

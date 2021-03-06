Command Line Interface for LZ4 library
============================================

Command Line Interface (CLI) can be created using the `make` command without any additional parameters.
There are also multiple targets that create different variations of CLI:
- `lz4` : default CLI, with a command line syntax close to gzip
- `lz4c` : Same as `lz4` with additional support legacy lz4 commands (incompatible with gzip)
- `lz4c32` : Same as `lz4c`, but forced to compile in 32-bits mode


#### Aggregation of parameters
CLI supports aggregation of parameters i.e. `-b1`, `-e18`, and `-i1` can be joined into `-b1e18i1`.



#### Benchmark in Command Line Interface
CLI includes in-memory compression benchmark module for lz4.
The benchmark is conducted using a given filename.
The file is read into memory.
It makes benchmark more precise as it eliminates I/O overhead.

The benchmark measures ratio, compressed size, compression and decompression speed.
One can select compression levels starting from `-b` and ending with `-e`.
The `-i` parameter selects a number of iterations used for each of tested levels.



#### Usage of Command Line Interface
The full list of commands can be obtained with `-h` or `-H` parameter:
```
Usage :
      lz4 [arg] [input] [output]

input   : a filename
          with no FILE, or when FILE is - or stdin, read standard input
Arguments :
 -1     : Fast compression (default)
 -9     : High compression
 -d     : decompression (default for .lz4 extension)
 -z     : force compression
 -f     : overwrite output without prompting
 -h/-H  : display help/long help and exit

Advanced arguments :
 -V     : display Version number and exit
 -v     : verbose mode
 -q     : suppress warnings; specify twice to suppress errors too
 -c     : force write to standard output, even if it is the console
 -t     : test compressed file integrity
 -m     : multiple input files (implies automatic output filenames)
 -l     : compress using Legacy format (Linux kernel compression)
 -B#    : Block size [4-7](default : 7)
 -BD    : Block dependency (improve compression ratio)
--no-frame-crc : disable stream checksum (default:enabled)
--content-size : compressed frame includes original size (default:not present)
--[no-]sparse  : sparse mode (default:enabled on file, disabled on stdout)
Benchmark arguments :
Benchmark arguments :
 -b#    : benchmark file(s), using # compression level (default : 1)
 -e#    : test all compression levels from -bX to # (default: 1)
 -i#    : iteration loops [1-9](default : 3), benchmark mode only
 ```

#### License

All files in this directory are licensed under GPL-v2.
See [COPYING](COPYING) for details.
The text of the license is also included at the top of each source file.

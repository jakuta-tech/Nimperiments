# Nimless
A template of sorts to build Offensive Security Tooling in Nim without the C Standard Library or the Nim Runtime. Features the bare minimum. *Should* compile as-is. Tested on Windows 10 22H2. *Should* be PIC as dependencies are manually resolved through custom `GetModuleHandle` and `GetProcAddress` functions and, by using stack strings, everything should be within the .text section. YMMV.

## Examples
The examples folder includes Work-in-Progress and Proof-of-Concept programs. Files directly under the root folder are the main program logic and are to be used in combination with the utils folder, linker script ("script.ld") and Nim configuration file. It is a playground while I explore the capabilities and limitations of going without the Nim runtime and C standard libraries. You may need to play with compiler and linker switches in the configuration file or on the command line to enable/disable various things.

- Use command line switch (or add to configuration file) `-d:MalDebug` to enable printing to the console.
- Use command line switch (or add to configuration file) `-d:mingw` to cross-compile from Linux.
- Use command line switch (or add to configuration file) `-o="EvilLsassTwin.exe"` to change the executable file name.
- Use command line switches `--skipUserCfg`, `--skipParentCfg`, and `--skipProjCfg` if compiling Nim projects in the same directory or in subdirectories as "Nimless" projects.
- Uncomment (`--l:"-Wl,-subsystem,windows"`) in the configuration file to prevent a console from opening. Not *required* but you should do this if you don't enable printing to console. They kind of go hand in hand.

Compile with `nim c filenameHere.nim`

### Local Injection with Function Pointer Execution
Self explanatory. Use with utils folder, linker script and Nim configuration file. Not tested as PIC.

### Evil Lsass Twin
The *original* Evil Lsass Twin version without any custom functionality, obfuscation, or other evasive features. It also does not attempt to duplicate existing handles to LSASS, even though that was a feature of the original. There were a few things I'd have to change to get that feature working without the Nim Runtime, so for the time being (and it may never be added here) it's out. 

Since it's here in the Nimless folder, that obviously means this version works without the Nim Runtime or C Standard Library. Tested and works as PIC. Also included is a Python script to extract the .text section and write to a new file. File names are hardcoded and will need to be changed! 

Had to do some nasty, hacky things to get this working as PIC. Code refresh incoming! 
#### Notes
First, you cannot enabling printing to the console if you intend to use this as shellcode (PIC). It's fine to enable that otherwise.

~~Second, you need to change lines 112-121 in `main.nim` to *your* server's IP address. This is a character array and it looks awful. The stack string macro was causing issues here and this was the quick and dirty way to get PIC working.~~

Second, change line 58 in `main.nim` to include *your* server's IPv4 address. This *was* a character array and it *did* look awful before. This has since been changed to a stack string which *does* still cause linker errors under normal circumstances. Instead of *actually* fixing it, I figured out you can use `--l:"-Wl,--allow-multiple-definition"` to workaround the issue. Not ideal from a developer standpoint but it works. 

Third, if and only if you want to use a different port, change line 59.

Fourth, should you want to remove another indicator, change the temporary file name defined on line 56.
#### Usage
If you've used Evil Lsass Twin before, you know how to use this version. 

Compile -> Set up a listener on your server (ex: `nc -lvnp 9001 > lsassDump.dmp`) -> Run executable on Windows target

To use as PIC:

Compile -> Change and run `pic.py` script -> Use output file as desired

The `pic.py` script will write the raw bytes of the .text section to a new file. This is your shellcode. What you do with it is up to you. 

Ideas:
1. Embed the file into a shellcode loader with a language such as with Golang's `embed` directive or Nim's `slurp` function.
2. Parse the file with a custom script, encode/encrypt (hex, base64, UUID, MAC, AES, XOR) the raw bytes, and copy/paste into file (*don't really recommend. 5KB in size*).
3. Parse the file with a custom script that adds the encoded/encrypted bytes to a shellcode loader automatically
4. Cobalt Strike `shinject` command or other similar C2 functionality

## Notes
- Some Winim converters (such as `winstrConverterStringToLPWSTR`) may cause compiler and linker errors as these attempt to reference standard library and Win32 API functions.
- Nim Arrays (and Sequences) seem to cause linker errors. Memory allocation and use of UncheckedArray type works as a workaround.

  *Update: this may not be the case. Looking into this further. Char array seems to be fine. StackString macro also uses byte arrays.*
  
  *Update 2: Issue seems to be connected to the `complexXor` function in `stackstr.nim`. Linker tends to complain about multiple defintions for it. Workaround, as described above, is telling the linker to allow this. Need to actually fix at some point though*
- Oddity: May not be able to use `emit` pragma to directly write C/C++ code.

## Credit
Everything here is **heavily** based on (and/or often unmodified from) the [Writing Nimless](https://github.com/m4ul3r/writing_nimless/tree/main) repository. 

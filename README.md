## WinAllocTracer

[Pin-based tool](https://software.intel.com/en-us/articles/pin-a-dynamic-binary-instrumentation-tool)

Potential uses of this `Pintool` include, but are not limited to:
* Log all memory (de)allocations (`RtlAllocateHeap`, `RtlReAllocateHeap`, `RtlFreeHeap`, `VirtualAllocEx`, and `VirtualFreeEx`)
* Detect `invalid allocations`
* Detect `double frees`
* Detect `memory leaks`
* Detect `use after frees`

### Status

* This is work in progress
* Proof of concept code, tested in a small number of situations

### Building 

* Built with Pin 3.13-98189/3.11-97998 & Visual Studio 2019 (`x86` and `x64`)
* If you run into problems please have a look here: https://software.intel.com/sites/default/files/managed/8e/f5/PinCRT.pdf (the pdf is mirrored [here](PinCRT.pdf))

Since I use `windows.h` in my Pintool, and Pin uses some base types that conflict with Windows types, we have to wrap the `windows.h` file as follows.

```
namespace WINDOWS
{
#include <windows.h>
}
```

Items that reside in the `windows.h` are then referenced using the `WINDOWS::` prefix. Because of this there's an extra define you'll need to add to the compilation flags:

```
/D_WINDOWS_H_PATH_="$(ORIGINAL_WINDOWS_H_PATH)"
```

This is documented in the **pdf** file mentioned above. To add the `define` above, one possible solution is:

1. open the project's properties (Right click -> `Properties`)
2. go to `C/C++` -> `Preprocessor`
3. edit `Preprocessor Definitions`
4. add: `_WINDOWS_H_PATH_=C:\Program Files (x86)\Windows Kits\10\Include\10.0.18362.0\um`

Note: Make sure you adjust the path above to your SDK version in use.

**Warning**: If you are playing with 64-bit binaries use Pin 3.11-97998. Version 3.13-98189 has some weird string conversion bug when tracing 64-bit binaries.

### Using the Pintool

```
C:\pin>pin -t source\tools\WinAllocTracer\Release\WinAllocTracer.dll -- C:\TARGET\testcase.exe
[+] Loading C:\TARGET\testcase.exe, Image id = 1
    Low Adress : 1638400, High Address : 1765375
[+] Loading C:\Windows\syswow64\KERNELBASE.dll, Image id = 2
    Low Adress : 1979580416, High Address : 1979871231
[+] Loading C:\Windows\syswow64\kernel32.dll, Image id = 3
    Low Adress : 1984888832, High Address : 1986002943
[+] Loading C:\Windows\SysWOW64\ntdll.dll, Image id = 4
    Low Adress : 2005532672, High Address : 2007105535
[+] Loading C:\Windows\SysWOW64\vcruntime140d.dll, Image id = 5
    Low Adress : 1854865408, High Address : 1854971903
[+] Loading C:\Windows\SysWOW64\ucrtbased.dll, Image id = 6
    Low Adress : 1720320000, High Address : 1721843711
[+] Loading C:\Windows\SysWOW64\api-ms-win-core-localization-l1-2-0.dll, Image id = 7
    Low Adress : 1884225536, High Address : 1884237823
[+] Loading C:\Windows\SysWOW64\api-ms-win-core-processthreads-l1-1-1.dll, Image id = 8
    Low Adress : 1879834624, High Address : 1879846911
[+] Loading C:\Windows\SysWOW64\api-ms-win-core-file-l1-2-0.dll, Image id = 9
    Low Adress : 1879769088, High Address : 1879781375
[+] Loading C:\Windows\SysWOW64\api-ms-win-core-timezone-l1-1-0.dll, Image id = 10
    Low Adress : 1884356608, High Address : 1884368895
[+] Loading C:\Windows\SysWOW64\api-ms-win-core-file-l2-1-0.dll, Image id = 11
    Low Adress : 1884291072, High Address : 1884303359
[+] Loading C:\Windows\SysWOW64\api-ms-win-core-synch-l1-2-0.dll, Image id = 12
    Low Adress : 1879900160, High Address : 1879912447
[+] Unloading image C:\TARGET\testcase.exe
[+] Unloading image C:\Windows\syswow64\KERNELBASE.dll
[+] Unloading image C:\Windows\syswow64\kernel32.dll
[+] Unloading image C:\Windows\SysWOW64\ntdll.dll
[+] Unloading image C:\Windows\SysWOW64\vcruntime140d.dll
[+] Unloading image C:\Windows\SysWOW64\ucrtbased.dll
[+] Unloading image C:\Windows\SysWOW64\api-ms-win-core-localization-l1-2-0.dll
[+] Unloading image C:\Windows\SysWOW64\api-ms-win-core-processthreads-l1-1-1.dll
[+] Unloading image C:\Windows\SysWOW64\api-ms-win-core-file-l1-2-0.dll
[+] Unloading image C:\Windows\SysWOW64\api-ms-win-core-timezone-l1-1-0.dll
[+] Unloading image C:\Windows\SysWOW64\api-ms-win-core-file-l2-1-0.dll
[+] Unloading image C:\Windows\SysWOW64\api-ms-win-core-synch-l1-2-0.dll

C:\pin>type memprofile.out
[+] Memory tracing for PID = 4532

 Image Name          : C:\TARGET\testcase.exe
 Image Load offset   : 0x1638400
 Image Low address   : 0x1638400
 Image High address  : 0x1765375
 Image Start address : 0x1638400
 Image Size mapped   : 126976
 Image Type          : 2

[+] Started tracing after 'main()' call

[*] RtlAllocateHeap(0x005b0000, 8, 32)    = 0x5cc628
[*] RtlFreeHeap(0x005b0000, 0, 0x5cc628)
[Use After Free] Chunk: 0x5cc632  Instruction: 0x1a1a55 mov al, byte ptr [edx+ecx*1]
[Use After Free] Chunk: 0x5cc63c  Instruction: 0x1a1a69 mov byte ptr [edx+ecx*1], al
[*] RtlAllocateHeap(0x005b0000, 8, 64)    = 0x5cea78
[*] RtlAllocateHeap(0x005b0000, 8, 128)   = 0x5cdd30
[*] RtlAllocateHeap(0x005b0000, 8, 256)   = 0x5cddb8
[*] RtlFreeHeap(0x005b0000, 0, 0x5cdd30)
[*] RtlAllocateHeap(0x005b0000, 8, 512)   = 0x5cdec0
[*] RtlFreeHeap(0x5cdec0) called from RtlHeapRealloc()
[*] RtlHeapReAlloc(0x005b0000, 8, 0x5cdec0, 1024)   = 0x5cdec0
[*] RtlFreeHeap(0x005b0000, 0, 0x5cdec0)
[*] VirtualAllocEx(0xffffffff, 0, 327680, 8192, 1)    = 0x2720000
[*] VirtualFreeEx(0xffffffff, 0x2720000, 0, 8000)
[Double Free] Memory at address 0x2720000 has been freed more than once (Caller IP: 0x75fef07f)
[Memory Leak] Memory at address 0x5cea78 has been allocated but not freed
[Memory Leak] Memory at address 0x5cddb8 has been allocated but not freed
```

### Debugging

To debug the `Pintool` use the `-pause_tool` switch and specify the number of seconds to wait until you attach the debugger to its process.

```
C:\pin>pin.exe -pause_tool 20 -t source\tools\MallocTracer\Release\MallocTracer.dll -- C:\TARGET\testcase.exe 
Pausing for 20 seconds to attach to process with pid 1568
```

If you want to `break` when a memory issue is found, use `-appdebug` and attach to the remote `gdb` server. A `breakpoint` will trigger everytime a memory issue is found (`double free` and `use after free` only for now actually).

```
C:\pin>pin.exe -appdebug -t source\tools\WinAllocTracer\Release\WinAllocTracer.dll -- C:\TARGET\testcase.exe
(...)
Application stopped until continued from debugger.
Pin ready to accept debugger connection on port 60094
(...)
```

### Acknowledgment

* This code is based on the sample `Pin` tools distributed as part of the `Pin` package.
* Started as a PoC for [http://deniable.org/reversing/binary-instrumentation](http://deniable.org/reversing/binary-instrumentation).

Also, lots of inspiration from:
* http://github.com/joxeankoret/membugtool
* https://github.com/JonathanSalwan/PinTools


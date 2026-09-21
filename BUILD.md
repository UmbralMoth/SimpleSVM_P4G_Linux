# Build Instructions for SimpleSvm P4G Linux Version

## Prerequisites
- Windows 10/11 SDK
- Visual Studio 2019/2022 with C++ Desktop Development workload
- WDK (Windows Driver Kit) 10.0.19041 or later

## Building SimpleSvm.sys

### Option 1: Visual Studio (Recommended)
1. Open `SimpleSvm.sln` in Visual Studio
2. Select Configuration: `Release` | Platform: `x64`
3. Build Solution
4. Output: `x64/Release/SimpleSvm.sys`

### Option 2: Command Line (MSBuild)
```cmd
msbuild SimpleSvm.sln /p:Configuration=Release /p:Platform=x64
```

### Option 3: clang-cl (if available)
```cmd
clang-cl /DPOOL_NX_OPTIN=1 /I. /I..\include /MD /O2 /GS- /Gy /Zc:strictStrings /Zc:inline /Zc:throwingNew /Zc:referenceBinding /Zc:__cplusplus /Zc:wchar_t /Zc:forScope /Zc:rvalueCast /fp:fast /EHsc /nologo /c SimpleSvm.cpp /FoSimpleSvm.obj
clang-cl /DPOOL_NX_OPTIN=1 /I. /I..\include /MD /O2 /GS- /Gy /Zc:strictStrings /Zc:inline /Zc:throwingNew /Zc:referenceBinding /Zc:__cplusplus /Zc:wchar_t /Zc:forScope /Zc:rvalueCast /fp:fast /EHsc /nologo /c x64.asm /Foasm.obj
link /DRIVER /SUBSYSTEM:NATIVE,6.00 /MACHINE:X64 /RELEASE /OUT:SimpleSvm.sys SimpleSvm.obj asm.obj /ENTRY:DriverEntry
```

## Key Changes from Original P4G Version (for Linux/Proton compatibility)

1. **REMOVED KVA Shadow Check** in `DriverEntry` (line ~2830):
   - Original: `if (!IsKVAShadowDisabled()) return STATUS_HV_FEATURE_UNAVAILABLE;`
   - This was blocking driver load on Linux/Proton

2. **P4G-Specific CPUID Spoofing** in `SvHandleCpuid`:
   - Leaf 0x1: AMD Ryzen 9 5900X features (0x00A20F10, 0x00180800, 0x7AD8320B, 0x178BFBFF)
   - Leaf 0x80000002: "AMD Ryzen 9 5900X"
   - Leaf 0x80000003: "21 X-Croper seco"
   - Leaf 0x80000004: " sro    "

3. **DR3/DR7-Based Syscall Hook** (P4G style):
   - Uses `TargetDR3 = 0x7FFE0FF0` for process detection
   - Checks `DR7 & 0xF0000040 == 0x40` for debug register setup
   - `TargetSysHandler` for syscall interception

4. **KUSER_SHARED_DATA Spoofing** with full feature disable:
   - Disables MONITORX, RDTSCP, RDPID, RDRAND, XSAVE, AVX, AVX2
   - Processor features matching P4G expectations

5. **Process Tracking via CPUID 0x336933 and 0x1337**:
   - 0x336933: Captures TargetSysHandler (syscall entry)
   - 0x1337: Captures TargetProcessId for KUSER spoofing

## Directory Structure for Deployment

```
DenuvOwO/
├── reflex.ini          # Configuration (autoload=true for Linux)
├── reflex_amd          # AMD loader stub
├── reflex_intel        # Intel loader stub
├── reflex.dll          # Main DLL
└── driver_amd/
    └── SimpleSvm.sys   # Compiled driver (place here after build)
```

## Testing on Linux/Proton

1. Build `SimpleSvm.sys` on Windows
2. Copy to `driver_amd/SimpleSvm.sys`
3. Copy entire folder to Linux game directory
4. Ensure `reflex.ini` has `autoload = true`
5. Run game through Proton

## Debugging Tips

- Enable `SV_DEBUG_BREAK()` in code for kernel debugging
- Check `SvDebugPrint` output in kernel debugger
- Monitor `CounterUpdater` thread for process tracking
- Verify CPUID spoofing with `cpuid` command in guest

## Files Modified from Pragmata Base

- `SimpleSvm.cpp`: P4G CPUID values, DR3/DR7 syscall hook, KUSER spoofing
- `x64.asm`: Syscall hook assembly (P4G style)
- `reflex.ini`: P4G target configuration

## Notes

- This uses AMD SVM (SimpleSvm) path, not Intel VT-x (HyperDbg)
- Works on AMD CPUs with nested virtualization (KVM)
- For Intel CPUs on Linux, Intel VT-x path would be needed (not included)
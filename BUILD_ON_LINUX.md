# Building SimpleSvm P4G Driver on Linux

## ⚠️ Important: Kernel Driver Build Requires Windows

**This driver cannot be fully built on Linux** because it requires:
- Windows Driver Kit (WDK) - only runs on Windows
- `ntifs.h`, `wdm.h`, `ntddk.h` - WDK-only kernel headers
- Kernel libraries (`ntoskrnl.lib`, `hal.lib`, etc.) - Windows only

## Recommended: GitHub Actions (Free)

The repository includes a GitHub Actions workflow at `.github/workflows/build.yml` that:
- Runs on Windows runners with VS 2022 + WDK preinstalled
- Builds `SimpleSvm.sys` automatically
- Uploads the driver as an artifact
- Creates releases on tags

### To use:
```bash
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/YOURUSER/SimpleSvm-P4G.git
git push -u origin main
```
Then enable Actions in the repo settings.

---

## Alternative: Windows VM

If you have a Windows VM:
1. Install **Visual Studio 2022** with "Desktop development with C++"
2. Install **Windows Driver Kit (WDK)** for VS 2022
3. Open `SimpleSvm.sln` → Build `Release|x64`
4. Output: `x64/Release/SimpleSvm.sys`

---

## Minimal Linux Compilation Test (Won't Produce Working Driver)

If you just want to test syntax/type-checking on Linux:

```bash
# Install clang with Windows target
nix-shell -p llvmPackages_18.clang llvmPackages_18.lld

# This will FAIL on ntifs.h but shows other errors
clang-cl /DPOOL_NX_OPTIN=1 \
  /I. \
  /I/home/umbralmoth/Downloads/DenuvOwO/DenuvOwO_SRC-Persona\ 4/HyperKD/include \
  /c SimpleSvm.cpp /FoSimpleSvm.obj
```

### Missing on Linux:
| Header | Source | Available on Linux? |
|--------|--------|---------------------|
| `ntifs.h` | WDK | ❌ No |
| `wdm.h` | WDK | ❌ No |
| `ntddk.h` | WDK | ❌ No |
| `intrin.h` | Compiler | ✅ clang built-in |
| `stdarg.h` | Standard | ✅ clang built-in |
| `pshpack1.h` | Windows SDK | ⚠️ Partial (mingw) |

---

## Why the WDK is Required

The driver uses kernel APIs:
```c
PsLookupProcessByProcessId
PsAcquireProcessExitSynchronization
MmGetVirtualForPhysical
IoAllocateMdl
MmProbeAndLockPages
ExAllocatePool2
KeStackAttachProcess
__readdr, __readmsr, __writemsr, __rdtsc  // Compiler intrinsics
```

These are **kernel-mode only** - no user-mode equivalent exists.

---

## Summary

| Method | Works? | Effort | Cost |
|--------|--------|--------|------|
| GitHub Actions | ✅ Yes | Low | Free |
| Windows VM | ✅ Yes | Medium | License |
| Linux native | ❌ No | High | Free |
| Wine/Proton | ❌ No | - | - |

**Use GitHub Actions** - it's the intended way to build Windows kernel drivers without a local Windows machine.
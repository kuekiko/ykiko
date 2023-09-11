---
title: "Windows反调试总结"
created_Time: 2023-09-11 16:53:00 +0000 UTC
lastmod: 2023-09-11 16:55:00 +0000 UTC
author: "ukiko"
last_edit_author: "ukiko"
draft: false
categories: [二进制安全]
description: "总结Win下常见的反调试手段"
tags: [Windows,调试]
date: 2021-08-22
---

# Windows反调试总结

## 0x00 前言

在对Windows程序进行分析时，会发现很多软件会加入反调试技术防止核心功能被破解，以及在进行恶意软件分析时也有很多样本无法直接进行调试。

本文介绍了常用的反破解和反逆向保护技术，也就是Windows平台中的反调试方法。攻和防其实是相对的，只有了解了调试的原理，才能更深入的进行对抗，

## 0x01 PEB反调试

### (1) PEB块

PEB（环境进程块）是Windows操作系统中每个进程都有的一个数据结构，它包含了进程相关的信息。PEB是在用户模式下的一个结构，与在内核模式下的EPROCESS结构相对应。PEB为进程提供了关于其自身的信息，例如加载的模块列表、启动参数、程序的基地址等。

32位程序中在`fs:[0x30]`处可以读取到PEB的指针，可以使用`*PEB`指针进行读取。

64位程序中PEB定义在`gs:[0x60]`处，但通常会有地址随机化，所以一般都是先读取TEB（**`gs:[0x30]`**）再读取PEB。

PEB中有一些字段（标志位）能够被用作检测是否被调试：

1. **BeingDebugger：**bool字段，当进程被调试时，会被设置为`true`。

	```c
	// 检测代码，常见
	PEB* peb = (PEB*)__readfsdword(0x30);
	if (peb->BeingDebugged) {
	    ExitProcess(0);
	}
	// 汇编中类似
	mov eax,dword ptr fs:[0x30]
	```



1. **NtGlobalFlag：**包含了与调试和堆相关的标志，当进程在调试器下运行时，某些标志（如**`FLG_HEAP_ENABLE_TAIL_CHECK`**、**`FLG_HEAP_ENABLE_FREE_CHECK`**和**`FLG_HEAP_VALIDATE_PARAMETERS`**）可能会被设置。该字段在32位程序中位于`PEB`的`0x68`的偏移处，在64位程序中位于0xBC偏移处。

	一般来说，在32位程序中`(NtGlobalFlag & 0x70) == True` 则说明被调试状态。

	```c
	FLG_HEAP_ENABLE_TAIL_CHECK (0x10)
	FLG_HEAP_ENABLE_FREE_CHECK (0x20)
	FLG_HEAP_VALIDATE_PARAMETERS (0x40)
	
	// 检测的汇编代码
	mov eax, fs:[30h]
	mov al, [eax+68h]
	and al, 70h
	cmp al, 70h
	je being_debugged
	```



1. **LoaderLock：**模块加载和卸载的锁。尝试在没有获取这个锁的情况下访问加载的模块列表可能会导致程序崩溃。

### (2) **IsDebuggerPresent**

除了直接读取PEB判断是否被调试以外，还可以通过使用kernel32.dll中的**IsDebuggerPresent**这个API进行判断。这个API的实际原理也是读取PEB中的**`BeingDebugged`**** **字段进行判断。

直接调用IsDebuggerPresent() 如果为返回值为True则为调试状态、如果为False则为没有被调试

`bRet `**`=`**` IsDebuggerPresent();`

## 0x02 **Nt**

### (1) **CheckRemoteDebuggerPresent**

**`CheckRemoteDebuggerPresent`** 是Windows API中的一个函数，用于检测指定的进程是否由调试器调试。

实际是调用`NtQueryInformationProcess`的`ProcessDebugPort`参数来判断的。

```c
// debugapi.h
BOOL CheckRemoteDebuggerPresent(
  HANDLE hProcess,
  PBOOL  pbDebuggerPresent
);
```

参数

- **`hProcess`**: 这是一个句柄，指向要检查的进程。如果此句柄是当前进程的句柄，函数将检查当前进程。

- **`pbDebuggerPresent`**: 这是一个指向变量的指针，该变量在函数返回时将被设置为 **`TRUE`**（如果进程正在被调试）或 **`FALSE`**（如果进程没有被调试）。

如果函数成功，返回值为非零。如果函数失败，返回值为零。

```c
//使用代码
BOOL bIsDebugged = FALSE;
CheckRemoteDebuggerPresent(GetCurrentProcess(), &bIsDebugged);
if (bIsDebugged) {
    printf("The process is being debugged.\n");
    ExitProcess(1);
}
```

### (2) **NtQuerySystemInformation**

**`NtQuerySystemInformation`**是Windows NT内核模式函数，用于查询各种系统信息。这个函数在**`ntdll.dll`**中定义，但它主要是为内部使用和驱动程序设计的。

实际是查询SystemKernelDebuggerInformation

```c
NTSTATUS NtQuerySystemInformation(
  SYSTEM_INFORMATION_CLASS SystemInformationClass,
  PVOID SystemInformation,
  ULONG SystemInformationLength,
  PULONG ReturnLength
);
```

- **`SystemInformationClass`**：一个枚举值，指定要查询的系统信息的类型。

- **`SystemInformation`**：一个指针，指向一个缓冲区，该缓冲区用于接收请求的系统信息。

- **`SystemInformationLength`**：指定**`SystemInformation`**缓冲区的大小（以字节为单位）。

- **`ReturnLength`**：一个可选的输出参数，如果提供，它将接收实际返回的信息的大小（以字节为单位）。

如果函数成功，返回状态码**`STATUS_SUCCESS`**。如果函数失败，返回一个NTSTATUS错误代码。

使用**`NtQuerySystemInformation`**** **进行反调试，有以下方法：

1. **检查进程的父进程**:

	- 使用**`SystemProcessInformation`**信息类，可以获取所有系统进程的列表。

	- 检查当前进程的父进程是否是调试器（例如：**`Visual Studio`**, **`OllyDbg`**, **`WinDbg`**等）。



1. **检查调试对象句柄**:

	- 使用**`SystemHandleInformation`**信息类，可以获取系统中所有打开的句柄。

	- 检查是否有与调试相关的句柄（例如，调试对象句柄）。



1. **检查调试端口**:

	- 使用**`SystemKernelDebuggerInformation`**信息类，可以检查系统是否有内核调试器连接。

	- 如果返回的结构中**`KernelDebuggerEnabled`**字段为**`TRUE`**，则可能存在调试器。



1. **检查系统时间**:

	- 使用**`SystemTimeOfDayInformation`**信息类，可以查询系统时间。

	- 通过比较两次查询之间的时间差，可以检测到调试器的存在，因为在单步执行或暂停执行时，时间差可能会异常地大。



1. **检查线程的上下文**:

	- 使用**`SystemThreadInformation`**信息类，可以获取线程的上下文。

	- 检查线程的上下文中的某些标志，如**`TrapFlag`**，以确定是否在单步模式下运行，这是调试的一个标志。



### (3) **NtClose**

其实就是一个`CloseHandle`、如果有调试器的情况下关闭一个无效的句柄则会触发一个异常、可以用`VEH`进行接收并处理

如果有调试器存在的话`NtClose`就会触发一个异常、则可以捕获这个异常
来判断是否被调试器调试状态

### (4) **NtQueryInformationProcess**

**`NtQueryInformationProcess`**是一个Windows Native API函数，它用于查询与指定进程相关的信息。

```c
NTSTATUS NtQueryInformationProcess(
  HANDLE           ProcessHandle,
  PROCESSINFOCLASS ProcessInformationClass,
  PVOID            ProcessInformation,
  ULONG            ProcessInformationLength,
  PULONG           ReturnLength
);
```

- **`ProcessHandle`**：要查询的进程的句柄。

- **`ProcessInformationClass`**：要查询的信息的类型。可以取值**`ProcessDebugPort`****、****`ProcessDebugObjectHandle`****、****`ProcessDebugFlags`**等。

- **`ProcessInformation`**：一个指针，指向接收查询结果的缓冲区。

- **`ProcessInformationLength`**：缓冲区的大小。

- **`ReturnLength`**：如果非NULL，它是一个指针，指向一个变量，该变量接收返回的信息的实际大小。

对于反调试，一般检查**ProcessInformationClass中的值**：

1. **检查****`DebugPort`**:

	- 使用**`ProcessDebugPort`**信息类，可以检查进程是否被调试。

	- 如果返回的值不为0，那么进程可能正在被调试。



1. **检查****`DebugFlags`**:

	- 使用**`ProcessDebugFlags`**信息类，可以检查进程的调试标志。

	- 如果返回的值为0，那么进程可能正在被调试。



1. **检查****`DebugObject`**:

	- 使用**`ProcessDebugObjectHandle`**信息类，可以检查进程是否有调试对象句柄。

	- 如果返回的句柄有效，那么进程可能正在被调试。



1. **检查父进程**:

	- 使用**`ProcessBasicInformation`**信息类，可以获取进程的基本信息，其中包括父进程的ID。

	- 检查父进程是否是调试器（例如：**`Visual Studio`**, **`OllyDbg`**, **`WinDbg`**等）。



### (5) **NtSetInformationThread**

**`NtSetInformationThread`** 是 Windows 的一个 Native API 函数，它允许开发者设置关于指定线程的各种信息。

```c
NTSTATUS NtSetInformationThread(
  HANDLE          ThreadHandle,
  THREADINFOCLASS ThreadInformationClass,
  PVOID           ThreadInformation,
  ULONG           ThreadInformationLength
);
```

- **`ThreadHandle`**：要设置信息的线程的句柄。

- **`ThreadInformationClass`**：要设置的信息的类型。常用**ThreadBasicInformation、ThreadHideFromDebugger。**

- **`ThreadInformation`**：一个指针，指向包含要设置的信息的缓冲区。

- **`ThreadInformationLength`**：缓冲区的大小。

返回**`STATUS_SUCCESS`**** 则**操作成功，其他 **`NTSTATUS`** 值则操作失败。

在反调试中使用 **`ThreadHideFromDebugger`** 信息类。当一个线程使用这个信息类调用 **`NtSetInformationThread`** 时，该线程会变得对调试器不可见。这意味着，如果一个调试器试图暂停、检查或修改这个线程，它会失败。

### (6) **NtDuplicateObject**

**`NtDuplicateObject`** 是 Windows 的一个 Native API 函数，用于复制对象句柄。这允许进程创建一个新的句柄，该句柄与原始句柄具有相同的访问权限，并指向相同的内核对象。

```c
NTSTATUS NtDuplicateObject(
  HANDLE      SourceProcessHandle,
  HANDLE      SourceHandle,
  HANDLE      TargetProcessHandle,
  PHANDLE     TargetHandle,
  ACCESS_MASK DesiredAccess,
  ULONG       HandleAttributes,
  ULONG       Options
);
```

- **`SourceProcessHandle`**：源进程的句柄，其中包含要复制的对象句柄。

- **`SourceHandle`**：要复制的对象的句柄。

- **`TargetProcessHandle`**：目标进程的句柄，其中将创建新的对象句柄。

- **`TargetHandle`**：指向新复制的对象句柄的指针。

- **`DesiredAccess`**：新句柄的请求访问权限。

- **`HandleAttributes`**：新句柄的属性。

- **`Options`**：控制复制操作的选项。

返回**`STATUS_SUCCESS`**** 则**操作成功，其他 **`NTSTATUS`** 值则操作失败。

**`NtDuplicateObject`****在内核中内核会检测是否有调试器、有调试器则发出一个异常**

### (7) **NtQueryObejct**

**`NtQueryObject`** 是 Windows 的一个 Native API 函数，用于查询系统对象的信息。这个函数提供了一种方法来获取关于系统中对象（如文件、句柄、进程、线程等）的详细信息。

```c
NTSTATUS NtQueryObject(
  HANDLE                   Handle,
  OBJECT_INFORMATION_CLASS ObjectInformationClass,
  PVOID                    ObjectInformation,
  ULONG                    ObjectInformationLength,
  PULONG                   ReturnLength
);
```

- **`Handle`**：要查询的对象的句柄。

- **`ObjectInformationClass`**：要查询的信息的类型。这是一个枚举值，可以是 **`ObjectNameInformation`**、**`ObjectTypeInformation`** 等。

- **`ObjectInformation`**：指向接收查询结果的缓冲区的指针。

- **`ObjectInformationLength`**：缓冲区的大小。

- **`ReturnLength`**：实际返回的信息的大小。

返回**`STATUS_SUCCESS`**** 则**操作成功，其他 **`NTSTATUS`** 值则操作失败。

反调试主要是使用该API查询调试器的句柄是否存在来确定是否被调试。



## 0x03 TLS反调试

TLS全称`Thread Local Storage`，即**线程局部存储**。TLS是一种**方法**，通过这种方法，给定多线程进程中的每个线程可以分配位置来**存储特定于线程的数据**。通过TLS API (TlsAlloc)支持**动态绑定(运行时)**特定于线程的数据。Win32和Microsoft c++编译器现在除了现有的API实现外，还支持**静态绑定(加载时)**每个线程数据。

在PE (Portable Executable) 文件格式中，存在一个TLS目录，其中包含指向一系列回调函数的指针。当一个线程开始或结束时，这些回调函数会被调用。更重要的是，这些回调在程序的入口点 (**`main`** 或 **`WinMain`**) 之前就会被调用。

由于TLS回调在主程序入口点之前执行，因此它们可以用作反调试技术。调试器通常在主程序入口点上设置断点，但不会考虑TLS回调。因此，如果在TLS回调中放置反调试代码，那么在主程序开始执行之前，这些代码就会被执行。

知道了TLS的原理，实现反调试只需要将其他反调试代码在TLS中实现即可。比如将**`IsDebuggerPresent`****、****`NtQueryInformationProcess`**** **等调试检测放到TLS中实现。



## 0x04 时间差反调试

调试的时候，程序运行的时间会比正常运行时间久，所以我们可以根据运行时间的长短来判断是否运行在调试环境中。

使用读取CPU时钟计数器、时间计数相关API，**时间API**QueryPerformanceCounter、GetTickCount、GetSystemTime、GetLocalTime等

反调试的实现方式为使用**GetTickCount**取启动时间，获得从系统启动到现在所有毫秒数。如果在中间进行单步调试、则时间差一定大于1000毫秒、即为调试

```c
#include <windows.h>
#include <stdio.h>

int main() {
    DWORD start, end, elapsed;
    start = GetTickCount();
    // 一些可能会被调试的代码
    for (int i = 0; i < 1000000; i++);
    end = GetTickCount();
    elapsed = end - start;
    if (elapsed > SOME_THRESHOLD) {
        printf("Debugging detected!\n");
        exit(1);
    }
    // 正常的程序代码
    return 0;
}
```

## 0x05 其他一些反调试技巧

### （1）**STARTUPINFO**

**`STARTUPINFO`** 是一个结构体，它用于指定新进程的主窗口的外观（如窗口大小和位置）和行为（如标准输入/输出句柄）。当使用 **`CreateProcess()`** 函数创建新进程时，可以通过 **`STARTUPINFO`** 结构体来指定新进程的启动参数。

程序正常启动时，大多数 **`STARTUPINFO`** 结构体的字段都会被设置为0或默认值。但是，当程序在调试器下启动时，某些字段可能会被设置为非零值，因为调试器可能会修改这些字段以控制被调试程序的行为。

```c
#include <windows.h>
#include <stdio.h>

int main() {
    STARTUPINFO si;
    GetStartupInfo(&si);
    if (si.cbReserved2 != 0 || si.lpReserved2 != NULL) {
        printf("Debugging detected!\n");
        exit(1);
    }
    printf("No debugging detected.\n");
    return 0;
}
```

### （2）**SedebugPrivilege**

**`SeDebugPrivilege`** 是一个特殊的权限，允许进程打开其他进程进行读写，即使这些进程是由其他用户创建的。这个权限通常只授予管理员和调试器，程序正常启动不会具备这个权限。

系统启动的时候会启动一个核心进程**csrss.exe**，我们可以通过判断能否使用OpenProcess打开该进程来检查当前进程是否具有调试权限。有管理员权限和调试权限才打开这个 **csrss.exe**

```c
#include <windows.h>
#include <stdio.h>

int main() {
    DWORD csrssPID = 0; // 你需要获取到 csrss.exe 的 PID
    HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, csrssPID);

    if (hProcess != NULL) {
        printf("SeDebugPrivilege detected!\n");
        CloseHandle(hProcess);
    } else {
        printf("No SeDebugPrivilege detected.\n");
    }

    return 0;
}
```

### （3）**禁止键盘输入**

在函数头部加上这个禁止键盘输入的函数、然后在函数尾部恢复这个键盘输入、函数执行非常快、所以感受不到键盘有时候被禁止输入！所以这个方法有利于反单步调试(单步单步跟着键盘就失灵了）这个可以与时间差反调试进行联合使用！

### （4）**硬件断点检测**

可以获取当前线程的上下文、当前判断当前的调试寄存器DR0\DIR1\DR2\DIR3是否有值、如果这几个调试寄存器有值说明当前这个进程正在被调试

```c
CONTEXT pContext = {0};
pContext.EFlags = CONTEXT_ALL;
if (GetThreadContext(NtCurrentThread, &pContext))
{
    if (pContext.Dr0 || pContext.Dr1 || pContext.Dr2 || pContext.Dr3)
    {
        OUTPRINTF("DR寄存器(检测硬件断点)", TRUE);
    }
}
```

### （5）**检测硬件断点的地址**

异常方式检测硬件断点

反抗硬件断点调试

HOOK之后首先把DR寄存器全部清0然后再调用VEH、所以别人用的是你的VEH

HOOK这个函数然后还原之前的硬件断点

```c
PUCHAR dwEip = (PUCHAR)ExceptionInfo->ContextRecord->Eip;
//if (*dwEip == 0xCC)
if(ExceptionInfo->ExceptionRecord->ExceptionCode == 0xC0000005)//0xCC就是不可读
{
    if (ExceptionInfo->ContextRecord->Dr0 || ExceptionInfo->ContextRecord->Dr1 || ExceptionInfo->ContextRecord->Dr2 || ExceptionInfo->ContextRecord->Dr3)
    {
        OUTPRINTF("DR寄存器(检测到的硬件地址)", TRUE);
    }
    else
    {
        OUTPRINTF("DR寄存器(检测到的硬件地址)", FALSE);
    }
    ExceptionInfo->ContextRecord->Eip += 3;
    return EXCEPTION_CONTINUE_EXECUTION;
}
return EXCEPTION_CONTINUE_SEARCH;
```

### （6）**自内存CRC**

对抗CRC:下内存硬件断点、然后一步一步跟踪、Nop掉CRC即可

自内存CRC需要很早时期先计算一遍内存CRC校验和！然后后续在根据这个CRC校验值再来判断

```c
using namespace std;
typedef struct _CRC_HASHI
{
    PVOID m_pAddr;
    DWORD m_dwSize;
    DWORD m_dwHashVal;
}CRC_HASHI,*PCRC_HASHI;
vector<CRC_HASHI> g_crc_vtr;
```

反附加之前要先获取一次代码段的页面CRC校验和算出CRC、比如某些壳在链接的时候就已经算好了、这是最早的计算方式、越早越好

以下是代码、获取页面CRC

```c
HMODULE ImageBase = 0;
ImageBase = GetModuleHandle(NULL);
 
PIMAGE_DOS_HEADER pDos = NULL;
PIMAGE_NT_HEADERS pNt = NULL;
PIMAGE_SECTION_HEADER pSection = NULL;
DWORD dwStartAdr, dwSize;
 
pDos = PIMAGE_DOS_HEADER((ULONG_PTR)ImageBase);
if (pDos->e_magic != IMAGE_DOS_SIGNATURE)
{
    return;
}
pNt = PIMAGE_NT_HEADERS((ULONG_PTR)ImageBase + pDos->e_lfanew);
if (pNt->Signature != IMAGE_NT_SIGNATURE)
{
    return;
}
pSection = IMAGE_FIRST_SECTION(pNt);
g_crc_vtr.clear();
 
for (size_t i = 0; i < pNt->FileHeader.NumberOfSections; i++)
{
    if (pSection->Characteristics & IMAGE_SCN_MEM_EXECUTE)
    {
        CRC_HASHI ctx;
        //这里计算CRC值
        dwStartAdr = pSection->VirtualAddress + (ULONG_PTR)ImageBase;
        dwSize = pSection->Misc.VirtualSize;
        ctx.m_pAddr = (PVOID)dwStartAdr;
        ctx.m_dwSize = dwSize;
        ctx.m_dwHashVal = crc32((const void*)ctx.m_pAddr, dwSize);
        g_crc_vtr.push_back(ctx);
    }
    pSection++;
}
```

循环校验CRC

```c
PVOID pDbg =    GetProcAddress(GetModuleHandle(L"ntdll.dll"), "DbgBreakPoint");
byte bRet = 0xC3;
WriteProcessMemory(NtCurrentProcess, pDbg, &bRet, 1, 0);
```

### （7）**DbgBreakPoint**

调试器在附加的时候会走DbgBreakPoint函数、所以HOOK这个函数就可以改变调试器运转流程、从而达到反附加！

```c
PVOID pDbg = GetProcAddress(GetModuleHandle(L"ntdll.dll"), "DbgBreakPoint");
byte bRet = 0xC3;
WriteProcessMemory(NtCurrentProcess, pDbg, &bRet, 1, 0);
```

### （8）**注册表检测**

当程序利用调试器时，程序的注册表中的 JIT 值会被修改我们可以检查注册表里面是否有对应字符串。

解决 把 对应的字符串 修改为 0

### （9）**窗口检测**

`FindWindow`、`EnumWindows` 这两个函数 可以得到窗口的句柄。 判断窗口名称是否时对应的 字符串。

解决 把 对应的字符串 修改为 0

### （10）**父进程检测**

正常启动 父进程为 **exeplorer.exe 。**调试启动 父进程为 调试器。

利用 `NtQueryInformationProcess` 获得父进程的 PID

可能只检查程序的父进程名字，可以把调试器名字改为 **exeplorer.exe**

### （11）异常处理

正常运行的进程发生异常时，在SEH(Structured Exception Handling)机制的作用下，OS会接收异常，然后调用进程中注册的SEH处理。但是，若进程正被调试器调试，那么调试器就会先于SEH接收处理。利用该特征可判断进程是正常运行还是调试运行，然后根据不同的结果执行不同的操作，这就是利用异常处理机制不同的反调试原理。

1. **INT3 异常 EXCEPTION_BREAKPOINT**

	正常运行状态，则自动调用已经注册过的SEH; 若程序处于调试运行状态，则系统会停止运行程序将控制权转给调试器。



1. **SetUnhandledExceptionFilter()**

	该函数会检查进程是否处于调试状态，若是，就把异常传递给调试器，否则就弹个错误对话框，然后结束程序



1. **INT 2D**

	原为内核模式中用来触发断点异常的，也可以在用户模式下正常运行时触发异常.

	① 不会触发异常，只是忽略

	② INT 2D的下一条指令的第一个字节会被忽略。

	③ F7/F8单步命令跟踪INT 2D时，程序不会停在下条指令开始的地方，而是一直运行，直到遇到断点。



### （12）0xCC探测 __ 对应 INT 3

 若是关键位置检测到该指令，即可判断进程处于调试状态。检测时要注意不是所有的位置都可以，因为0xCC既可以是INT 3指令，也可以是其他指令的操作数`API断点` `校验和` → 采用比较特殊代码区域(易被下断点的区域)的校验和的值

### （13）单步检测

检测TF或0xCC实现反调试。

`TF检测` → 当EFLAGS的TF标志位被置1时，CPU将进入单步执行模式CPU执行1条指令后即触发一个EXCEPTION_SINGLE_STEP异常，然后TF会自动清零 第1种：主动触发TF异常、与SEH结合使用探测调试器。

### （14） 自调试

和linux 一样程序不允许同时被两个调试器调试。可以自己先调试运行自己，防止被另一个调试器继续调试。

1. **CreateProcess**

	进程第1次运行时会尝试访问同步内核对象，如果不存在，则说明当前进程第1次运行，创建一个内核对象，并以调试方式创建进程打开“自己”。这时若调试器首次调试运行进程则相当于在调试一个调试器，由于第2次运行的进程是被第1次运行的调试打开的，所以调试器也无法继续调试第2次运行的进程。



1. **DebugActiveProcess**

	自调试除上节讲的CreateProcess()以调试方式打开进程外，还可以选择正常创建自身，然后马上附加创建进程的操作来实现。DebugActiveProcess()就可以做到这一点。

	



## 参考文档

- [[原创]Windows最全反调试知识汇总-附实现代码-软件逆向-看雪-安全社区|安全招聘|kanxue.com](https://bbs.kanxue.com/thread-262200.htm#msg_header_h1_6)

- [【原创】反调试实战系列二 TLS反调试+CheckRemoteDebuggerPresent原理 - 『软件调试区』 - 吾爱破解 - LCG - LSG |安卓破解|病毒分析|www.52pojie.cn](https://www.52pojie.cn/thread-1490663-1-1.html)

- [Windows平台常见反调试技术梳理（上）-安全客 - 安全资讯平台 (anquanke.com)](https://www.anquanke.com/post/id/179709#h3-3)

- [Windows平台常见反调试技术梳理（下）-安全客 - 安全资讯平台 (anquanke.com)](https://www.anquanke.com/post/id/179710#h3-5)


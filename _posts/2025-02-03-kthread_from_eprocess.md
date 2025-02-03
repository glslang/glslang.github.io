---
layout: default
title: "Getting the address of a KTHREAD from EPROCESS using WinDbg"
date: 2025-02-03 20:00:00 -0000
categories: posts
---

# Getting the address of a KTHREAD from EPROCESS using WinDbg

This post demonstrates how to get the address of a KTHREAD from an EPROCESS structure using WinDbg.

If an attacker has an arbitrary read primitive, these steps can be replicated in an exploit.

The offsets in this post are for Windows 10, version 20H2.

## Steps

WinDbg can provide this information using the `!process` command.

```
kd> !process 0 2 exploit.exe
PROCESS ffff9003e19ae080
    SessionId: 1  Cid: 2010    Peb: daaa0d9000  ParentCid: 099c
    DirBase: 52318000  ObjectTable: ffffda0acaaca940  HandleCount:  41.
    Image: exploit.exe

        THREAD ffff9003df611080  Cid 2010.0e68  Teb: 000000daaa0da000 Win32Thread: 0000000000000000 WAIT: (Executive) KernelMode Alertable
            ffff9003e3f32658  NotificationEvent
```

The ETHREAD is at 0xffff9003df611080.

To manually obtain this value, get the address of the EPROCESS structure and read the `ThreadListHead` field.

```
kd> dt nt!_EPROCESS ffff9003e19ae080 -y ThreadListHead
   +0x5e0 ThreadListHead : _LIST_ENTRY [ 0xffff9003`df611568 - 0xffff9003`df611568 ]
```

The `ThreadListHead` field is a pointer to a LIST_ENTRY structure.

From the above there's only one thread associated with the process.

```
kd> dt nt!_LIST_ENTRY ffff9003e19ae080+5e0
 [ 0xffff9003`df611568 - 0xffff9003`df611568 ]
   +0x000 Flink            : 0xffff9003`df611568 _LIST_ENTRY [ 0xffff9003`e19ae660 - 0xffff9003`e19ae660 ]
   +0x008 Blink            : 0xffff9003`df611568 _LIST_ENTRY [ 0xffff9003`e19ae660 - 0xffff9003`e19ae660 ]
```

Let's dump the ETHREAD structure,

```
kd> dt nt!_ETHREAD ffff9003df611080 -y ThreadListEntry
   +0x4e8 ThreadListEntry : _LIST_ENTRY [ 0xffff9003`e19ae660 - 0xffff9003`e19ae660 ]
```

At offset 0x4e8 is the `ThreadListEntry` field.

```
kd> dt nt!_LIST_ENTRY ffff9003df611080+4e8
 [ 0xffff9003`e19ae660 - 0xffff9003`e19ae660 ]
   +0x000 Flink            : 0xffff9003`e19ae660 _LIST_ENTRY [ 0xffff9003`df611568 - 0xffff9003`df611568 ]
   +0x008 Blink            : 0xffff9003`e19ae660 _LIST_ENTRY [ 0xffff9003`df611568 - 0xffff9003`df611568 ]
```

Going back to the EPROCESS structure, and subtracting the offset of the `ThreadListEntry` field, from the dereferencing of the `Flink` field, gives the address of the ETHREAD structure.

```
kd> dt nt!_LIST_ENTRY ffff9003e19ae080+5e0
 [ 0xffff9003`df611568 - 0xffff9003`df611568 ]
   +0x000 Flink            : 0xffff9003`df611568 _LIST_ENTRY [ 0xffff9003`e19ae660 - 0xffff9003`e19ae660 ]
   +0x008 Blink            : 0xffff9003`df611568 _LIST_ENTRY [ 0xffff9003`e19ae660 - 0xffff9003`e19ae660 ]
kd> ? 0xffff9003`df611568-4e8
Evaluate expression: -123128669728640 = ffff9003`df611080
```

Which is the ETHREAD address returned by WinDbg's process command. The KTHREAD matches the address since it is the first field in the ETHREAD structure.

# Conclusion

This post documents the steps to get the address of a KTHREAD from an EPROCESS structure using WinDbg.

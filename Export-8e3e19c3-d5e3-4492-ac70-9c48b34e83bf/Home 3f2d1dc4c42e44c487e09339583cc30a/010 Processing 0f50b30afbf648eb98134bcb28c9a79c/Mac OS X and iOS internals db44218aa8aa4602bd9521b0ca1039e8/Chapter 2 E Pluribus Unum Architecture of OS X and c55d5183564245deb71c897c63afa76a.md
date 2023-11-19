# Chapter 2 E Pluribus Unum: Architecture of OS X and iOS

# OS X Architectural Overview

![Untitled](Chapter%202%20E%20Pluribus%20Unum%20Architecture%20of%20OS%20X%20and%20c55d5183564245deb71c897c63afa76a/Untitled.png)

# The user Experience Layer

## Aqua

The system's first user-mode process, launchd (which is covered in great depth in Chapter 6) is responsible for starting the GUI. 

## Quicklook

**Bundle**:For now, suffice it to consider a bundle as a directory hierarchy conforming to a fixed structure.

**UTI**: Uniform Type Identifier, which is essentially just reverse DNS notation.

The idea of using the same namespace for software originated with Java. To prevent namespace conflict, **Sun (now Oracle)** noted that DNS can be used — albeit in reverse — to provide a hierarchy that closely resembles a file system.

## Spotlight

The brain behind spotlight is an indexing server, mds, located in the MetaData framework, which is part of the system's core services.(/`System/Library/Frameworks/CoreServices.framework/Frameworks/Metadata.framework/Support/mds)`. This is a daemon with no GUI. 

Every time a file operation occurs — creation, modification, or deletion — the kernel notifies this daemon. This notification mechanism, called `fsevents`, 

Another little documented feature is controlling Spotlight (particularly, mds) by creating files in various paths: For example, creating a .metadata_never_index hidden file in a directory will prevent its indexing (originally designed for removable media).

# Darwin - The Unix Core

Leopard (10.5) was the first version of OS X to be UNIX-certified. 

## The Shell

Either telnet or SSH can be easily enabled on OS X by editing the appropriate property list file (telnet.plist, or ssh.plist) in /System/Library/LaunchDaemons. Simply set the Disabled key to false, (or remove it altogether). 

# UNIX System Directories

- **/bin:** Unix binaries. This is where the common UNIX commands (for example, ls, rm, mv, df) are.
- /**sbin**: System binaries. These are binaries used for system administration, such as file-system management, network configuration, and so on.
- /**usr**: The User directory. This is not meant for users, but is more like Windows' program files in that third-party software can install here.
- /**usr**: Contains in it bin, sbin, and lib. /usr/lib is used for shared objects (think, Windows DLLs and \windows\system32). This directory also contains the include/ subdirectory, where all the standard C headers are.
- /**etc**: Et Cetera. A directory containing most of the system configuration files; for example, the password file (/etc/passwd). In OS X, this is a symbolic link to /private/etc.
- /**dev**: BSD device files. These are special files that represent hardware devices on the system (character and block devices).
- /**tmp**: Temporary directory. The only directory in the system that is world-writable (permissions: rwxrwxrwx). In OS X, this is a symbolic link to /private/tmp.
- /**var**: Various. A directory for log files, mail store, print spool, and other data. In OS X, this is a symbolic link to /private/var.

## OS X-Specific Directories

- /**Applications**: Default base for all applications in system.
- /**Developer:If** XCode is installed, the default installation point for all developer tools.
- /**Library**: Data files, help, documentation, and so on for system applications.
- /**Network**: Virtual directory for neighbor node discovery and access.
- /**System**: Used for System files. It contains only a Library subdirectory, but this directory holds virtually every major component of the system, such as frameworks (/System/Library/Frameworks), kernel modules (/System/Library/Extensions), fonts, and so on.
- /**Users**: Home directory for users. Every user has his or her own directory created here.
- /**Volumes**: Mount point for removable media and network file systems.
- /**Cores**: Directory for core dumps, if enabled. Core dumps are created when a process crashes, if the ulimit(1) command allows it, and contain the core virtual memory image of the process. Core dumps are discussed in detail in Chapter 4, “Process Debugging.

## iOS File System Idiosyncrasies

- /**Applications** may be a symbolic link to /var/stash/Applications. This is a feature of the jailbreak, not of iOS.
- There is no /Users, but a /User — which is a symbolic link to /var/mobile
- There is no /Volumes (and no need for it,  or for disk arbitration, as iOS doesn't have any way to add more storage to a given system)
- /**Developer** is populated only if the i-Device is selected as “Use for development” from within XCode. In those cases, the DeveloperDiskImage.dmg included in the iOS SDK is mounted onto the device.

# Interlude: Bundles

<aside>
💡 The term “bundle” is actually used to describe two different terms in Mac OS: The first is the directory structure described in this section (also sometimes called “package”). The second is a file object format of a shared-library object which has to be explicitly loaded by the process (as opposed to normal libraries, which are implicitly loaded). This is also sometimes referred to as a plug-in.

</aside>

Apple defines bundles as “a standardized hierarchical structure that holds executable code and the resources used by that code.”

![Untitled](Chapter%202%20E%20Pluribus%20Unum%20Architecture%20of%20OS%20X%20and%20c55d5183564245deb71c897c63afa76a/Untitled%201.png)

# Applications and Apps

## iOS上原生和安装的区别：

Additionally, iOS distinguishes between the default applications provided by Apple, which reside in /Applications (or /var/stash/Applications in older jailbreak-versions of iOS), and App Store purchased ones, which are in /var/mobile/Applications. 

When downloaded from the App Store (or elsewhere), applications are packaged as an .ipa file. this is really nothing more than a zip file (and may be opened with unzip(1))

## info.plist

The Info.plist file, which resides in the Contents/ subdirectory of Applications (and of most other bundles), holds the bundle's metadata. 

Property lists are stored in one of three formats:

- XML
- Binary
- JSON

## CodeResources

“The last important file an application contains is CodeResources, which is a symbolic link to _CodeSignature/CodeResources.”

它的作用是可以帮助判断这个App是不是完整的或者被修改了

### Application default settings

“A system administrator or power user can access and manipulate defaults using the defaults(1) command — a generally preferable approach to direct editing of the plist files.”

### Launching Default Applications

# Frameworks

是基于系统Darwin之上仅用于Apple的组件库，来封装系统底层调用

## Framework Bundle Format

框架也是Bundle，和应用类似，但是主要的区别是框架会存在多个版本，以适应兼容性

## Top Level Frameworks

### Carbon

Carbon is the name given to the OS 9 legacy programming interfaces. Carbon has been declared deprecated, though many applications, including Apple's own, still rely on it. Even though many of its interfaces are specifically geared for OS 9 compatibility, many new interfaces have been added into it, and it shows no sign of disappearing.

### Cocoa

Cocoa is the preferred application programming environment. It is the modern day incarnation of the NeXTSTEP environment, as is evident by the prefix of many of its base classes — **NS**, short for NeXTSTEP/Sun. The preferred language for programming with Cocoa is Objective C, although it can be accessed from Java and AppleScript as well.

# Libraries

大部分Libraries都是一个symbol link

Libraries是Framworks下面的层级

# Other Application Type

“The Application and App bundles discussed so far aren't the only types of applications that can be created. OS X (and, to a degree iOS) supports several other types of Applications as well.”

## Java（OS X only）

## Widgets

Widgets就是一个个HTML/Javascript小页面？

## BSD/Mach Native

Though the preferred language for both iOS and OS X is Objective-C, native applications may be coded in C/C++, and may choose to forego frameworks, working directly with the system libraries and the low-level interfaces of BSD and Mach instead.

这样可以直接使用UNIX版本的开源代码，编译成类似于Linux的RPM/APT类型的命令程序

# System Calls

应用不能直接访问系统资源，必须通过System Call

OS  X的system call有两种类型

## POSIX

Leopard开始OS X和POSIX完全兼容，但是怎么又有部分内容不兼容呢？

> “**Portable Operating System Interface**, more commonly known as POSIX”
> 
- **System call prototypes**: All POSIX system calls, regardless of underlying implementation, have the same prototype .This ensures that POSIX-compatible code can be ported — at the source level — between any POSIX compatible operating system.
- **System call numbers:** The key POSIX functions, in addition to the fixed prototype, have well-defined system call numbers. OS X does not support this, however, because its object format, Mach-O, is incompatible with ELF. What's more, its system call numbers deviate from those of the standard.

## Mach System Calls

# A High-Level View of XNU

The core of Darwin, and of all of OS X, is its Kernel, XNU. XNU  is itself made up of several components:

- The Mach microkernel
- The BSD layer
- libKern
- I/O Kit

## Mach

Mach **microkernel,** which handles only the most primitive responsibilities of the operating system: 微内核的作用

- Process and thread abstractions
- Virtual memory management
- Task scheduling
- Interprocess communication and messaging

Any additional functionality, such as file and device access, has to be implemented on top of it — and that is exactly what the BSD layer does.

## The BSD Layer

The BSD layer provides higher-level abstractions, including, among others:

- The UNIX Process model
- The POSIX threading model (Pthread) and its related synchronization primitives
- UNIX Users and Groups
- The Network stack (BSD Socket API)
- File system access
- Device access (through the /dev directory)

## libkern

XNU和其它内核不一样，I/O kit，也就是设备驱动，是通过C++来实现，所以为了支持C++包含了一个libkern库

## I/O Kit：利用C++编程又快又好 ：）

Apple's most important modification to XNU was the introduction of the I/O Kit device-driver framework. This is a complete, self-contained execution environment in the kernel, which enables developers to quickly create device drivers that are both elegant and stable. It achieves that by establishing a restricted **C++** environment (of libkern), with the most important functionality offered by the language — **inheritance and overloading.**

可以继承和重载，所以可以迅速开发

另外就是可以实现面向对象编程环境
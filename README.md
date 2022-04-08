# Setup debugserver
How to install debugserver for iOS from macOS

# Requirements
- macOS
- homebrew
- hdiutil
- Xcode
- Command Line Tools for Xcode
- iproxy (usbmuxd)

# Install Xcode 

[Xcode 12.4 for macOS Catalina (login to download)](https://download.developer.apple.com/Developer_Tools/Xcode_12.4/Xcode_12.4.xip)

You can download older versions of Xcode and the Command Line Tools from Apples developer site. [Xcode Downloadable List](https://developer.apple.com/download/all/?q=xcode)

Example:
Xcode's version is 12.4 then download 
- Xcode 12.4
- Command Line Tools for Xcode 12.4


You can check which Xcode version is supported by your OS here:  [Xcode Support Device](https://developer.apple.com/support/xcode/)


# Installation

# Extracting debugserver


- `cd /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/DeviceSupport/<iOS version>` 
```
- Example: iDevice version 12.0.1 => 12.0

cd /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/DeviceSupport/12.0

```
- `hdiutil attach DeveloperDiskImage.dmg`

# Copy to debugserver to Desktop
- `cp /Volumes/DeveloperDiskImage/usr/bin/debugserver ~/Desktop` 

# Convert to arm64
- ```lipo -thin arm64 debugserver  -output debugserver_arm64```



# Connecting to iDevice
Open new Terminal for:
- ```iproxy 6666 6666```
- ```iproxy 2222 22```
Then SSH into iDevice (new Terminal)
- ```ssh -p 2222 root@localhost```

# Copy debugserver_arm64 to iphone
- ```scp -P 2222 debugserver_arm64 root@localhost:/usr/bin/debugserver_arm64```

# Signing debugserver (via ldid)
- Download : [debugserver.xml](https://www.dropbox.com/s/j50oc0ikjh2x6jd/debugserver.xml?dl=0) then put it into `/usr/share/entitlements`
- ```ldid -S/usr/share/entitlements/debugserver.xml /usr/bin/debugserver_arm64``` (in SSH)

# Use inject command on iDevice
- `inject /usr/bin/debugserver_arm64 `

``` 
  iPhone:~ root# inject /usr/bin/debugserver_arm64
	got persisted port!
	Injecting to trust cache...
	/usr/bin/debugserver_arm64: OK
	Actually injecting 1 keys
	1 new hashes to inject
	Successfully injected [1/1] to trust cache.
```

# Test success installation
- `debugserver_arm64`
```
iPhone:~ root# debugserver_arm64 
	debugserver-@(#)PROGRAM:LLDB  PROJECT:lldb-900.3.85
	 for arm64.
	Usage:
	  debugserver host:port [program-name program-arg1 program-arg2 ...]
	  debugserver /path/file [program-name program-arg1 program-arg2 ...]
	  debugserver host:port --attach=<pid>
	  debugserver /path/file --attach=<pid>
	  debugserver host:port --attach=<process_name>
	  debugserver /path/file --attach=<process_name>


```

- Start Settings app in iPhone -> `ps -ax | grep Preferences`

```

iPhone:~ root# ps -ax | grep Preferences
22686 ??         0:01.22 /Applications/Preferences.app/Preferences
22688 ttys000    0:00.00 grep Preferences

```

- Attach to process `debugserver_arm64  localhost:6666 -a Preferences`

- Now, open a new Mac console and run `lldb`
```
(lldb) platform select remote-ios
(lldb) process connect connect://localhost:6666
```

- -> Wait 1-2 min and, finally, you'll get the result


```
Result:
	Process 400 stopped
	* thread #1: tid = 0x118f, 0x38bfda58 libsystem_kernel.dylib`mach_msg_trap + 20, queue = 'com.apple.main-thread', stop reason = signal SIGSTOP
	    frame #0: 0x38bfda58 libsystem_kernel.dylib`mach_msg_trap + 20
	libsystem_kernel.dylib`mach_msg_trap:
	->  0x38bfda58 <+20>: pop    {r4, r5, r6, r8}
	    0x38bfda5c <+24>: bx     lr

	libsystem_kernel.dylib`mach_msg_overwrite_trap:
	    0x38bfda60 <+0>:  mov    r12, sp
	    0x38bfda64 <+4>:  push   {r4, r5, r6, r8}

(lldb)continue			-> thats all ...done.
```


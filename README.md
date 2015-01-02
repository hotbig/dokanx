# DokanX

## This is a fork of dokan 0.6.0
Because the original dokan isn't maintained anymore, I have decided to fork it.   
If you don't know about dokan, you should read [this document](http://dokan-dev.net/en/docs/) first. 

## What Improved
* Visual Studio 2013 solution file is provided with makefile and ddkbuild. Now you can compile filesystem driver very easily.
* All dokan library dll code has been recompiled in C++
* Using precompiled header for driver. It makes building driver much faster.
* Your usermode filesystem implementation should return their result code as NTSTATUS(Not the win32 status code).
This gives your application more control.
* Use better logger.
* Applied PREfast analyzer(static analyzing driver source code)

## Build Setting
* Download and install the [WDK 7.1.0](http://www.microsoft.com/en-us/download/details.aspx?id=11800)
* You should set the Windows environment variables DOKANX_PATH and WIN7BASE before compile driver.

```bash
# Note. Do not enclose WIN7BASE environment value in double quotes.  
# I guess ddkbuild can't recognize it.
DOKANX_PATH=YOURWORKSPACE\dokanx
WIN7BASE=C:\WinDDK\7600.16385.1
```
* Choose build configuration `debug_win7` or `release_win7` regardless your actual target. The driver binary(.sys) will works on any targets. 

### If you succeed to build, you may have
* dokanx.dll
* dokanx.lib
* mirrorfs.exe
* dokanx_win7.sys
* dokanx_mount.exe
* dokanx_control.exe

WDK 8.x are not supported yet. But this doesn't mean dokan can't run on Windows 8 or later. 

## How to mount mirrorfs?
Before mounting a filesystem, you need to *register* and *start* the filesystem driver.  
This can be done with [CreateService](http://msdn.microsoft.com/en-us/library/windows/desktop/ms682450(v=vs.85).aspx) and [StartService](http://msdn.microsoft.com/en-us/library/windows/desktop/ms686321(v=vs.85).aspx) function. You could write your own code for registering/starting driver if you want -maybe when your product is ready to deploy for end-user. At that time, [this simple wrapper](https://github.com/BenjaminKim/dokanx/blob/master/Common/WinNT/NtServiceCtrl.cpp) could be helpful to you.  
But just for testing, you don't need to know the much things like that. [There is a tool for them.](http://www.osronline.com/article.cfm?article=157)
You can easily *registering* and *starting* dokanx driver by osrloader.
* Move `dokanx_win7.sys` to `%SystemRoot%\System32\drivers\dokanx.sys`
* Set this path to osrloader's `Driver Path` and write `dokanx` to `Display Name`
* Click `Register Service` and `Start Service`

If there was no problem, you are ready to mount a volume.  
Note that `Stop Service` function doesn't work. This maybe `dokanx.sys`'s problem. It is very hard to write stopping device-driver code in Windows world. You need to reboot to stop the driver.

## How to test your own filesystem?

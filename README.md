# WinGo
An extendable cross-platform Windows remote administration tool written in Go
# About
WinGo is a tool similar to PsExec by Mark Russinovich, but rather than being tied to running the client on Windows, you can run it on Windows, MacOS, or Linux. WinGo was designed to be extendable by the end-user and allows you to write extensions in C# that are loaded automatically at runtime on the remote administration target. This is possible because the WinGo server-side shell runs within powershell on your target. This also eliminates the need to ship a large binary over the network like PsExec and it allows the server-side shell to run both itself and its extensions entirely in memory. 
# Usage
```
WinGo <address> <username> <domain>
./WinGo 192.168.7.142 Administrator TESTDOMAIN
Password: 
[*] Initiating remote WinGo session...
[*] Loading extensions...
[+] Successfully loaded extension: cmd
[+] Successfully loaded extension: process
[+] Successfully loaded extension: services
[+] Successfully loaded extension: shell

___       ______       _________      
__ |     / /__(_)________  ____/_____ 
__ | /| / /__  /__  __ \  / __ _  __ \
__ |/ |/ / _  / _  / / / /_/ / / /_/ /
____/|__/  /_/  /_/ /_/\____/  \____/ 
                                                                          

Windows Remote Administration Shell v1.0

WinGo C:\WINDOWS\system32> 
```
# Folders
There are two folders currently associated with WinGo. They are 'downloads' and 'extensions'. The downloads folder will store any files transferred from the remote host via the 'get' command, and extensions holds any standard api extensions as well as your own custom extensions. 
# Extensions
Extensions are simply C# source code files. When writing your own extension, the source file name must be unique, should be lowercase, and must have no file extension. In addition, a second file should be created with the same name as your extension but saved with a .refs file extension. The .refs file must contain a list of C# assembly references required for your source code to function, with each dll being separated by a comma. If you forget to include an assembly reference for a particular class that is used in your source, the shell will throw an exception and your extension will not be loaded. The majority of extensions will only require System.dll and System.Core.dll. For reference, you can look at the existing extensions that ship with WinGo as part of the standard api. 
# Standard API
A number of standard api routines are exposed as part of the core functionality of WinGo and their source is contained in the extensions folder under cmd, process, services, and shell. Their usage is as follows:
## Shell
cd - Change the current shell directory. This works similarly to cmd.exe in that you can change directories using accessible environment variables, using fully qualified paths, or using paths available in your current directory.
```
Examples:
cd %TMP% - changes directory to your temp folder
WinGo C:\> cd %TMP%


WinGo C:\WINDOWS\TEMP>

cd Users - change directory to Users folder from your current directory
WinGo C:\> cd Users


WinGo C:\Users>

cd "Program Files" - change directory to Program Files folder from your current directory. Note the required use of double quotes. In general, any command that is followed by another argument that contains a space should be denoted by the use of double quotes, otherwise the command will likely fail. 
WinGo C:\> cd "Program Files"


WinGo C:\Program Files>
```
dir - get a complete directory listing for either the current directory (if no path is supplied) or for a specified path. Environment variables are allowed here as well.
```
Examples:
WinGo C:\> dir
10/30/2015 12:24:24 AM  <DIR>           $Recycle.Bin
4/3/2019 11:29:08 AM    <DIR>           Documents and Settings
2/13/2016 5:22:58 AM    <DIR>           Logs
9/15/2018 12:33:50 AM   <DIR>           PerfLogs
9/15/2018 12:33:50 AM   <DIR>           Program Files
9/15/2018 12:33:50 AM   <DIR>           Program Files (x86)
9/15/2018 12:33:50 AM   <DIR>           ProgramData
4/3/2019 11:29:07 AM    <DIR>           Recovery
4/3/2019 11:27:11 AM    <DIR>           System Volume Information
9/14/2018 11:09:26 PM   <DIR>           Users
9/14/2018 11:09:26 PM   <DIR>           Windows
5/6/2019 10:02:11 AM    <DIR>           Windows.old
10/30/2015 1:13:43 AM           400228  bootmgr
10/30/2015 1:13:44 AM           1       BOOTNXT
4/3/2019 11:27:12 AM            1207959552      pagefile.sys
4/3/2019 11:27:12 AM            16777216        swapfile.sys


WinGo C:\>
```
del - deletes a file either from the current directory (if no complete path is provided) or a specified file path
```
Examples:
WinGo C:\> dir
10/30/2015 12:24:24 AM  <DIR>           $Recycle.Bin
4/3/2019 11:29:08 AM    <DIR>           Documents and Settings
2/13/2016 5:22:58 AM    <DIR>           Logs
9/15/2018 12:33:50 AM   <DIR>           PerfLogs
9/15/2018 12:33:50 AM   <DIR>           Program Files
9/15/2018 12:33:50 AM   <DIR>           Program Files (x86)
9/15/2018 12:33:50 AM   <DIR>           ProgramData
4/3/2019 11:29:07 AM    <DIR>           Recovery
4/3/2019 11:27:11 AM    <DIR>           System Volume Information
9/14/2018 11:09:26 PM   <DIR>           Users
9/14/2018 11:09:26 PM   <DIR>           Windows
5/6/2019 10:02:11 AM    <DIR>           Windows.old
10/30/2015 1:13:43 AM           400228  bootmgr
10/30/2015 1:13:44 AM           1       BOOTNXT
5/13/2019 1:29:57 PM            4141488 filetodelete
4/3/2019 11:27:12 AM            1207959552      pagefile.sys
4/3/2019 11:27:12 AM            16777216        swapfile.sys


WinGo C:\> del filetodelete
[+] Success

WinGo C:\>
```
copy - copies a file from the current directory (if no complete path is provided) to the provided destination or from the provided full path to the provided destination
```
Examples:
WinGo C:\> copy c:\windows\system32\cmd.exe test.exe
[+] Success

WinGo C:\> dir
10/30/2015 12:24:24 AM  <DIR>           $Recycle.Bin
4/3/2019 11:29:08 AM    <DIR>           Documents and Settings
2/13/2016 5:22:58 AM    <DIR>           Logs
9/15/2018 12:33:50 AM   <DIR>           PerfLogs
9/15/2018 12:33:50 AM   <DIR>           Program Files
9/15/2018 12:33:50 AM   <DIR>           Program Files (x86)
9/15/2018 12:33:50 AM   <DIR>           ProgramData
4/3/2019 11:29:07 AM    <DIR>           Recovery
4/3/2019 11:27:11 AM    <DIR>           System Volume Information
9/14/2018 11:09:26 PM   <DIR>           Users
9/14/2018 11:09:26 PM   <DIR>           Windows
5/6/2019 10:02:11 AM    <DIR>           Windows.old
10/30/2015 1:13:43 AM           400228  bootmgr
10/30/2015 1:13:44 AM           1       BOOTNXT
4/3/2019 11:27:12 AM            1207959552      pagefile.sys
4/3/2019 11:27:12 AM            16777216        swapfile.sys
5/13/2019 1:33:03 PM            278528  test.exe


WinGo C:\>
```
env - Get a list of all accessible environment variables
```
Examples:
WinGo C:\> env
COMPUTERNAME=DESKTOP-7T587IE
PUBLIC=C:\Users\Public
LOCALAPPDATA=C:\WINDOWS\system32\config\systemprofile\AppData\Local
PSModulePath=WindowsPowerShell\Modules;C:\Program Files\WindowsPowerShell\Modules;C:\WINDOWS\system32\WindowsPowerShell\v1.0\Modules
PROCESSOR_ARCHITECTURE=AMD64
Path=C:\WINDOWS\system32;C:\WINDOWS;C:\WINDOWS\System32\Wbem;C:\WINDOWS\System32\WindowsPowerShell\v1.0\;C:\WINDOWS\System32\OpenSSH\;C:\WINDOWS\system32\config\systemprofile\AppData\Local\Microsoft\WindowsApps
CommonProgramFiles(x86)=C:\Program Files (x86)\Common Files
ProgramFiles(x86)=C:\Program Files (x86)
PROCESSOR_LEVEL=6
ProgramFiles=C:\Program Files
PATHEXT=.COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC;.CPL
USERPROFILE=C:\WINDOWS\system32\config\systemprofile
SystemRoot=C:\WINDOWS
ALLUSERSPROFILE=C:\ProgramData
DriverData=C:\Windows\System32\Drivers\DriverData
ProgramData=C:\ProgramData
PROCESSOR_REVISION=9e0a
USERNAME=DESKTOP-7T587IE$
CommonProgramW6432=C:\Program Files\Common Files
CommonProgramFiles=C:\Program Files\Common Files
OS=Windows_NT
PROCESSOR_IDENTIFIER=Intel64 Family 6 Model 158 Stepping 10, GenuineIntel
ComSpec=C:\WINDOWS\system32\cmd.exe
PROMPT=$P$G
SystemDrive=C:
TEMP=C:\WINDOWS\TEMP
NUMBER_OF_PROCESSORS=2
APPDATA=C:\WINDOWS\system32\config\systemprofile\AppData\Roaming
TMP=C:\WINDOWS\TEMP
ProgramW6432=C:\Program Files
windir=C:\WINDOWS
USERDOMAIN=WORKGROUP


WinGo C:\>
```
## Services
services - Gets a complete list of all installed services on the remote server (includes device drivers)

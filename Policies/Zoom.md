# Installation Location
```
%APPDATA%\Zoom
```

# Code integrity policy methodology utilized
This policy was generated using the `FilePublisher` file level with a file `Hash` level as a fallback. The reason `Hash` was used as a fallback was that the following 3rd party libraries, while signed, do not have a populated version info resource:

```
%APPDATA%\Zoom\bin\libfaac.dll
%APPDATA%\Zoom\bin\turbojpeg.dll
```

* It would not be recommend to build a policy based on `FileName` or `FilePath` considering the software is installed in a user-writeable directory.
* It is my personal preference to not create a blanket allow list for a signer. Allowing all code to execute that is signed by a particular vendor, while minimizing policy maintenance, effectively eliminates the need to perform software baselining in the future for that particular signer. In my opinion, we always want visibility into the software build/packaging practices of every installation.

# Commands used to generate the code integrity policy

```powershell
$ZoomSigners = Get-SystemDriver -ScanPath $Env:APPDATA\Zoom -NoShadowCopy -UserPEs

New-CIPolicy -DriverFiles $ZoomSigners -FilePath 'Zoom.xml' -Level 'FilePublisher' -Fallback 'Hash'
```

# Software build hygeine observations

Striking the right balance between a policy that isn't too broad while remaining maintainable (i.e. avoidance of file hash rules) relies upon the following software build best practices:

1) All executable code be signed.
2) A [version info resource](https://docs.microsoft.com/en-us/windows/win32/menurc/versioninfo-resource) with a consistent `OriginalFilename` and consistently incremented `FileVersion`.

Fortunately, all executable code is signed using a single code signing certificate.

Unfortunately, the following version info inconsistencies are present:

The following files do not have a populated version info resource:

```
%APPDATA%\Zoom\bin\libfaac.dll
%APPDATA%\Zoom\bin\turbojpeg.dll
```

An issue was filed with Zoom on March 19, 2020 informing them of the above issue.

Overall, I am pleased with the consistent signing practices, relative lack of complexity of executable code in use, and the use of a single installation directory where all installation and uninstallation code is stored.

# Signers

### Subject
`CN="Zoom Video Communications, Inc.", O="Zoom Video Communications, Inc.", L=San Jose, S=California, C=US, SERIALNUMBER=4969967, OID.2.5.4.15=Private Organization, OID.1.3.6.1.4.1.311.60.2.1.2=Delaware, OID.1.3.6.1.4.1.311.60.2.1.3=US`

### Thumbprint
`0F9ADA46756C17EFFFD467D10654E2A766566CB3`

### Files

```
%APPDATA%\Zoom\bin\annoter.dll
%APPDATA%\Zoom\bin\aomagent.dll
%APPDATA%\Zoom\bin\asproxy.dll
%APPDATA%\Zoom\bin\CmmBrowserEngine.dll
%APPDATA%\Zoom\bin\Cmmlib.dll
%APPDATA%\Zoom\bin\CptControl.exe
%APPDATA%\Zoom\bin\CptHost.exe
%APPDATA%\Zoom\bin\CptInstall.exe
%APPDATA%\Zoom\bin\CptService.exe
%APPDATA%\Zoom\bin\CptShare.dll
%APPDATA%\Zoom\bin\DuiLib.dll
%APPDATA%\Zoom\bin\Installer.exe
%APPDATA%\Zoom\bin\libeay32.dll
%APPDATA%\Zoom\bin\libfaac.dll
%APPDATA%\Zoom\bin\mcm.dll
%APPDATA%\Zoom\bin\msaalib.dll
%APPDATA%\Zoom\bin\npzoomplugin.dll
%APPDATA%\Zoom\bin\nydus.dll
%APPDATA%\Zoom\bin\reslib.dll
%APPDATA%\Zoom\bin\ssb_sdk.dll
%APPDATA%\Zoom\bin\ssleay32.dll
%APPDATA%\Zoom\bin\tp.dll
%APPDATA%\Zoom\bin\turbojpeg.dll
%APPDATA%\Zoom\bin\util.dll
%APPDATA%\Zoom\bin\viper.dll
%APPDATA%\Zoom\bin\XmppDll.dll
%APPDATA%\Zoom\bin\zAutoUpdate.dll
%APPDATA%\Zoom\bin\zChatApp.dll
%APPDATA%\Zoom\bin\zChatUI.dll
%APPDATA%\Zoom\bin\zCrashReport.dll
%APPDATA%\Zoom\bin\zCrashReport.exe
%APPDATA%\Zoom\bin\zData.dll
%APPDATA%\Zoom\bin\zlt.dll
%APPDATA%\Zoom\bin\zmb.dll
%APPDATA%\Zoom\bin\Zoom_launcher.exe
%APPDATA%\Zoom\bin\Zoom.exe
%APPDATA%\Zoom\bin\zTscoder.exe
%APPDATA%\Zoom\bin\zVideoApp.dll
%APPDATA%\Zoom\bin\zVideoUI.dll
%APPDATA%\Zoom\bin\zWebService.dll
%APPDATA%\Zoom\bin\zWinRes.dll
%APPDATA%\Zoom\bin\zzhost.dll
%APPDATA%\Zoom\uninstall\Installer.exe
%APPDATA%\Zoom\ZoomDownload\Installer.exe
```

### Notes

The following 3rd party libraries were identified:

* [`libfaac.dll`](https://www.audiocoding.com/) - "FAAC is an open-source MPEG-4 and MPEG2 AAC au­dio en­co­der."
* [`turbojpeg.dll`](https://libjpeg-turbo.org/) - "libjpeg-turbo is a JPEG image codec that uses SIMD instructions (MMX, SSE2, AVX2, NEON, AltiVec) to accelerate baseline JPEG compression and decompression on x86, x86-64, ARM, and PowerPC systems, as well as progressive JPEG compression on x86 and x86-64 systems."
* [`XmppDll.dll`](https://camaya.net/gloox/) - "gloox is a rock-solid, full-featured Jabber/XMPP client library, written in clean ANSI C++. It makes writing spec-compliant clients easy and allows for hassle-free integration of Jabber/XMPP functionality into existing applications."
* `libeay32.dll` and `ssleay32.dll` - [The OpenSSL Project](http://www.openssl.org/)
* [`zCrashReport.dll` and `zCrashReport.exe`](http://crashrpt.sourceforge.net) - "CrashRpt is a free open-source library designed for intercepting exceptions in your C++ program, collecting technical information about the crash and sending error reports over the Internet to software vendor."
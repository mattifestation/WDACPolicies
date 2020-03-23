# Installation Location
```
%LOCALAPPDATA%\Microsoft\Teams
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin
```

# Code integrity policy methodology utilized
This policy was generated using the `FilePublisher` file level and `ProductName` file name level. Doing so maximizes the longevity of the policy by basing it on the hash of the issuing certificate which will have a much longer lifetime while only being applicable to the files that match the specified `ProductName` property in the version info resource of each executable file. A file level that relies upon signing should be used when all executable code is consistently signed which Teams fortunately is.

* It would not be recommend to build a policy based on `FileName` or `FilePath` considering the software is installed in user-writeable directories.
* It is my personal preference to not create a blanket allow list for all Microsoft-signed code. Microsoft-signed code is just as vulnerable to exploitable/abusable flaws as code signed by other publishers. Note that I make a clear distinction between Windows-signed code (in-box Windows code required to boot the OS) and Microsoft-signed code. In my opinion, Microsoft code shouldn't get a more permissable pass over non-Microsoft code.

# Commands used to generate the code integrity policy

```powershell
$TeamsFiles1 = Get-SystemDriver -ScanPath "$Env:LOCALAPPDATA\Microsoft\Teams" -NoShadowCopy -UserPEs
$TeamsFiles2 = Get-SystemDriver -ScanPath "$Env:LOCALAPPDATA\Microsoft\TeamsMeetingAddin" -NoShadowCopy -UserPEs

# Due to the poor design of Get-SystemDriver, what they return is a single List object. We can merge them this way:
$TeamsFiles1.AddRange($TeamsFiles2)

# Get rid of any signer info where the cmdlet thinks that it is a device driver. There are no device drivers in this list. It's good to get rid of them prior to calling New-CIPolicy so that driver rules are not created.
$TeamsFiles = $TeamsFiles1 | Where-Object { $_.UserMode }

# Now, build the policy
New-CIPolicy -DriverFiles $TeamsFiles -FilePath 'Teams.xml' -Level 'FilePublisher' -SpecificFileNameLevel 'ProductName'
```

# Software build hygeine observations

Striking the right balance between a policy that isn't too broad while remaining maintainable (i.e. avoidance of file hash rules) relies upon the following software build best practices:

1) All executable code be signed.
2) A [version info resource](https://docs.microsoft.com/en-us/windows/win32/menurc/versioninfo-resource) with a consistent `OriginalFilename` and consistently incremented `FileVersion`.

Fortunately, all executable code is signed.

Unfortunately, the following version info inconsistencies are present:

The following files were missing both the `OriginalFileName` property and the `FileVersion` property:

```
%LOCALAPPDATA%\Microsoft\Teams\current\ffmpeg.dll
%LOCALAPPDATA%\Microsoft\Teams\current\osmesa.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\v8-profiler-torycl\build\Release\profiler.node
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\slimcore\bin\onnxruntime.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\office-int-win\build\Release\office-int-win.node
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\node-spellcheckr\build\Release\spellchecker.node
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\native-utils\build\Release\native-utils.node
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\modern-osutils\build\Release\modern-osutils.node
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\media-hid\build\Release\media-hid.node
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\keytar4\build\Release\keytar.node
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\keytar3\build\Release\keytar.node
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\keyboard-layout\build\Release\keyboard-layout-manager.node
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\adal3-win\build\Release\adal-win.node
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\adal2-win\build\Release\adal-win.node
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\@msteams\node-locale-info-provider\build\Release\node-locale-info-provider.node
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\@msteams\electron-modules-package-utils\build\Release\package-utils.node
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\@microsoft\fasttext-languagedetector\build\Release\fastText-languagedetector.node
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\@microsoft\electron-windows-interactive-notifications\build\Release\InteractiveNotifications.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\@microsoft\electron-windows-interactive-notifications\build\Release\notifications_bindings.node
```

Fortunately, all executable code has a populated `ProductName` field, which can be used as the basis for an alternative `SpecificFileNameLevel` argument when calling `New-CIPolicy`. Because `FileVersion` is used inconsistently though, there is no way to build a policy enforcing actual minimum file versions without also requiring file hash rules for files that don't have a `FileVersion` property specified.

All files utilized the `Microsoft Teams` product name except for `msvcp140.dll` and `vcruntime140.dll` which use the `Microsoft® Visual Studio® 2015` product name. Once you start building policies where Visual C Runtime libraries like these are encountered more frequently, you may want to consider building a policy dedicated to the C Runtime library.

# Signers

## Signer #1

### Subject
`CN=Microsoft 3rd Party Application Component, O=Microsoft Corporation, L=Redmond, S=Washington, C=US`

### Thumbprint
`3F8F5784D54D72625EEEC7365691D65C9F1A8280`

### Files

```
%LOCALAPPDATA%\Microsoft\Teams\Update.exe
%LOCALAPPDATA%\Microsoft\Teams\current\ffmpeg.dll
%LOCALAPPDATA%\Microsoft\Teams\current\libEGL.dll
%LOCALAPPDATA%\Microsoft\Teams\current\libGLESv2.dll
%LOCALAPPDATA%\Microsoft\Teams\current\Squirrel.exe
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\v8-profiler-torycl\build\Release\profiler.node
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\node-spellcheckr\build\Release\spellchecker.node
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\keytar4\build\Release\keytar.node
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\keytar3\build\Release\keytar.node
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\keyboard-layout\build\Release\keyboard-layout-manager.node
```

### Notes

The following 3rd party libraries were identified:

* `Update.exe` and `Squirrel.exe` are the [Squirrel.Windows](https://github.com/Squirrel/Squirrel.Windows) installation/update framework. Both [identified as "LOLBins"](https://twitter.com/MrUn1k0d3r/status/1143928885211537408).
* [`ffmpeg.dll`](https://www.ffmpeg.org/)
* [`libEGL.dll`](https://www.mesa3d.org/egl.html)
* [`libGLESv2.dll`](https://github.com/google/angle/tree/master/src/libGLESv2)
* [`v8-profiler-torycl`](https://github.com/torycl/v8-profiler) - "v8-profiler provides node bindings for the v8 profiler and integration with node-inspector"
* [`node-spellcheckr`](https://github.com/atom/node-spellchecker) - "SpellChecker Node Module"
* [`keytar`](https://github.com/atom/node-keytar) - "A native Node module to get, add, replace, and delete passwords in system's keychain. On macOS the passwords are managed by the Keychain, on Linux they are managed by the Secret Service API/libsecret, and on Windows they are managed by Credential Vault."
* [`keyboard-layout`](https://github.com/atom/keyboard-layout) - "Read and observe the current keyboard layout."

## Signer #2

### Subject
`CN=Microsoft Corporation, O=Microsoft Corporation, L=Redmond, S=Washington, C=US`

### Thumbprint
`B10607FB914700B40F794610850C1DE0A21566C1`

### Files

```
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-core-console-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-core-console-l1-2-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-core-datetime-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-core-debug-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-core-errorhandling-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-core-file-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-core-file-l1-2-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-core-file-l2-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-core-handle-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-core-heap-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-core-interlocked-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-core-libraryloader-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-core-localization-l1-2-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-core-memory-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-core-namedpipe-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-core-processenvironment-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-core-processthreads-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-core-processthreads-l1-1-1.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-core-profile-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-core-rtlsupport-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-core-string-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-core-synch-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-core-synch-l1-2-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-core-sysinfo-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-core-timezone-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-core-util-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-crt-conio-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-crt-convert-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-crt-environment-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-crt-filesystem-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-crt-heap-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-crt-locale-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-crt-math-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-crt-multibyte-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-crt-private-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-crt-process-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-crt-runtime-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-crt-stdio-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-crt-string-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-crt-time-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\api-ms-win-crt-utility-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\d3dcompiler_47.dll
%LOCALAPPDATA%\Microsoft\Teams\current\msvcp140.dll
%LOCALAPPDATA%\Microsoft\Teams\current\osmesa.dll
%LOCALAPPDATA%\Microsoft\Teams\current\Teams.exe
%LOCALAPPDATA%\Microsoft\Teams\current\ucrtbase.dll
%LOCALAPPDATA%\Microsoft\Teams\current\vccorlib140.dll
%LOCALAPPDATA%\Microsoft\Teams\current\vcruntime140.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\adal-meetingaddin.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\adal2-meetingaddin.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-core-console-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-core-datetime-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-core-debug-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-core-errorhandling-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-core-file-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-core-file-l1-2-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-core-file-l2-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-core-handle-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-core-heap-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-core-interlocked-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-core-libraryloader-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-core-localization-l1-2-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-core-memory-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-core-namedpipe-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-core-processenvironment-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-core-processthreads-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-core-processthreads-l1-1-1.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-core-profile-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-core-rtlsupport-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-core-string-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-core-synch-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-core-synch-l1-2-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-core-sysinfo-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-core-timezone-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-core-util-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\API-MS-Win-core-xstate-l2-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-crt-conio-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-crt-convert-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-crt-environment-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-crt-filesystem-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-crt-heap-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-crt-locale-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-crt-math-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-crt-multibyte-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-crt-private-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-crt-process-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-crt-runtime-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-crt-stdio-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-crt-string-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-crt-time-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\api-ms-win-crt-utility-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\Microsoft.Applications.Telemetry.Windows.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\Microsoft.Teams.AddinLoader.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\Microsoft.Teams.AuthLib.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\Microsoft.Teams.Diagnostics.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\Microsoft.Teams.MeetingAddin.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\msvcp140.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\Newtonsoft.Json.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\System.Net.Http.Formatting.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\ucrtbase.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\vcruntime140.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\zh-TW\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\zh-CN\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\tr-TR\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\sv-SE\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\ru-RU\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\pt-BR\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\pl-PL\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\nl-NL\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\nb-NO\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\ko-KR\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\ja-JP\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\it-IT\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\fr-FR\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\fi-FI\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\es-ES\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\de-DE\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\da-DK\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x86\cs-CZ\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\adal-meetingaddin.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\adal2-meetingaddin.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-core-console-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-core-datetime-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-core-debug-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-core-errorhandling-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-core-file-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-core-file-l1-2-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-core-file-l2-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-core-handle-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-core-heap-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-core-interlocked-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-core-libraryloader-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-core-localization-l1-2-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-core-memory-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-core-namedpipe-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-core-processenvironment-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-core-processthreads-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-core-processthreads-l1-1-1.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-core-profile-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-core-rtlsupport-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-core-string-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-core-synch-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-core-synch-l1-2-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-core-sysinfo-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-core-timezone-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-core-util-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-crt-conio-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-crt-convert-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-crt-environment-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-crt-filesystem-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-crt-heap-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-crt-locale-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-crt-math-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-crt-multibyte-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-crt-private-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-crt-process-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-crt-runtime-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-crt-stdio-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-crt-string-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-crt-time-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\api-ms-win-crt-utility-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\Microsoft.Applications.Telemetry.Windows.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\Microsoft.Teams.AddinLoader.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\Microsoft.Teams.AuthLib.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\Microsoft.Teams.Diagnostics.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\Microsoft.Teams.MeetingAddin.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\msvcp140.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\Newtonsoft.Json.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\System.Net.Http.Formatting.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\ucrtbase.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\vcruntime140.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\zh-TW\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\zh-CN\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\tr-TR\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\sv-SE\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\ru-RU\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\pt-BR\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\pl-PL\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\nl-NL\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\nb-NO\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\ko-KR\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\ja-JP\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\it-IT\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\fr-FR\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\fi-FI\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\es-ES\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\de-DE\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\da-DK\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\meeting-addin\1.0.19350.3\x64\cs-CZ\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\assets\TeamsIconSet.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\slimcore\bin\onnxruntime.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\slimcore\bin\RtmCodecs.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\slimcore\bin\RtmControl.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\slimcore\bin\RtmMediaManager.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\slimcore\bin\RtmPal.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\slimcore\bin\RTMPLTFM.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\slimcore\bin\sharing-indicator.node
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\slimcore\bin\skypert.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\slimcore\bin\slimcore.node
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\slimcore\bin\ssScreenVVS2.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\office-int-win\build\Release\office-int-win.node
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\native-utils\build\Release\native-utils.node
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\msft-wam\build\Release\wam.node
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\modern-osutils\build\Release\modern-osutils.node
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\media-hid\build\Release\media-hid.node
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\adal3-win\build\Release\adal-win.node
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\adal3-win\build\Release\adal.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\adal2-win\build\Release\adal-win.node
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\adal2-win\build\Release\adal.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\@msteams\node-locale-info-provider\build\Release\node-locale-info-provider.node
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\@msteams\electron-modules-package-utils\build\Release\package-utils.node
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\@microsoft\fasttext-languagedetector\build\Release\fastText-languagedetector.node
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\@microsoft\electron-windows-interactive-notifications\build\Release\InteractiveNotifications.dll
%LOCALAPPDATA%\Microsoft\Teams\current\resources\app.asar.unpacked\node_modules\@microsoft\electron-windows-interactive-notifications\build\Release\notifications_bindings.node
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\adal-meetingaddin.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\adal2-meetingaddin.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-core-console-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-core-datetime-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-core-debug-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-core-errorhandling-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-core-file-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-core-file-l1-2-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-core-file-l2-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-core-handle-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-core-heap-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-core-interlocked-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-core-libraryloader-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-core-localization-l1-2-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-core-memory-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-core-namedpipe-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-core-processenvironment-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-core-processthreads-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-core-processthreads-l1-1-1.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-core-profile-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-core-rtlsupport-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-core-string-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-core-synch-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-core-synch-l1-2-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-core-sysinfo-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-core-timezone-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-core-util-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\API-MS-Win-core-xstate-l2-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-crt-conio-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-crt-convert-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-crt-environment-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-crt-filesystem-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-crt-heap-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-crt-locale-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-crt-math-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-crt-multibyte-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-crt-private-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-crt-process-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-crt-runtime-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-crt-stdio-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-crt-string-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-crt-time-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\api-ms-win-crt-utility-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\Microsoft.Applications.Telemetry.Windows.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\Microsoft.Teams.AddinLoader.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\Microsoft.Teams.AuthLib.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\Microsoft.Teams.Diagnostics.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\Microsoft.Teams.MeetingAddin.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\msvcp140.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\Newtonsoft.Json.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\System.Net.Http.Formatting.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\ucrtbase.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\vcruntime140.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\zh-TW\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\zh-CN\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\tr-TR\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\sv-SE\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\ru-RU\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\pt-BR\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\pl-PL\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\nl-NL\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\nb-NO\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\ko-KR\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\ja-JP\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\it-IT\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\fr-FR\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\fi-FI\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\es-ES\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\de-DE\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\da-DK\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x86\cs-CZ\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\adal-meetingaddin.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\adal2-meetingaddin.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-core-console-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-core-datetime-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-core-debug-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-core-errorhandling-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-core-file-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-core-file-l1-2-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-core-file-l2-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-core-handle-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-core-heap-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-core-interlocked-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-core-libraryloader-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-core-localization-l1-2-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-core-memory-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-core-namedpipe-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-core-processenvironment-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-core-processthreads-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-core-processthreads-l1-1-1.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-core-profile-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-core-rtlsupport-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-core-string-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-core-synch-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-core-synch-l1-2-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-core-sysinfo-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-core-timezone-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-core-util-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-crt-conio-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-crt-convert-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-crt-environment-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-crt-filesystem-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-crt-heap-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-crt-locale-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-crt-math-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-crt-multibyte-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-crt-private-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-crt-process-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-crt-runtime-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-crt-stdio-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-crt-string-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-crt-time-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\api-ms-win-crt-utility-l1-1-0.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\Microsoft.Applications.Telemetry.Windows.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\Microsoft.Teams.AddinLoader.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\Microsoft.Teams.AuthLib.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\Microsoft.Teams.Diagnostics.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\Microsoft.Teams.MeetingAddin.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\msvcp140.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\Newtonsoft.Json.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\System.Net.Http.Formatting.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\ucrtbase.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\vcruntime140.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\zh-TW\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\zh-CN\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\tr-TR\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\sv-SE\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\ru-RU\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\pt-BR\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\pl-PL\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\nl-NL\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\nb-NO\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\ko-KR\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\ja-JP\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\it-IT\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\fr-FR\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\fi-FI\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\es-ES\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\de-DE\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\da-DK\Microsoft.Teams.MeetingAddin.resources.dll
%LOCALAPPDATA%\Microsoft\TeamsMeetingAddin\1.0.19350.3\x64\cs-CZ\Microsoft.Teams.MeetingAddin.resources.dll
```

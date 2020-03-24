# Installation Location
```
%windir%\System32
```

# Code integrity policy methodology utilized
This policy was generated using the `FilePublisher` file level and `ProductName` file name level. Doing so maximizes the longevity of the policy by basing it on the hash of the issuing certificate which will have a much longer lifetime while only being applicable to the files that match the specified `ProductName` property in the version info resource of each executable file. A file level that relies upon signing should be used when all executable code is consistently signed which Teams fortunately is.

* It would not be recommend to build a policy based on `FileName` or `FilePath` considering the software is installed in user-writeable directories.
* It is my personal preference to not create a blanket allow list for all Microsoft-signed code. Microsoft-signed code is just as vulnerable to exploitable/abusable flaws as code signed by other publishers. Note that I make a clear distinction between Windows-signed code (in-box Windows code required to boot the OS) and Microsoft-signed code. In my opinion, Microsoft code shouldn't get a more permissable pass over non-Microsoft code.

# Commands used to generate the code integrity policy

```powershell
# Start a file trace for the installation
packageinspector.exe start C: -path vcredist_x64.exe

# Run the installer - vcredist_x64.exe

# Stop the trace and save the list of installed files to a text file
packageinspector.exe stop C: -out list -listPath VCRedistInstalledFiles.txt

# Create a directory where I will copy all the installed files to.
# Note: I first confirmed that there were no filename collisions
mkdir VisualCRuntimeFiles

# Copy the installed files to the new, temp directory.
# This directory will be the specified scan path for Get-SystemDriver
Get-Content VCRedistInstalledFiles.txt | Get-ChildItem | Copy-Item -Destination VisualCRuntimeFiles

$VCRuntimeFiles = Get-SystemDriver -ScanPath VisualCRuntimeFiles -UserPEs -NoShadowCopy

# Get rid of the files it thought were drivers
$VCRuntimeFiles = $VCRuntimeFiles | Where-Object { $_.UserMode }

# Key off ProductName as this property is used consistently and will reduce code integrity policy complexity.
New-CIPolicy -DriverFiles $VCRuntimeFiles -FilePath 'VisualCRuntime.xml' -Level 'FilePublisher' -Fallback Hash -SpecificFileNameLevel 'ProductName'
```

# Software build hygeine observations

Everything was signed consistently and version info resource properties were consistently applied to PE files.

# Signers

## Signer #1

### Subject
`CN=Microsoft Corporation, O=Microsoft Corporation, L=Redmond, S=Washington, C=US`

### Thumbprint
`9DC17888B5CFAD98B3CB35C1994E96227F061675`

### Files

```
vcredist_x64.exe
%windir%\System32\vcruntime140_1.dll
%windir%\System32\vcruntime140.dll
%windir%\System32\vcomp140.dll
%windir%\System32\vccorlib140.dll
%windir%\System32\vcamp140.dll
%windir%\System32\msvcp140_2.dll
%windir%\System32\msvcp140_1.dll
%windir%\System32\msvcp140.dll
%windir%\System32\mfcm140u.dll
%windir%\System32\mfcm140.dll
%windir%\System32\mfc140u.dll
%windir%\System32\mfc140rus.dll
%windir%\System32\mfc140kor.dll
%windir%\System32\mfc140jpn.dll
%windir%\System32\mfc140ita.dll
%windir%\System32\mfc140fra.dll
%windir%\System32\mfc140esn.dll
%windir%\System32\mfc140enu.dll
%windir%\System32\mfc140deu.dll
%windir%\System32\mfc140cht.dll
%windir%\System32\mfc140chs.dll
%windir%\System32\mfc140.dll
%windir%\System32\concrt140.dll
%ProgramData%\Package Cache\{F3241984-5A0E-4632-9025-AA16E0780A4B}v14.20.27508\packages\vcRuntimeMinimum_amd64\cab1.cab
%ProgramData%\Package Cache\{7b178cda-9740-4701-a92a-f168d213b343}\VC_redist.x64.exe
%ProgramData%\Package Cache\{4931385B-094D-4DC5-BD6A-5188FE9C51DF}v14.20.27508\packages\vcRuntimeAdditional_amd64\cab1.cab
```

## Signer #2

### Subject
`CN=Microsoft Corporation, O=Microsoft Corporation, L=Redmond, S=Washington, C=US`

### Thumbprint
`C3A3D43788E7ABCD287CB4F5B6583043774F99D2`

### Files

```
%windir%\Installer\3814638.msi
%windir%\Installer\3814634.msi
%ProgramData%\Package Cache\{F3241984-5A0E-4632-9025-AA16E0780A4B}v14.20.27508\packages\vcRuntimeMinimum_amd64\vc_runtimeMinimum_x64.msi
%ProgramData%\Package Cache\{4931385B-094D-4DC5-BD6A-5188FE9C51DF}v14.20.27508\packages\vcRuntimeAdditional_amd64\vc_runtimeAdditional_x64.msi
```
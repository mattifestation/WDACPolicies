# Installation Location
```
%CommonProgramFiles%\VMware
%ProgramFiles%\VMware\VMware Tools
```

# Code integrity policy methodology utilized
This policy was generated using the `WHQLFilePublisher` file level considering there were device drivers included. Fallback file levels used were `FilePublisher` and then `Hash`. `FilePublisher` was used as the default for user-mode code and `Hash` in the case where a PE did not contain a version info resource or in the case of allowing script/MSI code.

# Commands used to generate the code integrity policy

Due to the relative complexity of the software installation and with a mixture of kernel and user-mode code, crafting this policy was slightly more involved.

```powershell
# Get signer information for the setup files on the VMWare Tools installer ISO.
# The ISO was mounted to D: in my case.
$VMWareInstallerDiscDriverFiles = Get-SystemDriver -ScanPath D:\ -NoShadowCopy
$VMWareInstallerDiscUserFiles = Get-SystemDriver -ScanPath D:\ -UserPEs -NoShadowCopy
$VMWareInstallerDiscUserFiles = $VMWareInstallerDiscUserFiles | Where-Object { $VMWareInstallerDiscDriverFiles.FilePath -notcontains $_.FilePath }

# Exclude *ver.dll DLLs. These are not signed but they are resource-only DLLs
$VMWareFilesDrivers = Get-SystemDriver -ScanPath "$Env:ProgramFiles\Common Files\VMware" -NoShadowCopy | ? { !$_.FilePath.EndsWith('ver.dll') }
$VMWareFilesDrivers = $VMWareFilesDrivers | ? { !$_.FilePath.EndsWith('ver.dll') }

$VMWareFilesUser1 = Get-SystemDriver -ScanPath "$Env:ProgramFiles\Common Files\VMware" -NoShadowCopy -UserPEs
# Exclude Windows Vista files - I don't personally require these
# Exclude *ver.dll DLLs. These are not signed but they are resource-only DLLs
# Exclude any file that was determined to be a device driver. These often get included in user-mode rulesets and drivers have no business loading in user mode.
$VMWareFilesUser1 = $VMWareFilesUser1 | Where-Object { ($VMWareFilesDrivers.FilePath -notcontains $_.FilePath) -and (!$_.FilePath.EndsWith('ver.dll')) -and (($_.FilePath -notmatch 'Vista')) }

$VMWareFilesUser2 = Get-SystemDriver -ScanPath "$Env:ProgramFiles\VMware\VMware Tools" -NoShadowCopy -UserPEs

[Microsoft.SecureBoot.UserConfig.DriverFile[]] $AllVMWareFiles = $VMWareInstallerDiscDriverFiles + $VMWareInstallerDiscUserFiles + $VMWareFilesDrivers + $VMWareFilesUser1 + $VMWareFilesUser2

New-CIPolicy -DriverFiles $AllVMWareFiles -FilePath 'VMWareTools.xml' -Level 'WHQLFilePublisher' -Fallback 'FilePublisher, Hash'
```

# Software build hygeine observations

Everything was consistently signed. Additionally, all device drivers are WHQL-signed.

The following files were missing a version info resource, resulting in the need to allow by hash (due to me selecting the `FilePublisher` rule):

```
%CommonProgramFiles%\VMware\Drivers\vss\comreg.exe
%ProgramFiles%\VMware\VMware Tools\VMware VGAuth\libxmlsec-openssl.dll
%ProgramFiles%\VMware\VMware Tools\VMware VGAuth\libxmlsec.dll
```

# Signers

## Signer #1

### Subject
`E=noreply@vmware.com, CN="VMware, Inc.", O="VMware, Inc.", L=Palo Alto, S=California, C=US`

### Thumbprint
`E4A2E2000A9A21B45EA38BF3A71C10270C6737AB`

### Files

```
%CommonProgramFiles%\VMware\Drivers\vss\comreg.exe
%CommonProgramFiles%\VMware\Drivers\vss\VCBSnapshotProvider.dll
%ProgramFiles%\VMware\VMware Tools\7za.exe
%ProgramFiles%\VMware\VMware Tools\deployPkg.dll
%ProgramFiles%\VMware\VMware Tools\gio-2.0.dll
%ProgramFiles%\VMware\VMware Tools\glib-2.0.dll
%ProgramFiles%\VMware\VMware Tools\glibmm-2.4.dll
%ProgramFiles%\VMware\VMware Tools\gmodule-2.0.dll
%ProgramFiles%\VMware\VMware Tools\gobject-2.0.dll
%ProgramFiles%\VMware\VMware Tools\gthread-2.0.dll
%ProgramFiles%\VMware\VMware Tools\guestStoreClient.dll
%ProgramFiles%\VMware\VMware Tools\hgfs.dll
%ProgramFiles%\VMware\VMware Tools\iconv.dll
%ProgramFiles%\VMware\VMware Tools\intl.dll
%ProgramFiles%\VMware\VMware Tools\libeay32.dll
%ProgramFiles%\VMware\VMware Tools\pcre.dll
%ProgramFiles%\VMware\VMware Tools\plugins\common\hgfsServer.dll
%ProgramFiles%\VMware\VMware Tools\plugins\common\hgfsUsability.dll
%ProgramFiles%\VMware\VMware Tools\plugins\common\vix.dll
%ProgramFiles%\VMware\VMware Tools\plugins\vmsvc\appInfo.dll
%ProgramFiles%\VMware\VMware Tools\plugins\vmsvc\autoLogon.dll
%ProgramFiles%\VMware\VMware Tools\plugins\vmsvc\autoUpgrade.dll
%ProgramFiles%\VMware\VMware Tools\plugins\vmsvc\bitMapper.dll
%ProgramFiles%\VMware\VMware Tools\plugins\vmsvc\deployPkgPlugin.dll
%ProgramFiles%\VMware\VMware Tools\plugins\vmsvc\disableGuestHibernate.dll
%ProgramFiles%\VMware\VMware Tools\plugins\vmsvc\diskWiper.dll
%ProgramFiles%\VMware\VMware Tools\plugins\vmsvc\guestInfo.dll
%ProgramFiles%\VMware\VMware Tools\plugins\vmsvc\guestStore.dll
%ProgramFiles%\VMware\VMware Tools\plugins\vmsvc\hwUpgradeHelper.dll
%ProgramFiles%\VMware\VMware Tools\plugins\vmsvc\powerOps.dll
%ProgramFiles%\VMware\VMware Tools\plugins\vmsvc\resolutionSet.dll
%ProgramFiles%\VMware\VMware Tools\plugins\vmsvc\timeSync.dll
%ProgramFiles%\VMware\VMware Tools\plugins\vmsvc\vmbackup.dll
%ProgramFiles%\VMware\VMware Tools\plugins\vmusr\darkModeSync.dll
%ProgramFiles%\VMware\VMware Tools\plugins\vmusr\desktopEvents.dll
%ProgramFiles%\VMware\VMware Tools\plugins\vmusr\dndcp.dll
%ProgramFiles%\VMware\VMware Tools\plugins\vmusr\unity.dll
%ProgramFiles%\VMware\VMware Tools\plugins\vmusr\vmtray.dll
%ProgramFiles%\VMware\VMware Tools\rpctool.exe
%ProgramFiles%\VMware\VMware Tools\rvmSetup.exe
%ProgramFiles%\VMware\VMware Tools\sigc-2.0.dll
%ProgramFiles%\VMware\VMware Tools\ssleay32.dll
%ProgramFiles%\VMware\VMware Tools\vmtools.dll
%ProgramFiles%\VMware\VMware Tools\vmtoolsd.exe
%ProgramFiles%\VMware\VMware Tools\VMToolsHook.dll
%ProgramFiles%\VMware\VMware Tools\VMToolsHook64.dll
%ProgramFiles%\VMware\VMware Tools\VMToolsHookProc.exe
%ProgramFiles%\VMware\VMware Tools\VMware VGAuth\gio-2.0.dll
%ProgramFiles%\VMware\VMware Tools\VMware VGAuth\glib-2.0.dll
%ProgramFiles%\VMware\VMware Tools\VMware VGAuth\gmodule-2.0.dll
%ProgramFiles%\VMware\VMware Tools\VMware VGAuth\gobject-2.0.dll
%ProgramFiles%\VMware\VMware Tools\VMware VGAuth\gthread-2.0.dll
%ProgramFiles%\VMware\VMware Tools\VMware VGAuth\iconv.dll
%ProgramFiles%\VMware\VMware Tools\VMware VGAuth\intl.dll
%ProgramFiles%\VMware\VMware Tools\VMware VGAuth\libeay32.dll
%ProgramFiles%\VMware\VMware Tools\VMware VGAuth\libxml2.dll
%ProgramFiles%\VMware\VMware Tools\VMware VGAuth\libxmlsec.dll
%ProgramFiles%\VMware\VMware Tools\VMware VGAuth\libxmlsec-openssl.dll
%ProgramFiles%\VMware\VMware Tools\VMware VGAuth\pcre.dll
%ProgramFiles%\VMware\VMware Tools\VMware VGAuth\ssleay32.dll
%ProgramFiles%\VMware\VMware Tools\VMware VGAuth\VGAuth.dll
%ProgramFiles%\VMware\VMware Tools\VMware VGAuth\VGAuthCLI.exe
%ProgramFiles%\VMware\VMware Tools\VMware VGAuth\VGAuthService.exe
%ProgramFiles%\VMware\VMware Tools\VMware VGAuth\vmtools.dll
%ProgramFiles%\VMware\VMware Tools\VMware VGAuth\VMwareAliasImport.exe
%ProgramFiles%\VMware\VMware Tools\VMwareHgfsClient.exe
%ProgramFiles%\VMware\VMware Tools\VMwareHostOpen.exe
%ProgramFiles%\VMware\VMware Tools\VMwareNamespaceCmd.exe
%ProgramFiles%\VMware\VMware Tools\VMwareResolutionSet.exe
%ProgramFiles%\VMware\VMware Tools\VMwareToolboxCmd.exe
%ProgramFiles%\VMware\VMware Tools\VMwareXferlogs.exe
%ProgramFiles%\VMware\VMware Tools\win32\vmGuestLib.dll
%ProgramFiles%\VMware\VMware Tools\win32\vmGuestLibJava.dll
%ProgramFiles%\VMware\VMware Tools\win64\vmGuestLib.dll
%ProgramFiles%\VMware\VMware Tools\win64\vmGuestLibJava.dll
```

## Signer #2

### Subject
`CN=Microsoft Windows Hardware Compatibility Publisher, O=Microsoft Corporation, L=Redmond, S=Washington, C=US`

### Thumbprint
`9D05A3DB6517CB4C46FDB645F655ADB548B6B918`

### Files

```
%CommonProgramFiles%\VMware\Drivers\hgfs\Win8\vmhgfs.sys
%CommonProgramFiles%\VMware\Drivers\hgfs\Win8\vmhgfs_x64.dll
%CommonProgramFiles%\VMware\Drivers\hgfs\Win8\vmhgfs_x86.dll
%CommonProgramFiles%\VMware\Drivers\memctl\Win8\vmmemctl.sys
%CommonProgramFiles%\VMware\Drivers\mouse\Win8\vmmouse.sys
%CommonProgramFiles%\VMware\Drivers\mouse\Win8\vmusbmouse.sys
%CommonProgramFiles%\VMware\Drivers\pvscsi\Win8\pvscsi.sys
%CommonProgramFiles%\VMware\Drivers\rawdsk\Win8\vmrawdsk.sys
%CommonProgramFiles%\VMware\Drivers\video_wddm\Vista\vm3dco.dll
%CommonProgramFiles%\VMware\Drivers\video_wddm\Vista\vm3ddevapi.dll
%CommonProgramFiles%\VMware\Drivers\video_wddm\Vista\vm3ddevapi64.dll
%CommonProgramFiles%\VMware\Drivers\video_wddm\Vista\vm3ddevapi64-debug.dll
%CommonProgramFiles%\VMware\Drivers\video_wddm\Vista\vm3ddevapi64-release.dll
%CommonProgramFiles%\VMware\Drivers\video_wddm\Vista\vm3ddevapi64-stats.dll
%CommonProgramFiles%\VMware\Drivers\video_wddm\Vista\vm3ddevapi-debug.dll
%CommonProgramFiles%\VMware\Drivers\video_wddm\Vista\vm3ddevapi-release.dll
%CommonProgramFiles%\VMware\Drivers\video_wddm\Vista\vm3ddevapi-stats.dll
%CommonProgramFiles%\VMware\Drivers\video_wddm\Vista\vm3dgl.dll
%CommonProgramFiles%\VMware\Drivers\video_wddm\Vista\vm3dgl64.dll
%CommonProgramFiles%\VMware\Drivers\video_wddm\Vista\vm3dglhelper.dll
%CommonProgramFiles%\VMware\Drivers\video_wddm\Vista\vm3dglhelper64.dll
%CommonProgramFiles%\VMware\Drivers\video_wddm\Vista\vm3dmp.sys
%CommonProgramFiles%\VMware\Drivers\video_wddm\Vista\vm3dmp_loader.sys
%CommonProgramFiles%\VMware\Drivers\video_wddm\Vista\vm3dmp-debug.sys
%CommonProgramFiles%\VMware\Drivers\video_wddm\Vista\vm3dmp-stats.sys
%CommonProgramFiles%\VMware\Drivers\video_wddm\Vista\vm3dservice.exe
%CommonProgramFiles%\VMware\Drivers\video_wddm\Vista\vm3dum.dll
%CommonProgramFiles%\VMware\Drivers\video_wddm\Vista\vm3dum_10.dll
%CommonProgramFiles%\VMware\Drivers\video_wddm\Vista\vm3dum_10-debug.dll
%CommonProgramFiles%\VMware\Drivers\video_wddm\Vista\vm3dum_10-stats.dll
%CommonProgramFiles%\VMware\Drivers\video_wddm\Vista\vm3dum_loader.dll
%CommonProgramFiles%\VMware\Drivers\video_wddm\Vista\vm3dum64.dll
%CommonProgramFiles%\VMware\Drivers\video_wddm\Vista\vm3dum64_10.dll
%CommonProgramFiles%\VMware\Drivers\video_wddm\Vista\vm3dum64_10-debug.dll
%CommonProgramFiles%\VMware\Drivers\video_wddm\Vista\vm3dum64_10-stats.dll
%CommonProgramFiles%\VMware\Drivers\video_wddm\Vista\vm3dum64_loader.dll
%CommonProgramFiles%\VMware\Drivers\video_wddm\Vista\vm3dum64-debug.dll
%CommonProgramFiles%\VMware\Drivers\video_wddm\Vista\vm3dum64-stats.dll
%CommonProgramFiles%\VMware\Drivers\video_wddm\Vista\vm3dum-debug.dll
%CommonProgramFiles%\VMware\Drivers\video_wddm\Vista\vm3dum-stats.dll
%CommonProgramFiles%\VMware\Drivers\vmci\device\Win8\vmci.sys
%CommonProgramFiles%\VMware\Drivers\vmci\sockets\Win8\vsock.sys
%CommonProgramFiles%\VMware\Drivers\vmci\sockets\Win8\vsocklib_x64.dll
%CommonProgramFiles%\VMware\Drivers\vmci\sockets\Win8\vsocklib_x86.dll
%CommonProgramFiles%\VMware\Drivers\vmxnet3\Win8\vmxnet3.sys
```

## Signer #3

### Subject
`CN="VMware, Inc.", O="VMware, Inc.", L=Palo Alto, S=California, C=US`

### Thumbprint
`40A149CE7CE886318EFA584C162D061143DFC25E`

### Files

```
%ProgramFiles%\VMware\VMware Tools\VMware VGAuth\VMWSU.dll
```

## Signer #4

### Subject
`CN="VMware, Inc.", O="VMware, Inc.", L=Palo Alto, S=California, C=US`

### Thumbprint
`274FDA789F9D0F1DEA5AE443918D9EFC4F221141`

### Files

```
%windir%\Installer\4927a.msi
```
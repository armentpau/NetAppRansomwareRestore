# RansomwareRestore

The RansomwareRestore module was created after my work had repeated malware infections, which took client's file servers offline for days at a time.
This module was developed to accelerate the identification, restoration, and cleanup of encrypted files in our environment.  Currently
this module can only restore from netapp snapshots, but more backup/restore method could be added over time.  The future versions of this module will be built to allow
for this change.

To use this module, the following two modules are required in order to work properly (when netapp is no longer the only restoration option possible, the dataontap module will no longer be required.

1. NTFSSecurity Original Version: http://ntfssecurity.codeplex.com Recommended version: https://github.com/armentpau/NTFSSecurity (this fixes some issues with hidden folders/files)
2. DataOnTap module for netapp (you need to be a netapp customer to download)

## Connect-Netapp

### PARAMETERS
#### netAppController `String`
The name of the netapp controller to connect to

#### vServer `String`
The name of the vserver to connect to.  Usually the format of this is svm_XXXXXXfs01

#### credential `PSCredential`
A credential object that can be passed to the script.  The format of the username is DOMAIN\Username with the domain in ALL UPPERCASE.  If the domain is not in all uppercase the login will fail.

#### Username `String`
The username of a user that has access to netapp that can be passed to the script.  The format of the username is DOMAIN\Username with the domain in ALL UPPERCASE.  If the domain is not in all uppercase the login will fail.  The script will prompt for the password through a secure form if this option is choosen.


## Get-EncryptedFiles

### SYNOPSIS
Generates a list of encrypted files on the file system.  

### DESCRIPTION
This script relies on the command get-vcpirandomewarefamily to determine how to search for encrypted files on the network share.  This script will, based on this information, be able to identify and output to the screen or to a variable a list of objects that have been encrypted.

### PARAMETERS
#### RansomWareFamily `Object`
An object identifying the family of RansomWare that has encrypted the network file share

#### searchPath `String`
The path to search for encrypted files


### EXAMPLES
#### -------------------------- EXAMPLE 1 --------------------------
PS C:\>
```powershell
get-EncryptedFiles -RansomWareFamily (get-RansomWareFamily -firstFilePath "c:\testtxt.txt" -secondFilePath "c:\testdoc.doc") -searchPath "C:\folder"
```
The parameter RansomWareFamily is expecting the output object from get-RansomWareFamily.  This can either be stored in a variable or called at runtime.
#### -------------------------- EXAMPLE 2 --------------------------
PS C:\>
```powershell
get-RansomWareFamily -firstFilePath "c:\testtxt.txt" -secondFilePath "c:\testdoc.doc" | get-EncryptedFiles -searchPath "c:\folder"
```
The parameter RansomWareFamily is expecting the output object from get-RansomWareFamily.  This can either be stored in a variable or called at runtime.
#### -------------------------- EXAMPLE 3 --------------------------
PS C:\>
```powershell
$family = get-RansomWareFamily -firstFilePath = "c:\testtxt.txt" -secondFilePath = "c:\testdoc.doc"

get-EncryptedFiles -RansomWareFamily $family -searchPath "c:\folder"
```
The parameter RansomWareFamily is expecting the output object from get-RansomWareFamily.  This can either be stored in a variable or called at runtime.


## Get-RansomwareFamily

### SYNOPSIS
Determines the type of RansomWare depending on the input of two files

### DESCRIPTION


### PARAMETERS
#### FirstFilePath `String`
The first file to analyze

#### SecondFilePath `String`
The second file to analyze


### EXAMPLES
#### -------------------------- EXAMPLE 1 --------------------------
PS C:\>
```powershell
get-EncryptedFiles -RansomWareFamily (get-RansomWareFamily -firstFilePath "c:\testtxt.txt" -secondFilePath "c:\testdoc.doc") -searchPath "C:\folder"
```

#### -------------------------- EXAMPLE 2 --------------------------
PS C:\>
```powershell
get-RansomWareFamily -firstFilePath "c:\testtxt.txt" -secondFilePath "c:\testdoc.doc" | get-EncryptedFiles -searchPath "c:\folder"
```

#### -------------------------- EXAMPLE 3 --------------------------
PS C:\>
```powershell
$family = get-RansomWareFamily -firstFilePath = "c:\testtxt.txt" -secondFilePath = "c:\testdoc.doc"

get-EncryptedFiles -RansomWareFamily $family -searchPath "c:\folder"
```


## Get-SnapshotList

### PARAMETERS
#### restoreDate `String`


## Remove-EncryptedFiles

### SYNOPSIS
Removes a list of encrypted files from the file share

### PARAMETERS
#### removeList `Object`
A list of files to remove.  This list is an object which should be provided by Get-VCPIEncryptedFiles or Get-VCPICleanupFiles


### NOTES

### EXAMPLES
#### -------------------------- EXAMPLE 1 --------------------------
PS C:\>
```powershell
Remove-EncryptedFiles -removelist $list
```
Use a list of files from Get-Cleanupfiles command to remove from the file server.  You can also use a list from Get-EncryptedFiles instead.  This is basically a cleanup function.
#### -------------------------- EXAMPLE 2 --------------------------
PS C:\>
```powershell
Remove-EncryptedFiles -removelist $list
```


## Restore-EncryptedFiles

### SYNOPSIS


### DESCRIPTION
This function works with other commands to restore files on the netapp file system from snapshots.  The command lists all of the available snapshots and then works on restoring the files.

### PARAMETERS
#### restoreList `Object`
A list of files to restore.  This list is an object which should be provided by Get-VCPIEncryptedFiles

#### netappFileServer `String`
The netapp file server to connect to.  This is usually in the format of svm_XXXXXfs01.

#### netappController `String`
The name of the netapp controller to connect to

#### netAppCredential `PSCredential`
A credential object that can be passed to the script.  The format of the username is DOMAIN\Username with the domain in ALL UPPERCASE.  If the domain is not in all uppercase the login will fail.

#### restoreBase `String`
The path of the netapp volume.  Usually this is in the format of /svm_XXXXXfs01_data

#### incidentNumber `String`
The Incident/Task number from Service now

#### restoreDate `String`
The date to restore the files to.  This is in the format of M/D/YYYY
Examples:
3/31/2016
3/1/2016
12/1/2016

#### fileSystemBase `Object`
The base of the file string in the file path to replace with the temporary share path. This is usually in the format of: \\XXXXXFS01\c$\svm_XXXXXFS01_data

#### Overwrite `[SwitchParameter]`
If this flag is selected the files will be deleted first in the original location and then copied over.  This is due to working with file paths longer than 256 characters

#### netappUserName `String`
The username of a user that has access to netapp that can be passed to the script.  The format of the username is DOMAIN\Username with the domain in ALL UPPERCASE.  If the domain is not in all uppercase the login will fail.  The script will prompt for the password through a secure form if this option is choosen.

#### ransomwareFamily `[]`
An object identifying the family of RansomWare that has encrypted the network file share, this object is from the Get-VCPIRansomwareFamily command


### EXAMPLES
#### -------------------------- EXAMPLE 1 --------------------------
PS C:\>
```powershell
Restore-EncryptedFiles -restoreList $list -netappFileServer svm_XXXXXXfs01 -netappController netapp01 -netAppCredential $credential -restoreBase /svm_XXXXXXfs01_data -incidentNumber INCTEST12 -restoreDate 3/27/2016 -fileSystemBase \\XXXXXXfs01\c$\svm_XXXXXXfs01_data -Overwrite -ransomwareFamily $encryptedFamily
```
Using the netappCredential parameter, you pass a credential object to the command.  This can easy be created by using the following command:
$credential = get-credential
#### -------------------------- EXAMPLE 2 --------------------------
PS C:\>
```powershell
Restore-EncryptedFiles -restoreList $list -netappFileServer svm_XXXXXXfs01 -netappController netapp01 -netAppCredential $credential -restoreBase /svm_XXXXXXfs01_data -incidentNumber INCTEST12 -restoreDate 3/27/2016 -fileSystemBase \\XXXXXXfs01\c$\svm_XXXXXXfs01_data -Overwrite -ransomwareFamily (get-RansomwareFamily -firstfile path c:\admin\test.txt -secondfilepath c:\admin\test2.txt)
```
Using the netappCredential parameter, you pass a credential object to the command.  This can easy be created by using the following command:
$credential = get-credential
#### -------------------------- EXAMPLE 3 --------------------------
PS C:\>
```powershell
Restore-EncryptedFiles -restoreList $list -netappFileServer svm_XXXXXXfs01 -netappController netapp01 -netAppUsername DOMAIN\Username -restoreBase /svm_XXXXXXfs01_data -incidentNumber INCTEST12 -restoreDate 3/27/2016 -fileSystemBase \\XXXXXXfs01\c$\svm_XXXXXXfs01_data -Overwrite -ransomwareFamily (get-RansomwareFamily -firstfile path c:\admin\test.txt -secondfilepath c:\admin\test2.txt)
```
Using the NetAppUsername parameter and filling in the username, you are prompted for your password


## Get-CleanupFiles

### SYNOPSIS
Gets a list of files left over by the ransomware that are not encrypted

### DESCRIPTION
This command retrieves a list of files left by the ransomware which are not encrypted.  These filese are usually the ransomware notification messages and are trypically .html, .txt and an image file (.png, .jpeg, etc)

### PARAMETERS
#### Filter `String`
Use the filter to filter the files on the file system by a unique value.  Typically this will be a specific name of the ransomeware notes that is in three different formats.  DO NOT USE WILDCARDS.

#### searchPath `String`
The root path to search for the files to cleanup.

### EXAMPLES
#### -------------------------- EXAMPLE 1 --------------------------
PS C:\>
```powershell
Get-CleanupFiles -filter "HelpDecrypt_" -searchPath "c:\"
```
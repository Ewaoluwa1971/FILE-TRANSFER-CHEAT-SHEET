# Table of Contents
- [1. SCP (Secure Copy)](#1-scp-secure-copy)
- [2. SFTP (SSH File Transfer)](#2-sftp-ssh-file-transfer)
- [3. RSYNC over SSH](#3-rsync-over-ssh)
- [4. FTP/FTPS Command Line](#4-ftpftps-command-line)
- [5. SMB/CIFS Client Mounts](#5-smbcifs-client-mounts)
- [6. Netcat (Unencrypted)](#6-netcat-unencrypted)
- [7. PowerShell Remoting (WinRM)](#7-powershell-remoting-winrm)
- [8. Tar over SSH](#8-tar-over-ssh)
- [9. Rclone (Cross-platform sync)](#9-rclone-cross-platform-sync)
- [10. cURL (Various protocols)](#10-curl-various-protocols)
- [11. LFTP (Advanced FTP)](#11-lftp-advanced-ftp)
- [12. TFTP (Trivial FTP)](#12-tftp-trivial-ftp)
- [13. Magic Wormhole](#13-magic-wormhole)
- [14. Azure/AWS CLI](#14-azureaws-cli)
- [15. WebDAV (curl)](#15-webdav-curl)
- [16. SSHFS (FUSE)](#16-sshfs-fuse)
- [17. NFS (Windows NFS Client)](#17-nfs-windows-nfs-client)
- [18. Base64 Encoding](#18-base64-encoding)
- [19. ZMODEM over SSH](#19-zmodem-over-ssh)
- [20. WINDOWS NATIVE](#20-windows-native)
- [21. PYTHON-BASED TRANSFER METHODS](#21-python-based-transfer-methods)

# Linux ↔ Windows CLI File Transfer Methods

## 1. SCP (Secure Copy)
**Requires OpenSSH on Windows**

### Linux → Windows
``` bash 
scp /path/to/file user@windows_ip:"C:\\path\\to\\destination"
 ```

### Windows → Linux (PowerShell)
```bash
scp "C:\path\to\file" user@linux_ip:/path/to/destination
```
## 2. SFTP (SSH File Transfer)

### Linux → Windows
```bash
sftp user@windows_ip:"C:\\path\\to\\remote" <<< $'put localfile'
```
### Windows → Linux (PowerShell)
```bash
sftp user@linux_ip:/remote/path <<< "get remotefile localfile"
```
## 3. RSYNC over SSH

### Linux → Windows (rsync on Windows required)
```bash
rsync -avz -e ssh /local/path/ user@windows_ip:"C:\\remote\\path"
```
### Windows → Linux (PowerShell with rsync)
```bash
rsync -avz -e ssh user@linux_ip:/remote/path/ "C:\local\path"
```
## 4. FTP/FTPS Command Line

### Linux → Windows
```bash
ftp -n windows_ip <<END_SCRIPT
quote USER username
quote PASS password
put localfile remotefile
quit
END_SCRIPT
```
### Windows → Linux (PowerShell)
```bash
(echo "USER username" && echo "PASS password" && echo "get remotefile localfile" && echo "quit") | ftp linux_ip
```
## 5. SMB/CIFS Client Mounts

### Linux → Windows (Mount first)
```bash
sudo mount -t cifs //windows_ip/share /mnt/win -o username=user,password=pass
cp /local/file /mnt/win/path/
```
### Windows → Linux (PowerShell)
```bash
net use Z: \\linux_ip\share /user:username password
copy "C:\file" Z:\path\
```
## 6. Netcat (Unencrypted)

### Linux → Windows
#### Windows listener (PowerShell):
```bash
nc -lvnp 1234 > file
```
#### Linux sender:
```bash
nc -w 3 windows_ip 1234 < file
```
### Windows → Linux
#### Linux listener:
```bash
nc -l -p 1234 > file
```
#### Windows sender (PowerShell):
```bash
nc linux_ip 1234 < file
```
## 7. PowerShell Remoting (WinRM)

### Windows → Linux
```bash
Copy-Item -Path "C:\file" -Destination "\\linux_ip\share\" -ToSession (New-PSSession -HostName linux_ip -UserName user)
```
## 8. Tar over SSH

### Linux → Windows
```bash
tar czf - /folder | ssh user@windows_ip "tar xzf - -C C:\\path"
```
### Windows → Linux (WSL/PowerShell)
```bash
tar -cf - "C:\folder" | ssh user@linux_ip "tar xf - -C /path"
```
## 9. Rclone (Cross-platform sync)

### Linux → Windows
```bash
rclone copy /local/path remote:windows_path --progress
```
### Windows → Linux
```bash
rclone copy remote:linux_path "C:\backup\" --progress
```
## 10. cURL (Various protocols)

### Linux → Windows (FTP)
```bash
curl -T file ftp://windows_ip/path/ --user user:pass
```
### Windows → Linux (PowerShell)
```bash
curl -u user:pass ftp://linux_ip/path/file -o file
```
## 11. LFTP (Advanced FTP)

### Linux → Windows
```bash
lftp -e "put /local/file -o C:\\remote\\file; quit" ftp://user:pass@windows_ip
```
## 12. TFTP (Trivial FTP)

### Linux → Windows
```bash
tftp -i windows_ip put file
```
### Windows → Linux (PowerShell)
```bash
tftp -i linux_ip get file
```
## 13. Magic Wormhole

### Linux → Windows
```bash
wormhole send file
```
#### Windows receiver (PowerShell):
```bash
wormhole receive CODE
```
## 14. Azure/AWS CLI

### Linux → Windows via Cloud
```bash
az storage blob upload --file localfile --container mycontainer --name remotefile
```
### Windows download (PowerShell):
```bash
az storage blob download --container mycontainer --name remotefile --file localfile
```
## 15. WebDAV (curl)

### Linux → Windows
```bash
curl -T file https://windows_ip/webdav/path/ --user user:pass
```
### Windows → Linux (PowerShell)
```bash
curl -u user:pass https://linux_ip/webdav/file -o file
```
## 16. SSHFS (FUSE)

### Linux → Windows (Windows needs SSHFS-Win)
```bash
net use Z: \\sshfs\user@linux_ip!22/remote/path
```
## 17. NFS (Windows NFS Client)

### Windows → Linux
```bash
mount -o anon linux_ip:/path Z:
```
## 18. Base64 Encoding

### Linux → Windows
```bash
base64 file | ssh user@windows_ip "base64 -d > file"
```
### Windows → Linux (PowerShell)
```bash
[Convert]::ToBase64String([IO.File]::ReadAllBytes("C:\file")) | ssh user@linux_ip "base64 -d > file"
```
## 19. ZMODEM over SSH

### Linux → Windows
```bash
sz file | ssh user@windows_ip
```
### Windows → Linux (requires ZMODEM client)
```bash
rz | ssh user@linux_ip
```
## 20. WINDOWS NATIVE
### Bitsadmin (Windows download):
```bash
bitsadmin /transfer job /download /priority high http://linux_ip/file C:\file
```
### Certutil (Base64):
```bash
certutil -encode file encoded & scp encoded user@linux_ip:/path/
certutil -decode encoded file
```
# 21. PYTHON-BASED TRANSFER METHODS

Python provides several built-in ways to transfer files between systems.
Note: Python 2.x reached end-of-life in 2020 - Python3 is recommended.

## HTTP Server Method (Simple File Sharing)

### Linux → Windows Transfer:
Start HTTP server on Linux (Python3):
python3 -m http.server 8000
Or Python 2.x:
python -m SimpleHTTPServer 8000

Windows download options:
Using curl (if installed):
curl http://linux_ip:8000/file -o file
Using PowerShell:
Invoke-WebRequest http://linux_ip:8000/file -OutFile file
Using Python on Windows (Python3):
python -c "import urllib.request; urllib.request.urlretrieve('http://linux_ip:8000/file', 'file')"
Python 2.x:
python -c "import urllib; urllib.urlretrieve('http://linux_ip:8000/file', 'file')"

### Windows → Linux Transfer:
Start HTTP server on Windows (Python3):
python -m http.server 8000
Python 2.x:
python -m SimpleHTTPServer 8000

Linux download options:
wget http://windows_ip:8000/file
or:
curl -O http://windows_ip:8000/file
Using Python on Linux (Python3):
python3 -c "import urllib.request; urllib.request.urlretrieve('http://windows_ip:8000/file', 'file')"
Python 2.x:
python -c "import urllib; urllib.urlretrieve('http://windows_ip:8000/file', 'file')"

## Direct Socket Transfer (Advanced)

### Linux → Windows Transfer:
First on Windows (receiver):
Python3:
python -c "import socket; s=socket.socket(); s.bind(('0.0.0.0',1234)); s.listen(); c,a=s.accept(); open('file','wb').write(c.recv(999999))"
Python 2.x:
python -c "import socket; s=socket.socket(); s.bind(('0.0.0.0',1234)); s.listen(); c,a=s.accept(); open('file','wb').write(c.recv(999999))"

Then on Linux (sender):
Python3:
python3 -c "import socket; s=socket.socket(); s.connect(('windows_ip',1234)); s.send(open('file','rb').read())"
Python 2.x:
python -c "import socket; s=socket.socket(); s.connect(('windows_ip',1234)); s.send(open('file','rb').read())"

### Windows → Linux Transfer:
First on Linux (receiver):
Python3:
python3 -c "import socket; s=socket.socket(); s.bind(('0.0.0.0',1234)); s.listen(); c,a=s.accept(); open('file','wb').write(c.recv(999999))"
Python 2.x:
python -c "import socket; s=socket.socket(); s.bind(('0.0.0.0',1234)); s.listen(); c,a=s.accept(); open('file','wb').write(c.recv(999999))"

Then on Windows (sender):
Python3:
python3 -c "import socket; s=socket.socket(); s.connect(('linux_ip',1234)); s.send(open('file','rb').read())"
Python 2.x:
python -c "import socket; s=socket.socket(); s.connect(('linux_ip',1234)); s.send(open('file','rb').read())"

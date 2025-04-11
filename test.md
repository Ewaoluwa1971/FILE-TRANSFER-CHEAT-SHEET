markdown
Copy

# Linux ↔ Windows CLI File Transfer Methods

A comprehensive guide to transferring files between Linux and Windows systems using command-line tools.

## Table of Contents
1. [SCP (Secure Copy)](#1-scp-secure-copy)
2. [SFTP (SSH File Transfer)](#2-sftp-ssh-file-transfer)
3. [RSYNC over SSH](#3-rsync-over-ssh)
4. [FTP/FTPS](#4-ftpftps-command-line)
5. [SMB/CIFS](#5-smbcifs-client-mounts)
6. [Netcat](#6-netcat-unencrypted)
7. [PowerShell Remoting](#7-powershell-remoting-winrm)
8. [Tar over SSH](#8-tar-over-ssh)
9. [Rclone](#9-rclone-cross-platform-sync)
10. [cURL](#10-curl-various-protocols)
11. [LFTP](#11-lftp-advanced-ftp)
12. [TFTP](#12-tftp-trivial-ftp)
13. [Magic Wormhole](#13-magic-wormhole)
14. [Azure/AWS CLI](#14-azureaws-cli)
15. [WebDAV](#15-webdav-curl)
16. [SSHFS](#16-sshfs-fuse)
17. [NFS](#17-nfs-windows-nfs-client)
18. [Base64 Encoding](#18-base64-encoding)
19. [ZMODEM over SSH](#19-zmodem-over-ssh)
20. [Windows Native Tools](#20-windows-native-tools)
21. [Python Methods](#21-python-based-transfer-methods)

---

### 1. SCP (Secure Copy)
**Requires OpenSSH on Windows**

```bash
# Linux → Windows
scp /path/to/file user@windows_ip:"C:\\path\\to\\destination"

# Windows → Linux (PowerShell)
scp "C:\path\to\file" user@linux_ip:/path/to/destination

2. SFTP (SSH File Transfer)
bash
Copy

# Linux → Windows
sftp user@windows_ip:"C:\\path\\to\\remote" <<< $'put localfile'

# Windows → Linux (PowerShell)
sftp user@linux_ip:/remote/path <<< "get remotefile localfile"

3. RSYNC over SSH
bash
Copy

# Linux → Windows (rsync on Windows required)
rsync -avz -e ssh /local/path/ user@windows_ip:"C:\\remote\\path"

# Windows → Linux (PowerShell with rsync)
rsync -avz -e ssh user@linux_ip:/remote/path/ "C:\local\path"

4. FTP/FTPS Command Line
bash
Copy

# Linux → Windows
ftp -n windows_ip <<END_SCRIPT
quote USER username
quote PASS password
put localfile remotefile
quit
END_SCRIPT

# Windows → Linux (PowerShell)
(echo "USER username" && echo "PASS password" && echo "get remotefile localfile" && echo "quit") | ftp linux_ip

5. SMB/CIFS Client Mounts
bash
Copy

# Linux → Windows (Mount first)
sudo mount -t cifs //windows_ip/share /mnt/win -o username=user,password=pass
cp /local/file /mnt/win/path/

# Windows → Linux (PowerShell)
net use Z: \\linux_ip\share /user:username password
copy "C:\file" Z:\path\

6. Netcat (Unencrypted)
bash
Copy

# Linux → Windows
# Windows listener (PowerShell):
nc -lvnp 1234 > file
# Linux sender:
nc -w 3 windows_ip 1234 < file

# Windows → Linux
# Linux listener:
nc -l -p 1234 > file
# Windows sender (PowerShell):
nc linux_ip 1234 < file

7. PowerShell Remoting (WinRM)
powershell
Copy

# Windows → Linux
Copy-Item -Path "C:\file" -Destination "\\linux_ip\share\" -ToSession (New-PSSession -HostName linux_ip -UserName user)

8. Tar over SSH
bash
Copy

# Linux → Windows
tar czf - /folder | ssh user@windows_ip "tar xzf - -C C:\\path"

# Windows → Linux (WSL/PowerShell)
tar -cf - "C:\folder" | ssh user@linux_ip "tar xf - -C /path"

9. Rclone (Cross-platform sync)
bash
Copy

# Linux → Windows
rclone copy /local/path remote:windows_path --progress

# Windows → Linux
rclone copy remote:linux_path "C:\backup\" --progress

10. cURL (Various protocols)
bash
Copy

# Linux → Windows (FTP)
curl -T file ftp://windows_ip/path/ --user user:pass

# Windows → Linux (PowerShell)
curl -u user:pass ftp://linux_ip/path/file -o file

11. LFTP (Advanced FTP)
bash
Copy

# Linux → Windows
lftp -e "put /local/file -o C:\\remote\\file; quit" ftp://user:pass@windows_ip

12. TFTP (Trivial FTP)
bash
Copy

# Linux → Windows
tftp -i windows_ip put file

# Windows → Linux (PowerShell)
tftp -i linux_ip get file

13. Magic Wormhole
bash
Copy

# Linux → Windows
wormhole send file
# Windows receiver (PowerShell):
wormhole receive CODE

14. Azure/AWS CLI
bash
Copy

# Linux → Windows via Cloud
az storage blob upload --file localfile --container mycontainer --name remotefile
# Windows download (PowerShell):
az storage blob download --container mycontainer --name remotefile --file localfile

15. WebDAV (curl)
bash
Copy

# Linux → Windows
curl -T file https://windows_ip/webdav/path/ --user user:pass

# Windows → Linux (PowerShell)
curl -u user:pass https://linux_ip/webdav/file -o file

16. SSHFS (FUSE)
bash
Copy

# Linux → Windows (Windows needs SSHFS-Win)
net use Z: \\sshfs\user@linux_ip!22/remote/path

17. NFS (Windows NFS Client)
bash
Copy

# Windows → Linux
mount -o anon linux_ip:/path Z:

18. Base64 Encoding
bash
Copy

# Linux → Windows
base64 file | ssh user@windows_ip "base64 -d > file"
# Windows → Linux (PowerShell)
[Convert]::ToBase64String([IO.File]::ReadAllBytes("C:\file")) | ssh user@linux_ip "base64 -d > file"

19. ZMODEM over SSH
bash
Copy

# Linux → Windows
sz file | ssh user@windows_ip
# Windows → Linux (requires ZMODEM client)
rz | ssh user@linux_ip

20. Windows Native Tools
powershell
Copy

# Bitsadmin (Windows download):
bitsadmin /transfer job /download /priority high http://linux_ip/file C:\file

# Certutil (Base64):
certutil -encode file encoded & scp encoded user@linux_ip:/path/
certutil -decode encoded file

21. Python-Based Transfer Methods
HTTP Server Method
bash
Copy

# Linux → Windows
python3 -m http.server 8000  # Python 2: python -m SimpleHTTPServer 8000

# Windows download options:
curl http://linux_ip:8000/file -o file  # or PowerShell:
Invoke-WebRequest http://linux_ip:8000/file -OutFile file

# Windows → Linux
python -m http.server 8000  # Python 2: python -m SimpleHTTPServer 8000
wget http://windows_ip:8000/file  # or: curl -O http://windows_ip:8000/file

Direct Socket Transfer
bash
Copy

# Linux → Windows
# Windows receiver (Python3):
python -c "import socket; s=socket.socket(); s.bind(('0.0.0.0',1234)); s.listen(); c,a=s.accept(); open('file','wb').write(c.recv(999999))"
# Linux sender:
python3 -c "import socket; s=socket.socket(); s.connect(('windows_ip',1234)); s.send(open('file','rb').read())"

Python FTP Library
python
Copy

# Linux → Windows (Python3)
python3 -c "from ftplib import FTP; ftp = FTP('windows_ip'); ftp.login('user', 'pass'); open('file','rb') as f: ftp.storbinary('STOR C:\\\\path\\\\file', f); ftp.quit()"

Requirements Summary

    SSH: OpenSSH on Windows (Windows 10+ built-in)

    PowerShell: Available on Windows and Linux

    Python: Available on both systems

    Special packages: rsync, rclone, wormhole may need installation
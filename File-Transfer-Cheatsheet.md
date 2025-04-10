# ↔️ Linux ↔ Windows CLI File Transfer Methods ↔️

This document provides a comprehensive list of command-line methods for transferring files between Linux and Windows systems.

## 1. SCP (Secure Copy) - Requires OpenSSH on Windows

```bash
# Linux → Windows
scp /path/to/file user@windows_ip:"C:\\path\\to\\destination"
scp -r /path/to/directory/ user@windows_ip:"C:\\path\\to\\destination" # Recursive

# Windows → Linux (PowerShell)
scp "C:\path\to\file" user@linux_ip:/path/to/destination
scp -r "C:\path\to\directory\" user@linux_ip:/path/to/destination # Recursive

2. SFTP (SSH File Transfer)
Bash

# Linux → Windows
sftp user@windows_ip:"C:\\path\\to\\remote" <<< $'put localfile'
sftp user@windows_ip:"C:\\path\\to\\remote" <<< $'mput file1 file2 directory/' # Multiple files/directories

# Windows → Linux (PowerShell)
sftp user@linux_ip:/remote/path <<< "get remotefile localfile"
sftp user@linux_ip:/remote/path <<< "mget file* directory/" # Multiple files/directories

3. RSYNC over SSH (rsync on Windows required)
Bash

# Linux → Windows
rsync -avz -e ssh /local/path/ user@windows_ip:"C:\\remote\\path"
rsync -avz -e ssh /local/dir/ user@windows_ip:"C:\\remote\\path" # Recursive

# Windows → Linux (PowerShell with rsync)
rsync -avz -e ssh "C:\local\path\" user@linux_ip:/remote/path/
rsync -avz -e ssh "C:\local\dir\" user@linux_ip:/remote/path/ # Recursive

4. FTP/FTPS Command Line
Bash

# Linux → Windows (FTP)
ftp -n windows_ip <<END_SCRIPT
quote USER username
quote PASS password
put localfile C:\\remote\\file
quit
END_SCRIPT

# Linux → Windows (FTPS - requires `lftp`)
lftp -e "set ftp:ssl-protect-data true; put localfile -o C:\\remote\\file; quit" ftps://user:pass@windows_ip

# Windows → Linux (PowerShell - FTP)
(echo "USER username" && echo "PASS password" && echo "put C:\local\file remotefile" && echo "quit") | ftp linux_ip

# Windows → Linux (PowerShell - FTPS - requires `curl`)
curl -k -T "C:\local\file" ftps://user:pass@linux_ip/remotefile

5. SMB/CIFS Client Mounts
Bash

# Linux → Windows (Mount first)
sudo mount -t cifs //windows_ip/share /mnt/win -o username=user,password=pass
cp /local/file /mnt/win/path/
sudo umount /mnt/win

# Windows → Linux (PowerShell)
net use Z: \\linux_ip\share /user:username password
copy "C:\file" Z:\path\
net use Z: /delete

6. Netcat (Unencrypted - Use with Caution!)
Bash

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
PowerShell

# Windows → Linux
$session = New-PSSession -HostName linux_ip -UserName user
Copy-Item -Path "C:\file" -Destination "\\linux_ip\share\" -ToSession $session
Remove-PSSession $session

8. Tar over SSH
Bash

# Linux → Windows
tar czf - /folder | ssh user@windows_ip "tar xzf - -C C:\\path"
tar czf - file1 file2 | ssh user@windows_ip "tar xzf - -C C:\\path" # Multiple files

# Windows → Linux (WSL/PowerShell with tar)
tar -cf - "C:\folder" | ssh user@linux_ip "tar xf - -C /path"
tar -cf - "C:\file1" "C:\file2" | ssh user@linux_ip "tar xf - -C /path" # Multiple files

9. Rclone (Cross-platform sync)
Bash

# Linux → Windows
rclone copy /local/path remote:windows_path --progress
rclone sync /local/path remote:windows_path --progress # Sync changes

# Windows → Linux
rclone copy remote:linux_path "C:\backup\" --progress
rclone sync remote:linux_path "C:\backup\" --progress # Sync changes

10. cURL (Various protocols)
Bash

# Linux → Windows (FTP)
curl -T file ftp://user:pass@windows_ip/path/

# Linux → Windows (SFTP - requires `libcurl` with SFTP support)
curl -T file sftp://user:pass@windows_ip/path/

# Windows → Linux (PowerShell - FTP)
curl -u user:pass ftp://linux_ip/path/file -o file

# Windows → Linux (PowerShell - SFTP - requires `curl` with SFTP support)
curl -u user:pass sftp://linux_ip/path/file -o file

11. LFTP (Advanced FTP)
Bash

# Linux → Windows
lftp -e "put /local/file -o C:\\remote\\file; quit" ftp://user:pass@windows_ip
lftp -e "mput /local/files*; quit" ftp://user:pass@windows_ip # Multiple files

# Windows → Linux (requires LFTP in a Windows environment like Cygwin)
lftp -e "get remotefile -o C:\\local\\file; quit" ftp://user:pass@linux_ip
lftp -e "mget remote_files*; quit" ftp://user:pass@linux_ip # Multiple files

12. TFTP (Trivial FTP - Minimal security)
Bash

# Linux → Windows (requires TFTP server on Windows)
tftp -i windows_ip -c put localfile remotefile

# Windows → Linux (PowerShell - requires TFTP server on Linux)
tftp -i linux_ip -c get remotefile localfile

13. Magic Wormhole (Secure P2P)
Bash

# Linux → Windows
wormhole send file
# Windows receiver (PowerShell):
wormhole receive CODE

# Linux sender for directory:
tar czf - directory | wormhole send -

# Windows receiver for directory (PowerShell):
wormhole receive CODE | tar xzf -

14. Azure/AWS CLI (Cloud Storage Intermediate)
Bash

# Linux → Windows via Azure
az storage blob upload --file localfile --container mycontainer --name remotefile --account-name <account_name> --account-key <account_key>
# Windows download (PowerShell):
az storage blob download --container mycontainer --name remotefile --file C:\localfile --account-name <account_name> --account-key <account_key>

# Linux → Windows via AWS
aws s3 cp localfile s3://bucket/path/
# Windows download (PowerShell - requires AWS CLI)
aws s3 cp s3://bucket/path/remotefile C:\localfile

15. WebDAV (curl)
Bash

# Linux → Windows (requires WebDAV server on Windows)
curl -T file https://user:pass@windows_ip/webdav/path/file

# Windows → Linux (PowerShell - requires WebDAV server on Linux)
curl -u user:pass -o "C:\local\file" https://linux_ip/webdav/file

16. SSHFS (Mount Linux on Windows - Requires WinFsp + SSHFS-Win)
Code snippet

# Mount Linux directory on Windows
net use Z: \\sshfs\user@linux_ip!22/remote/path
# Access files via the Z: drive
copy Z:\remote\file C:\local\
net use Z: /delete

17. NFS (Windows NFS Client - Requires NFS Server on Linux)
Code snippet

# Windows → Linux
mount -o anon \\linux_ip\path\to\share Z:
# Access files via the Z: drive
copy C:\local\file Z:\
mount Z: /delete

18. Base64 Encoding (for small text-based files)
Bash

# Linux → Windows
base64 file | ssh user@windows_ip "base64 -d > file"

# Windows → Linux (PowerShell)
[Convert]::ToBase64String([IO.File]::ReadAllBytes("C:\file")) | ssh user@linux_ip "base64 -d > file"

19. ZMODEM over SSH (Requires ZMODEM clients on both sides)
Bash

# Linux → Windows
sz file | ssh user@windows_ip
# On Windows, a ZMODEM receiving program needs to be running.

# Windows → Linux (requires ZMODEM client like `rzsz` on Linux)
rz | ssh user@linux_ip
# On Windows, a ZMODEM sending program needs to be running.

20. WINDOWS NATIVE
PowerShell

# Bitsadmin (Windows download from Linux HTTP/FTP server):
bitsadmin /transfer job /download /priority high http://linux_ip/file C:\file
bitsadmin /transfer job /download /priority high ftp://linux_ip/file C:\file

# Certutil (Base64 for manual transfer):
certutil -encode "C:\file" encoded.txt
# Manually copy encoded.txt and then decode on Linux:
base64 -d encoded.txt > file

21. PYTHON-BASED TRANSFER METHODS
HTTP Server Method (Simple File Sharing)
Python

# Start HTTP server on Linux (Python3):
python3 -m http.server 8000
# Start HTTP server on Linux (Python 2.x):
python -m SimpleHTTPServer 8000

# Windows download (PowerShell):
Invoke-WebRequest http://linux_ip:8000/file -OutFile file

# Start HTTP server on Windows (Python3):
python -m http.server 8000
# Start HTTP server on Windows (Python 2.x):
python -m SimpleHTTPServer 8000

# Linux download:
wget http://windows_ip:8000/file

Direct Socket Transfer (Advanced)
Python

# Linux → Windows Transfer:
# First on Windows (receiver - Python3):
python -c "import socket; s=socket.socket(); s.bind(('0.0.0.0',1234)); s.listen(); c,a=s.accept(); with open('file','wb') as f: f.write(c.recv(999999))"
# Then on Linux (sender - Python3):
python3 -c "import socket; s=socket.socket(); s.connect(('windows_ip',1234)); with open('file','rb') as f: s.sendall(f.read())"

# Windows → Linux Transfer:
# First on Linux (receiver - Python3):
python3 -c "import socket; s=socket.socket(); s.bind(('0.0.0.0',1234)); s.listen(); c,a=s.accept(); with open('file','wb') as f: f.write(c.recv(999999))"
# Then on Windows (sender - Python3):
python -c "import socket; s=socket.socket(); s.connect(('linux_ip',1234)); with open('file','rb') as f: s.sendall(f.read())"

Python FTP Library Method
Python

# Linux → Windows (Python3):
python3 -c "from ftplib import FTP; ftp = FTP('windows_ip'); ftp.login('username', 'password'); with open('localfile', 'rb') as f: ftp.storbinary('STOR C:\\\\path\\\\remotefile', f); ftp.quit()"

# Windows → Linux (Python3):
python -c "from ftplib import FTP; ftp = FTP('linux_ip'); ftp.login('username', 'password'); with open('C:\\\\path\\\\remotefile', 'rb') as f: ftp.storbinary('STOR remotefile', f); ftp.quit()"

🛠️ Requirements Summary:

    SSH: OpenSSH on Windows (Windows 10+ built-in)
    PowerShell: Available on Windows and Linux
    Python: Available on both systems
    Special packages: rsync, rclone, wormhole, lftp, tftp, winfsp, sshfs-win, nfs-client (Windows), nfs-common (Linux), aws-cli, azure-cli may need installation.
    Network: Ensure firewalls allow necessary ports for each method.

Choose the method that best suits your needs, security requirements, and installed tools. Remember to replace placeholders with your actual IP addresses, usernames, passwords, and file paths.
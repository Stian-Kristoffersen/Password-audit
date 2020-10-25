# Powershell remoting from Kali Linux

## Install powershell on Kali Linux

$ sudo apt install powershell gss-ntlmssp

# Create some symlinks to make it work

$ ln -s /usr/lib/x86_64-linux-gnu/libssl.so.1.0.2 libssl.so.1.0.0
$ ln -s /usr/lib/x86_64-linux-gnu/libcrypto.so.1.0.2 libcrypto.so.1.0.0

## Check to see if Domain controller have PSRemoting enabled:

$ sudo nmap -p 5985 192.168.38.102

If TCP port 5985, then Powershell remoting is Enabled on the remote server.
Make sure to get Admin credential and password for connecting.

## Make a powershell session from Kali linux

$ pwsh
$ $mysession = New-PSSession -ComputerName 192.168.38.102 -Authentication Negotiate -Credential contoso/Administrator

Type password - if connected, please continue to run remote commands.

Eksample:

PS / user/home> Invoke-Command -Session $mysession -ScriptBlock {hostname} 

# Password Audit (Active Directory)

## Dump hashes from Active Directory (AD)

Tool: Impacket-Secretsdump

## Dump hashes 1 (Using SMBExec)
	$ sudo impacket-secretsdump -use-vss -exec-method smbexec -just-dc-ntlm -user-status contoso/Administrator:vagrant@192.168.38.102 -outputfile contoso-hashes

## Dump hashes 2 (Using WMIExec)
	$ sudo impacket-secretsdump -use-vss -exec-method wmiexec -just-dc-ntlm -user-status contoso/Administrator:vagrant@192.168.38.102 -outputfile contoso-hashes

## Dump hashes 3 (Default)
	$ sudo impacket-secretsdump -just-dc-ntlm -user-status contoso/Administrator:vagrant@192.168.38.102 -outputfile contoso-hashes
*Warning - potential reboot of domain controller if userbase is large

## Look for LM hashes (Crack those first)
	$ grep -cv aad3b435b5

if that command returns anything greater than zero, you have LANMan hashes in your pwdump file, and you have a problem that you need to fix. 


## Cleaning of hashfile

Remove Machine Accounts

	$ grep -e "^.*[\$]:" contoso-hashes.ntds (this list machine accounts)
	$ grep -e "^.*[\$]:" -v contoso-hashes.ntds > hashes-no-machine-accounts.ntds


Remove Disabled Accounts
	
	$ grep -e "Enabled" -v hashes-no-machine-accounts.ntds (this list disabled accounts)
	$ grep -e "Enabled" hashes-no-machine-accounts.ntds > hashes-no-disabled-accounts.ntds 


## Crack the hashes

Tool: Hashcat 

	$ sudo hashcat -a 0 -m 1000 {NTLM hashfile} {wordlist} -r {rule} -w3 -O
	Examples:

	$ sudo hashcat -a 0 -m 1000 hashes-no-disabled-accounts.ntds ~/Documents/wordlists/rockyou.txt -r ~/Documents/rules/OneRuleToRuleThemAll.rule -w3 -O

Write cracked hashes to a file
	
	$ sudo hashcat -a 0 -m 1000 hashes-no-disabled-accounts.ntds ~/Documents/wordlists/rockyou.txt -r ~/Documents/rules/OneRuleToRuleThemAll.rule -w3 -O --username --show > cracked.txt


## Analyze the results

Show any passwords occurring more than once:

	$ cat cracked.txt | grep : | cut -d: -f3 | grep -e '[^\s]' | sort | uniq -c | sort -rn | grep -v -e '^\s*1 '

Lowercase everything, and show identical passwords:

	$ cat cracked.txt | grep : | cut -d: -f3 | grep -e '[^\s]' | tr '[:upper:]' '[:lower:]' | sort | uniq -c | sort -rn | grep -v -e '^\s*1 '

 Lowercase everything, remove numbers, and show identical passwords

	$cat cracked.txt | grep : | cut -d: -f3 | grep -e '[^\s]' | tr -d '[:digit:]' | tr '[:upper:]' '[:lower:]' | sort | uniq -c | sort -rn | grep -v -e '^\s*1 '

Create a TOP 10 passwords

	$ cat cracked.txt | grep : | cut -d: -f3 | grep -e '[^\s]' | sort | uniq -c | sort -rn | grep -v -e '^\s*1 ' | head -10

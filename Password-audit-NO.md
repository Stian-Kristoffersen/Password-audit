# Passord gjennomgang (Active Directory)

Stegene for en passord gjennomgang:
1. Hente ut passord hasher fra Active Directory (AD)
2. Lete etter LM hasher
3. Fjerne maskin-kontoer og brukere som er deaktivert
4. "Cracking" av passord
5. Analyse av funn


## Hente ut passord hasher fra Active Directory

Verktøy: Impacket-Secretsdump fra Core Impact
https://github.com/SecureAuthCorp/impacket

$ sudo impacket-secretsdump -use-vss -just-dc-ntlm -user-status contoso/administrator:vagrant@192.168.38.102 -outputfile contoso-hashes


## Lete etter LM hasher

$ grep -cv aad3b435b5 contoso-hashes.ntds


## Fjerne maskin-kontoer og brukere som er deaktivert 

Fjerne maskin-kontoer

$ grep -e "^.*[\$]:" contoso-hashes.ntds (lister ut maskin-kontoer)
$ grep -e "^.*[\$]:" -v contoso-hashes.ntds > hashes-no-machine-accounts.ntds (fjerner maskin-kontoer og lager en ny fil)

Fjerne brukere som er deaktivert

$ grep -e "Enabled" -v hashes-no-machine-accounts.ntds (lister ut dektiverte brukere)
$ grep -e "Enabled" hashes-no-machine-accounts.ntds > hashes-no-disabled-accounts.ntds (fjerner deaktiverte brukere og lager en ny fil)


## Cracking av passordhasher

Verktøy: Hashcat
https://hashcat.net/hashcat/

$ sudo hashcat -a 0 -m 1000 hashes-no-disabled-accounts.ntds ~/Documents/wordlists/rockyou.txt -r ~/Documents/rules/OneRuleToRuleThemAll.rule -w3 -O

Lagre resultatet i en fil (cracked.txt)
	
$ sudo hashcat -a 0 -m 1000 hashes-no-disabled-accounts.ntds ~/Documents/wordlists/rockyou.txt -r ~/Documents/rules/OneRuleToRuleThemAll.rule -w3 -O --username --show > cracked.txt 


## Analyse av funn

Show any passwords occurring more than once:

$ cat cracked.txt | grep : | cut -d: -f3 | grep -e '[^\s]' | sort | uniq -c | sort -rn | grep -v -e '^\s*1 '

Lowercase everything, and show identical passwords:

$ cat cracked.txt | grep : | cut -d: -f3 | grep -e '[^\s]' | tr '[:upper:]' '[:lower:]'| sort | uniq -c | sort -rn | grep -v -e '^\s*1 '

Lowercase everything, remove numbers, and show identical passwords

$ cat cracked.txt | grep : | cut -d: -f3 | grep -e '[^\s]' | tr -d '[:digit:]'| tr '[:upper:]' '[:lower:]' | sort | uniq -c | sort -rn | grep -v -e '^\s*1 '

Create a TOP 10 passwords:

$ cat cracked.txt | grep : | cut -d: -f3 | grep -e '[^\s]' | sort | uniq -c | sort -rn | grep -v -e '^\s*1 ' | head -10 








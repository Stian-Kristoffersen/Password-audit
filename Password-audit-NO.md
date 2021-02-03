# Passordrevisjon av Active Directory

Stegene for en passordrevisjon:
1. Hente ut passord-hasher fra Active Directory (AD)
2. Lete etter LM-hasher
3. Fjerne maskinkontoer og brukere som er deaktivert
4. Knekking/gjetting av passord
5. Analyse av funn


## Hente ut passord-hasher fra Active Directory

Verktøy: Impacket-Secretsdump fra Core Impact
https://github.com/SecureAuthCorp/impacket

$ sudo impacket-secretsdump -just-dc-ntlm -user-status contoso/administrator:vagrant@192.168.38.102 -outputfile contoso-hashes


## Lete etter LM-hasher

Leter etter AD-objekter som IKKE inneholder LM-strengen, hvis resultatet er lik 0 (ingen LM-hasher), så er alt bra/OK.

$ grep -cv aad3b435b5 contoso-hashes.ntds

## Eksempel på en liste med både LM og NTLM-hasher
$ grep -e "aad3b435b5" -v LM_NTLM-hashes.ntds (lister ut LM-hasher)

$ grep -e "aad3b435b5" -v LM_NTLM-hashes.ntds > LM-hashes.ntds


## Knekking av LM-hasher (bruteforce)

$ sudo hashcat -a 3 -m 3000 LM-hashes.ntds


## Fjerne maskin-kontoer og brukere som er deaktivert 

Fjerne maskin-kontoer

$ grep -e "^.*[\$]:" contoso-hashes.ntds (lister ut maskin-kontoer)

$ grep -e "^.*[\$]:" -v contoso-hashes.ntds > hashes-no-machine-accounts.ntds (fjerner maskin-kontoer og lager en ny fil)

Fjerne brukere som er deaktivert

$ grep -e "Enabled" -v hashes-no-machine-accounts.ntds (lister ut dektiverte brukere)

$ grep -e "Enabled" hashes-no-machine-accounts.ntds > hashes-no-disabled-accounts.ntds (fjerner deaktiverte brukere og lager en ny fil)


## Knekking av passordhasher

Verktøy: Hashcat
https://hashcat.net/hashcat/

## Hashcat attack modes
0 -> Dictionary attack 
1 -> Combinator attack
3 -> Brute-force attack and Mask attack
6-7 -> Hybrid attack

$ sudo hashcat -a 0 -m 1000 hashes-no-disabled-accounts.ntds ~/Documents/wordlists/rockyou.txt -r ~/Documents/rules/OneRuleToRuleThemAll.rule -w3 -O

Lagre resultatet i en fil (badpass.txt)
	
$ sudo hashcat -a 0 -m 1000 hashes-no-disabled-accounts.ntds ~/Documents/wordlists/rockyou.txt -r ~/Documents/rules/OneRuleToRuleThemAll.rule -w3 -O --username --show > badpass.txt 


## Analyse av funn

Show any passwords occurring more than once:
$ cat badpass.txt | grep : | cut -d: -f3 | grep -e '[^\s]' | sort | uniq -c | sort -rn | grep -v -e '^\s*1 ' | more

Lowercase everything, and show identical passwords:
$ cat badpass.txt | grep : | cut -d: -f3 | grep -e '[^\s]' | tr '[:upper:]' '[:lower:]'| sort | uniq -c | sort -rn | grep -v -e '^\s*1 ' | more

Lowercase everything, remove numbers, and show identical passwords
$ cat badpass.txt | grep : | cut -d: -f3 | grep -e '[^\s]' | tr -d '[:digit:]'| tr '[:upper:]' '[:lower:]' | sort | uniq -c | sort -rn | grep -v -e '^\s*1 ' | more

Create a TOP 10 passwords:
$ cat badpass.txt | grep : | cut -d: -f3 | grep -e '[^\s]' | sort | uniq -c | sort -rn | grep -v -e '^\s*1 ' | head -10 



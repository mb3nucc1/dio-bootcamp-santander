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

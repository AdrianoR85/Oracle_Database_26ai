# 📘 Guia Completo: Instalando Oracle Database Free usando Docker

Este guia mostra passo a passo como instalar o **Oracle Database 23ai Free** usando Docker em uma máquina nova, incluindo explicações dos comandos e como criar um usuário para se conectar com clientes como VS Code.

---

## 🧩 1. Pré-requisitos

Antes de começar, você precisa:

* 🐳 **Docker instalado e funcionando**
* 💻 Linux, macOS ou Windows com WSL2 (se estiver usando Windows)

Para verificar se o Docker está instalado:

```bash
docker --version
```

Para testar se está funcionando:

```bash
docker ps
```

📌 **Se aparecer `permission denied`**, execute:

```bash
sudo usermod -aG docker $USER
newgrp docker
```

Isso dá permissão ao seu usuário para executar o Docker sem `sudo`.

---

## 🎯 2. Login no container registry da Oracle

Antes de baixar a imagem, você precisa estar logado:

```bash
docker login container-registry.oracle.com
```

Você usará a mesma conta do site da Oracle.

---

## 📥 3. Baixar a imagem do Oracle Database Free

```bash
docker pull container-registry.oracle.com/database/free:latest
```

Esse comando baixa a imagem do banco Oracle.

📌 **Explicação:**

* `docker pull` → baixa uma imagem do registry
* `latest` → baixa a versão mais recente disponível

---

## 🚀 4. Criar e iniciar o container

Agora vamos rodar o banco:

```bash
docker run -d \
  --name oracle-free \
  -p 1521:1521 -p 5500:5500 \
  -e ORACLE_PWD=SenhaForte123 \
  container-registry.oracle.com/database/free:latest
```

📌 Explicação:

* `-d` → roda em background
* `--name` → nome do container
* `-p` → mapeia portas para acesso externo
* `-e ORACLE_PWD` → define senha do usuário `SYS`

Para verificar se está rodando:

```bash
docker ps
```

---

## 🔧 5. Entrar no banco via terminal (sqlplus)

Execute:

```bash
docker exec -it oracle-free sqlplus sys/SenhaForte123@localhost:1521/FREEPDB1 as sysdba
```

Se conectar, você está dentro do banco Oracle.

---

## 👤 6. Criar um usuário para aplicações

Como estamos em uma PDB, usuários locais precisam ser criados **dentro da PDB**.

Primeiro, certifique-se de que está na `FREEPDB1`:

```sql
ALTER SESSION SET container = FREEPDB1;
```

Agora crie o usuário:

```sql
CREATE USER dev IDENTIFIED BY DevPass123;
```

Dar permissões:

```sql
GRANT CONNECT, RESOURCE TO dev;
ALTER USER dev QUOTA UNLIMITED ON USERS;
```

---

## 🧪 7. Testar login com o novo usuário

```bash
docker exec -it oracle-free sqlplus dev/DevPass123@localhost:1521/FREEPDB1
```

Se aparecer `SQL>`, deu certo! 🎉

---

## 🛠️ 8. Conexão usando VS Code

No VS Code:

| Campo              | Valor      |
| ------------------ | ---------- |
| Host               | localhost  |
| Porta              | 1521       |
| SID / Service Name | FREEPDB1   |
| Usuário            | dev        |
| Senha              | DevPass123 |

Depois clique **Connect**.

---

## 📌 Resumo rápido dos comandos

| Ação                     | Comando                                                                                                                 |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------------- |
| Login na Oracle Registry | `docker login container-registry.oracle.com`                                                                            |
| Baixar imagem            | `docker pull container-registry.oracle.com/database/free:latest`                                                        |
| Rodar container          | `docker run -d --name oracle-free -p 1521:1521 -e ORACLE_PWD=SenhaForte123 container-registry.oracle.com/database/free` |
| Entrar no SQL            | `docker exec -it oracle-free sqlplus ...`                                                                               |
| Criar usuário            | `CREATE USER dev IDENTIFIED BY DevPass123;`                                                                             |

---

### 🚀 Pronto! Seu ambiente Oracle está configurado.

Se quiser, posso agora:

* Criar um script `.sh` para automatizar tudo
* Criar cheatsheet de comandos Oracle

Quer? 🙂

---

## 🔧 Script Automatizado (install_oracle.sh)

```bash
#!/bin/bash

CONTAINER_NAME="oracle-db"
IMAGE="container-registry.oracle.com/database/free:latest"

# 1. Login no container registry (precisa ter conta Oracle)
echo "🔐 Faça login no Oracle Container Registry (site): https://container-registry.oracle.com"
echo "Depois execute: docker login container-registry.oracle.com"

# 2. Pull da imagem
echo "📦 Baixando imagem Oracle..."
docker pull $IMAGE

# 3. Criação do container
echo "🚀 Criando container..."
docker run -d \
  --name $CONTAINER_NAME \
  -p 1521:1521 \
  -e ORACLE_PWD=Oracle123! \
  -v oracle-data:/opt/oracle/oradata \
  $IMAGE

echo "⏳ Aguardando inicialização (isso pode levar alguns minutos)..."
sleep 30

echo "📄 Logs do container:"
docker logs -f $CONTAINER_NAME
```

> 💡 Execute com: `chmod +x install_oracle.sh && ./install_oracle.sh`

---

## 🧪 Testando conexão rápida

Após o container subir:

```bash
docker exec -it oracle-db sqlplus / as sysdba
```

---

## ❗ Problemas comuns & Soluções

| Erro                                   | Causa                          | Solução                                                                 |
| -------------------------------------- | ------------------------------ | ----------------------------------------------------------------------- |
| `ORA-65096`                            | Tentando criar usuário no CDB  | Executar `ALTER SESSION SET CONTAINER=FREEPDB1;` antes do `CREATE USER` |
| `ORA-01017: Invalid username/password` | Serviço errado ou senha errada | Confirmar Service Name = `FREEPDB1`                                     |
| Cliente não conecta                    | Porta bloqueada                | Verifique `docker ps` e se `1521` está mapeado                          |
| Conta bloqueada                        | Muitas tentativas erradas      | `ALTER USER user ACCOUNT UNLOCK;`                                       |

---

## 📚 Oracle SQL Cheatsheet

### Criar usuário

```sql
CREATE USER dev IDENTIFIED BY DevPass123!;
```

### Permissões

```sql
GRANT CONNECT, RESOURCE TO dev;
ALTER USER dev QUOTA UNLIMITED ON USERS;
```

### Criar tabela

```sql
CREATE TABLE person (
 id NUMBER GENERATED BY DEFAULT AS IDENTITY,
 name VARCHAR2(50) NOT NULL,
 created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Inserir

```sql
INSERT INTO person (name) VALUES ('Batman');
```

### Consultar

```sql
SELECT * FROM person;
```

### Apagar tabela

```sql
DROP TABLE person;
```

---

## ♻️ Gerenciamento do Container

### Parar container:

```bash
docker stop oracle-db
```

### Iniciar novamente:

```bash
docker start oracle-db
```

### Ver logs:

```bash
docker logs -f oracle-db
```

### Acessar terminal do container:

```bash
docker exec -it oracle-db bash
```

---

## 🧰 Resetar instalação (opcional)

Se quiser reinstalar do zero:

```bash
docker stop oracle-db
docker rm oracle-db
docker volume rm oracle-data
```

Depois execute novamente o script.

---

## 🎓 Extra: Configurar persistência manual

Se não criou volume antes, crie:

```bash
docker volume create oracle-data
```

E então suba o container usando o volume:

```bash
docker run -d \
 --name oracle-db \
 -p 1521:1521 \
 -v oracle-data:/opt/oracle/oradata \
 -e ORACLE_PWD=Oracle123! \
 container-registry.oracle.com/database/free:latest
```

---

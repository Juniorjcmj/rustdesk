# Documentação de Instalação do Servidor RustDesk (HBBS + HBBR)

## **Introdução**
Este documento detalha os passos necessários para instalar, configurar e proteger um servidor **RustDesk** auto-hospedado, utilizando **HBBS (Servidor de Sinalização)** e **HBBR (Servidor de Relay)**.

---

## **1. Baixar os Binários do Servidor RustDesk**
Execute os seguintes comandos para baixar os binários do RustDesk:

```sh
wget https://github.com/rustdesk/rustdesk-server/releases/latest/download/hbbs-linux-amd64
wget https://github.com/rustdesk/rustdesk-server/releases/latest/download/hbbr-linux-amd64
```

Dê permissão de execução aos arquivos baixados:
```sh
chmod +x hbbs-linux-amd64 hbbr-linux-amd64
```

---

## **2. Rodar os Serviços Manualmente (Teste Inicial)**
Para verificar se os binários estão funcionando corretamente, execute:

### **Iniciar o Servidor de Relay (HBBR)**
```sh
./hbbr-linux-amd64
```

### **Iniciar o Servidor de Sinalização (HBBS)**
Abra uma nova sessão do terminal e execute:
```sh
./hbbs-linux-amd64
```

Se ambos os serviços iniciarem sem erros, prossiga para a configuração automatizada.

---

## **3. Criar Serviços do Systemd para HBBS + HBBR**
Para garantir que os serviços sejam iniciados automaticamente no boot do sistema, criamos arquivos de serviço do **Systemd**.

### **Criar o serviço para HBBR**
```sh
sudo nano /etc/systemd/system/rustdesk-hbbr.service
```

Adicione o seguinte conteúdo:
```ini
[Unit]
Description=RustDesk HBBR (Relay Server)
After=network.target

[Service]
ExecStart=/root/hbbr-linux-amd64
Restart=always
User=root
Group=root
WorkingDirectory=/root

[Install]
WantedBy=multi-user.target
```

Salve e saia (`Ctrl + X`, `Y`, `Enter`).

### **Criar o serviço para HBBS**
```sh
sudo nano /etc/systemd/system/rustdesk-hbbs.service
```

Adicione o seguinte conteúdo:
```ini
[Unit]
Description=RustDesk HBBS (Signaling Server)
After=network.target

[Service]
ExecStart=/root/hbbs-linux-amd64
Restart=always
User=root
Group=root
WorkingDirectory=/root

[Install]
WantedBy=multi-user.target
```

Salve e saia (`Ctrl + X`, `Y`, `Enter`).

Agora, **recarregue o Systemd e habilite os serviços**:
```sh
sudo systemctl daemon-reload
sudo systemctl enable rustdesk-hbbr
sudo systemctl enable rustdesk-hbbs
sudo systemctl start rustdesk-hbbr
sudo systemctl start rustdesk-hbbs
```

Verifique se os serviços estão rodando corretamente:
```sh
sudo systemctl status rustdesk-hbbr
sudo systemctl status rustdesk-hbbs
```

---

## **4. Configurar Chave Secreta para Restringir o Acesso**
Para restringir o acesso ao servidor **somente a usuários autorizados**, precisamos configurar uma **chave de autenticação**.

### **1️⃣ Criar um Arquivo de Chave Secreta**
```sh
mkdir -p /root/rustdesk
openssl rand -base64 32 > /root/rustdesk/key
cat /root/rustdesk/key  # Copie esse valor
```

### **2️⃣ Editar o Serviço HBBS para Exigir a Chave**
Abra o arquivo de serviço HBBS:
```sh
sudo nano /etc/systemd/system/rustdesk-hbbs.service
```

Modifique a linha `ExecStart` para incluir a chave secreta:
```ini
ExecStart=/root/hbbs-linux-amd64 -k /root/rustdesk/key
```

Agora, **recarregue e reinicie o RustDesk HBBS**:
```sh
sudo systemctl daemon-reload
sudo systemctl restart rustdesk-hbbs
```

Agora, **somente usuários com a chave secreta poderão acessar o servidor**.

---

## **5. Liberar Portas no Firewall**
RustDesk precisa das portas **21115, 21116 e 21117** abertas. Para liberar no **UFW**:
```sh
sudo ufw allow 21115/tcp
sudo ufw allow 21116/tcp
sudo ufw allow 21117/tcp
sudo ufw reload
```

Se estiver usando **iptables**:
```sh
sudo iptables -A INPUT -p tcp --dport 21115 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 21116 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 21117 -j ACCEPT
```

---

## **6. Configurar o Cliente RustDesk**
Agora, no **cliente RustDesk**, configure o servidor:

- **Servidor:** `66.70.155.247:21115`
- **Chave Secreta:** (A mesma gerada no Passo 4)

Se estiver rodando no **Windows**, inicie o RustDesk com a chave:
```sh
rustdesk.exe --server 66.70.155.247:21115 --key "SUA_CHAVE_SECRETA"
```
Se estiver no **Linux**, inicie assim:
```sh
rustdesk --server 66.70.155.247:21115 --key "SUA_CHAVE_SECRETA"
```

---

## **7. Verificando Logs e Solucionando Problemas**
Para ver logs do RustDesk no servidor:
```sh
journalctl -u rustdesk-hbbs -f
journalctl -u rustdesk-hbbr -f
```
Se houver problemas, reinicie os serviços:
```sh
sudo systemctl restart rustdesk-hbbs
sudo systemctl restart rustdesk-hbbr
```

---

## **🚀 Conclusão**
Agora o **RustDesk Server** está configurado corretamente, rodando como serviço e **somente usuários autorizados com a chave secreta podem se conectar**.

Se precisar de ajustes adicionais, entre em contato! 😊

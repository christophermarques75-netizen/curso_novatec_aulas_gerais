## 1) Quem pode realizar um failover?

**Resposta:**  
O failover pode ser realizado de forma **automática** ou **manual**.

- **Automática:** pelo sistema automatizado, softwares de monitoramento, roteadores, firewalls ou servidores em nuvem.
- **Manual:** por profissionais como administradores de redes, analistas de infraestrutura, entre outros responsáveis pelo ambiente.

---

## 2) Quais tipos de backups são comuns no mercado?

**Resposta:**  
Os tipos de backup mais comuns são:

- **Backup completo**
- **Backup incremental**
- **Backup diferencial**
- **Backup em tempo real (CDP)**
- **Backup de imagem do sistema**
- **Backup local**
- **Backup em nuvem**
- **Backup híbrido**

---

## 3) Explique as diferenças entre backup local e na nuvem.

**Resposta:**

- **Backup local:** os dados ficam gravados em um dispositivo físico próximo de você, como HD, SSD, servidor local ou unidade externa.
- **Backup em nuvem:** os dados são armazenados em servidores de um provedor, geralmente em um data center, que disponibiliza espaço para guardar suas informações.

---

## 4) Quais comandos posso usar no MySQL para realizar backups automaticamente?

**Resposta:**  
É comum utilizar o comando `mysqldump` para gerar backups do banco de dados.

### Exemplo:
```bash
mysqldump -u usuario -p'senha' nome_banco_dados > /caminho/local/arquivo.sql

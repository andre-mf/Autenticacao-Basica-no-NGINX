## Autenticação básica no NGINX

#### 1. Criar um arquivo de senhas

Use a ferramenta htpasswd para criar um arquivo de senhas.

Crie um arquivo de senhas e adicione um usuário:

```shell
docker run --rm -v /home/usuario/auth:/auth -it httpd htpasswd -c /auth/.htpasswd username
```

Você será solicitado a fornecer uma senha para o usuário. Para adicionar mais usuários, use o comando sem o **`-c`**:

```shell
docker run --rm -v /home/usuario/auth:/auth -it httpd htpasswd /auth/.htpasswd additional_user
```

#### 2. Gerando os certificados usados no modo SSL

##### 1. **Criar uma Autoridade Certificadora (CA)**

Primeiro, criaremos uma CA que será usada para assinar os certificados do servidor e dos clientes.

1. **Gerar a chave privada da CA:**
   
   ```shell
   openssl genpkey -algorithm RSA -out ca.key -aes256
   ```

2. **Criar o certificado da CA:** 
   
   ```shell
   openssl req -x509 -new -nodes -key ca.key -sha256 -days 3650 -out ca.crt
   ```

Você será solicitado a fornecer algumas informações sobre a CA.



##### 2. **Criar o certificado do servidor**

1. **Gerar a chave privada do servidor:**
   
   ```shell
   openssl genpkey -algorithm RSA -out server.key  
   ```

2. **Gerar um CSR (Certificate Signing Request) para o servidor:**
   
   ```shell
   openssl req -new -key server.key -out server.csr  
   ```

3. **Assinar o CSR do servidor com a CA para obter o certificado do servidor:**
   
   ```shell
   openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 365 -sha256   
   ```

#### 3. Configurar o NGINX para usar autenticação básica

Edite a configuração do NGINX para incluir autenticação básica:

```nginx
server {
    listen 443 ssl;
    server_name exemplo.com;

    ssl_certificate /etc/nginx/ssl/server.crt;
    ssl_certificate_key /etc/nginx/ssl/server.key;

    location / {
        auth_basic "Restricted Area";
        auth_basic_user_file /etc/nginx/.htpasswd;
        root   /usr/share/nginx/html;
        index index.html;
    }
}
```

##### Explicação das diretivas

* **`auth_basic "Restricted Area";`**: Define a mensagem a ser exibida na janela de autenticação.
* **`auth_basic_user_file /etc/nginx/.htpasswd;`**: Especifica o caminho para o arquivo de senhas criado anteriormente.

#### 4. Acessar o site

Agora, quando um cliente tentar acessar o site, ele será solicitado a fornecer um nome de usuário e uma senha. Após a autenticação bem-sucedida, o cliente terá acesso ao site.
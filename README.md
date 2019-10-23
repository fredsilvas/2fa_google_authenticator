# 2fa_google_authenticator

##Preparação

Antes da Instalação, prepare o ambiente realizando os seguintes comandos:

```
  $ yum install -y epel-release
  
  $ yum install -y qrencode git pam-devel
  
  $ yum group install "Development Tools"
```

##Instalação Google Authenticator

Após a preparação do ambiente, faça o clone do projeto google_authenticator_libpam para sua máquina com o comando:
git clone https://github.com/google/google-authenticator-libpam.git

Execute a instalação conforme descrito no site do projeto.
```shell
./bootstrap.sh
./configure
make
sudo make install
```

Após instalado, crie um link simbólico da lib p


**Configuração Pós Instalação**

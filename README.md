# 2fa_google_authenticator

## Preparação

Antes da Instalação, prepare o ambiente realizando os seguintes comandos:

```
  $ yum install -y epel-release
  
  $ yum install -y qrencode git pam-devel
  
  $ yum group install "Development Tools"
```

## Instalação Google Authenticator

Após a preparação do ambiente, faça o clone do projeto google_authenticator_libpam para sua máquina com o comando:
`git clone https://github.com/google/google-authenticator-libpam.git`

Execute a instalação conforme descrito no site do projeto.
```shell
./bootstrap.sh
./configure
make
sudo make install
```

Após instalado, crie um link simbólico da lib do Google Authenticator para o local correto:

```
cd /root
ln -s google-authenticator-libpam/.libs/pam_google_authenticator.la /usr/lib64/security/pam_google_authenticator.so
```

**Configuração Pós Instalação**

Após a instalação, edite o arquivo `/etc/pam.d/sshd` inserindo ao final no arquivo a seguinte linha: 
`auth required pam_google_authenticator.so nullok`

Em seguida, faça a edição do arquivo `/etc/ssh/sshd_config` e altere a opção ChallengeResponseAuthentication para yes:
`ChallengeResponseAuthentication yes`

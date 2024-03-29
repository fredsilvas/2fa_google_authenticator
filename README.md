# 2fa_google_authenticator

## Preparação

Antes da Instalação, prepare o ambiente realizando os seguintes comandos:

```
  $ yum install -y epel-release
  
  $ yum install -y qrencode git pam-devel
  
  $ yum group install -y "Development Tools"
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

## Configuração Pós Instalação

Após a instalação, edite o arquivo `/etc/pam.d/sshd` inserindo ao final no arquivo a seguinte linha: 
`auth required pam_google_authenticator.so nullok`

Em seguida, faça a edição do arquivo `/etc/ssh/sshd_config` e altere a opção ChallengeResponseAuthentication para yes:
`ChallengeResponseAuthentication yes`


É necessário criar um token para cada usuário, e a melhor forma disso acontecer é no momento do login via ssh. Para isso, devemos deixar padronizado o script de geração de token, e que isso esteja automaticamente vinculado ao usuário.
A melhor forma disso acontecer, é atualizar o arquivo `/etc/skel/.bashrc`, pois ao criar um usuário, este fará uma cópia deste arquivo para a home do usuaário criado.
Para isso abra o arquivo `/etc/skel/bashrc` e copie ao final do arquivo as seguintes linhas:


```
#------------------------------------------
HOME=$HOME
FILE=/.google_authenticator
FULLFILE=$HOME$FILE


ctrlc_count=0
function no_ctrlc()
{
    let ctrlc_count++
    echo
    if [[ $ctrlc_count == 1 ]]; then
        echo ""
        echo "Não é possível continuar o login sem a criação do token de segurança!"
        echo "Você Será Deslogado!"
        sleep 5
        PID=`pgrep -U $LOGNAME -f sshd`
        #echo $PID
        kill $PID
        exit
    fi
}

if [ ! -f $FULLFILE ]; then

    trap no_ctrlc SIGINT
    echo ""
    echo ""
    echo "Atenção... este é seu primeiro login via SSH, e será necessário criar um token para autenticação em 2 etapas."
    echo "Antes de continuar, faça a instalação em seu celular de algum aplicativo TOTP, como por exemplo: Google Authenticator, TOTP Authenticator ou FreeOTP Authenticator"
    echo "Sem a criação do token não será possível realizar o login"
    echo "Siga as instruções da tela para concluir com sucesso."
    echo "Em caso de problemas ou dificuldades, entre em contato com xxxx@xxxx.xxx.xx"
    echo ""
    echo -n "Digite 'y ou Y' para continuar, ou qualquer outra tecla para cancelar e ser deslogado: "
    read input

    if [[ $input == "Y" || $input == "y" ]]; then
    #   trap no_ctrlc SIGINT
        google-authenticator -t -d -f -r3 -R30 -W
        OUT=$?

        if [ $OUT -eq 0 ]; then
            echo ""
            echo ""
            echo "Você criou a sua chave de segurança com sucesso! Seu usuário será deslogado e para a realização do login utilizando 2FA."
            sleep 5
            PID=`pgrep -U $LOGNAME -f sshd`
            #echo $PID
            kill $PID
        fi

    else
        echo ""
        echo "Não é possível continuar o login sem a criação do token de segurança!"
        echo "Você Será Deslogado!"
        sleep 5
        PID=`pgrep -U $LOGNAME -f sshd`
        #echo $PID
        kill $PID
        exit
    fi

fi

#------------------------------------------
```

## Instalação no Celular

Para geração do token, instale no celular (Android), qualquer um dos aplicativos TOTP (Time-based One-time Password). Os aplicativos abaixo foram testados e funcionam corretamente.

* Google Authenticator (https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2&hl=en)
* TOTP Authenticator - 2FA with Backup & Restore (https://play.google.com/store/apps/details?id=com.authenticator.authservice2&hl=en)
* FreeOTP Authenticator (https://play.google.com/store/apps/details?id=org.fedorahosted.freeotp&hl=en)

Após instalar o APP, scaneie o QR-Code mostrado ao configurar o Token.


## Uso

Ao logar no ssh de uma máquina, informe primeiramente a sua senha de acesso. Caso ainda não tenha configurado o token, o sistema criará o token e mostrará o QR-Code para ser scaneado pelo celular.
Caso já exista token criado, após informar corretamente a senha, será solicitado o token (gerado pelo APP).
Informe o token gerado e se correto, será feito o login.


## TODO


* Reabilitar o SELinux, colocando apenas as regras para funcionamento do 2FA adequadamente;

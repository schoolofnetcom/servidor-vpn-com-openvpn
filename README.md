<h3><u>Instalando o OpenVPN</u></h3>


```$ apt-get update```

```$ sudo apt-get install openvpn```


<h3><u>Configurando o CA</h3></u>

Instalação do Easy-RSA


```$ wget https://github.com/OpenVPN/easy-rsa/releases/download/v3.0.7/EasyRSA-3.0.7.tgz```

```$ tar xvzf EasyRSA-3.0.7.tgz```

```$ cd EasyRSA-3.0.7```

```$ cp vars.example vars```

```$ vim vars```

Mudar os dados da empresa nas linhas 95-100 e salvar

Iniciar o PKI

```$ ./easyrsa init-pki```

Iniciar o CA

```$ ./easyrsa build-ca nopass```

Deixar o campo Common Name em branco

Com isto são criados o ca.crt, que é o arquivo de certificado público para garantir a autenticação de servidor/cliente na mesma rede, e o arquivo /private/ca.key, que é usado para assinar chaves e certificados entre servidores e clientes para ficar apenas na máquina CA.

<h3><u>Configurando chaves para o servidor do OpenVPN</h3></u>


Crie uma outra pasta e execute os seguintes comandos

```$ wget https://github.com/OpenVPN/easy-rsa/releases/download/v3.0.7/EasyRSA-3.0.7.tgz```

```$ tar xvzf EasyRSA-3.0.7.tgz```

```$ cd EasyRSA-3.0.7```

```$ ./easyrsa init-pki```

```$ ./easyrsa gen-req server nopass```

Enter para manter o nome 'server'

```$ cd /<path>/EasyRSA-3.0.7/pki/reqs/```

Copiar o arquivo server.req para o servidor via WinSCP ou outro gerenciador para a CA

Também copiar o arquivo server.req para a autoridade de certificação.

```$ cd ..```

```$ cd private```

Copiar o arquivo server.key para o diretorio do openvpn

```$ cp server.key /etc/openvpn/```

Feito isto, verifique com o servidor que possue a Autoridade de Certificação se o pedido é válido.

Importe a requisição para o CA, na pasta EasyRSA-3.0.7/

Na sequência, execute o seguinte comando para importar um pedido em certificado.

```$ ./easyrsa import-req server.req server```

Assinar a solicitação para emissão de certificado.

```$ ./EasyRSA sign-req server server```

Com isto o certificado é assinado, gerando um arquivo server.crt, considerando o servidor como válido/confiável.

Copiar o certificado do CA EasyRSA-3.0.7/pki/issued/server.crt e o arquivo EasyRSA-3.0.7/pki/ca.crt para os outros servidores no diretório /root/

No ambiente do servidor, copiar ambos os arquivos para /etc/openvpn/

```$ cp ca.crt server.crt /etc/openvpn/```

Para proteger a troca de chaves entre redes, e necessário gerar uma chave de Diffie-Hellman.

Em ./ReasyRSA-3.0.7, digite o seguinte comando:

```$ ./esayrsa gen-dh```

Com o processo concluído, gere outra chave HMEC para garantir a verificação de integridade TLS do servidor.

```$ openvpn --genkey --secret ta.key```

Copiar o ta.key e o dh.pem para o penvpn

```$ cp ta.key /etc/openvpn/```

```$ cp pki/dh.pem /etc/openvpn/```

Com isto, as chaves para o servidor estão prontas.

<h3><u>Gerando chaves do cliente</u></h3>


No servidor do OpenVPN, crie um diretorio no home

```$ mkdir -p client-conf/keys```

```$ chmod 700 -R cliente-conf/```

Navegue para o diretório /EasyRSA-3.0.7 para gerar o certificado do cliente

```$ ./easyrsa gen-req client1 nopass```

Pressione enter para deixar o nome padrão.

Copie a chave client1.key para o diretório cliente-conf/keys

```$ cp pki/private/cliente1.key ~/cliente-conf/keys/```

Copie o cliente1.req localizado na pasta pki/reqs/cliente1.req para o servidor de CA e para a gere outra cópia para a própria pasta EasyRSA-3.0.7/

Agora é necessário importar o client1.req para realizar a assinatura do certificado

```$ ./easyrsa import-req cliente1.req client1```

Para assinar a requisição, execute o seguinte comando:

```$ ./easyrsa sign-req cliente client1```

Confirmar a solicitação e o certificado para o cliente foi gerado.

Copiar o arquivo client1.crt gerado em pki/issued/ para a raiz do servidor do OpenVPN.

Copiar o cliente1.req do servidor do OpenVPN para o diretório client-conf/keys/

Copiar os arquivos ca.crt e ta.key para o cliente para que o servidor CA valide o cliente.

```$ cp /etc/openvpn/ca.crt ~/client-conf/keys/```

```$ cp /etc/openvpn/ta.key ~/client-conf/keys/```

<h3><b>Configurando serviço do OpenVPN</b></h3>


Configurar o serviço do OpenVPN para usar as credenciais.

```$ cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz /etc/openvpn/```

```$ cd etc/openvpn/```

```$ gzip -d /etc/openvpn/server.conf.gz```

```$ vim server.conf```

Ao abrir o arquivo, retire o comentário das linhas

```tls-auth ta.key 0```

```cipher AES-256-CBC```

Abaixo da linha ```cipher AES-256-CBC```, adicione a seguinte linha:

```auth SHA256```

Localize a linha com o seguinte texto:

```dh dh2048.pem```

E substitua-o para:

```dh dh.pem```

Verifique se as linhas:

```ca ca.crt```

```cert server.crt```

```key server.key```

tem o mesmo nome dos arquivos gerados anteriormente.

Busque as linhas que contêm:

```;user nobody```

```;group nogroup```

E remova o sinal de comentário ";", salve o arquivo e saia do editor.


<h3><u>Ajustando configurações de rede</u></h3>


No servidor OpenVPN, acesse ao arquivo:

```$ vim /etc/sysctl.conf```

No arquivo, busque pela linha:

```net.ipv4.ip_forward=1```

E remova o sinal de comentario "#" e execute o comando

```sysctl -p```

Esteja certo que a porta 1154 está livre para o OpenVPN usar para as conexões.


<h3><u>Inicializando o OpenVPN</u></h3>


No servidor OpenVPN execute o seguinte comando:

```$ sudo systemctl start openvpn@server```

```$ sudo systemctl status openvpn@server```

Com isto o servidor OpenVPN está configurado com sucesso. Com o comando

```$ ifconfig```

É possível verificar o 'tun0' ativo com uma interface de rede disparando novos endereços de IP disponível no servidor.

Para configurar a inicialização automática na configuração do servidor, execute o comando:

```$ sudo systemctl enable openvpn@server```

Para criar a configuração do cliente


<h3><u>Configurando a infraestrutura dos clientes</u></h3>


No servidor OpenVPN crie uma pasta para configuração dos clientes:

```$ mkdir client-conf/files```
```cp /usr/share/dop/openvpn/examples/sample-config-files/client.conf ~/client-conf/```
```vim client-conf```

No arquivo, busque a linha com a seguinte descrição:

```remote my-server-1 1194```

Modifique para o endereço para o do OpenVPN

```remote 000.000.000.000 1194``` Onde o endereço ip deve ser o do servidor vpn

Localize a linha

```;user nobody"``` 

e 

```";group nogroup"```

 e remova o símbolo de comentário ";"

 Busque as linhas:

```ca ca.crt```

```cert client.crt```

```key client.key```

e comente-as usando "#" na frente de cada linha.

Busque a linha:

```tls-auth ta.key 1```

e comente usando "#" na frente da linha.

Localize a linha:

```cipher AES-256-CBC```

e abaixo dela, coloque as seguintes linhas:

```auth SHA256```
```key-direction 1```

Salve e saia do arquivo.

Com a seguinte configuração, vamos concatenar todos os arquivos em um único arquivo

```vim make config.sh```

Inclua o seguinte texto:
```
KEY_DIR=~/client-conf/keys
OUTPU_DIR=~/client-conf/files
BASE_CONFIG=~/client-conf/base.conf

cat ${BASE_CONFIG} \
    <(echo -e '<ca>')> \
    ${KEY_DIR}/ca.crt \
    <(echo -e '<\ca>\n<cert>')> \
    ${KEY_DIR}/${1}.crt \
    <(echo -e '</cert>\n<key>')> \
    ${KEY_DIR}/${1}.key \
    <(echo -e '</key>\n<tls-auth>')> \
    ${KEY-DIR}/ta.key \
    <(echo -e '</tls-auth>')> \
    > ${OUTPUT_DIR}/${1}.ovpn
```

Salve o arquivo e feche

Execute o script:

``` ./make config.sh client1```

```cd files```

O arquivo client1.ovpn foi criado.

Envie este arquivo para a pasta do openvpn do cliente.


<h3><u>Conectando na VPN pelo cliente Linux</u></h3>


Uma vez no cliente-linux, instale o pacote do OpenVPN


```apt-get update```

```apt-get install openvpn```

Copie o arquivo client1.ovpn

```cp client1.ocpn /etc/openvpn/```

Abra o arquivo client.ovpn

```vim client.ovpn```

Localize a linha

```verb 3```

Abaixo dela, insira as seguintes linhas:

```script-security 2```
```up /etc/openvpn/update-resolv-conf```
```down /etc/openvpn/update-resolv-conf```

Salve e saia do arquivo.

Execute o comando:

```openvpn --config client1.ovpn```

Com isto, a conexão VPN.

Para testar, execute o seguinte comando:

```ifconfig```

Localize o primeiro ip na linha abaixo do "tun0", iniciando em "inet", e copie o ip.

Execute o ping:

```ping <ip>``` onde <ip> é o ip copiado no passo anterior.

Verifique que existe a comunicação via túnel criptografado.

Edite o arquivo padrão do OpenVPN

```vim /etc/default/openvpn```

No arquivo, localize a linha:

```#AUTOSTART="all"```

Remova o "#" para habilitar a função para que o OpenVPN possa executar todos os arquivos conf.

Renomeie o arquivo client1.ovpn da seguinte maneira:

```cp client1.ovpn client.conf```

Em seguida execute o comando:

```systemctl deamon-reload```
```systemcrl start openvpn@client.service```
```systemcrl status openvpn@client.service```

Agora o log exibe o VPN inicializado com sucesso, rodando em background.


<h3><u>Conectando pelo VPN pelo Windows</u></h3>


Baixe o OpenVPN no Windows pelo site apenvpn.net/community-downloads/

Baixe o executavel de acordo com a sua versão do Windows.

Avance até finalizar a instalação.

Copie o arquivo clien1.ovpn gerado anteriormente na pasta OpenVPN/config em Arquivos de Programas.

Abra o programa OpenVPN como administrador.

Na barra inferior, proximo ao relógio, localize o OpenVPN, clique no botão direito do mouse e cliete em conectar

Ele detectou o arquivo e já conectou na VPN com sucesso.


<h3><u>Revogando Certificados</u></h3>


Acesse o servidor CA

Execute o seguinte comando:

```./easyrsa revoke client1```

Confirme com "yes"

Desta maneira, o certificado fica revogado.

Contudo, umas lista de revogação de sertificados precisa ser criado no servidor CA con o seguinte comando:

```./easyrsa gen-crl```

O arquivo pki/crl.pem é criado. Transfira este arquivo para o servidorVPN e na pasta home do servidor CA.

Transfira o arquivo também para a pasta /etc/openvpn/ do servidor VPN.

Abra o arquivo de configuração do OpenVPN.

```vim /etc/openvpn/server.conf```

Localize a linha:

```explicit-exit-notify 1```

Abaixo dela, coloque a seguinte linha:

```crl-verify crl.pem```

Salve e saia do arquivo.

Execute o seguinte comando para dar efeito às alterações*:

```systemctl restart openvpn@server```
```systemctl status openvpn@server``` para verificar o resultado.

*verifique se o permissão está em 600 no crl.pem.

Desta maneira, o cliente perde o acesso ao OpenVPN.

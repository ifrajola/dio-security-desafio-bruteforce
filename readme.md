*Desafio BruteForce DIO*

Criado ambiente virtualizado via VirtualBox
Utilizando Kali (Medusa) + Metasploitable 2 (DVWA)

Simulação de ataque BruteForce em ambiente vulnerável

\*Etapas\*
\*Configurando ambiente\*
\*1 - Baixar VirtualBox\*
Link: https://www.virtualbox.org/wiki/Downloads

\*2 - Baixar a imagem do Kali Linux preparado para ambiente virtualizado\*
Link: https://www.kali.org/get-kali/#kali-virtual-machines

\*3 - Baixar Metasploitable 2 através do site Rapid7 ou acessando o SourceForge\*
Link1: https://docs.rapid7.com/metasploit/metasploitable-2/
Link2: https://sourceforge.net/projects/metasploitable/

\*4 - Criar VM Kali Linux com o arquivo vbox já baixado\*
Após baixar na etapa 2, volte ao Virtual box e selecione a opção "open/abrir" e selecione o arquivo ".vbox". E siga as opções para configurar a VM
Clique na máquina Kali listada, navegue até Configurações > Rede > Altere a opção "Connected to/Ligado a" para "Host Only / Placa de rede exclusiva de hospedeiro

\*5 - Criar VM Metasploitable 2 utlizando o arquivo de disco vmdk\*
Com o arquivo baixado na etapa 2, volte ao Virtual box e selecione a opção "new/novo" e configure a máquina com as opções
VM Name: metasploitable-2
OS: Linux
OS Distribution: Ubuntu \*IMPORTANTE: Se não configurar isso, a VM pode dar Kernel Panic ao iniciar\*
Vá até a ultima opção em "Create new virtual disk / Criar novo disco virtual" e selecione o disco ".vmdk" baixado.
Navegue até Configurações > Rede > Altere a opção "Connected to/Ligado a" para "Host Only / Placa de rede exclusiva de hospedeiro

\*Iniciando as configurações e ataques\*
\*6 Ligue as VMs\*
Acesse as máquinas com os usuários padrão de cada VM

\*Kali Linux\*
Usuário: kali
Senha: kali

\*Metasploitable2\*
Usuário: msfadmin
Senha: msfadmin

\*7 - Colete as informações de ip da máquina vulneravel\*
Na VM metasploitable2, execute o comando
ip a
Após esse comando o ip da máquina será informado (veja a pasta images se precisar de apoio "metasplitable1.jpg")
Neste simulado a máquina estava com o IP 192.168.56.102

\*7 Enumeração com Kali\*
Acessando sua máquina Kali e com a informação do IP comece a realizar as primeiras configurações 
Abra o terminal da máquina utilizando o conjunto de teclas (Ctrl + Alt + T)
Antes de iniciar o teste de ataque verique se a máquina está acançável via comando ping 
"ping IP" / Nesse simulado o comando será "ping 192.168.56.102"
Cancele o comando utilizando "Ctrl + C"

Verificado que a VM está acessível pela rede, iniciaremos nossa etapa de enumeração
Execute o comando para utilizar a ferramenta de enumeração nativa do Kali Enum4Linux
"enum4linux -a 192.168.56.102 | tee enum4\_output.txt"

Esse comando irá realizar a enumeração das tecnologias presentes no ambiente, e a saída ficará visível no terminal, e também no arquivo "enum4\_output.txt" que configuramos utilizando "| tee" na frente do IP
(Verifique a pasta nas images "enum1.jpg / enum2.jpg")

Também vamos usar o nmap, outra ferramanta de investigação de redes para coletar mais informações do alvo a fim de entender se há alguma tecnologia vulnerável, para isso devemos utilizar mais parâmetros no comando, em nosso ambiente usaremos o comando abaixo:
"nmap -sV -p 21,22,80,445,139 192.168.56.102"
O comando acima invoca o Nmap para realizar a busca no alvo (192.168.56.102) e utiliza o parâmetro "-p" para definir as portas que iremos buscar, uma busca maior de portas levaria mais tempo, e essas portas informadas são mais comuns de estarem com programas vulneráveis. O "-sV" do comando retorna qual a versão da tecnologia utilizada naquela porta (Veja a imagem "enum3.jpg")
As portas que estamos testando são as portas padrão dos protocolos SMB, SSH, Telnet, HTTP, FTP, e NetBIOS. 

\*8 Criando Wordlists para ataques\* 
Com as informações carregadas, precisamos iniciar nossas configurações do ambiente para utilizar o software Medusa
Como é um ambiente iremos utilizar o material com menos usuários fornecidos pela DIO, mas em caso de ambientes e testes maiores, procurar por "Common10k" no google fornece listas com grandes números de usuários / senhas para ataques de bruteforce mais elaborados. Segue um para estudo posterior:
https://github.com/danielmiessler/SecLists/blob/master/Passwords/Common-Credentials/10k-most-common.txt

Vamos criar a primeira lista de usuários que utilizaremos em nossos ataques

"echo -e "user\\nmsfadmin\\nadmin\\nroot" > users.txt"
Vai criar a lista de usuários
"echo -e "123456\\npassword\\nqwerty\\nmsfadmin" > pass.txt"
Vai criar a lista de senhas
PS.: O "\\n" presente em cada comando é muito importante estar no lugar correto, pois ele determina uma "quebra de linha" o que faz cada usuário ser lido por vez. Ao invés de "\\nmsfadmin" o software transforma o "\\n" em um "enter" e o texto vira "msfadmin"
(Veja a imagem "list1.jpg" para ver o como fica a lista utilizando um leitor de texto no linux)

Iremos utilizar essa lista posteriormente, mas já deixamos criada para acelerar o processo:
"echo -e "user\\nmsfadmin\\nservice" > smb\_users.txt"
"echo -e "password\\n123456\\nWelcome123\\nmsfadmin" > senhas\_spray.txt"


\*9 Realizando o primeiro ataque de força bruta com Medusa em um servidor ftp\*
Agora que já temos IP da máquina a ser atacada, informações enumeradas do alvo, uma lista de usuários e senhas para tentativas de ataque, vamos a primeira tentativa, utilizando o protocolo ftp

A ferramenta Medusa tem a capacidade de realizar diversos ataques simultâneos utilizando listas pré-determinadas e parâmetros configurados para o ataque.
Nesse momento inicial testaremos com o comando abaixo:
"medusa -h 192.168.56.102 -U users.txt -P pass.txt -M ftp -t 6"
O comando tem parâmetros para definir o host que será atacado "-h IP", a lista de usuário "-U arquivo.txt", a lista de senhas "-P arquivo.txt" o procolo/método de ataque "-M" e quantidade de tentativas por segundo "-t" 
Dessa lista sairá um usuário e senha vulnerável (msfadmin/msfadmin) que iremos acessar utilizando o comando abaixo
"ftp 192.168.56.102"
Após esse comando inserir usuário e senha testados e verificar que o acesso estará feito (Imagem "ftp1.jpg" está disponível para apoio na pasta images)

\*10 Coletando informações de formulário web vulnerável\*
Com nosso primeiro ataque feito, utilizaremos nosso arsenal de usuário e senha para um novo ataque, mas agora em uma página web vulnerável.
como verificamos no comando nmap a porta 80 aberta para ser atacada, o serviço web está exposto e com uma ferramenta chamada DVWA
DVWA é uma ferramenta para estudos que fornece uma aplicação web com diversas vulnerabilidades para serem estudadas e exploradas
Sabendo que já temos lista de usuário e senha precisamos da URL e da página de forms para acesso via navegador:
http://192.168.56.102/dvwa/login.php 
Essa é a URL que será atacada. Ao cair na página de login e tentar usar um usuário qualquer, vejamos que a página retorna um "login failed". Essa é a informação que precisamos para o ataque.
Junto com isso, vamos coletar qual é a requisição enviada para o servidor para validar  se usuário e senha estão corretos. No navegador, utilize as teclas "Ctrl + Shift + I) e irá abrir o painel de depuração das páginas web do navegador (Firefox, nesse simulado).
Navegue no painel até Network > clique na lista na página que retornou o protocolo 302, e navegue até a opção "request". Precisamos disso para coletar todos os parâmetros desse novo ataque. (Veja a imagem "web1.jpg" para apoio)

\*11 Montando ataque via Hydra para Web\*
Com as informações coletadas acima, voltamos ao terminal e utilizaremos o Hydra para esse novo ataque. Uma ferramenta com mais robustez para ataques web. Os parâmetros serão diferentes para atacar esse novo ambiente
"hydra 192.168.56.102 http-post-form "/dvwa/login.php:username=^USER^\&password=^PASS^\&Login=Login:Login failed" -L users.txt -P pass.txt -vV"
Agora estamos passando além de host, usuário e senha, vamos passar um novos parâmetros para informar onde na página iremos inserir as informações. O "Login failed" é o que determina a falha ou sucesso nesse caso. Se receber a informação diferente de falha, significa que conseguimos uma credencial para acessar o ambiente. (Veja a imagem "web2.jpg" que apresenta o teste executado)
Com isso, conseguiremos mais uma credencial vulneravel para acesso ao ambiente web "admin, password" e acessaremos o ambiente (Imagem "web3.jpg" disponível para apoio)

\*12 Ataque de bruteforce em SMB usando medusa\*
Com a lista de usuário e senha que fizemos na etapa 7, agora vamos para um ataque em um SMB, utilizaremos de recursos já apresentados anteriormente para tentativa de ataque, com parâmetros alterados para protocolo SMB
"medusa -h 192.168.56.102 -U smb\_users.txt -t 2  -P senhas\_spray.txt -M smbnt -t 2 -T 50" 
Essa lista foi criada através da enumeração na etapa 7, e agora passamos os parametros para SMB no -M e mudamos a quantidade de hosts para 50 por vez. No nosso simulado não é necessário, mas é de grande recomendação para ambientes reais.
Ao rodar o comando iremos ter o retorno de uma "ACCOUNT FOUND" e assim conseguiremos acessar o ambiente via SMB (msfadmin, msfadmin)
Acessaremos usando o comando
"smbclient -L //192.168.56.102 -U msfadmin" depois iremos inserir a senha verificada acima.
(O acesso está apresentado na imagem "smb1.jpg")

Assim finalizamos esse simulado de ataques com 3 ataques diferentes apresentados: FTP, WEB(HTTP) e SMB.



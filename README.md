# Ataque Brute Force em diferentes servicos (FTP,Web,SMB) com Kali Linux e Medusa
## DESAFIO Dio.me.
### simulando um Ataque de Brute Force de Senhas com Medusa e Kali Linux, utilizando diferentes serviços (FTP, Web, SMB) em ambiente controlado, para simular cenários de ataque de força bruta e exercitar medidas de prevenção.

### :computer: Configuração do Ambiente de Laboratório
* Para a simulação dos cenários de ataque, foram utilizadas duas máquinas virtuais: Kali Linux e Metasploitable 2.
O Metasploitable foi executado no VirtualBox utilizando rede interna (host-only), enquanto o Kali Linux foi configurado no Windows Subsystem for Linux (WSL).

* #### Configuração do Metasploitable 2 (VirtualBox)

1- Baixe a imagem do Metasploitable 2 (distribuição destinada a testes e aprendizado).

2- Extraia o arquivo compactado e importe a VM no VirtualBox.

3- Acesse: Configurações → Rede → Conectado a: Placa de rede exclusiva do hospedeiro (host-only).

4- Inicie a máquina virtual para acessar o ambiente Metasploitable.


* #### 🐧 Instalação do Kali Linux no WSL

- Abra o PowerShell como administrador e execute:
  
wsl --install

- O comando habilita os componentes necessários para o WSL e instala a distribuição padrão (Ubuntu).
Após a execução, reinicie o sistema.

- Instale o Kali Linux via Microsoft Store.

- Após o primeiro acesso:
  
1- Configure o usuário e a senha.

2- Atualize os pacotes:
   
sudo apt update

* 🪟Ativação da Interface Gráfica (Win-KeX)
  
- Instale o Win-KeX:
  
sudo apt install -y kali-win-kex

- Inicie a interface gráfica:

kex --win -s.
* #### 🔧Instalação do Medusa
- Agora iremos instalar a ferramenta MEDUSA para ataque de Brute Force, a ferramenta ja vem pré-instalada no sistema do kali, Caso não esteja instalado ou precise ser reinstalado, ele pode ser instalado usando o gerenciador de pacotes apt com o seguinte comando:

  sudo apt update && sudo apt install medusa

- Este comando atualiza a lista de pacotes e instala o Medusa junto com suas dependências.

### 🚨Executando ataques simulados Brute Force(força bruta) FTP File Transfer Protocol (Protocolo de Transferência de Arquivos)
- Para começar temos que saber o ip da máquina que será atacada, para saber o ip no mestasploitable é só digitar "ip a" e veremos assim o ip que utilizaremos para atacar maquina teste
- para verificar se está tudo certo, abra o terminal no kali linux e podemos dar um ping no terminal digitando "ping -c (quantidade de pacotes) (numero ip do metsploitable)", exemplo: "ping -c 3 192.168.56.102", se receber resposta saberemos que maquina recebendo comunicação corretamente.
- Agora sim podemos começar com ataque FTP, para fazer esse ataque teremos que verificar quais portas está vulneravel no nosso alvo, para verificar se o alvo está exposto ou não.
- para isso é utilizado a ferramenta NMAP para fazer o scan e verificar qual porta estão abertas.
- após verificar podemos agora utilizar o comando ftp seguido do ip da máquina alvo "ftp 192.168.56.102" para tentar conectar com o alvo, se conectar ai saberemos que está ativo e precisaremos agora utilizar a forca bruta para descobrir usuário e senha da maquina alvo.
- Para darmos continuidade precisaremos criar wordlist(arquivos em txt, onde cada linha para representar ou login ou senha)
- Para criar wordlist digite comando "echo -e "user\nmsfadmin\nadmin\nroot" > users.txt" para usuário e "echo -e "123456\npassword\nqwerty\nmsfadmin" > pass.txt" para senhas
- veja que no comando após ">" é selecionado o nome da wordlist da sua preferência.
- depois de criado utilizaremos a ferramenta MEDUSA para fazer ataque, com seguinte parametros "medusa -h 192,168,56,102 -U (user.txt) -P (pass.txt) -M ftp -t 6" veja que user.txt e pass.txt são as wordlist criadas "-M" é tipo de serviço e "-t" quantidade de Threads que deseja utilizar.
- Após executar veremos que a ferramenta medusa ira tentar encontra login e senhas válidas ou não, se encontrar a ferramenta mostrará em qual login e senha teve sucesso.
- Após isso é só executar novamente o comando ftp seguido do ip da máquina alvo e conectar login e senha que a ferramenta medusa teve sucesso no brute force feito.

### 🚨Executar ataques simulados Brute Force(força bruta) de automação de tentativas em formulário web (DVWA)
- Para ataque em formulario de login em sistemas web teremos que analisar o nosso navegador para conseguir capturar parâmetros que servidor espera receber.
- Após analisar-mos vamos criar as wordlist para tentar simular com medusa todas as combinações possíveis para conseguir-mos achar usuário e senha do sitema web.
- Para criar wordlist é só fazer igual no ataque anterior.
- Agora vamos utilizar a ferramenta MEDUSA  para para conseguir-mos achar usuário e senha do sitema web
- esse seria código para executar no medusa para fazer nosso ataque:
  
  "medusa -h 192.168.56.102 -U users.txt -P pass.txt -M http \
   -m PAGE:'dvwa\login.php'\
   -m FORM:'username=^USER^&password=^PASS^&Login=Login' \
   -m 'FAIL=Login failed' -t 6"
  
- note que -U e -P = nome da wordlist criada
- -M(maiúsculo) = tipo de serviço
- -m(minúsculo) = caminho e corpo da requisição e tipo de resposta esperada se estiver falha
- -t = quantidade de Threads
- Após executar veremos que a ferramenta medusa ira tentar encontra login e senhas válidas ou não, se encontrar a ferramenta mostrará em qual login e senha teve sucesso.

### 🚨Executando ataques password spraying em SMB(Server Message Block) com enumeração de usuários.
- SMB(Server Message Block) usado principalmente para fornecer acesso compartilhado a arquivos, impressoras, portas seriais e outras comunicações entre dispositivos em uma rede. Ele permite que aplicativos em um computador leiam, escrevam e solicitem serviços de programas em servidores remotos, tornando possível acessar recursos de forma semelhante ao armazenamento local.
- Agora utilizamos a ferramenta "enum4linux" que serve para enumeração de usuários
- Para rodar a ferramenta precisamos utilizar código: "enum4linux -a 192.168.56.102 | tee enum4_output.txt"
- -a = todas as técnica de enumeração
- tee enum4_output.txt = para salvar saída do comando em um arquivo
- Após executar o comando é só abrir o arquivo que acabou de ser salvo utilizando o comando:"less enum4_output.txt"
- Assim conseguimos explorar o arquivo com nomes e sistemas reais de usuários do sistema
- Para darmos continuidade precisaremos criar wordlist novamente só que agora criaremos wordlist de senhas com conceito de password spraying que diferente do Brute Force o password spraying testa poucas senhas definidas em muitos usuários, sendo mais dificil de ser detectado
- código para criar wordlist login: "echo -e "user\nmsfadmin\nservice" > smb_users.txt"
- código para criar wordlist senhas:"echo -e "password\n123456\nWelcome\nmsfadmin" > senhas_spray.txt"
- Agora execute a ferramneta MEDUSA e de comando: "medusa -h 192,168,56,102 -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50"
- note que -U e -P = nome da wordlist criada
- -M(maiúsculo) = tipo de serviço
- -t = quantidade de Threads
- -T(maiúsculo) = limite de host paralelo
- excute o código e a ferramenta medusa irá mostrar usuário e senha em "ACCOUNT FOUND"  que significa que medusa conseguiu acesso a usuário e senha de administrador
- Para verificar se realmente teve sucesso a conta de administrador vamos usar o comando:"smbclient -L //192.18.56.102 -U msfadmin"
- -U = usuário capturado com medusa
- assim que executar código se estiver correto vai pedir senha q será a mesma capturada pela medusa e pronto terá o acesso a administrador capturado pela ferramenta medusa.

#### 📖Esses testes demonstram de forma prática como um agente mal-intencionado pode comprometer um serviço exposto devido a configurações inadequadas ou à falta de atualizações. Uma simples porta aberta em um serviço FTP legado pode servir como vetor inicial de intrusão, permitindo que o atacante obtenha acesso não autorizado ao sistema e, potencialmente, comprometa toda a infraestrutura da organização. Da mesma forma, ambientes corporativos configurados de forma inadequada tornam-se alvos suscetíveis a ataques de força bruta e password spraying em protocolos como SMB, possibilitando a escalada de privilégios e o acesso a redes inteiras por meio dessas vulnerabilidades.
#### 🔍Para mitigar esses riscos, recomenda-se a desativação de serviços FTP desnecessários ou a substituição por protocolos mais modernos que implementem mecanismos de autenticação robustos. Medidas adicionais incluem: implementação de bloqueios após múltiplas tentativas de autenticação, limitação de tentativas por endereço IP, aplicação de bloqueios temporários, utilização de autenticação multifator (MFA), monitoramento contínuo de logs com geração de alertas, adoção de políticas de senhas fortes e realização de auditorias de segurança de forma regular.

  



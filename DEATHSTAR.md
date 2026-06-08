#Relatório de Teste de Penetração DEATHSTAR

##Introdução

Este relatório apresenta os resultados de um teste de penetração realizado em um servidor localizado no ambiente de laboratório, com o objetivo de identificar vulnerabilidades exploráveis que pudessem permitir acesso não autorizado ao sistema.
O teste foi conduzido utilizando ferramentas comuns de segurança ofensiva no Kali Linux, seguindo etapas típicas de um processo de Pentest: reconhecimento, enumeração, exploração e obtenção de acesso ao sistema.

##Escopo do Teste

Item	Informação

IP do servidor:	192.168.56.202

Serviços identificados:	SSH (22) e HTTP (80)

Ambiente:	Rede local de laboratório

##Metodologia

O teste foi realizado seguindo as etapas abaixo:

1.	Reconhecimento da rede

2.	Enumeração de serviços

3.	Descoberta de diretórios

4.	Análise de código fonte

5.	Ataques de força bruta

6.	Quebra de hash de senha

7.	Acesso remoto ao servidor


##Reconhecimento de Rede

Inicialmente foi utilizado o Netdiscover no Kali Linux para identificar dispositivos ativos na rede local.
Comando utilizado:
netdiscover	

<img width="886" height="164" alt="image" src="https://github.com/user-attachments/assets/c4df836b-f418-4d1b-a1ae-557f7d93f985" />

 
A ferramenta permitiu identificar o endereço IP do servidor alvo:
192.168.56.202

##Enumeração de Serviços

Após identificar o IP do alvo, foi realizada uma varredura de portas utilizando Nmap para identificar serviços ativos.

Comando utilizado:
 nmap
 <img width="886" height="95" alt="image" src="https://github.com/user-attachments/assets/b53078b8-5929-4710-afd8-5bf4aa4c0661" />
<img width="886" height="363" alt="image" src="https://github.com/user-attachments/assets/69ce65ab-e605-4639-a594-ce5a2d273358" />


##Resultado da varredura:

Porta	Serviço

22	SSH

80	HTTP

Esses serviços indicam que o servidor permite acesso remoto via SSH e possui um servidor web ativo.
Descoberta de Diretórios
Foram utilizadas ferramentas de enumeração de diretórios web, incluindo:

•	Dirb

•	Feroxbuster

•	Scripts do Nmap

<img width="886" height="640" alt="image" src="https://github.com/user-attachments/assets/49d52310-61dc-470f-bd4c-d9ab105c56e9" />

 
Durante a análise foi identificado o arquivo:

/robots.txt

O arquivo estava acessível com código 200.

Descoberta de Diretório Oculto

Ao acessar o arquivo robots.txt, foi encontrado um diretório oculto:

<img width="480" height="195" alt="image" src="https://github.com/user-attachments/assets/de259066-9316-484f-895b-2e0bb451420b" />

/0VVNZMMGM1

Acessando este diretório, foi encontrada uma página de login solicitando usuário e senha.

 <img width="473" height="185" alt="image" src="https://github.com/user-attachments/assets/7f31b7fc-b9eb-4a66-a68e-1e5a8f9ac463" />

Análise de Código Fonte

Durante a inspeção do código fonte da página, foram encontrados dados que indicavam que a senha do usuário poderia ser obtida gerando o hash MD5 da página 

Obtenção do Hash MD5
O arquivo foi baixado utilizando o comando:
wget <URL do arquivo Star_Wars.pdf>

<img width="886" height="356" alt="image" src="https://github.com/user-attachments/assets/4219735c-4b06-45f6-9d9d-43bdebfd209c" />

 
Após o download, foi gerado o hash MD5 do arquivo
 
O valor gerado foi utilizado como senha do usuário, permitindo acesso à aplicação web.
 
<img width="886" height="525" alt="image" src="https://github.com/user-attachments/assets/4efc70cb-2b1b-488b-bc57-9235d8a20adf" />


Nova Informação Obtida

Após o login bem-sucedido, foi exibida uma página com conteúdo sobre Star Wars.

Durante a análise do código fonte, foi encontrada uma mensagem que revelava senhas de acesso


Descoberta da Senha do Usuário
A página estreladamorte2023.html foi acessada.

<img width="886" height="428" alt="image" src="https://github.com/user-attachments/assets/0d4cca6d-05ee-4bd7-beef-cf20ceadb360" />

 
O conteúdo da página foi salvo em um arquivo

Este arquivo foi utilizado como wordlist para um ataque de força bruta.
 
<img width="886" height="127" alt="image" src="https://github.com/user-attachments/assets/52e556c5-9341-4c57-94a8-eec7b054fd0d" />


Ataque de Força Bruta com Hydra
Foi utilizado o Hydra para testar as possíveis senhas do usuário

<img width="886" height="217" alt="image" src="https://github.com/user-attachments/assets/d353c833-9050-4dd0-a12f-0eb37492995d" />

 
Comando utilizado:
hydra -l darth -P senha_d@.txt 192.168.56.202 http-post-form
Resultado obtido:

<img width="855" height="473" alt="image" src="https://github.com/user-attachments/assets/51e07194-2a42-4cc2-8987-0104a37cec11" />


Após autenticação bem-sucedida, foi possível acessar a página

 <img width="834" height="247" alt="image" src="https://github.com/user-attachments/assets/4bf3f0d5-2781-428b-8530-902cc6349a21" />


Descoberta de Novo Usuário
Na análise do código fonte da página, foi encontrada a mensagem:

<img width="886" height="206" alt="image" src="https://github.com/user-attachments/assets/7b20bcf6-cd0f-408b-9f67-1e6dcacc0bf6" />

Geração de Wordlist Numérica

Foi utilizada a ferramenta Crunch para gerar todas as combinações possíveis de senhas com x dígitos numéricos.

<img width="886" height="283" alt="image" src="https://github.com/user-attachments/assets/bd6a04dc-f8eb-454f-be7b-979f19870e52" />

 
Comando utilizado:
crunch 7 7 0123456789 -o senhas_kenobi.txt






Ataque de Força Bruta no Usuário K@

O arquivo gerado foi utilizado em um novo ataque com Hydra.

Comando utilizado:

hydra -l k@ -P senhas_k@.txt 192.168.56.202 http-post-form

<img width="886" height="345" alt="image" src="https://github.com/user-attachments/assets/c6af69bc-232c-479a-86c8-06dc24cd9841" />

 
Após diversas tentativas, foi possível obter acesso à aplicação e visualizar o arquivo shadow do sistema.

 <img width="814" height="347" alt="image" src="https://github.com/user-attachments/assets/f385eb65-af3e-4ba1-b26a-1ead21edb6c0" />

 <img width="886" height="33" alt="image" src="https://github.com/user-attachments/assets/8831603a-3e92-4cf6-87f9-d7c5148bb634" />

 Análise do Arquivo Shadow
 
No arquivo shadow foi identificado o seguinte usuário:

leia
O hash associado era do tipo:
SHA512-crypt

Quebra do Hash de Senha
O hash foi copiado para um arquivo chamado:
hash.txt

<img width="420" height="88" alt="image" src="https://github.com/user-attachments/assets/847de76c-0d75-4273-9387-918cfdede1e1" />

 
Foi utilizada a ferramenta John the Ripper com a wordlist rockyou.

Comando utilizado:

john hash.txt --wordlist=rockyou.txt

<img width="886" height="209" alt="image" src="https://github.com/user-attachments/assets/5af85b04-273c-452d-9413-670a8bdb448d" />

 
Senha descoberta:

<img width="642" height="276" alt="image" src="https://github.com/user-attachments/assets/644ff36d-37cc-4abc-9691-48992a8ae8cc" />

 
O acesso ao servidor foi realizado com sucesso.

##Impacto da Vulnerabilidade

As vulnerabilidades encontradas permitiram:

•	Descoberta de diretórios ocultos

•	Exposição de credenciais em código fonte

•	Ataques de força bruta

•	Acesso ao arquivo shadow

•	Quebra de hash de senha

•	Acesso remoto ao servidor via SSH

Isso demonstra falhas críticas de segurança na aplicação e no sistema operacional.

##Recomendações de Segurança

Para mitigar as vulnerabilidades identificadas, recomenda-se:

1.	Remover credenciais expostas em código fonte

2.	Implementar proteção contra ataques de força bruta

3.	Utilizar políticas de senha fortes

4.	Restringir acesso a arquivos sensíveis como /etc/shadow

5.	Implementar autenticação multifator (MFA)

6.	Utilizar monitoramento de tentativas de login

 Resultado Final do Pentest
 
Foi possível obter acesso remoto ao servidor através de SSH, demonstrando comprometimento completo do sistema.
 
<img width="487" height="138" alt="qw" src="https://github.com/user-attachments/assets/61c16c08-537c-46ef-8688-0da97fcd733b" />

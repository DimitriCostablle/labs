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


##netdiscover	
 <img width="886" height="164" alt="image" src="https://github.com/user-attachments/assets/8065f9cf-007b-4029-997e-45afcb4c85a6" />

A ferramenta permitiu identificar o endereço IP do servidor alvo:
192.168.56.202

##Enumeração de Serviços

Após identificar o IP do alvo, foi realizada uma varredura de portas utilizando Nmap para identificar serviços ativos.
Comando utilizado:
 
 <img width="886" height="95" alt="image" src="https://github.com/user-attachments/assets/add63320-7286-405c-9d72-d3aadd5cf7ae" />

<img width="886" height="363" alt="image" src="https://github.com/user-attachments/assets/f222d0d6-6aee-4b75-8f6c-6211e2068b3d" />



##Resultado da varredura:

Porta	Serviço

22	SSH

80	HTTP

Esses serviços indicam que o servidor permite acesso remoto via SSH e possui um servidor web ativo.

##Descoberta de Diretórios
Foram utilizadas ferramentas de enumeração de diretórios web, incluindo:

•	Dirb

•	Feroxbuster

•	Scripts do Nmap
<img width="886" height="640" alt="image" src="https://github.com/user-attachments/assets/14710b22-458f-4949-9df3-de7ee85e59d0" />

 
Durante a análise foi identificado o arquivo:
/robots.txt
O arquivo estava acessível com código 200.

##Descoberta de Diretório Oculto
Ao acessar o arquivo robots.txt, foi encontrado um diretório oculto:
 
/0VVNZMMGM1
Acessando este diretório, foi encontrada uma página de login solicitando usuário e senha.


##Análise de Código Fonte
Durante a inspeção do código fonte da página, foram encontradas credenciais do user1 

Isso indicava que a senha do usuário user1 poderia ser obtida gerando o hash MD5 do arquivo Star_Wars.pdf

##Obtenção do Hash MD5
 <img width="886" height="356" alt="image" src="https://github.com/user-attachments/assets/c6b0f908-3e8a-4375-b180-eba8395e5727" />

Após o download, foi gerado o hash MD5 do arquivo:
md5sum Star_Wars.pdf

O valor gerado foi utilizado como senha do user1 , permitindo acesso à aplicação web.
 <img width="886" height="525" alt="image" src="https://github.com/user-attachments/assets/63ff6ea2-8f79-467a-970d-d17cfc6f0cf4" />


##Nova Informação Obtida
Após o login bem-sucedido, foi exibida uma página com conteúdo sobre Star Wars.
Durante a análise do código fonte, foi encontrada a seguinte mensagem:
Remember: deixei a senha de darth para logar em restrict98712
na página estreladamorte2023.html.
 

##Descoberta da Senha do User2 A página estreladamorte2023.html foi acessada.
 <img width="886" height="428" alt="image" src="https://github.com/user-attachments/assets/04769cae-3e46-4328-8aae-5ae1fa184923" />

O conteúdo da página foi salvo em um arquivo de texto.
Este arquivo foi utilizado como wordlist para um ataque de força bruta.
 


##Ataque de Força Bruta com Hydra
Foi utilizado o Hydra para testar as possíveis senhas do user2.
 <img width="886" height="217" alt="image" src="https://github.com/user-attachments/assets/eccdc91e-4957-4963-a06b-daf5bab7d3b7" />

Comando utilizado:
hydra -l user2 -P senha_user1.txt 192.168.56.202 http-post-form
Resultado obtido:

<img width="547" height="303" alt="2" src="https://github.com/user-attachments/assets/10f7ef8e-b0ba-49a8-b1bc-e44dc6175356" />

 
Após autenticação bem-sucedida, foi possível acessar a página:
chewbacca.php

 <img width="834" height="247" alt="image" src="https://github.com/user-attachments/assets/7755c904-f677-4b6e-859e-ee5dca63b8f4" />


##Descoberta de Novo Usuário

Na análise do código fonte da página, foram encontradas novas informações
 
Login em outra página, user3, senha numerica
Geração de Wordlist Numérica
Foi utilizada a ferramenta Crunch para gerar todas as combinações possíveis de senhas com 7 dígitos numéricos.

<img width="567" height="181" alt="5" src="https://github.com/user-attachments/assets/4dbf8ef4-20b0-442c-8f24-025317526660" />

 
Comando utilizado:
crunch 7 7 0123456789 -o senhas_user3.txt






##Ataque de Força Bruta no Usuário user3
O arquivo gerado foi utilizado em um novo ataque com Hydra.
Comando utilizado:
hydra -l user3 -P senhas_user3.txt 192.168.56.202 http-post-form

<img width="567" height="221" alt="1" src="https://github.com/user-attachments/assets/6d78671a-2dd7-411e-ae77-4840627e3ca9" />

 
Após diversas tentativas, foi possível obter acesso à aplicação e visualizar o arquivo shadow do sistema.

 
 
 Análise do Arquivo Shadow
 
No arquivo shadow foi identificado o user4

O hash associado era do tipo:
SHA512-crypt

##Quebra do Hash de Senha

O hash foi copiado para um arquivo chamado:
hash.txt

<img width="420" height="88" alt="image" src="https://github.com/user-attachments/assets/a718b817-baf0-4f10-ad14-3b8183d57fc1" />

 
Foi utilizada a ferramenta John the Ripper com a wordlist rockyou.

Comando utilizado:

john hash.txt --wordlist=rockyou.txt
 
Senha descoberta

Acesso SSH ao Servidor

Com as credenciais obtidas, foi realizado acesso remoto via SSH.

Comando utilizado:

ssh user4@192.168.56.202
 
O acesso ao servidor foi realizado com sucesso.

<img width="409" height="177" alt="4" src="https://github.com/user-attachments/assets/6432cef1-0f6a-4d41-aa2a-a1a79e2c1c61" />


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

##Resultado Final do Pentest
Foi possível obter acesso remoto ao servidor através de SSH, demonstrando comprometimento completo do sistema.

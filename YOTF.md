#Relatório de Teste de Invasão: Ambiente Year of the Fox
Introdução e Coleta de Informações
O teste de invasão iniciou-se com a fase de reconhecimento passivo e ativo, visando mapear a superfície de ataque do alvo. Através do protocolo SMB, utilizou-se a ferramenta enum4linux para identificar a estrutura do diretório e possíveis contas de usuário.

 <img width="900" height="105" alt="image" src="https://github.com/user-attachments/assets/2e33b5f8-078e-4d71-96a6-7e26e8e6cba7" />
<img width="519" height="130" alt="image" src="https://github.com/user-attachments/assets/040dcbc2-2b77-4a66-87f7-dc4f5627c14e" />

 
A análise revelou a existência de dois usuários principais, fox e rascal, além de um compartilhamento de rede denominado yotf. Esta etapa é crucial, pois estabelece os alvos para ataques de força bruta e direciona a exploração de serviços específicos, como o servidor web identificado na porta 80.
 

<img width="439" height="401" alt="image" src="https://github.com/user-attachments/assets/85ee7ce9-6f41-4d3a-9e30-7b3a70bb8e99" />




Exploração de Vulnerabilidades Web
Ao acessar a aplicação web, identificou-se um sistema de busca de arquivos. Devido à necessidade de autenticação, foi realizado um ataque de dicionário utilizando a ferramenta Hydra contra o formulário de login, utilizando a lista de senhas rockyou.txt.
Comando utilizado: hydra -l rascal -P /usr/share/wordlists/rockyou.txt 10.67.138.207 http-get /
 
<img width="709" height="95" alt="image" src="https://github.com/user-attachments/assets/a46ab27f-2c0d-4725-bd0d-5a643db6a533" />
<img width="714" height="106" alt="image" src="https://github.com/user-attachments/assets/91667d2e-7f7c-4085-8332-55bef31c1726" />

 
Com a credencial obtida (rascal:rhianna), foi possível acessar o painel de busca. Através da interceptação de requisições com o Burp Suite, constatou-se que o parâmetro de pesquisa era vulnerável a Injeção de Comandos de Sistema (OS Command Injection). Ao inserir caracteres de controle (como ; ou &&), o servidor permitiu a execução de comandos arbitrários.
 
 <img width="621" height="355" alt="image" src="https://github.com/user-attachments/assets/d6683712-a1a4-4d2f-bd55-52208be620ce" />
<img width="625" height="593" alt="image" src="https://github.com/user-attachments/assets/88837cff-bdc8-4496-9ceb-e5cee12428e1" />


Para consolidar o acesso, foi executado um payload de Reverse Shell em PHP, redirecionando o terminal da vítima para a máquina do atacante, estabelecendo uma conexão estável como o usuário www-data.
 
 <img width="886" height="116" alt="image" src="https://github.com/user-attachments/assets/05834e94-5601-4496-b997-61405781e94b" />
<img width="772" height="186" alt="image" src="https://github.com/user-attachments/assets/7bcd432b-3410-488a-b25a-3fa8a4cc5e4d" />







Enumeração Local e Pós-Exploração com LinPeas
Imediatamente após o acesso inicial como www-data, foi necessário compreender as permissões internas do sistema. Para isso, transferiu-se e executou-se o script LinPeas (Linux Privilege Escalation Awesome Script).
 
 <img width="623" height="159" alt="image" src="https://github.com/user-attachments/assets/33907fd6-e200-4489-9533-58349b0e5ddb" />
<img width="613" height="186" alt="image" src="https://github.com/user-attachments/assets/939aa38f-3af3-41ab-a93e-3af9ed0bf8cd" />


Movimentação Lateral e Pivoting
Uma vez dentro do sistema, a fase de pós-exploração revelou arquivos de configuração e hashes de senhas no diretório /var/www/files. No entanto, o serviço SSH estava configurado para aceitar conexões apenas localmente (127.0.0.1), impedindo o acesso direto externo.
Para contornar essa restrição, utilizou-se a técnica de Local Port Forwarding com a ferramenta socat, criando um túnel que mapeou a porta 22 (SSH) da vítima para a porta 1234 da máquina atacante.
 
<img width="653" height="189" alt="image" src="https://github.com/user-attachments/assets/bcd40bab-67ec-4ee1-89e8-5fcb444340d1" />
<img width="655" height="209" alt="image" src="https://github.com/user-attachments/assets/dc82aa11-02ec-49a6-8f7f-b8dd991d2142" />

 
Com o túnel estabelecido, foi realizado um novo ataque de força bruta via SSH através da porta redirecionada, permitindo o acesso ao usuário fox com a senha mommy1.
 
 <img width="530" height="55" alt="image" src="https://github.com/user-attachments/assets/6c994e1c-aad7-4e54-99e0-5fa006851e64" />
<img width="805" height="97" alt="image" src="https://github.com/user-attachments/assets/7ed39caf-9b2c-447e-9281-6209e23235d2" />


Escalação de Privilégios para Root
O objetivo final do teste era a obtenção de privilégios administrativos. O script de enumeração linpeas.sh foi carregado para a máquina para identificar vetores de escalação.

<img width="342" height="64" alt="image" src="https://github.com/user-attachments/assets/8f5a089d-b558-43c1-b643-af38afc9e8e5" />

 
A vulnerabilidade crítica identificada foi uma configuração permissiva no arquivo sudoers, onde o usuário fox podia executar o binário /usr/sbin/shutdown como root. Contudo, este binário possuía uma falha de design: ele invocava o comando poweroff sem especificar seu caminho absoluto.
Explorando o PATH Hijacking, foi criado um script malicioso chamado poweroff no diretório /tmp, contendo uma instrução para executar um shell root. Ao adicionar /tmp ao início da variável de ambiente $PATH, o sistema executou o script falso com privilégios elevados.
Comandos utilizados: echo "/bin/bash" > /tmp/poweroff chmod +x /tmp/poweroff export PATH=/tmp:$PATH sudo /usr/sbin/shutdown
 
<img width="536" height="325" alt="image" src="https://github.com/user-attachments/assets/cd0e69fb-37c3-4c56-ad94-24e28f51810f" />

<img width="409" height="58" alt="image" src="https://github.com/user-attachments/assets/71765f56-3218-4c88-a228-79c2a6176657" />

 
Acho que a flag não está aqui...

5. Conclusão
O sucesso do teste demonstrou que falhas simples de configuração, como a falta de caminhos absolutos em comandos de sistema e a ausência de sanitização em formulários web, podem levar ao comprometimento total de um servidor Linux. Recomenda-se a atualização das políticas de privilégios e o saneamento rigoroso de entradas de usuário na camada de aplicação.
<img width="567" height="189" alt="flag" src="https://github.com/user-attachments/assets/16501be4-7c92-4026-ac7f-6aeae297ece0" />

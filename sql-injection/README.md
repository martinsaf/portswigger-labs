# O write-up detalhado (modelo abaixo)

# Lab: [Nome e Número do Lab]

**Categoria:** [e.g., SQL Injection]
**Tipo:** [e.g., Blind SQLi com time delays]
**Dificuldade:** [Apprentice / Practitioner]

## 1. Explicação da Vulnerabilidade (O "Porquê")
[Explica aqui a falha com as tuas palavras. O que o input do utilizador está a controlar? Porque é que o código é vulnerável? Sê conciso e técnico.]

*Exemplo:* "O parâmetro `trackingId` no cookie de sessão é concatenado diretamente numa query SQL sem sanitização. Como a resposta da aplicação não reflete erros nem dados, mas a execução do comando `pg_sleep(10)` causa um atraso visível, podemos inferir a injeção de forma cega."

## 2. Passos de Exploração (O "Como")
[Documenta os passos lógicos da exploração, incluindo becos sem saída.]

1.  **Identificação:** Injetei uma `'` simples e a aplicação lançou um erro 500, indicando uma potencial SQLi.
2.  **Confirmação (Base de dados):** Usei `' OR 1=1-- -` para verificar se a lógica da query podia ser manipulada.
3.  **Confirmação (Cega):** Testei com `' AND '1'='1` (condição verdadeira) vs. `' AND '1'='2` (condição falsa) para confirmar a injeção cega.
4.  **Enumeração com Payloads Condicionais:** Utilizei `' AND (SELECT CASE WHEN (1=1) THEN 'a' ELSE 'b' END)='a'-- -` para confirmar a execução condicional.
5.  **Exfiltração da Palavra-passe:** Automatizei a extração da password do administrador caractere a caractere com o Intruder do Burp Suite, usando o payload `' AND (SELECT SUBSTRING(password,§1§,1) FROM users WHERE username='administrator')='§a§'-- -`.

## 3. Deteção e Monitorização (A Ligaçao ao CV)
[Esta é a secção que te destaca. Como detetarias este ataque no teu laboratório Wazuh? Em que logs?]

**Produto de Segurança:** Wazuh SIEM com Sysmon / Auditd.
**Fontes de Dados:** Web Server Logs (Apache), Sysmon Event ID 1 (Process Creation).

- **Hipótese de Deteção 1 (Web Server Logs):** Um elevado volume de pedidos (HTTP 200/500) com strings de SQL (`SELECT`, `SUBSTRING`, `SLEEP`) no User-Agent ou parâmetros da query para o mesmo IP de origem.
- **Hipótese de Deteção 2 (Sysmon):** Criação de processos de shell suspeitos (`cmd.exe`, `powershell.exe`) pelo processo do servidor web (`w3wp.exe` ou `httpd`), originados por um ataque de injeção de comandos pós-exploração.
- **Regra Wazuh (Exemplo Conceptual):**

```xml
<rule id="100010" level="12">
  <if_sid>31101</if_sid> <!-- Apache error log -->
  <match>SUBSTRING|SLEEP|UNION SELECT|pg_sleep</match>
  <description>Possível tentativa de SQL Injection cega detetada em logs do Apache.</description>
  <group>sql_injection,web_attack,</group>
  <mitre>
    <id>T1190</id> <!-- Exploit Public-Facing Application -->
  </mitre>
</rule>
```

## 4. Remediação (A Mentalidade de Engenheiro)
[Corrigir o problema, não apenas detetá-lo.]

- **Causa Raiz:** Concatenação de input não confiável numa query SQL.
- **Correção Permanente:** Substituir a query dinâmica por **Prepared Statements** com parâmetros vinculados. Para casos limites (nomes de tabelas dinâmicos), usar uma lista de permissões estrita no backend. **Nunca** confiar em sanitização de input como única defesa.

**Ligações MITRE ATT&CK:** [T1190 - Exploit Public-Facing Application], [T1059 - Command and Scripting Interpreter]

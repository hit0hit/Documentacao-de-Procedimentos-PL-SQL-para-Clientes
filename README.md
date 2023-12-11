# FOR_INSERT
# Documentação de Procedimentos PL/SQL para Clientes

A seguir estão dois procedimentos PL/SQL utilizados para inserir e atualizar informações de clientes no Oracle Database. Esses procedimentos são projetados para trabalhar com datas em diferentes formatos e realizar operações em duas tabelas relacionadas: `CLIENTE` e `CLIENTE_INFORMACAES`.

## Inserção de Clientes

O procedimento a seguir é responsável por inserir informações de clientes na ausência de registros correspondentes no banco de dados. Ele lida com diferentes formatos de data e realiza a inserção em duas tabelas relacionadas.

```sql
DECLARE
   ID_CLIENTE_V NUMBER;
BEGIN
   FOR cliente_data IN (
      SELECT 
         CASE
            WHEN INSTR(F.DATA, '/') = 2 THEN TO_DATE(F.DATA, 'MM/DD/YYYY HH24:MI:SS')
            WHEN INSTR(F.DATA, '/') = 3 THEN TO_DATE(F.DATA, 'DD/MM/YYYY HH24:MI:SS')
            ELSE NULL
         END AS DATA,
         F.RAZAO_SOCIAL,
         F.NOME_RESPONSAVEL,
         F.WHATSAPP,
         F.INVESTIMENTO,
         F.EMAIL,
         F.ID_EMPRESA_FK,
         F.NOME_FANTASIA,
         F.CPF_CNPJ,
         F.ENDERECO,
         F.TELEFONE,
         F.LINKEDIN,
         F.CARGO,
         F.SITE_WEB
      FROM CLIENTE_DATA_TEMP_FILE F
      LEFT JOIN CLIENTE A ON A.WHATSAPP = F.WHATSAPP
      WHERE A.WHATSAPP IS NULL
   ) LOOP
      BEGIN
         -- Inserção na tabela CLIENTE
         INSERT INTO cLIENTE (
            CARIMBO_DE_DATA_HORA,
            NOME,
            WHATSAPP,
            VALOR_DE_INVESTIMENTO,
            GMAIL,
            ID_EMPRESA_FK
         ) VALUES (
            cliente_data.DATA,
            cliente_data.NOME_RESPONSAVEL,
            cliente_data.WHATSAPP,
            cliente_data.INVESTIMENTO,
            cliente_data.EMAIL,
            cliente_data.ID_EMPRESA_FK
         )
         RETURNING ID INTO ID_CLIENTE_V;

         -- Inserção na tabela CLIENTE_INFORMACAES
         INSERT INTO CLIENTE_INFORMACAES (
            ID_CLIENTE_FK,
            "NOME/RAZAO_SOCIAL",
            FANTASIA,
            "CNPJ/CPF",
            FONE_CONTATO,
            LINKEDIN,
            NOME_CONTATO,
            CARGO_COBTATO,
            ENDERECO,
            EMAIL,
            SITE_WEB,
            ID_EMPRESA_FK
         ) VALUES (
            ID_CLIENTE_V,
            cliente_data.RAZAO_SOCIAL,
            cliente_data.NOME_FANTASIA,
            cliente_data.CPF_CNPJ,
            cliente_data.TELEFONE,
            cliente_data.LINKEDIN,
            cliente_data.NOME_RESPONSAVEL,
            cliente_data.CARGO,
            cliente_data.ENDERECO,
            cliente_data.EMAIL,
            cliente_data.SITE_WEB,
            cliente_data.ID_EMPRESA_FK
         );

      EXCEPTION
         WHEN OTHERS THEN
            -- Lida com exceções, se ocorrerem
            NULL;
      END;
   END LOOP;
   -- Confirma as alterações no banco de dados
   COMMIT;
END;
```

Este procedimento realiza a inserção de clientes verificando se já existem registros com o mesmo número de WhatsApp na tabela `CLIENTE`.

## Atualização de Clientes

O procedimento a seguir é responsável por atualizar informações de clientes quando já existe um registro correspondente na tabela `CLIENTE`. Ele lida com diferentes formatos de data e realiza a atualização em duas tabelas relacionadas.

```sql
DECLARE
   ID_CLIENTE_V NUMBER;
BEGIN
   FOR cliente_data IN (
      SELECT 
         CASE
            WHEN INSTR(F.DATA, '/') = 2 THEN TO_DATE(F.DATA, 'MM/DD/YYYY HH24:MI:SS')
            WHEN INSTR(F.DATA, '/') = 3 THEN TO_DATE(F.DATA, 'DD/MM/YYYY HH24:MI:SS')
            ELSE NULL
         END AS DATA,
         F.RAZAO_SOCIAL,
         F.NOME_RESPONSAVEL,
         F.WHATSAPP,
         F.INVESTIMENTO,
         F.EMAIL,
         F.ID_EMPRESA_FK,
         F.NOME_FANTASIA,
         F.CPF_CNPJ,
         F.ENDERECO,
         F.TELEFONE,
         F.LINKEDIN,
         F.CARGO,
         F.SITE_WEB
      FROM CLIENTE_DATA_TEMP_FILE F
      LEFT JOIN CLIENTE A ON A.WHATSAPP = F.WHATSAPP
      WHERE A.WHATSAPP IS NOT NULL
   ) LOOP
      BEGIN
         -- Atualização na tabela CLIENTE
         UPDATE CLIENTE SET 
            CARIMBO_DE_DATA_HORA = cliente_data.DATA,
            NOME = cliente_data.NOME_RESPONSAVEL,
            WHATSAPP = cliente_data.WHATSAPP,
            VALOR_DE_INVESTIMENTO = cliente_data.INVESTIMENTO,
            GMAIL = cliente_data.EMAIL,
            ID_EMPRESA_FK = cliente_data.ID_EMPRESA_FK
         RETURNING ID INTO ID_CLIENTE_V;

         -- Atualização na tabela CLIENTE_INFORMACAES
         UPDATE CLIENTE_INFORMACAES SET
            "NOME/RAZAO_SOCIAL" = cliente_data.RAZAO_SOCIAL,
            FANTASIA = cliente_data.NOME_FANTASIA,
            "CNPJ/CPF" = cliente_data.CPF_CNPJ,
            FONE_CONTATO = cliente_data.TELEFONE,
            LINKEDIN = cliente_data.LINKEDIN,
            NOME_CONTATO = cliente_data.NOME_RESPONSAVEL,
            CARGO_COBTATO = cliente_data.CARGO,
            ENDERECO = cliente_data.ENDERECO,
            EMAIL = cliente_data.EMAIL,
           

 SITE_WEB = cliente_data.SITE_WEB,
            ID_EMPRESA_FK = cliente_data.ID_EMPRESA_FK
         WHERE ID_CLIENTE_FK = ID_CLIENTE_V;

      EXCEPTION
         WHEN OTHERS THEN
            -- Lida com exceções, se ocorrerem
            NULL;
      END;
   END LOOP;
   -- Confirma as alterações no banco de dados
   COMMIT;
END;
```

Este procedimento realiza a atualização de clientes verificando se já existem registros com o mesmo número de WhatsApp na tabela `CLIENTE`.

Lembre-se de personalizar nomes de tabelas, colunas e lógica conforme necessário para atender aos requisitos específicos do seu aplicativo Oracle PL/SQL.

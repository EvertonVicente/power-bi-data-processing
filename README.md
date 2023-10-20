# Desafio do Módulo 3: Processamento de Dados Simplificado com Power BI

## Etapa 1: Criação de uma instância na Azure para MySQL

Nesta fase, realizei a configuração de uma instância na Microsoft Azure para acomodar um servidor MySQL. Comecei criando uma conta na plataforma Azure e, em seguida, segui as instruções para estabelecer um servidor de banco de dados MySQL flexível. Durante o processo de configuração, configurei parâmetros essenciais, como o nome do servidor, informações de acesso, etc.

## Etapa 2: Criação do Banco de Dados a partir de um Script no GitHub

Para criar o banco de dados, adotei um script SQL disponível no GitHub [aqui](https://github.com/julianazanelatto/power_bi_analyst/blob/main/M%C3%B3dulo%203/Desafio%20de%20Projeto/script_bd_company.sql). 

```sql
-- Exemplo do que executei no MySQL:
SHOW DATABASES;
SHOW TABLES;
DESCRIBE departament;
DESCRIBE dependent;
DESCRIBE dept_locations;
DESCRIBE employee;
DESCRIBE project;
DESCRIBE works_on;
```

No entanto, ao implementar o banco de dados, deparei com desafios relacionados à tabela "employee," mais precisamente na coluna "Super_ssn." Esta seção discute as soluções que empreguei para superar as complexidades durante o processo de inserção de dados nessa tabela.

***Passo 1:** Temporariamente removi a restrição de chave estrangeira.

***Passo 2:** Inseri funcionários sem supervisores (atribuindo NULL à coluna "Super_ssn").

***Passo 3:** Inseri os registros dos supervisores.

***Passo 4:** Restaurei a restrição de chave estrangeira.

***Passo 5:** Atualizei os valores em "Super_ssn."

Essa abordagem viabilizou a inserção bem-sucedida dos dados na tabela "employee," resolvendo os obstáculos relacionados a referências de chave estrangeira e valores nulos na coluna "Super_ssn."

##  Etapa 3: Integração do Power BI com MySQL na Azure

Realizei a integração do Microsoft Power BI com o banco de dados MySQL hospedado na plataforma Azure. Durante o processo, deparei-me com um problema relativo à versão do conector "mysql-connector-net-8.1.0", que não funcionava corretamente. Para solucionar esse impasse, optei por fazer o download e empregar uma versão anterior do conector, a "8.0.32", que se mostrou compatível e permitiu uma integração bem-sucedida.

## Etapa 4: Análise de Problemas na Base de Dados para Preparação da Transformação de Dados
Antes de iniciar a etapa de transformação de dados, realizei uma análise detalhada da base de dados com o objetivo de identificar possíveis desafios que poderiam impactar a qualidade e integridade dos dados. Nesta seção, enfatizo os principais problemas encontrados e as soluções que foram adotadas para resolvê-los.

# Diretrizes para Transformação dos Dados

## 1 - Verificação de Cabeçalhos e Tipos de Dados:

Inicialmente, examinei os cabeçalhos e tipos de dados em todas as tabelas do projeto para garantir consistência e conformidade com os requisitos do Power BI. Essa ação visou evitar problemas na importação dos dados no Power BI.

## 2 - Conversão de Valores Monetários para Double Preciso: 

Notei que os valores monetários na coluna "Salary" da tabela "employee" eram armazenados como decimais de precisão fixa. Para uma representação mais precisa, modifiquei o tipo de dados para "double" no Power BI, o que possibilita cálculos mais precisos.

## 3 - Verificação de Valores Nulos e Avaliação da Remoção:

Identifiquei valores nulos na coluna "Super_ssn" da tabela "employees." Conduzi uma análise de impacto e decidi não remover os registros nulos, uma vez que eles possuíam significado no contexto do banco de dados.

## 4 - Identificação de Colaboradores Sem Gerente:

Após uma análise aprofundada, identifiquei que os registros na coluna "Super_ssn" representavam os gerentes. Confirmei que todos os colaboradores tinham um gerente atribuído, exceto um.

## 5 - Verificação de Departamentos sem Gerente:

Verifiquei a tabela "departament" e assegurei que não havia departamentos sem gerente. Todos os departamentos tinham um gerente atribuído.

## 6 - Consideração de Preenchimento de Lacunas nos Departamentos: 

Em função dos dados, não foi necessário preencher lacunas nos departamentos, uma vez que todos possuíam gerentes atribuídos.

## 7 - Verificação do Número de Horas nos Projetos:

Fui à tabela “Project” e agreguei informações da tabela "works_on" para somar as horas de projeto.

## 8 - Separação de Colunas Complexas: 

Não identifiquei colunas complexas que necessitassem de análises especiais.

## 9 - Mesclagem de Consultas entre Employee e Department: 

No Power BI, mesclei as consultas das tabelas "employee" e "department" com base na coluna "Dno." Isso possibilitou a criação de uma tabela "employee" com os nomes dos departamentos associados a cada colaborador. Utilizei uma junção interna para incluir apenas colaboradores com departamentos atribuídos.

## 10 - Remoção de Colunas Desnecessárias:

Durante a mesclagem das tabelas, eliminei colunas que não seriam utilizadas nas elaborações das visualizações no Power BI, visando manter um modelo de dados eficiente.

## 11 - Junção de Colaboradores e Nomes dos Gerentes: 

Conduzi a junção das tabelas "employee" e "department" diretamente no Power BI através da funcionalidade de mesclagem de consultas. Também elaborei uma consulta SQL onde efetuei um "Left Join" com uma subquery, a título de estudo.

```sql
-- Exemplo do que executei no MySQL:

SELECT 
b.GERENTE AS "GERENTE",
CONCAT(a.Fname,' ',a.Lname) AS "COLABORADOR"
FROM employee a 
LEFT JOIN (SELECT concat(b.Fname, ' ',b.lname) AS "GERENTE",
			a.Super_ssn AS "Super_ssn"
			FROM employee a 
			INNER JOIN employee b ON a.Super_ssn = b.Ssn
			group by a.Super_ssn) b ON b.Super_ssn = a.Super_ssn
ORDER BY GERENTE,COLABORADOR
;

```
## 12 - Concatenação de Nomes e Sobrenomes em uma Única Coluna: 

No Power BI, criei uma nova coluna calculada que mesclou os campos "Nome" e "Sobrenome" em uma única coluna denominada "Nome."

## 13 - Combinação de Nomes de Departamentos e Localizações: 

No Power BI, criei uma nova coluna que combinou os nomes dos departamentos e localizações para criar uma chave única para cada combinação.

## 14 - Explicação da Utilização de Mesclagem ao Invés de Atribuição: 

No Power BI, optei por utilizar a funcionalidade de mesclagem de consultas, pois meu objetivo era criar um novo conjunto de dados com base na junção das tabelas "employee" e "department." A mesclagem permitiu a criação de uma tabela combinada que preservava a integridade dos dados originais, enquanto a atribuição (ou criação de colunas calculadas) é geralmente empregada para realizar cálculos baseados em dados existentes.

## 15 - Agrupamento de Dados para Contagem de Colaboradores por Gerente: 

Criei uma tabela resumida que agrupou os dados por gerente, possibilitando a contagem do número de colaboradores supervisionados por cada gerente.

## 16 - Eliminação de Colunas Desnecessárias em Cada Tabela: 

Limpei as tabelas, removendo colunas que não seriam utilizadas na elaboração dos relatórios, garantindo a eficiência do modelo de dados.

# Conclusão

Neste desafio do Módulo 3, percorremos um processo abrangente de configuração, integração e transformação de dados usando o Microsoft Power BI e um servidor MySQL na Microsoft Azure. Cada etapa foi projetada para superar desafios específicos e garantir a qualidade dos dados e a eficiência no Power BI.

Ao final deste projeto, conseguimos criar com sucesso um ambiente de banco de dados funcional, integrá-lo ao Power BI e realizar transformações de dados relevantes. Identificamos e solucionamos problemas comuns em bases de dados, como valores nulos, formatos de dados e chaves estrangeiras.

Essa experiência nos permitiu aprimorar nossas habilidades em processamento de dados e preparação para análises avançadas. Além disso, aprendemos a utilizar as ferramentas disponíveis na Microsoft Azure e a solucionar problemas relacionados à integração com o Power BI.

Este projeto representa um passo significativo na jornada de aprimoramento das nossas habilidades em análise de dados e preparação de informações para tomada de decisões mais embasadas.

Agradeço a todos os envolvidos no processo e estou entusiasmado para aplicar essas habilidades em futuros desafios e projetos.

Se você tiver alguma dúvida ou comentário, sinta-se à vontade para compartilhar. Estou sempre aberto a discussões e troca de conhecimentos.



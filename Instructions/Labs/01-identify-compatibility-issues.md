---
lab:
  title: Identificar problemas de compatibilidade na migração SQL
---

# Identificar problemas de compatibilidade na migração SQL

Em nosso cenário, você foi solicitado a avaliar a preparação de um banco de dados herdado do SQL Server para migração para o Banco de Dados SQL do Azure. Sua tarefa é executar uma avaliação do banco de dados herdado e identificar possíveis problemas de compatibilidade ou alterações que precisam ser feitas antes da migração. Você também deve analisar o esquema do banco de dados e identificar todos os recursos ou as configurações sem suporte no Banco de Dados SQL do Azure.

Este exercício levará, aproximadamente, **15** minutos.

> **Observação**: para realizar este exercício, será necessário ter acesso a uma assinatura do Azure para criar recursos do Azure. Se você não tiver uma assinatura do Azure, crie uma [conta gratuita](https://azure.microsoft.com/free/?azure-portal=true) antes de começar.

## Antes de começar

Para fazer este exercício, verifique se você cumpre os seguintes pré-requisitos antes de continuar:

- Você precisará do SQL Server 2019 ou posterior, juntamente com o banco de dados leve do [**AdventureWorksLT**](https://learn.microsoft.com/sql/samples/adventureworks-install-configure#download-backup-files) que seja compatível com sua instância do SQL Server específica.
- Baixar e instalar o [Azure Data Studio](https://learn.microsoft.com/sql/azure-data-studio/download-azure-data-studio). Se já estiver instalado, atualize-o para garantir a utilização da versão mais recente.
- Um usuário do SQL com acesso de leitura ao banco de dados de origem.

## Restaurar um banco de dados do SQL Server e executar um comando

1. Selecione o botão Iniciar do Windows e digite SSMS. Selecione **Microsoft SQL Server Management Studio 18** na lista.  

1. Quando o SSMS for aberto, observe que a caixa de diálogo **Conectar ao Servidor** será pré-preenchida com o nome de instância padrão. Selecione **Conectar**.

1. Selecione a pasta**Bancos de Dados** e **Nova Consulta**.

1. Na janela Nova consulta, copie e cole o T-SQL abaixo. Certifique-se de que o nome e o caminho do arquivo de backup do banco de dados correspondam ao arquivo de backup real. Se isso não acontecer, o comando falhará. Execute a consulta para restaurar o banco de dados.

    ```sql
    RESTORE DATABASE AdventureWorksLT
    FROM DISK = 'C:\<FolderName>\AdventureWorksLT2019.bak'
    WITH RECOVERY,
          MOVE 'AdventureWorksLT2019_Data' 
            TO 'C:\<FolderName>\AdventureWorksLT2019.mdf',
          MOVE 'AdventureWorksLT2019_Log'
            TO 'C:\<FolderName>\AdventureWorksLT2019.ldf';
    ```

    > **Observação**: Certifique-se de que você tenha o arquivo de backup leve [AdventureWorks](https://learn.microsoft.com/sql/samples/adventureworks-install-configure#download-backup-files) no computador do SQL Server antes de executar o comando T-SQL.

1. Uma mensagem de sucesso será exibida após a conclusão da restauração.

1. Execute o comando a seguir no banco de dados do **AdventureWorksLT** na instância do SQL Server.

```sql
ALTER TABLE [SalesLT].[Customer] ADD [Next] VARCHAR(5);
```

## Instale e inicie a extensão de migração do Azure para o Azure Data Studio

Siga estas etapas para instalar a extensão de migração. Se a extensão de migração do Azure já estiver instalada, você poderá ignorar essas etapas.

1. Abra o gerenciador de extensões no Azure Data Studio. s

1. Pesquise ***Migração SQL do Azure*** e instale a extensão. Depois de instalá-la, a extensão de Migração SQL do Azure estará na lista de extensões instaladas.

1. Selecione o ícone **Conexões** e então selecione **Nova conexão**. 

1. Na nova guia **Conexão**, digite o nome do servidor. Selecione **Opcional (falso)** para a opção **Criptografar**.

1. Selecione **Conectar**. 

1. Para iniciar a extensão de migração do Azure, basta clicar com o botão direito do mouse no nome da instância de origem e selecionar **Gerenciar**. 

1. No menu do servidor, em **Geral**, selecione **Migração de SQL do Azure**. Isso o levará à página principal da extensão de Migração SQL do Azure.

    > **Observação**: se você não conseguir ver a opção **Migração SQL do Azure** no menu do servidor, ou se a página Migração SQL do Azure não carregar, reabra o Azure Data Studio.

## Executar a avaliação de compatibilidade

A avaliação de compatibilidade ajuda a identificar possíveis problemas de migração e fornece orientações detalhadas sobre como resolvê-los antes do início do processo de migração. Economizando, assim, tempo e recursos significativos. 

Execute a extensão de migração do Azure para o Azure Data Studio, em seguida execute a avaliação de compatibilidade e, por último, veja os resultados de um destino do Banco de Dados SQL do Azure.

1. No painel da Migração de SQL do Azure, selecione **Migrar para o SQL do Azure** para abrir o assistente de migração.

1. Na **Etapa 1: Bancos de dados para avaliação**, selecione o banco de dados *AdventureWorks* e escolha **Avançar**.

1. Na **Etapa 2: Resultados da avaliação e recomendações de SKU**, aguarde a conclusão da avaliação e selecione **Avançar**.

## Examinar os resultados da avaliação

Agora você pode analisar as recomendações geradas pela extensão de migração.

1. Na **Etapa 3: Plataforma de destino e resultados de avaliação**, selecione o **Banco de Dados SQL do Azure** como a plataforma de destino.

1. Escolha o banco de dados *AdventureWorks*. Reserve um momento para analisar os resultados da avaliação no lado direito.
    
    > **Observação**: podemos ver que a coluna `Next` adicionada anteriormente foi sinalizada, pois ela pode resultar em um erro no Banco de Dados SQL do Azure.

1. Escolha **Instância Gerenciada de SQL do Azure** como a plataforma de destino do **Banco de Dados SQL do Azure**.
    
    > **Observação**: a coluna `Next` não está mais sinalizada na Instância Gerenciada de SQL do Azure. Por quê? 
    >
    >Isso significa que a coluna `Next` pode ser usada com segurança na Instância Gerenciada de SQL do Azure.

1. Selecione **Salvar relatório de avaliação** para salvar o relatório no formato JSON.

1. Reserve um momento para analisar o arquivo JSON e as propriedades dele.

## Corrigir o problema

1. Execute o comando T-SQL a seguir no banco de dados *AdventureWorks*.

    ```sql
    ALTER TABLE [SalesLT].[Customer] DROP COLUMN [Next];
    ```

1. Volte para a **Etapa 2: Página de resultados da avaliação e recomendações de SKU** no assistente e selecione **Atualizar avaliação**.

1. Selecione o **Banco de Dados SQL do Azure** como a plataforma de destino.

1. Escolha o banco de dados *AdventureWorks*.

    > **Observação:** O banco de dados está pronto para migrar.

Você aprendeu a avaliar a preparação de um banco de dados do SQL Server para migração para o Banco de Dados SQL do Azure. Ao resolver problemas de compatibilidade e fazer alterações de esquema essenciais ou relatá-las, você deu um passo importante na mitigação de possíveis problemas técnicos que podem surgir no futuro no Banco de Dados SQL do Azure.

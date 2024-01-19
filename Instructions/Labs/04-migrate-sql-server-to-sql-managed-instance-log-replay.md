---
lab:
  title: Migrar bancos de dados do SQL Server para a Instância Gerenciada de SQL do Azure usando o Serviço de Reprodução de Log
---

# Migrar bancos de dados do SQL Server para a Instância Gerenciada de SQL do Azure usando o Serviço de Reprodução de Log

Neste exercício, você aprenderá a migrar um banco de dados do SQL Server para a Instância Gerenciada de SQL do Azure usando o Serviço de Reprodução de Logs. 

Você começará implantando uma Instância Gerenciada de SQL do Azure. Em seguida, usará o Serviço de Reprodução de Log para executar uma migração online de um banco de dados do SQL Server para a Instância Gerenciada de SQL do Azure usando o LRS (Serviço de Reprodução de Log). Você também aprenderá a monitorar o processo de migração no PowerShell.

Este exercício levará, aproximadamente, **45** minutos.

> **Observação**: para realizar este exercício, será necessário ter acesso a uma assinatura do Azure para criar recursos do Azure. Se você não tiver uma assinatura do Azure, crie uma [conta gratuita](https://azure.microsoft.com/free/?azure-portal=true) antes de começar.

## Antes de começar

Para executar este exercício, você precisa:

| Item | Descrição |
| --- | --- |
| **Servidor de destino** | Uma instância gerenciada de SQL do Azure Iremos criá-lo durante este exercício.|
| **Servidor de origem** | Uma instância do SQL Server 2019 ou [posterior](https://www.microsoft.com/en-us/sql-server/sql-server-downloads) instalada em um servidor de sua preferência. |
| **Banco de dados de origem** | O banco de dados leve do [AdventureWorks](https://learn.microsoft.com/sql/samples/adventureworks-install-configure) a ser restaurado na Instância do SQL Server. |
| **Azure Data Studio** | Instale o [Azure Data Studio](https://learn.microsoft.com/sql/azure-data-studio/download-azure-data-studio) no mesmo servidor onde o banco de dados de origem está localizado. Se já estiver instalado, atualize-o para garantir a utilização da versão mais recente. |

## Restaurar um banco de dados do SQL Server

Vamos restaurar o banco de dados do *AdventureWorksLT* na instância do SQL Server. Esse banco de dados serve como o banco de dados de origem para este exercício de laboratório. Você pode pular essas etapas se o banco de dados já estiver restaurado.

1. Selecione o botão Iniciar do Windows e digite SSMS. Selecione **Microsoft SQL Server Management Studio 18** na lista.  

1. Quando o SSMS for aberto, observe que a caixa de diálogo **Conectar ao Servidor** é preenchida previamente com o nome da instância padrão. Selecione **Conectar**.

1. Selecione a pasta**Bancos de Dados** e **Nova Consulta**.

1. Na nova janela de consulta, copie e cole o T-SQL a seguir. Execute a consulta para restaurar o banco de dados.

    ```sql
    RESTORE DATABASE AdventureWorksLT
    FROM DISK = 'C:\LabFiles\AdventureWorksLT2019.bak'
    WITH RECOVERY,
          MOVE 'AdventureWorksLT2019_Data' 
            TO 'C:\LabFiles\AdventureWorksLT2019.mdf',
          MOVE 'AdventureWorksLT2019_Log'
            TO 'C:\LabFiles\AdventureWorksLT2019.ldf';
    ```

    > **Observação**: certifique-se de que o nome e o caminho do arquivo de backup do banco de dados no exemplo acima correspondam ao arquivo de backup real. Caso contrário, o comando poderá falhar.

1. Uma mensagem de sucesso será exibida após a conclusão da restauração.

## Implantar uma Instância Gerenciada de SQL do Azure

Crie uma Instância Gerenciada de SQL do Azure seguindo estas etapas:

1. Entre no [portal do Azure](https://portal.azure.com/learn.docs.microsoft.com?azure-portal=true) e selecione **Criar um recurso** no canto superior esquerdo.
1. Procure **Instância gerenciada**, selecione **Instância gerenciada do SQL do Azure** e selecione **Criar**.
1. Preencha o formulário da instância gerenciada do SQL usando as informações na seguinte tabela:

    |  | Valor sugerido |
    |---|---|
    | **Assinatura** | Sua assinatura. |
    | **nome da instância gerenciada** | Qualquer nome válido. |
    | **logon do administrador da instância gerenciada** | Qualquer nome de usuário válido. Não use "serveradmin", pois essa é uma função reservada no nível de servidor. |
    | **Senha** | Qualquer senha com mais de 16 caracteres que atenda aos requisitos de complexidade. |
    | **Fuso horário** | O fuso horário a ser observado pela instância gerenciada. |
    | **Ordenação** | A ordenação que você quer usar para a instância gerenciada. Se você migrar bancos de dados do SQL Server, verifique a ordenação de origem usando SELECT SERVERPROPERTY(N'Collation') e use esse valor. |
    | **Localização** | A região do Azure na qual você quer criar a instância gerenciada. |
    | **Rede virtual** | Selecione "Criar uma rede virtual" ou escolha uma rede virtual e uma sub-rede válidas. |
    | **Habilitar o ponto de extremidade público** | Marque esta opção para habilitar um ponto de extremidade público, que ajuda os clientes fora do Azure a acessar o banco de dados. |
    | **Permitir acesso de** | Selecione nos serviços do Azure, na Internet ou sem acesso. |
    | **Tipo de conexão** | Escolha entre um tipo de conexão de Proxy e Redirecionamento. |
    | **Grupo de recursos** | Um grupo de recursos novo ou existente. |

1. Selecione **Tipo de preço** para dimensionar os recursos de computação e armazenamento e examinar as opções de tipo de preço. 
1. Quando tiver terminado, selecione **Aplicar** para salvar sua seleção e selecione **Criar** para implantar a instância gerenciada.
1. Selecione o **ícone Notificações** para exibir o status da implantação.
1. Selecione **Implantação em andamento** para abrir a janela de instância gerenciada e monitorar melhor o progresso da implantação.

## Crie uma conta e contêiner do Armazenamento de Blobs do Azure

Crie uma conta de Armazenamento de Blobs do Azure na mesma região que sua Instância Gerenciada de SQL do Azure. É aqui que você armazenará seus backups de banco de dados para migração.

1. Vá para o [portal do Azure](https://portal.azure.com) e entre com as credenciais da sua conta.
1. No menu à esquerda, selecione **Todos os serviços** e pesquise *“Contas de armazenamento”*. Selecione **Contas de armazenamento** para abrir a página de contas de armazenamento.
1. Na página Contas de armazenamento, selecione **+ Adicionar** para criar uma nova conta de armazenamento.
1. Na guia **Básicos** da página **Criar conta de armazenamento**, selecione a assinatura que você deseja usar para a conta de armazenamento. Em seguida, selecione o grupo de recursos que contém sua Instância Gerenciada de SQL do Azure.
1. Insira um nome exclusivo para a conta de armazenamento. 
    
    > **Observação:** O nome precisa ter entre 3 e 24 caracteres e pode conter somente letras minúsculas e números.

1. Selecione o local (região) em que a Instância Gerenciada de SQL do Azure está localizada.
1. Selecione a camada de desempenho da conta de armazenamento.
1. Selecione **blobStorage** para o tipo de conta da conta de armazenamento. 
1. Selecione **LRS (armazenamento com redundância local)** para a opção de Replicação da conta de armazenamento.
1. Examine e selecione **Examinar + criar** para criar a conta de armazenamento.
1. Depois que a conta de armazenamento for criada, vá para a página da conta de armazenamento e selecione a opção **Contêineres** no menu à esquerda. Em seguida, selecione **+ Contêiner** para criar um novo contêiner. Insira um nome para o contêiner e selecione o nível de acesso público. 
1. Selecione o botão **Criar** para criar o contêiner.

Depois de concluir essas etapas, você terá uma conta do Armazenamento de Blobs do Azure na mesma região que sua Instância Gerenciada de SQL do Azure e um contêiner em que você pode armazenar seus backups de banco de dados para migração.

## Backup de um banco de dados do SQL Server

Vamos criar um backup completo do banco de dados *AdventureWorksLT* na instância do SQL Server, seguido por um diferencial e backups de log com `CHECKSUM` habilitado. 

1. Selecione o botão Iniciar do Windows e digite SSMS. Selecione **Microsoft SQL Server Management Studio 18** na lista.  
1. Quando o SSMS for aberto, observe que a caixa de diálogo **Conectar ao Servidor** será pré-preenchida com o nome da instância padrão. Selecione **Conectar**.
1. Selecione a pasta**Bancos de Dados** e **Nova Consulta**.
1. Na janela Nova consulta, copie e cole o T-SQL abaixo. Execute a consulta para restaurar o banco de dados.

    ```sql
    BACKUP DATABASE AdventureWorksLT
    TO DISK = 'C:\LabFiles\AdventureWorksLT_full.bak'
    WITH CHECKSUM;

    BACKUP DATABASE AdventureWorksLT
    TO DISK = 'C:\LabFiles\AdventureWorksLT_diff.dif'
    WITH DIFFERENTIAL, CHECKSUM;

    BACKUP LOG AdventureWorksLT
    TO DISK = 'C:\LabFiles\AdventureWorksLT_log.trn'
    WITH CHECKSUM;
    ```

    > **Observação**: Verifique se o caminho do arquivo no exemplo acima corresponde ao caminho do arquivo real. Caso contrário, o comando poderá falhar.

1. Uma mensagem de sucesso será exibida após a conclusão da restauração.
1. Se você estiver executando uma versão do SQL Server (começando com o SQL Server 2012 SP1 CU2 e o SQL Server 2014), poderá fazer backups do SQL Server diretamente para sua conta de Armazenamento de Blobs usando a opção nativa do SQL Server `BACKUP TO URL`. 

    ```sql
    CREATE CREDENTIAL [https://<mystorageaccountname>.blob.core.windows.net/<containername>] 
    WITH IDENTITY = 'SHARED ACCESS SIGNATURE',  
    SECRET = '<SAS_TOKEN>';  
    GO
    
    -- Take a full database backup to a URL
    BACKUP DATABASE [AdventureWorksLT]
    TO URL = 'https://<mystorageaccountname>.blob.core.windows.net/<containername>/<databasefolder>/AdventureWorksLT_full.bak'
    WITH INIT, COMPRESSION, CHECKSUM
    GO
    
    -- Take a differential database backup to a URL
    BACKUP DATABASE [AdventureWorksLT]
    TO URL = 'https://<mystorageaccountname>.blob.core.windows.net/<containername>/<databasefolder>/AdventureWorksLT_diff.bak'  
    WITH DIFFERENTIAL, COMPRESSION, CHECKSUM
    GO
    
    -- Take a transactional log backup to a URL
    BACKUP LOG [AdventureWorksLT]
    TO URL = 'https://<mystorageaccountname>.blob.core.windows.net/<containername>/<databasefolder>/AdventureWorksLT_log.trn'  
    WITH COMPRESSION, CHECKSUM
    ```

    > **Observação:** Se você decidir usar essa opção, poderá ignorar a próxima seção **Copiar arquivos de backup para a conta de Armazenamento do Azure**.

## Copiar arquivos de backup para a conta de Armazenamento do Azure

Agora, vamos copiar os arquivos de backup para a conta do Armazenamento de Blobs do Azure que você criou anteriormente.

1. Vá para o [portal do Azure](https://portal.azure.com) e entre com as credenciais da sua conta.
1. No menu à esquerda, selecione **contas de armazenamento** e, em seguida, selecione a conta de armazenamento que você criou anteriormente.
1. Na página de visão geral da conta de armazenamento, role para baixo até a seção do **serviço Blob** e selecione **Contêineres**. Selecione o contêiner criado anteriormente.
1. Selecione **Carregar** na parte superior do contêiner. Na página **Carregar blob**, selecione **Pasta** para selecionar a pasta que contém os arquivos de backup ou selecione **Arquivos** para escolher arquivos de backup individuais. Depois de selecionar os arquivos, selecione **Carregar** para iniciar o processo de upload.

## Validar o acesso

É importante validar se o SQL Server e sua Instância Gerenciada de SQL podem acessar sua conta de Armazenamento de Blobs com êxito. Para isso, execute uma consulta de teste de exemplo para determinar se sua instância gerenciada é capaz de acessar o backup no contêiner.

1. Conecte-se à Instância Gerenciada de SQL por meio do SSMS.
1. Abra um novo editor de Consultas e execute o comando.

```sql
CREATE CREDENTIAL [https://<mystorageaccountname>.blob.core.windows.net/databases] 
WITH IDENTITY = 'SHARED ACCESS SIGNATURE' 
, SECRET = '<sastoken>' 

RESTORE HEADERONLY 
FROM URL = 'https://<mystorageaccountname>.blob.core.windows.net/<containername>/<backup_file_name>.bak'
```
1. Repita esse processo conectado em sua instância de SQL Server.

## Usar o Serviço de Reprodução de Logs para restaurar arquivos de backup

Você usará o LRS (Serviço de Reprodução de Log) para restaurar os arquivos de backup do Armazenamento de Blobs do Azure para a Instância Gerenciada de SQL do Azure. O LRS é um serviço gratuito baseado na tecnologia de envio de logs do SQL Server.

1. Na página de visão geral da conta de armazenamento, role para baixo até a seção do **serviço Blob** e selecione **Contêineres**. Selecione o contêiner em que os arquivos de backup são armazenados.
1. Selecione **Gerar SAS** na parte superior da página do contêiner. Na página **Gerar assinatura de acesso compartilhado**, selecione as permissões que deseja conceder, defina a hora de início e de expiração para o token SAS e selecione **Gerar SAS e cadeia de conexão**. O token SAS será exibido no campo **Token SAS**, e, em seguida, copie-o.
1. Use o PowerShell para se conectar à sua conta do Azure executando o cmdlet `Connect-AzAccount`.

    ```powershell
    Login-AzAccount
    Select-AzSubscription -SubscriptionId <subscription ID>
    ```

1. Use o cmdlet `Start-AzSqlInstanceDatabaseLogReplay` para iniciar o Serviço de Reprodução de Logs para o banco de dados que você deseja restaurar. Você precisará fornecer o nome do grupo de recursos, o nome da instância, o nome do banco de dados, o URI do contêiner de armazenamento e o token SAS copiado anteriormente.

```PowerShell
Import-Module Az.Sql

Start-AzSqlInstanceDatabaseLogReplay -ResourceGroupName "YourResourceGroupName" -InstanceName "YourInstanceName" -Name "YourDatabaseName" -StorageContainerUri "https://yourstorageaccount.blob.core.windows.net/yourcontainer" -StorageContainerSasToken "YourSasToken"
```

## Monitorar o progresso da migração

Você pode usar o cmdlet `Get-AzSqlInstanceDatabaseLogReplay` para monitorar o progresso do Serviço de Reprodução de Logs. Esse cmdlet retorna informações sobre o status atual do serviço, incluindo o último arquivo de backup de log que foi restaurado.

1. Execute o seguinte código do PowerShell.

```powershell
# Import the Az.Sql module
Import-Module Az.Sql

# Set the resource group name, instance name, and database name
$resourceGroupName = "YourResourceGroupName"
$instanceName = "YourInstanceName"
$databaseName = "YourDatabaseName"

# Get the log replay status
$logReplayStatus = Get-AzSqlInstanceDatabaseLogReplay -ResourceGroupName $resourceGroupName -InstanceName $instanceName -Name $databaseName

# Display the log replay status
$logReplayStatus | Format-List
```

## Executar a substituição de migração

Depois que o backup do banco de dados completo for restaurado na instância de destino da instância gerenciada do Banco de Dados SQL do Azure, o banco de dados estará disponível para a substituição de migração.

1. Quando você estiver pronto para concluir a migração de banco de dados online, selecione **Iniciar substituição**.
1. Interrompa todo o tráfego de entrada para os bancos de dados de origem.
1. Obtenha o backup da parte final do log, disponibilize o arquivo de backup no compartilhamento de rede SMB e aguarde até que esse backup de log de transações final seja restaurado.
1. Nesse ponto, você verá **Alterações pendentes** definido como 0.
1. Selecione **Confirmar**e, em seguida, **Aplicar**.

    ![Tela de substituição de migração](../media/3-migration-cutover-screen.png)

1. Quando o status de migração de banco de dados mostrar **Concluído**, conecte seus aplicativos à nova instância de destino da instância gerenciada do Banco de Dados SQL do Azure.
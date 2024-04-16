---
lab:
  title: Comparar os custos locais do Azure para migração
---

# Comparar os custos locais do Azure para migração

O TCO (custo total de propriedade) é uma ferramenta que você pode usar durante um projeto de modernização da plataforma de dados para avaliar a diferença de custo que a migração pode ter.

No varejista global, é esperado que a modernização da plataforma de dados gere uma economia significativa, mas a diretoria pediu que você estimasse a economia da forma mais precisa possível.

Aqui, você calculará o TCO (custo total de propriedade) da migração para o Azure usando a calculadora de TCO.

Este exercício levará aproximadamente **30** minutos.

## Calcule o TCO (custo total de propriedade)

1. Abra uma nova guia do navegador e navegue até a [calculadora de TCO do Azure](https://azure.microsoft.com/pricing/tco/calculator/).
1. Em **Definir suas cargas de trabalho**, exclua todas as cargas de trabalho existentes na seção **Servidores**.

## Adicione a carga de trabalho do banco de dados

1. Em **Bancos de dados**, selecione **+ Adicionar Banco de dados**.
1. Na caixa de texto **Nome**, digite **Contabilização**.
1. Na seção **Origem**, escolha estes valores:

    | Propriedade | Valor |
    | --- | --- |
    | Banco de dados | **Microsoft SQL Server** |
    | Licença | **Empresa** |
    | Ambiente | **Servidores Físicos** |
    | Sistema operacional | **Windows** |
    | Licença do Sistema Operacional | **Datacenter** |
    | Servidores | **1** |
    | Procs por servidor | **1** |
    | Núcleo(s) por proc | **4** |
    | RAM (GB) | **64** |
    | Otimizar por | **CPU** |
    | SQL Server 2008/2008R2 | **Alterna para selecionar este valor** |

1. Na seção **Destino**, escolha estes valores:

    | Propriedade | Valor |
    | --- | --- |
    | Serviço | **VM do SQL Server** |
    | Tipo de disco | **SSD** |
    | IOPS | **5000** |
    | Armazenamento do SQL Server | **32 GB** |
    | Backup do SQL Server | **32 GB** |

    > [!NOTE]
    > Os SSDs são recomendados para cargas de trabalho de produção no Azure.

## Adicione as cargas de trabalho de armazenamento e rede

1. Em **Armazenamento**, selecione **+ Adicionar armazenamento**.
1. Na caixa de texto **Nome**, digite **Contabilização de discos locais**e, em seguida, insira estes valores:

    | Propriedade | Valor |
    | --- | --- |
    | Tipo de armazenamento | **Disco Local/SAN** |
    | Tipo de disco | **HDD** |
    | Capacity | **3 TB** |
    | Backup | **1 TB** |
    | Archive | **0 TB** |

1. Em **Rede**, nos controles de **Largura de banda de saída**, selecione **1 GB**.
1. Na parte inferior da página, selecione **Avançar**.

## Ajustar as suposições

1. Na seção **Ajustar as suposições**, na lista **Moeda**, selecione sua moeda preferida.
1. Em **Cobertura do Software Assurance (fornece Benefício Híbrido do Azure)**, selecione **cobertura do Software Assurance do Windows Server** habilitando a alternância.
1. Selecione **SQL Server cobertura do Software Assurance** habilitando a alternância.

    > [!NOTE]
    > Use os links fornecidos na seção **Software Assurance** para saber mais sobre a garantia disponível. 

1. Em **GRS (armazenamento com redundância geográfica)**, verifique se a opção **O GRS replica seus dados para uma região secundária que está a centenas de quilômetros da região primária** está desabilitada.
1. Em **custos da Máquina Virtual**, garanta que a opção **Habilitar isso para a Calculadora não recomendar as máquinas virtuais da série B** esteja habilitada.

    > [!NOTE]
    > As máquinas virtuais da série B não têm a proporção de oito de memória para vCore recomendada para cargas de trabalho do SQL Server.

1. Em **Custos de eletricidade**, na caixa de texto **Preço por hora de kW**, insira um valor realista para seu local.

    > [!NOTE]
    > Você pode encontrar preços de eletricidade aproximados em [Preços de eletricidade global](https://www.statista.com/statistics/263492/electricity-prices-in-selected-countries/). Esses preços estão em USD ($). Converta-os em um valor aproximado em sua moeda preferida.

1. Em **Custos de armazenamento**, deixe todos os valores em seus padrões ou ajuste-os se parecerem inaceitáveis.
1. Em **Custos de mão de obra de TI**, deixe todos os valores em seus padrões ou ajuste-os se parecerem inaceitáveis.
1. Em **Outras suposições**, expanda cada seção e examine os custos associados.
1. Na parte inferior da página, selecione **Avançar**.

## Investigar o relatório de cinco anos

1. Na página **Exibir relatório**, observe que o padrão de **Período** é de **5 anos**.
1. Role para baixo o relatório e investigue o detalhamento estimado de custos para os sistemas locais e o Azure. Anote estas informações:

    - Qual é o componente de custo mais significativo do local?
    - Qual será a maior economia se você decidir migrar para o Azure?

1. Expanda cada seção por vez e investigue o detalhamento dos custos.

## Investigar o relatório de três anos

1. Role até a parte superior da página e, em seguida, na caixa de texto **Período**, selecione **3 anos**.
1. Role para baixo o relatório e investigue o detalhamento estimado de custos para os sistemas locais e o Azure. Anote estas informações:

    - Qual é o componente de custo mais significativo do local?
    - Qual será a maior economia se você decidir migrar para o Azure?

1. Expanda cada seção por vez e investigue o detalhamento dos custos.

Você usou a calculadora de TCO do Azure para identificar as diferenças de custo entre as implantações locais e do Azure para o servidor de contabilidade da Adatum Corporation e seus bancos de dados associados.

---
title: Azure Event Hubs 名前空間を別のリージョンに移動する | Microsoft Docs
description: この記事では、Azure Event Hubs 名前空間を現在のリージョンから別のリージョンに移動する方法について説明します。
services: event-hubs
author: spelluru
ms.service: event-hubs
ms.topic: how-to
ms.custom: subject-moving-resources
ms.date: 04/14/2020
ms.author: spelluru
ms.reviewer: shvija
ms.openlocfilehash: 5b96bf1c538b3c5589a1993a0353292fadd0936d
ms.sourcegitcommit: 366e95d58d5311ca4b62e6d0b2b47549e06a0d6d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/01/2020
ms.locfileid: "82690483"
---
# <a name="move-an-azure-event-hubs-namespace-to-another-region"></a>Azure Event Hubs 名前空間を別のリージョンに移動する
既存の Event Hubs 名前空間をあるリージョンから別のリージョンに移動するには、さまざまなシナリオがあります。 たとえば、テスト用に同じ構成で名前空間を作成することができます。 [ディザスター リカバリー計画](event-hubs-geo-dr.md#setup-and-failover-flow)の一部として、別のリージョンにセカンダリ名前空間を作成することもできます。

> [!NOTE]
> この記事では、既存の Event Hubs 名前空間の Azure Resource Manager テンプレートをエクスポートし、そのテンプレートを使用して、別のリージョンに同じ構成設定で名前空間を作成する方法について説明します。 ただし、このプロセスでは、まだ処理されていないイベントは移動されません。 元の名前空間のイベントは、削除する前に処理する必要があります。

## <a name="prerequisites"></a>前提条件

- お使いのアカウントで使用されるサービスと機能が、ターゲット リージョンでサポートされていることを確認してください。
- プレビュー機能については、お使いのサブスクリプションがターゲット リージョンのホワイトリストに登録されていることを確認してください。
- 名前空間のイベント ハブに対して**キャプチャ機能**を有効にしている場合は、Event Hubs 名前空間を移動する前に、[Azure Storage または Azure Data Lake Store Gen 2](../storage/common/storage-account-move.md) アカウントまたは [Azure Data Lake Store Gen 1](../data-lake-store/data-lake-store-migration-cross-region.md) アカウントを移動します。 この記事で説明されている内容と同様の手順に従って、Storage と Event Hubs の両方の名前空間を含むリソース グループを他のリージョンに移動することもできます。 
- Event Hubs 名前空間が **Event Hubs クラスター**にある場合は、この記事の手順を行う前に、**ターゲット リージョン**に[専用クラスターを作成](event-hubs-dedicated-cluster-create-portal.md)します。 [GitHub 上のクイックスタート テンプレート](https://github.com/Azure/azure-quickstart-templates/tree/master/201-eventhubs-create-cluster-namespace-eventhub/)を使用して Event Hubs クラスターを作成することもできます。 このテンプレートから JSON の名前空間の部分を削除して、クラスターのみを作成します。 

## <a name="prepare"></a>準備
まず、Resource Manager テンプレートをエクスポートします。 このテンプレートには、Event Hubs 名前空間を説明する設定が含まれています。

1. [Azure portal](https://portal.azure.com) にサインインします。

2. **[すべてのリソース]** を選択し、Event Hubs 名前空間を選択します。

3. **[設定]**  >  **[テンプレートのエクスポート]** を選択します。

4. **[テンプレートのエクスポート]** ページで **[ダウンロード]** を選択します。

    ![Resource Manager テンプレートをダウンロードする](./media/move-across-regions/download-template.png)

5. ポータルからダウンロードした .zip ファイルを見つけて、選択したフォルダーにそのファイルを解凍します。

   この zip ファイルには、テンプレートとテンプレートをデプロイするためのスクリプトから構成される .json ファイルが含まれています。


## <a name="move"></a>詳細ビュー

テンプレートをデプロイして、ターゲット リージョンに Event Hubs 名前空間を作成します。 


1. Azure Portal で、 **[リソースの作成]** を選択します。

2. **[Marketplace を検索]** で「**template deployment**」と入力し、**Enter** キーを押します。

3. **[テンプレートのデプロイ]** を選択します。

4. **［作成］** を選択します

5. **[Build your own template in the editor] \(エディターで独自のテンプレートをビルド\)** を選択します。

6. **[ファイルの読み込み]** を選択し、手順に従って、前のセクションでダウンロードした **template.json** ファイルを読み込みます。

7. **[保存]** を選択してテンプレートを保存します。 

8. **[カスタム デプロイ]** ページで、次の手順を行います。 

    1. Azure **サブスクリプション**を選択します。 

    2. 既存の**リソース グループ**を選択するか、新しく作成することができます。 ソース名前空間が Event Hubs クラスター内にあった場合は、ターゲット リージョンにクラスターを含むリソース グループを選択します。 

    3. ターゲットの**場所**またはリージョンを選択します。 既存のリソース グループを選択した場合、この設定は読み取り専用になります。 

    4. **[設定]** セクションで、次の手順を行います。
    
        1. 新しい**名前空間名**を入力します。 

            ![Resource Manager テンプレートをデプロイする](./media/move-across-regions/deploy-template.png)

        2. ソース名前空間が **Event Hubs クラスター**に含まれていた場合は、**リソース グループ**の名前を入力し、**Event Hubs クラスター**を**外部 ID** の一部としてします。 

              ```
              /subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<CLUSTER'S RESOURCE GROUP>/providers/Microsoft.EventHub/clusters/<CLUSTER NAME>
              ```   
        3. 名前空間のイベント ハブでイベントのキャプチャにストレージ アカウントを使用する場合は、リソース グループ名と `StorageAccounts_<original storage account name>_external` フィールドのストレージ アカウントを指定します。 
            
            ```
            /subscriptions/0000000000-0000-0000-0000-0000000000000/resourceGroups/<STORAGE'S RESOURCE GROUP>/providers/Microsoft.Storage/storageAccounts/<STORAGE ACCOUNT NAME>
            ```    
    5. **[上記の使用条件に同意する]** チェック ボックスをオンにします。 
    
    6. 次に、 **[Select Purchase]\(購入の選択\)** を選択してデプロイ プロセスを開始します。 

## <a name="discard-or-clean-up"></a>破棄またはクリーンアップ
デプロイ後に、最初からやり直す場合は、**ターゲット Event Hubs 名前空間**を削除し、この記事の「[準備](#prepare)」と「[移動](#move)」のセクションに示されている手順を繰り返します。

変更をコミットし、Event Hubs 名前空間の移動を完了するには、**ソース Event Hubs 名前空間**を削除します。 名前空間を削除する前に、名前空間内のすべてのイベントを処理したことを確認します。 

Azure portal を使用して Event Hubs 名前空間 (ソースまたはターゲット) を削除するには、次の手順を行います。

1. Azure portal の上部にある検索ウィンドウで「**Event Hubs**」と入力し、検索結果から **[Event Hubs]** を選択します。 Event Hubs 名前空間が一覧に表示されます。

2. 削除するターゲット名前空間を選択し、ツールバーの **[削除]** を選択します。 

    ![名前空間の削除 - ボタン](./media/move-across-regions/delete-namespace-button.png)

3. **[リソースの削除]** * ページで、選択したリソースを確認し、「**yes**」と入力して削除を確定し、 **[削除]** を選択します。 

## <a name="next-steps"></a>次のステップ

このチュートリアルでは、Azure Event Hubs 名前空間をあるリージョンから別のリージョンに移動し、ソース リソースをクリーンアップしました。  リージョン間でのリソースの移動と Azure でのディザスター リカバリーの詳細については、以下を参照してください。


- [リソースを新しいリソース グループまたはサブスクリプションに移動する](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-move-resources)
- [Azure VM を別のリージョンに移動する](https://docs.microsoft.com/azure/site-recovery/azure-to-azure-tutorial-migrate)

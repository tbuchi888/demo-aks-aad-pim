# demo-aks-aad-pim
## 0. はじめに
これは、Azure resource role について PIM の対象化および割当を行う、ARM テンプレートのサンプルです
* PIM のサンプルとして AKS クラスタへの 'Azure Kubernetes Service RBAC Writer' Azure ロール を 対象としています
  * AKS での AAD を利用した RBAC の事前設定についてはこちらを参考に設定してください
    * https://learn.microsoft.com/ja-jp/azure/aks/manage-azure-rbac
* PIMの対象化 "Microsoft.Authorization/roleEligibilityScheduleRequests"
  * https://learn.microsoft.com/en-us/azure/templates/microsoft.authorization/roleassignmentschedulerequests?pivots=deployment-language-arm-template

* PIMの割当 "Microsoft.Authorization/roleAssignmentScheduleRequests"
  * https://learn.microsoft.com/en-us/azure/templates/microsoft.authorization/roleassignmentschedulerequests?pivots=deployment-language-arm-template

* Azure resource への PIM の概念や設定パラメータ等に精通していない場合は、事前に Portal UI での操作等を説明している以下公式ドキュメントを一読ください
  * https://learn.microsoft.com/ja-jp/azure/active-directory/privileged-identity-management/pim-resource-roles-assign-roles

> **Warning**
> * 現時点では、Azure リソースの PIM を利用した場合 AKS の Namespace 単位でのスコープ設定はできないようです
>  * Portal UI から Namespace を指定できず、ARM テンプレートでは指定すると、404 エラーとなります
> * Azure Active Directory (Azure AD) と AKS の間で統合認証を利用した、Kubernetes 認可に Azure RBAC を使用においてスコープとして Namespace 単位でロールの割当が可能ですが、 AKS namespace 単位で JIT（Just-in-Time）アクセスを実現したい場合は、PIM 特権アクセスグループ（Preview）の利用を検討してください
> * [Configure just-in-time cluster access with Azure AD and AKS](https://learn.microsoft.com/en-us/azure/aks/managed-aad#configure-just-in-time-cluster-access-with-azure-ad-and-aks)

## 1. 使い方
PIMの対象化と割当（特権のアクティブ化）として以下2種類の ARM テンプレートを用意
* PIM の対象化（対象となったアカウントは、自分で特権のアクティブ化をリクエストできる）
  * ['arm-roleEligibilityScheduleRequests.json'](https://github.com/tbuchi888/demo-aks-aad-pim/blob/main/arm-roleEligibilityScheduleRequests.json)
* PIMの割当(特権アサインのアクティブ化)
  * ['arm-roleAssignmentScheduleRequests.json'](https://github.com/tbuchi888/demo-aks-aad-pim/blob/main/arm-roleAssignmentScheduleRequests.json)
  
### 1.1. パラメータ（共通） 
以下を自身の環境や用途に応じて変更して利用
* principalId: 対象の SP または、USER の ObejectID を事前に確認の上設定
* roleDefinitionId: 設定したい Azure リソースのロール のIDを、"/subscriptions/~"からはじまる形でフルで設定
    * Azure Portal の Access control (IAM) や `az role definition list --name "Azure Kubernetes Service RBAC Writer"`等で確認
* scope: 対象の Azure resouce を設定します。なお、前述の通り、AKS の Namespace 単位での指定はできません。
* id: 変更不要です。リクエスト毎に、ユニークなguidを生成します。
* requestType: 利用者からのリクエストの場合、SelfActivate, 管理者からの場合は AdminAssign, AdminUpdate 等を設定します。
* justification: リクエストの理由などを正当性を記載します
* startDateTime: 開始日を ISO 8601 形式で設定します。日本時間GMT+9での設定例）2023-02-16T18:00:00.00+09:00
* duration: 有効な期間を ISO 8601 形式で設定します。30分 PT30M、365日 P365D 

### 1.2. az コマンド での ARM テンプレート 実行例

```
# roleEligibilityScheduleRequest
az deployment group create --resource-group YOUR-RESOURCE-GROUP --template-file ./arm-roleEligibilityScheduleRequests.json
# roleAssignmentScheduleRequest
az deployment group create --resource-group YOUR-RESOURCE-GROUP --template-file ./arm-roleAssignmentScheduleRequests.json
```

### 1.3. 実行結果の例

```
$ az deployment group create --resource-group YOUR-RESOURCE-GROUP --template-file ./arm-roleEligibilityScheduleRequests.json
{
  "id": "/subscriptions/YOUR-SUBSCRIPTION/resourceGroups/YOUR-RESOURCE-GROUP/providers/Microsoft.Resources/deployments/arm-roleEligibilityScheduleRequests",
  "location": null,
  "name": "arm-roleEligibilityScheduleRequests",
  "properties": {
    "correlationId": "9cdccb62-fe5c-4000-96b2-a90f7f854d4a",
    "debugSetting": null,
    "dependencies": [],
    "duration": "PT6.0765989S",
    "error": null,
    "mode": "Incremental",
    "onErrorDeployment": null,
    "outputResources": [
      {
        "id": "/subscriptions/YOUR-SUBSCRIPTION/resourceGroups/YOUR-RESOURCE-GROUP/providers/Microsoft.ContainerService/managedClusters/YOUR-AKS/providers/Microsoft.Authorization/roleEligibilityScheduleRequests/83e71fba-4f29-4dc3-a920-97608015cff7",
        "resourceGroup": "YOUR-RESOURCE-GROUP"
      }
    ],
    "outputs": {},
    "parameters": {
      "duration": {
        "type": "String",
        "value": "P365D"
      },
      "id": {
        "type": "String",
        "value": "83e71fba-4f29-4dc3-a920-97608015cff7"
      },
      "justification": {
        "type": "String",
        "value": "THIS IS TEST VIA ARM TEMPLATE."
      },
      "principalId": {
        "type": "String",
        "value": "YOUR-SP-OR-USER-OBJECT-ID"
      },
      "requestType": {
        "type": "String",
        "value": "AdminUpdate"
      },
      "roleDefinitionId": {
        "type": "String",
        "value": "/subscriptions/YOUR-SUBSCRIPTION/providers/Microsoft.Authorization/roleDefinitions/a7ffa36f-339b-4b5c-8bdf-e2c188b2c0eb"
      },
      "scope": {
        "type": "String",
        "value": "/subscriptions/YOUR-SUBSCRIPTION/resourceGroups/YOUR-RESOURCE-GROUP/providers/Microsoft.ContainerService/managedClusters/YOUR-AKS"
      },
      "startDateTime": {
        "type": "String",
        "value": "2023-02-16T18:00:00.00+09:00"
      }
    },
    "parametersLink": null,
    "providers": [
      {
        "id": null,
        "namespace": "Microsoft.Authorization",
        "providerAuthorizationConsentState": null,
        "registrationPolicy": null,
        "registrationState": null,
        "resourceTypes": [
          {
            "aliases": null,
            "apiProfiles": null,
            "apiVersions": null,
            "capabilities": null,
            "defaultApiVersion": null,
            "locationMappings": null,
            "locations": [
              null
            ],
            "properties": null,
            "resourceType": "roleEligibilityScheduleRequests",
            "zoneMappings": null
          }
        ]
      }
    ],
    "provisioningState": "Succeeded",
    "templateHash": "8651735680548649475",
    "templateLink": null,
    "timestamp": "2023-02-17T01:16:10.100559+00:00",
    "validatedResources": null
  },
  "resourceGroup": "YOUR-RESOURCE-GROUP",
  "tags": null,
  "type": "Microsoft.Resources/deployments"
}
$ az deployment group create --resource-group YOUR-RESOURCE-GROUP --template-file ./arm-roleAssignmentScheduleRequests.json
{
  "id": "/subscriptions/YOUR-SUBSCRIPTION/resourceGroups/YOUR-RESOURCE-GROUP/providers/Microsoft.Resources/deployments/arm-roleAssignmentScheduleRequests",
  "location": null,
  "name": "arm-roleAssignmentScheduleRequests",
  "properties": {
    "correlationId": "798170bc-3f6a-41f2-ac1a-e16ded0e5e72",
    "debugSetting": null,
    "dependencies": [],
    "duration": "PT7.0195961S",
    "error": null,
    "mode": "Incremental",
    "onErrorDeployment": null,
    "outputResources": [
      {
        "id": "/subscriptions/YOUR-SUBSCRIPTION/resourceGroups/YOUR-RESOURCE-GROUP/providers/Microsoft.ContainerService/managedClusters/YOUR-AKS/providers/Microsoft.Authorization/roleAssignmentScheduleRequests/d1f2d969-f908-420f-bd5d-dc25c1f64bbf",
        "resourceGroup": "YOUR-RESOURCE-GROUP"
      }
    ],
    "outputs": {},
    "parameters": {
      "duration": {
        "type": "String",
        "value": "PT90M"
      },
      "id": {
        "type": "String",
        "value": "d1f2d969-f908-420f-bd5d-dc25c1f64bbf"
      },
      "justification": {
        "type": "String",
        "value": "THIS IS TEST VIA ARM TEMPLATE."
      },
      "principalId": {
        "type": "String",
        "value": "YOUR-SP-OR-USER-OBJECT-ID"
      },
      "requestType": {
        "type": "String",
        "value": "AdminAssign"
      },
      "roleDefinitionId": {
        "type": "String",
        "value": "/subscriptions/YOUR-SUBSCRIPTION/providers/Microsoft.Authorization/roleDefinitions/a7ffa36f-339b-4b5c-8bdf-e2c188b2c0eb"
      },
      "scope": {
        "type": "String",
        "value": "/subscriptions/YOUR-SUBSCRIPTION/resourceGroups/YOUR-RESOURCE-GROUP/providers/Microsoft.ContainerService/managedClusters/YOUR-AKS"
      },
      "startDateTime": {
        "type": "String",
        "value": "2023-02-16T18:00:00.00+09:00"
      }
    },
    "parametersLink": null,
    "providers": [
      {
        "id": null,
        "namespace": "Microsoft.Authorization",
        "providerAuthorizationConsentState": null,
        "registrationPolicy": null,
        "registrationState": null,
        "resourceTypes": [
          {
            "aliases": null,
            "apiProfiles": null,
            "apiVersions": null,
            "capabilities": null,
            "defaultApiVersion": null,
            "locationMappings": null,
            "locations": [
              null
            ],
            "properties": null,
            "resourceType": "roleAssignmentScheduleRequests",
            "zoneMappings": null
          }
        ]
      }
    ],
    "provisioningState": "Succeeded",
    "templateHash": "8187991026471893073",
    "templateLink": null,
    "timestamp": "2023-02-16T09:03:10.577982+00:00",
    "validatedResources": null
  },
  "resourceGroup": "YOUR-RESOURCE-GROUP",
  "tags": null,
  "type": "Microsoft.Resources/deployments"
}
```

## 2. その他
### 2.1. 参考 PIM 特権アクセスグループ（Preview）の Powershell（非推奨）の利用について
以下は、[既に廃止が決定している(廃止予定日 2023/06/30 )](https://jpazureid.github.io/blog/azure-active-directory/how-to-determine-depreacated-azuread-msol/)の Powershell の旧版 Preview コマンドのため、非推奨ではありますが廃止までの期間限定で利用可能

```
# (1)  PowerShell を管理者にて起動し以下のコマンドを実行、Azure AD PowerShell インストール済みの場合は、競合するためいったんアンインストール
Uninstall-Module AzureAD

# (2) Azure AD Preview PowerShell をインストール
Install-Module -Name AzureADPreview

# (3) テナント ID を設定し、以下のコマンドを実行し、テナントの管理者でサインイン
$tenantid = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
Connect-AzureAD -TenantId $tenantid

# (4) グループに追加したいユーザーのオブジェクト ID、グループのオブジェクト ID を変数に設定し、以下のコマンドを実行してロール定義 ID を取得
$userid = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" 
$groupid = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
Get-AzureADMSPrivilegedRoleDefinition -ProviderId "aadGroups" -ResourceId $groupid

# (5) 上記 `Get-AzureADMSPrivilegedRoleDefinition` コマンドの出力結果の、
# Member（ Owner ではなく、2段目の Memberの Id であることに注意） の ID の値を以下のように`$RoleDefinitionId`変数へ設定
# 出力例
# Get-AzureADMSPrivilegedRoleDefinition -ProviderId "aadGroups" -ResourceId $groupid
# 
# Id                      : aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa
# ResourceId              : XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX
# ExternalId              : XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX
# DisplayName             : Owner
# SubjectCount            :
# EligibleAssignmentCount :
# ActiveAssignmentCount   :
#
# Id                      : bbbbbbbb-bbbb-bbbb-bbbb-bbbbbbbbbbbb
# ResourceId              : XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX
# ExternalId              : XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX
# DisplayName             : Member
# SubjectCount            :
# EligibleAssignmentCount :
# ActiveAssignmentCount   :
# 

$RoleDefinitionId = "bbbbbbbb-bbbb-bbbb-bbbb-bbbbbbbbbbbb"

# (6) スケジュール等を設定し、以下の `Open-AzureADMSPrivilegedRoleAssignmentRequest` コマンドを実行し、資格のある割り当てを実行
# 割当対象化ではなく、アクティブ化したいときは、 `-AssignmentState` "Eligible" の代わりに "Active" を設定
$schedule = New-Object Microsoft.Open.MSGraph.Model.AzureADMSPrivilegedSchedule
$schedule.Type = "Once"
$schedule.StartDateTime = "2023-02-08T14:00:00.00+09:00"
$schedule.endDateTime = "2023-03-09T14:00:00.00+09:00"
 
Open-AzureADMSPrivilegedRoleAssignmentRequest -ProviderId aadgroups -Schedule $schedule -ResourceId $groupid  -RoleDefinitionId $RoleDefinitionId  -SubjectId $userid -AssignmentState "Eligible" -Type "AdminAdd" -Reason test
```

---

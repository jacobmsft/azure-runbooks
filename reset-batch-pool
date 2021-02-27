#
# reset-batch-pool - Resets and azure batch pool - resizes to zero and back to current state (removes the cached containers if running batch for containers)
# be sure to import the included modules into the automation account
# 
param
(
		[string]
		[Parameter(Mandatory=$false)] 
		$batchAccountName = 'batch_account',

		[string]
		[Parameter(Mandatory=$false)] 
		$poolName = 'batch_pool'
)

Set-StrictMode -Version Latest 

Import-Module AzureRM.Profile
Import-Module AzureRM.Batch

# authenticate
$connectionName = "AzureRunAsConnection"
try
{
    # Get the connection "connectionName "
    $servicePrincipalConnection=Get-AutomationConnection -Name $connectionName         

    "Logging in to Azure..."
    Add-AzureRmAccount `
        -ServicePrincipal `
        -TenantId $servicePrincipalConnection.TenantId `
        -ApplicationId $servicePrincipalConnection.ApplicationId `
        -CertificateThumbprint $servicePrincipalConnection.CertificateThumbprint 
}
catch {
    if (!$servicePrincipalConnection)
    {
        $ErrorMessage = "Connection $connectionName not found."
        throw $ErrorMessage
    } else{
        Write-Error -Message $_.Exception
        throw $_.Exception
    }
}

Write-Output "----------------------------"
Write-Output "----------------------------"
Write-Output "----------------------------"
Write-Output "*** Begin Batch Pool Reset: "(Get-Date)

#set the context
Set-AzureRmContext -SubscriptionId $servicePrincipalConnection.SubscriptionId  


$batchContext = Get-AzureRmBatchAccount -AccountName $batchAccountName
$pool = Get-AzureBatchPool -Id $poolName -BatchContext $batchContext

# get the current values
$TargetDedicatedComputeNodes = $pool.TargetDedicatedComputeNodes
$TargetLowPriorityComputeNodes = $pool.TargetLowPriorityComputeNodes       

# resize to zero nodes
Write-Output "Resizing pool to zero nodes"
Start-AzureBatchPoolResize -Id "defaultpool" -TargetDedicatedComputeNodes 0 -TargetLowPriorityComputeNodes 0 -BatchContext $batchContext

Write-Output "Waiting for allocation state to become Steady"

Do {
    Write-Output "Sleeping 10 seconds..."
    Start-Sleep 10
    $pool = Get-AzureBatchPool -Id $poolName -BatchContext $batchContext
}
While ($pool.AllocationState -ne "Steady")

# resize back to previous state
Write-Output "Resizing pool back to old values: TargetDedicatedComputeNodes: $TargetDedicatedComputeNodes, TargetLowPriorityComputeNodes: $TargetLowPriorityComputeNodes"
Start-AzureBatchPoolResize -Id "defaultpool" -TargetDedicatedComputeNodes $TargetDedicatedComputeNodes -TargetLowPriorityComputeNodes $TargetLowPriorityComputeNodes -BatchContext $batchContext

Do {
    Start-Sleep 10
    $pool = Get-AzureBatchPool -Id $poolName -BatchContext $batchContext
    Write-Output $pool
}
While ($pool.AllocationState -ne "Steady")

Write-Output "----------------------------"
Write-Output "----------------------------"
Write-Output "----------------------------"
Write-Output "*** End Batch Pool Reset: "(Get-Date)








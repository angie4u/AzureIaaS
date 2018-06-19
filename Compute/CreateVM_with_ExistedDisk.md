### 기존 가상 머신 디스크를 사용하여, 가상 머신을 다시 생성하는 PowerShell 스크립트입니다.

> 가상 머신 크기에 대한 문서는 [여기](https://docs.microsoft.com/ko-kr/azure/virtual-machines/windows/sizes)를 참고합니다. 지역별로 제공하는 가상 머신 크기는 상이하므로, 정확한 크기를 확인하고자 할 경우, 아래 *PowerShell*을 통해 가능합니다.
```powershell
Get-AzureRmLocation # 서비스를 제공하고 있는 지역 확인
Get-AzureRmVMSize -Location '<지역>'
```
```powershell
# 해당 스크립트는 가상 머신과 관련된 모든 리소스가 같은 리소스 그룹에 들어 있다고 가정하고 작성되었습니다.

$ResourceGroupName = "<리소스 그룹 이름>"
$VMSize = "<가상 머신 크기>"
$Location = "<지역>"
$AVSName = "<가용성 집합>"
$NICName = "<NIC 이름>"
$diagstorageName = "<부트 진단 저장소 계정 이름>"
$OSDiskName = "<OS 디스크 이름>"
$DataDiskName = "<데이터 디스크 이름>" # 데이터 디스크가 있는 경우, 이름 명시 필요

$avs = Get-AzureRmAvailabilitySet -ResourceGroupName $ResourceGroupName -Name $AVSName # 가용성 집합을 사용하지 않는 경우, 해당 줄 주석 처리 필요
$nic = Get-AzureRmNetworkInterface -Name $NICName -ResourceGroupName $ResourceGroupName

$vm = New-AzureRmVMConfig -VMName $VMName -VMSize $VMSize -AvailabilitySetId $avs.id # 가용성 집합을 사용하지 않는 경우, -AvailabilitySetId 주석 처리 필요
$vm = Add-AzureRmVMNetworkInterface -VM $vm -Id $nic.Id
$vm = Set-AzureRmVMBootDiagnostics -VM $vm -Enable -ResourceGroupName $resourceGroupName -StorageAccountName $diagstoragename

$OSDisk = Get-AzureRmDisk | ? {$_.Name -eq $OSDiskName}
$vm = Set-AzureRmVMOSDisk -VM $vm -ManagedDiskId $OSDisk.Id -CreateOption Attach -Windows

# 데이터 디스크가 없는 경우, 아래 2줄 주석 처리
$dataDisk = Get-AzureRmDisk | ? {$_.Name -eq $DataDiskName}
$vm = Add-AzureRmVMDataDisk -VM $vm -Name $DataDiskName -CreateOption Attach -ManagedDiskId $dataDisk.Id -Lun 1

New-AzureRmVM -VM $vm -ResourceGroupName $resourceGroupName -Location $location -LicenseType "Windows_Server"
```
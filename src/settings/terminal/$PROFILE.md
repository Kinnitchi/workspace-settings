```
$MaximumHistoryCount = 20000

Import-Module posh-git
Import-Module PSReadLine
Import-Module Get-ChildItemColor
Import-Module Terminal-Icons
Import-Module DockerCompletion

oh-my-posh init pwsh --config "$HOME\AppData\Local\Programs\oh-my-posh\themes\kinnitchi.omp.json" | Invoke-Expression

Set-PSReadLineKeyHandler -Key Tab -Function MenuComplete

function Set-PoshGitStatus {
  $global:GitStatus = Get-GitStatus
  $env:POSH_GIT_STRING = Write-GitStatus -Status $global:GitStatus
}
New-Alias -Name 'Set-PoshContext' -Value 'Set-PoshGitStatus' -Scope Global -Force

$HistoryFilePath = Join-Path ([Environment]::GetFolderPath('UserProfile')) .ps_history
Register-EngineEvent PowerShell.Exiting -Action { Get-History | Export-Clixml $HistoryFilePath } | out-null
if (Test-path $HistoryFilePath) { Import-Clixml $HistoryFilePath | Add-History }

Set-PSReadLineOption -HistorySearchCursorMovesToEnd
Set-PSReadlineKeyHandler -Key UpArrow -Function HistorySearchBackward
Set-PSReadlineKeyHandler -Key DownArrow -Function HistorySearchForward
Set-PSReadLineOption -ShowToolTips
Set-PSReadLineOption -PredictionSource History
Set-Alias which Get-Command
Set-Alias open Invoke-Item

function ll() { Get-ChildItem | Format-Table }
function la() { Get-ChildItem | Format-Wide }
function lb() { Get-ChildItem | Format-List }

Set-Alias ls la
Set-Alias l lb

function sdd() { Set-Location "C:\Downloads" }
function sdp() { Set-Location "C:\Users\igori\OneDrive\Documentos\WORKSPACE\" }
function sdf() { Set-Location "C:\Users\igori\OneDrive\Documentos\WORKSPACE\FLUIG" }
function opd() { open "C:\Downloads" }
function opp() { open "C:\Users\igori\OneDrive\Documentos\WORKSPACE\" }
function cdp() { code $PROFILE }
function cdh() { code "$HOME\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt" }
function cds() { code "$HOME\AppData\Local\oh-my-posh\kinnitchi.omp.json" }
function cdf() { code "C:\Users\igori\OneDrive\Documentos\WORKSPACE\FLUIG" }
function cdp() { code "C:\Users\igori\OneDrive\Documentos\WORKSPACE\" }
function md5    { Get-FileHash -Algorithm MD5 $args }
function sha1   { Get-FileHash -Algorithm SHA1 $args }
function sha256 { Get-FileHash -Algorithm SHA256 $args }
function tail { Get-Content $args -Tail 30 -Wait }
function take {
  New-Item -ItemType directory $args
  Set-Location "$args"
}
```
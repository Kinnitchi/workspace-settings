# ==============================================
# PowerShell 7 Profile - Igor Custom
# ==============================================

# ===== Core Settings =====
$MaximumHistoryCount = 20000
$HistoryFilePath = Join-Path $HOME ".ps_history"

if (Test-Path $HistoryFilePath){ 
    Import-Clixml $HistoryFilePath | Add-History
}

Register-EngineEvent PowerShell.Exiting -Action {
    Get-History | Export-Clixml $HistoryFilePath
} | Out-Null

# ===== Module Imports =====
$modules = @('posh-git', 'PSReadLine', 'Get-ChildItemColor', 'Terminal-Icons', 'DockerCompletion')
foreach ($mod in $modules) {
    if (Get-Module -ListAvailable -Name $mod) {
        Import-Module $mod -ErrorAction SilentlyContinue
    }
}

# ===== Oh-My-Posh Theme =====
$ompTheme = Join-Path $HOME "AppData\Local\Programs\oh-my-posh\themes\catppuccin_mocha.omp.json"
oh-my-posh init pwsh --config $ompTheme | Invoke-Expression

# ===== PSReadLine Options =====
Set-PSReadLineKeyHandler -Key Tab -Function MenuComplete
Set-PSReadLineKeyHandler -Key UpArrow -Function HistorySearchBackward
Set-PSReadLineKeyHandler -Key DownArrow -Function HistorySearchForward
Set-PSReadLineOption -HistorySearchCursorMovesToEnd
Set-PSReadLineOption -ShowToolTips
Set-PSReadLineOption -PredictionSource History

# ===== Git Integration =====
function Set-PoshGitStatus {
    $global:GitStatus = Get-GitStatus
    $env:POSH_GIT_STRING = Write-GitStatus -Status $global:GitStatus
}
New-Alias -Name 'Set-PoshContext' -Value 'Set-PoshGitStatus' -Scope Global -Force

# ===== Aliases =====
Set-Alias which Get-Command
Set-Alias open Invoke-Item
Set-Alias ls la
Set-Alias l lb

# ===== Directory Shortcuts =====
$workspace = "C:\Users\Kinnitchi\OneDrive\Documentos\WORKSPACE"
$fluig     = Join-Path $workspace "FLUIG"
$downloads = "C:\Downloads"

function sdd { Set-Location $downloads }
function sdp { Set-Location $workspace }
function sdf { Set-Location $fluig }

function opd { open $downloads }
function opp { open $workspace }

# ===== Quick Code Editing =====
function cdp { code $PROFILE }
function cdh { code (Join-Path $HOME "AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt") }
function cds { code "$HOME\AppData\Local\oh-my-posh\spaceship.omp.json" }
function cdf { code $fluig }

# ===== File Browsing & Hashing =====
function ll { Get-ChildItem | Format-Table }
function la { Get-ChildItem | Format-Wide }
function lb { Get-ChildItem | Format-List }

function md5    { Get-FileHash -Algorithm MD5 $args }
function sha1   { Get-FileHash -Algorithm SHA1 $args }
function sha256 { Get-FileHash -Algorithm SHA256 $args }

function tail { Get-Content $args -Tail 30 -Wait }

function take {
    param([string]$name)
    New-Item -ItemType Directory -Name $name -Force | Out-Null
    Set-Location $name
}

# ===== Ferramentas de Qualidade de Vida =====
function h {
    if (Get-Command fzf -ErrorAction SilentlyContinue) {
        Get-History | Sort-Object -Property Id -Descending | Select-Object -ExpandProperty CommandLine | fzf
    } else {
        Write-Warning "fzf não está instalado." 
    }
}

function fzf-find {
    if (Get-Command rg, fzf -ErrorAction SilentlyContinue) {
        $preview = if (Get-Command bat -ErrorAction SilentlyContinue) {
            '--preview "bat --style=numbers --color=always {}"'
        } else {
            '--preview "Get-Content {} | Out-String"'
        }
        Invoke-Expression "rg --files | fzf $preview"
    } else {
        Write-Warning "rg ou fzf não estão instalados."
    }
}

# ===== Navegação com fzf =====
function fcd {
    if (Get-Command fzf -ErrorAction SilentlyContinue) {
        Set-Location (Get-ChildItem -Directory | Select-Object -ExpandProperty FullName | fzf)
    } else {
        Write-Warning "fzf não está instalado."
    }
}

function vf {
    if (Get-Command fzf -ErrorAction SilentlyContinue) {
        code (Get-ChildItem -Recurse -File | Select-Object -ExpandProperty FullName | fzf)
    } else {
        Write-Warning "fzf não está instalado."
    }
}

# ===== Dev Tools =====
function serve {
    if (Test-Path "package.json") {
        npm run dev
    } elseif (Test-Path "pom.xml") {
        mvn spring-boot:run
    } elseif (Test-Path "build.gradle") {
        gradle bootRun
    } else {
        Write-Host "Projeto não reconhecido"
    }
}

function npmi { npm install }
function npms { npm start }
function npmb { npm run build }

function mvnc { mvn clean install }
function mvnt { mvn test }

function gpull { git pull origin (git branch --show-current) }
function gpush { git push origin (git branch --show-current) }

# ===== Menu de Ambientes de Projeto =====
function menu-projeto {
    $choices = @(
        @{ Label = "🚧 ALURA";     Path = Join-Path $workspace "\ALURA" },
        @{ Label = "🧪 CLONE"; Path = Join-Path $workspace "\CLONE" },
        @{ Label = "🚀 FLUIG";    Path = Join-Path $workspace "\FLUIG" }
        @{ Label = "🚀 PROJECTS";    Path = Join-Path $workspace "\PROJECTS" }
    )

    Write-Host "Selecione um ambiente:" -ForegroundColor Cyan
    for ($i = 0; $i -lt $choices.Count; $i++) {
        Write-Host "$($i + 1)) $($choices[$i].Label)"
    }

    $selection = Read-Host "Digite o número da opção"
    if ($selection -match '^[1-9]$' -and $selection -le $choices.Count) {
        $target = $choices[$selection - 1]
        Set-Location $target.Path
        Write-Host "Você está agora em: $($target.Label) → $($target.Path)" -ForegroundColor Green
    } else {
        Write-Warning "Opção inválida."
    }
}

Set-Alias projeto menu-projeto


function main-menu {
    $menu = @(
        @{ Label = "📁 Navegar por ambientes de projeto";   Action = { menu-projeto } }
        @{ Label = "🚀 Rodar projeto atual (serve)";        Action = { serve } }
        @{ Label = "📝 Editar profile";                     Action = { code $PROFILE } }
        @{ Label = "🐙 GitHub: abrir repositório";          Action = { gh-browse } }
        @{ Label = "🐳 Docker: listar containers";          Action = { docker ps -a | Out-Host } }
        @{ Label = "🌿 Git: trocar de branch/tag";          Action = { git-switch } }
        @{ Label = "❌ Sair";                               Action = { return } }
    )

    Write-Host "`n🔧 Menu Principal:" -ForegroundColor Cyan
    for ($i = 0; $i -lt $menu.Count; $i++) {
        Write-Host "$($i + 1)) $($menu[$i].Label)"
    }

    $choice = Read-Host "Escolha uma opção"
    if ($choice -match '^[1-9]$' -and $choice -le $menu.Count) {
        Clear-Host
        & $menu[$choice - 1].Action
    } else {
        Write-Warning "Opção inválida."
    }
}

Set-Alias menu main-menu

function gh-browse {
    if (-not (Get-Command gh -ErrorAction SilentlyContinue)) {
        Write-Warning "GitHub CLI (gh) não está instalado."
        return
    }

    try {
        gh repo view --web
    } catch {
        Write-Warning "Falha ao abrir repositório. Verifique se está em um repositório git."
    }
}

function git-switch {
    if (-not (Get-Command fzf -ErrorAction SilentlyContinue)) {
        Write-Warning "fzf não está instalado."
        return
    }

    $branches = git for-each-ref --format='%(refname:short)' refs/heads refs/remotes refs/tags | Sort-Object -Unique
    if ($branches) {
        $selection = $branches | fzf --prompt "Escolha branch/tag: "
        if ($selection) {
            git checkout $selection
        }
    } else {
        Write-Warning "Nenhuma referência encontrada."
    }
}

# ===== Verificação de Dependências no Startup =====
$tools = @('git', 'npm', 'fzf', 'code', 'bat', 'rg')
$missingTools = @()

foreach ($tool in $tools) {
    if (-not (Get-Command $tool -ErrorAction SilentlyContinue)) {
        $missingTools += $tool
    }
}

if ($missingTools.Count -gt 0) {
    Write-Warning "As seguintes ferramentas não estão instaladas: $($missingTools -join ', ')"
}
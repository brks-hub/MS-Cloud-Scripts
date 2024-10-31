# Pre-requisites: Install-Module Microsoft.Graph, Install-Module IntuneWin32App, PSIntuneAuth, and Azure.Identity

# Variables for registered app credentials
$ClientID = Read-Host -Prompt "Enter your Client ID"
$TenantID = Read-Host -Prompt "Enter your Tenant ID"
$ClientSecret = Read-Host -Prompt "Enter your Client Secret" -AsSecureString

# Connect to Microsoft Graph using client credentials
$ClientSecretPlainText = [Runtime.InteropServices.Marshal]::PtrToStringAuto([Runtime.InteropServices.Marshal]::SecureStringToBSTR($ClientSecret))
Connect-MSIntuneGraph -ClientId $ClientID -TenantId $TenantID -ClientSecret $ClientSecretPlainText

# Create working directory for the application, set download location, and download installer
$appfolder = New-Item -Path ".\apps" -Name "Chrome" -ItemType Directory -Force
$downloadcache = New-Item -Path ".\" -Name "Downloaded" -ItemType Directory -Force
$downloadsource = 'https://dl.google.com/tag/s/appguid%253D%257B8A69D345-D564-463C-AFF1-A69D9E530F96%257D%2526iid%253D%257BBEF3DB5A-5C0B-4098-B932-87EC614379B7%257D%2526lang%253Den%2526browser%253D4%2526usagestats%253D1%2526appname%253DGoogle%252520Chrome%2526needsadmin%253Dtrue%2526ap%253Dx64-stable-statsdef_1%2526brand%253DGCEB/dl/chrome/install/GoogleChromeEnterpriseBundle64.zip?_ga%3D2.8891187.708273100.1528207374-1188218225.1527264447'
$filename = "ChromeEnterprise.zip"
$downloaddestination = $downloadcache
Start-BitsTransfer -Source $downloadsource -Destination $downloadcache\$filename | Out-Null

# Extract Chrome enterprise archive with the -Force parameter to overwrite any existing files
Expand-Archive -LiteralPath $downloadcache\ChromeEnterprise.zip -DestinationPath $downloadcache -Force
Move-Item -Path $downloadcache\Installers\GoogleChromeStandaloneEnterprise64.msi -Destination $appfolder -Force

# Logo download
$logoURL = "https://logos-world.net/wp-content/uploads/2020/08/Google-Chrome-Logo-700x394.png"
$LogoFileName = "chrome.png"
Invoke-WebRequest -Uri $logoURL -OutFile $appfolder\$LogoFileName

# Create the intunewin file with Force parameter to overwrite if it exists
$Source = $appfolder
$SetupFile = "GoogleChromeStandaloneEnterprise64.msi"
$Destination = $appfolder
$CreateAppPackage = New-IntuneWin32AppPackage -SourceFolder $Source -SetupFile $SetupFile -OutputFolder $Destination -Verbose -Force
$IntuneWinFile = $CreateAppPackage.Path

# Check if $IntuneWinFile is not empty before proceeding
if (-not [string]::IsNullOrEmpty($IntuneWinFile)) {
    # Get IntuneWin file Meta data and assign intunewin file location variable
    $IntuneWinMetaData = Get-IntuneWin32AppMetaData -FilePath $IntuneWinFile
} else {
    Write-Host "Failed to create the .intunewin file. Please check the packaging process."
    return
}

# Define application name, description, and publisher info
$Displayname = "Google Chrome"
$Description = "Google Chrome x64"
$Publisher = "Google"

# Create file exists detection rule
$DetectionRule = New-IntuneWin32AppDetectionRuleMSI -ProductCode $IntuneWinMetaData.ApplicationInfo.MsiInfo.MsiProductCode

# Create requirement rule with supported operating system version corrected
$RequirementRule = New-IntuneWin32AppRequirementRule -Architecture x64 -MinimumSupportedOperatingSystem "W10_1909"

# Create an icon from the image file
$ImageFile = "$appfolder\$LogoFileName"
$Icon = New-IntuneWin32AppIcon -FilePath $ImageFile

# Build and upload the application to Intune
Add-IntuneWin32App -FilePath $IntuneWinFile -DisplayName $DisplayName -Description $Description -Publisher $Publisher -InstallExperience "user" -RestartBehavior "suppress" -DetectionRule $DetectionRule -RequirementRule $RequirementRule -Icon $Icon -Verbose

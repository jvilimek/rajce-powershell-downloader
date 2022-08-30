# Rajce PowerShell Downloader

Album downloader for Rajce.net photo album service. Simple use. Works with password protected albums too. Tested in **Powershell Core 7.1** in Windows 10 and Linux Ubuntu 20.4.


```powershell
#version 1.0
$url = "https://jam53.rajce.idnes.cz/Kvetouci_klivie_2021"
$userName = ""
$userPassword = ""
$downloadLocation = "album" #will be downloaded in local folder. Full path is supported, e.g. d:\pictures\album

Write-Host "$(Get-Date) Downloading $url …";
$ProgressPreference = 'SilentlyContinue'

New-Item -Path $downloadLocation -Type Directory -Force | Out-Null;
[Microsoft.PowerShell.Commands.WebRequestSession]$session = new-object Microsoft.PowerShell.Commands.WebRequestSession
$userAgent = "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.159 Safari/537.36";
$request1 = Invoke-WebRequest -Uri $url -UseBasicParsing -UserAgent $userAgent -WebSession $session
$body = $null;
If ("" -ne $userName -and "" -ne $userPassword) { $body = "login=$userName&code=$userPassword";}
$requestLogin = Invoke-WebRequest -Uri $url -UseBasicParsing -UserAgent $userAgent -WebSession $session -Method POST -Body $body

function getJsonVariable([string]$variableName){
	$fullVariableDeclaration = "var $variableName = ";
	$lineRaw = $requestLogin.Content  -split '\r?\n' | Select-String $fullVariableDeclaration;
	$lineRaw = $lineRaw -replace $fullVariableDeclaration,''
	$lineRaw = $lineRaw.Replace('filename', 'filename2')
	$lineRaw = $lineRaw -replace ';',''
	return $lineRaw | ConvertFrom-Json;
}
$photosLocation = getJsonVariable -variableName 'storage';
$photos = getJsonVariable -variableName 'photos';
$cnt = $($photos.Count);
Write-Host "$(Get-Date) Found $cnt photos";
$i = 1;
Foreach($photo in $photos) {
	$photoFileName = $photo.fileName;
	Write-Host "$(Get-Date) Downloading $photoFileName ($i/$cnt) …"  -ForegroundColor DarkGray;
	$url = "${photosLocation}images/${photoFileName}?ver=0";
	$outFile = Join-Path -Path $downloadLocation -ChildPath $photo.fileName;
	Invoke-WebRequest -Uri $url -UseBasicParsing -UserAgent $userAgent -WebSession $session -OutFile $outFile | Out-Null;
	$i++;
}
Write-Host "Finished." -ForegroundColor Green;
```

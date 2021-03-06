# escape=`

FROM mcr.microsoft.com/windows/servercore/iis
FROM mcr.microsoft.com/dotnet/framework/aspnet:4.8-windowsservercore-ltsc2019

ARG DOTNETCOREHOSTING_EXE=c:\\install\\DotNetCore.1.1.0-WindowsHosting.exe
ARG PUBLISHING_SITE_NAME=sitecore.publishing
ARG SITE_WEB_ROOT="C:\\inetpub\\wwwroot"
ARG ASSET_ROOT="C:\\assets"
ARG PUBLISHING_HOST="Sitecore.Framework.Publishing.Host.exe"
ARG PUBLISHING_PACKAGE
ARG CONN_STRING_CORE
ARG CONN_STRING_MASTER
ARG CONN_STRING_WEB

ADD https://aka.ms/dotnetcore_windowshosting_1_1_0 ${DOTNETCOREHOSTING_EXE}

SHELL ["powershell"]

# Install Prerequisites
RUN Start-Process $env:DOTNETCOREHOSTING_EXE -ArgumentList '/install', '/passive', '/norestart' -NoNewWindow -Wait; `	
	Import-Module ServerManager; Add-WindowsFeature Web-Scripting-Tools;

RUN powershell -NoProfile -Command Remove-Item -Recurse C:\inetpub\wwwroot\*

SHELL ["powershell", "-NoProfile", "-Command", "$ErrorActionPreference = 'Stop';"]

COPY ./ ${ASSET_ROOT}

WORKDIR $SITE_WEB_ROOT

# Copy Publishing Service files
RUN $packagePath = Join-Path $env:ASSET_ROOT $env:PUBLISHING_PACKAGE; `
	Expand-Archive -Path $packagePath -DestinationPath $env:SITE_WEB_ROOT; 

# Set Conn strings
RUN	$coreArgs = 'configuration setconnectionstring core """' + $env:CONN_STRING_CORE + '"""'; `
	$masterArgs = 'configuration setconnectionstring master """' + $env:CONN_STRING_MASTER + '"""'; `
	$webArgs = 'configuration setconnectionstring web """' + $env:CONN_STRING_WEB + '"""'; `
	Start-Process $env:PUBLISHING_HOST -ArgumentList $coreArgs -NoNewWindow -Wait; ` 
    Start-Process $env:PUBLISHING_HOST -ArgumentList $masterArgs -NoNewWindow -Wait; ` 
	Start-Process $env:PUBLISHING_HOST -ArgumentList $webArgs -NoNewWindow -Wait; 

# Upgrade schema
RUN Start-Process $env:PUBLISHING_HOST -ArgumentList 'schema upgrade --force' -NoNewWindow -Wait; 

# For references - to use when there are issues in upgrading the schemas
# RUN Start-Process Sitecore.Framework.Publishing.Host.exe -ArgumentList 'configuration setconnectionstring core "\"Data Source=sqldev;Initial Catalog=sitecorerc-ase-app-prod-rg-core-db_Primary;User Id=sa;Password=HASH-epsom-sunset-cost7!;\""' -NoNewWindow -Wait;
# RUN Start-Process Sitecore.Framework.Publishing.Host.exe -ArgumentList 'configuration setconnectionstring master "\"Data Source=sqldev;Initial Catalog=sitecorerc-ase-app-prod-rg-master-db_Primary;User Id=sa;Password=HASH-epsom-sunset-cost7!;\""' -NoNewWindow -Wait;
# RUN Start-Process Sitecore.Framework.Publishing.Host.exe -ArgumentList 'configuration setconnectionstring web "\"Data Source=sqldev;Initial Catalog=royalcanin_Web_Primary;User Id=sa;Password=HASH-epsom-sunset-cost7!;\""' -NoNewWindow -Wait;

RUN Start-Process Sitecore.Framework.Publishing.Host.exe -ArgumentList 'configuration set sitecore:publishing:instancename -v rc-sitecore-docker-pubsvc' -NoNewWindow -Wait;

# install the iis with hosts for the publishing service
RUN $args = 'iis install --hosts --force --sitename ' + $env:SITE_NAME + ' --apppool ' + $env:SITE_NAME; `
	Start-Process $env:PUBLISHING_HOST -ArgumentList $args -NoNewWindow -Wait;  
	
EXPOSE 80

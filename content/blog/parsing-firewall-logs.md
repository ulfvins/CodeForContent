+++
author = "Richard Ulfvin"
date = "2018-05-31"
description = "Mega Geek Week presentation"
linktitle = ""
title = "Parsing Windows Firewall Logs"
type = "post"

+++

Last night me and my coworker Jonas was invited to do and evening presenting at the Mega Geek Week venue in Stockholm. The topic was on how to handle the Windows Firewall and some helpful tips and tricks to help you figure out how the network communication was setup between windows systems. 

The basic steps we presented was: 

1. How to enable firewall logging.
2. How to parse log files to get all communicating IPs and ports used.
3. How to build basic but more granular rules based on IP and port restrictions.

## The Powershell Function for analysis
This was the code we used in the presentation.

{{< highlight powershell >}}
Function Get-FirewallStatistics{

    [CmdletBinding()]
    Param (
        [Parameter(Position=0)]
        [String[]]
        $LogFiles = @("$env:windir\system32\LogFiles\Firewall\pfirewall.log")
    )
     

    Begin
    {
        $normalizedCommunication = [System.Collections.ArrayList]::New()

        If(Test-Path "$env:TEMP\service-names-port-numbers.csv")
        {
            
            Write-Verbose "Common port list is located."
        }
        Else
        {
            Write-Verbose "Common port list is missing, downloading..."
            Try
            {
                $url = "https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.csv"
                $output = "$env:TEMP\service-names-port-numbers.csv"
                Invoke-WebRequest -Uri $url -OutFile $output
            }
            Catch
            {
                Write-Verbose "Error in downloading Description from IANA, proceeding without detailed port inforamtion"
            }
        }

        $portInformation = Import-Csv -Path "$env:TEMP\service-names-port-numbers.csv" |
        Select-Object 'Service Name', 'Port Number', 'Transport Protocol', 'Description' -ErrorAction SilentlyContinue
        
        # Default dynamic ports: 49152 - 65535
        $DynamicPorts = netsh int ipv4 show dynamicport tcp
        $DynamicPorts = ($DynamicPorts -match "Start").Split(":")[1].Trimstart()

        $logHeaders = @(
            'date', 
            'time', 
            'action', 
            'protocol', 
            'src-ip', 
            'dst-ip', 
            'src-port', 
            'dst-port', 
            'size', 
            'tcpflags', 
            'tcpsyn', 
            'tcpack', 
            'tcpwin', 
            'icmptype', 
            'icmpcode', 
            'info', 
            'path'
        )

        Write-Verbose "Prereqs are loaded!"
    }

    Process
    {
       
        Function AddOrUpdateProtocolStatistics($connection) {

            if ($connection.protocol -eq "ICMP") {
                $port = "$($connection.icmptype):$($connection.icmpcode)"
            }
            elseif ($connection.'dst-port' -as [int] -ge $DynamicPorts -as [int]) {
                $port = "Dynamic"
            }
            else {
                $port = $connection.'dst-port'
            }

            $protocolStatistics = $normalizedCommunication | 
            Where-Object {$_.Protocol -eq $connection.protocol -and $_.Port -eq $port}

            if ($protocolStatistics.Count -eq 0) {

                $protocolStatistics = [PSCustomObject]@{
                    Protocol        = $connection.protocol
                    Port            = $port
                    ConnectionCount = 1
                    Description     = ($portInformation | 
                            Where-Object {$_.'Port Number' -eq $connection.'dst-port' -and 
                            $_.'Transport Protocol' -eq $connection.protocol} | 
                            Select-Object -First 1).Description
                    SourceIps       = [System.Collections.ArrayList]::new()
                }

                $null = $protocolStatistics.SourceIps.Add($connection.'src-ip')
                $null = $normalizedCommunication.Add($protocolStatistics)

            }
            else {
                $protocolStatistics.ConnectionCount++
                if (!($protocolStatistics.SourceIps.Contains($connection.'src-ip'))) {
                    $null = $protocolStatistics.SourceIps.Add($connection.'src-ip')
                }
            }
        }

        foreach ($logFile in $LogFiles) {

            Write-Verbose "About to parse file: $logFile"

            if($logFile -eq "$env:windir\system32\LogFiles\Firewall\pfirewall.log")
            {
                Write-Verbose "Local logfile: $logFile , locked for read, copying to temp location..."
                Copy-Item -Path $logFile -Destination "$env:Temp\pfirewall.log" -Force
                $logFile = "$env:Temp\pfirewall.log"
            }
            
            $file = Get-ChildItem -Path $logFile -ErrorAction SilentlyContinue

            if($file)
            {
                foreach ($line in [System.IO.File]::ReadLines($file.FullName)) {

                    if ($line -match "^#" -or $line -eq $null -or $line -eq "")
                    { 
                        continue
                    }
        
                    $connection = $line | ConvertFrom-Csv -Delimiter ' ' -Header $logHeaders
                    if ($connection.path -eq 'RECEIVE' -and $connection.action -eq 'ALLOW') {
                        AddOrUpdateProtocolStatistics($connection)
                    }
                }
            }
        }

        Return $normalizedCommunication | Sort-Object ConnectionCount -Descending
    }

    End
    {
    }
}
{{< /highlight >}}
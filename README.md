# Introduction
We're working a lot at our home office these days. Several people already found inventive solutions to make working in the home office more comfortable. One of these ways is to automate activities in your home automatation system based on your status on Microsoft Teams.

Microsoft provides the status of your account that is used in Teams via the Graph API. To access the Graph API, your organization needs to grant consent for the organization so everybody can read their Teams status. Since my organization didn't want to grant consent, I needed to find a workaround, which I found in monitoring the Teams client logfile for certain changes.

For Home Assistant, this script makes use of two sensors that are created in Home Assistant up front:
* sensor.teams_status
* sensor.teams_activity

sensor.teams_status displays that availability status of your Teams client based on the icon overlay in the taskbar on Windows. sensor.teams_activity shows if you are in a call or not based on the App updates deamon, which is paused as soon as you join a call.

For Jeedom, you need to create a Virtual with 2 sensors and type "other" :
* status
* activity
Once saved, the command's ID are visible.

# Important
This solution is created to work with Home Assistant or Jeedom. It will work with any home automation platform that provides an API, but you probably need to change the PowerShell code.

Please check your language for the settings! Powershell hasn't have a good implementation for UTF8, we need to replace value in variables. Currently English and French are good. Maybe you need to add characters at this line in the script (and share it!) :
```Powershell
$output = $Text.Replace('Ã©','é').Replace('Ã´','ô')
```

# Requirements
## Home Assistant
* Create the three Teams sensors in the Home Assistant configuration.yaml file
```yaml
input_text:
  teams_status:
    name: Microsoft Teams status
    icon: mdi:microsoft-teams
  teams_activity:
    name: Microsoft Teams activity
    icon: mdi:phone-off

sensor:
  - platform: template
    sensors:
      teams_status: 
        friendly_name: "Microsoft Teams status"
        value_template: "{{states('input_text.teams_status')}}"
        icon_template: "{{state_attr('input_text.teams_status','icon')}}"
        unique_id: sensor.teams_status
      teams_activity:
        friendly_name: "Microsoft Teams activity"
        value_template: "{{states('input_text.teams_activity')}}"
        unique_id: sensor.teams_activity

```
* Generate a Long-lived access token ([see HA documentation](https://developers.home-assistant.io/docs/auth_api/#long-lived-access-token))
* Copy and temporarily save the token somewhere you can find it later
* Restart Home Assistant to have the new sensors added

## Jeedom
* Add the `Virtuel` plugin if you don't have it
* Create a virtuel `teams mode` with 2 infos command, type `other`
* Save and copy the ID's that were genereted

# Configuration
* Copy the Settings.ps1 file to Settings.local.ps1 and:
  * Replace `<Insert token>` with the token you generated
  * Replace `<UserName>` with the username that is logged in to Teams and you want to monitor
  * Replace `<HAURL>` or `<Jeedom URL>` with the URL to your Home Assistant / Jeedom server
  * Adjust the language settings to your preferences
# Installation
Start a elevated PowerShell prompt, browse to the TeamsStatus folder and run the following command:
```powershell
New-Item -Path "c:\Scripts\" -ItemType Directory
Copy-Item -Path .\nssm.exe -Destination "c:\Scripts\"
Copy-Item -Path .\Settings.local.ps1 -Destination "c:\Scripts\"
Copy-Item -Path .\Get-TeamsStatus.ps1 -Destination "c:\Scripts\"
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned
Unblock-File c:\Scripts\Settings.local.ps1
Unblock-File c:\Scripts\Get-TeamsStatus.ps1
Start-Process -FilePath "c:\Scripts\nssm.exe" -ArgumentList 'install "Microsoft Teams Status Monitor" "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" "-command "& { . C:\Scripts\Get-TeamsStatus.ps1 }"" ' -NoNewWindow -Wait
Start-Service -Name "Microsoft Teams Status Monitor"
```

After completing the steps below, start your Teams client and verify if the status and activity is updated as expected.
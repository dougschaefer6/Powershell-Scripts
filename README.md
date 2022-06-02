# Teams-Rooms-PS
Repository for Powershell scripts for Microsoft Teams accounts

Prerequisites:

// Azure AD PowerShell for Graph module

Install-Module -Name AzureAD

// Exchange Online PowerShell Module

Install-Module -Name ExchangeOnlineManagement

// Microsoft Teams PowerShell Module

Install-Module -Name MicrosoftTeams



Identify room -> Resource exists -> if no, create resource in exchange -> configure calendar auto accept -> set password to never expire -> assign teams license -> configure teams rooms

//Create Room Mailbox

New-Mailbox -Name $acctUpn -Alias $MailBoxAlias -Room -DisplayName $DisplayName -EnableRoomMailboxAccount $true -RoomMailboxPassword (ConvertTo-SecureString -String $Password -AsPlainText -Force)

// Set Outlook to auto-accept meeting invites and add additional response

Set-CalendarProcessing -Identity $MailboxAlias -AutomateProcessing AutoAccept -AddOrganizerToSubject $false -DeleteComments $false -DeleteSubject $false -RemovePrivateProperty $false -ProcessExternalMeetingMessages $true -AdditionalResponse "This is a Microsoft Teams Meeting Room!"


// Set password to never expire and set usage location

Set-AzureADUser -ObjectID $acctUpn -PasswordPolicies DisablePasswordExpiration

Set -AzureADUser -ObjectID ConferenceRoom01@contoso.com UsageLocation 'US'

// Create an object for a single license type

$License = New-Object -TypeName Microsoft.Open.AzureAD.Model.AssignedLicense

$License.SkuId = $ADLicense

// Create an object for a multiple license type

$Licenses = New-Object -TypeName Microsoft.Open.AzureAD.Model.AssignedLicenses

// Add the single license object to the multiple license object

$Licenses.AddLicenses = $License

// Assign the license to the resource account

Set-AzureADUserLicense -ObjectId ConferenceRoom01@contoso.com -AssignedLicenses $Licenses

// OPTIONAL SETTINGS
// Additional information to be included in responses to meeting requests

$AdditionalResponse=@" <hr style="font-family: Calibri, Arial, Helvetica, sans-serif; font-size: 12pt; color: rgb(0, 0, 0);"><p align="center"><img width="291" height="137"src="https://images.contoso.com/msc17_collaboration_002.png"></p><p align="center">This room is equipped with One Touch Join - Did you make this a Microsoft Teams or Skype for Business meeting?</p><p align="center">Become a meeting ninja - <a href="https://aka.ms/TeamsRoomVideo">watch this short video</a>!</p><p align="center"><br></p><hr><img style="margin-right: auto; margin-left: auto; float: none; display: block;"src="https://images.contoso.com/2020/03/contoso-icon.png">"@

Set-CalendarProcessing -Identity $MailboxAlias -AddAdditionalResponse $true -AdditionalResponse $AdditionalResponse

// prevent mailbox from filling up with voicemail

New-CsTeamsCallingPolicy -Identity NoVoiceMail -Description "Do not permit user to have voicemail" -AllowVoicemail AlwaysDisabled

Grant-CsTeamsCallingPolicy -Identity MTR-TVW-Conference1 -PolicyName NoVoiceMail

// MailTips enablement, tells users a message in the InfoBar in Outlook

Set-Mailbox -Identity $MailboxAlias -MailTip "This room is equipped to support Microsoft Teams meetings"

// Assigning Phone Number

$acctUpn= 'MTR-SEA-FocusRoom1@teamsroomslab.com'

$PhoneNumber='+14255553223'

#Get Location ID via Get-CsOnlineLisLocation

$LocationID='c04255ef-0aac-4cc8-b5fe-510c4578f792'

Set-CsPhoneNumberAssignment -Identity $AcctUPN -PhoneNumber $PhoneNumber -LocationID $LocationID -PhoneNumberType CallingPlan

// Places

Set-Place 'MTR-TVW-Conference' -IsWheelChairAccessible $true -Capacity 12 -DisplayDeviceName $True // see more examples online for advanced controls of Set-Place





// BULK USER PROVISIONING

$path="c:\temp\teamsrooms.csv" */identify path location
$csv=Import-Csv -path $path */define csv variable and action
ForEach ($Room in $csv) */identify file location, cycle through csv by $Room variable
{

	#Set variables
	$accUpn=$Room.AccountUpn
	$MailboxName=$Room.MailboxName
	$MailboxAlias=$Room.MailboxAlias
	$DisplayName=$Room.DisplayName
	$Password=$Room.Password
	$AdditionalResponse=$Room.AdditionalResponse
	$PhoneNumber=$Room.PhoneNumber
	$UsageLocation=$Room.UsageLocation
	#Get Location ID via Get-CsOnlineLisLocation
	$LocationID=Room.LocationID
}

// push XML file

$movefile = "<path>";
$targetDevice = "\\<Device fqdn> \Users\Skype\AppData\Local\Packages\Microsoft.SkypeRoomSystem_8wekyb3d8bbwe\LocalState\SkypeSettings.xml"; 
Copy-Item $movefile $targetDevice

// enable remote PS https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/enable-psremoting?view=powershell-7.2

This functionality is off by default. You need to enable remote PowerShell for your environment on the Microsoft Teams Rooms system to perform the operations below. Refer to the documentation on Enable-PSRemoting for information about how to enable remote PowerShell.

For example, you can enable Remote PowerShell as follows:

Sign in as Admin on a Microsoft Teams Rooms device.
Open an elevated PowerShell command prompt.
Enter the following command: Enable-PSRemoting -SkipNetworkProfileCheck -Force
Open the Local Security Policy and add the Administrators security group to Security Settings > Local Policies > User Rights Assignment > Access this computer from the network.


This is a working example of the XML manifest used by Microsoft for the 'out of date active-x blocking feature' 
(tested on 8.1 ie11, but should be no different for 8-11, IE7)
I've only tested it with flash 16. I haven't tested it 

The original file is here http://go.microsoft.com/fwlink/?LinkId=517023 

The modified file goes into %LOCALAPPDATA%\microsoft\Internet Explorer\VersionManager

A good reference is https://technet.microsoft.com/en-us/library/dn761713.aspx 


For the file not get updated and overwritten you will need to add:
	>Although we strongly recommend against it, if you don’t want your computer to automatically download the updated version list from Microsoft, run 	the following command from a command prompt:
	
	reg add "HKCU\Software\Microsoft\Internet Explorer\VersionManager" /v DownloadVersionList /t REG_DWORD /d 0 /f


caveats:
	-IE needs to be restarted to take effect. GPO/etc distro methods can be delayed as well [well, unless you WMI exec GPUPdate force or 	whatever you people do]
	-intranet and trusted sites are not blocked by default (good)
	-it's a whitelist, you need to ammend it or dl ms's one (ignore it says ie11, it's 8-11). Read more about the supported versions here https://technet.microsoft.com/en-us/library/dn761713.aspx 
	- by overriding the list [and if you don't it will get overwritten] and disabling MS list updates you're possibly shoting yourself in the foot if other controls on the list get own and people with managed PCs are offsite depending on the distribution method and enforcement styles
	- no one guarantees the format of the XML won't change. 
	- you can probably play with the timestamps and time windows at the top instead of disabling auto-update [effectively setting the XML to expire after a few days and to resume MS updates for it] 
	- you should have your trusted sites list ready and preferably controlled via GPO (IEAK is not good but could work too)
	- if you don't have trusted sites managed, you can use >Turn off blocking of outdated ActiveX controls for Internet Explorer on specific domains
	- you need to distribute it - note the debug settings go into the USER Registry hive, and the XML file goes into the %localdata%\
		- probably means user gpo or computer + looback user GPO [? I haven't tried GPO deploying it myself yet]
	- or startup script or SCCM package.
	- manage user settings 
		e.g. update buttons[because everyone's users' should go and d/l install things, 
		you're free to point the url in the unsafe version to whereever too instead of adobe], 
		allow to run anyway buttons [in case you are paranoid that users will click run if a nice picture above tells em to like with that windword macro vector [sigh]],  http://go.microsoft.com/fwlink/?LinkId=444484 (GPO templates) 
- you should test reverting the XML update setting and distributing it along with testing this just in case. 

- classid (<blocklistentry key=') may change in the future [e.g. flash 17]






The guide here is really quite good: 
https://technet.microsoft.com/en-us/library/dn761713.aspx


Here's how you can get the ocx and active-x dll file properties for file and product versions for the XML if you need to yourself

For testing and enumerating current controls - you can do it via 
	HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Policies\Ext

	AuditModeEnabled 
	dword32=1

	(or better - get the IE ADMx/l templates and test via the GPO or LGPO )
	http://www.microsoft.com/en-us/download/details.aspx?id=40905  - NB that template for some reason hasn't been updated for the Advanced_EnableSSL3Fallback bits 


The loaded Active-X csv in %LOCALAPPDATA%\Microsoft\Internet Explorer\AuditMode\VersionAuditLog.CSV
[you could probably leave it on for future reference]

IE's add-on properties window gives you a bit more info:

	Name:                   Shockwave Flash Object
	Publisher:              Microsoft Windows Third Party Application Component
	Type:                   ActiveX Control
	Architecture:           32-bit and 64-bit
	Version:                16.0.0.257
	File date:              ‎Friday, ‎16 ‎January ‎2015, ‏‎12:51 AM
	Date last accessed:     ‎Today, ‎24 ‎January ‎2015, ‏‎2 minutes ago
	Class ID:               {D27CDB6E-AE6D-11CF-96B8-444553540000}
	Use count:              21
	Block count:            0
	File:                   Flash.ocx
	Folder:                 C:\Windows\System32\Macromed\Flash

Watch  %LOCALAPPDATA%\Microsoft\Internet Explorer\AuditMode\VersionAuditLog.CSV

When testing if you get it correctly it will go from

	https://www.adobe.com/software/flash/about/,C:\Windows\SysWOW64\Macromed\Flash\Flash.ocx,16.0.0.257,16.0.0.257,Allowed,Not in blocklist,EPM 	Compatible

to the following if your version bits are incorrect:
	https://www.adobe.com/software/flash/about/,C:\Windows\SysWOW64\Macromed\Flash\Flash.ocx,16.0.0.257,16.0.0.257,Allowed,Version not in blocklist,EPM 	Compatible

to 
	https://www.adobe.com/software/flash/about/,C:\Windows\SysWOW64\Macromed\Flash\Flash.ocx,16.0.0.257,16.0.0.257,Blocked,Out of date,EPM Compatible

	
with the final one, you will be rewarded with the GUI message about it being blocked. 
	

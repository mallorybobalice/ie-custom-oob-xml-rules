<h2>What is this?</h2>
<br/>This is a [rudementarily tested] working example of modifying the XML manifest used by Microsoft for the 'out of date active-x blocking feature' to add flash 16.0.0.257 and 16.0.0.287 to a blocklist. [or rather not whitelist them]
<br/>
<br/>Tested on 8.1 ie11, but should be no different for 8-11, IE7
<br/>I've only tested it with flash 16. I haven't tested it with 15, 14, etc. 
<br/>
<br/>The original file is here http://go.microsoft.com/fwlink/?LinkId=517023 
<br/>^permaupdated link
<br/>
<br/>The modified file goes into %LOCALAPPDATA%\microsoft\Internet Explorer\VersionManager
<br/>(userprofile - because IE auto-updates it by default)
<br/>
<br/>A good reference is https://technet.microsoft.com/en-us/library/dn761713.aspx 
<br/>
<br/>
<br/><h2>For the file not get updated and overwritten you will need to add:</h2> (or maybe to play with timestamps in the XML headers but I haven't tried that...blocklist version="5" ttlHigh="50" ttlLow="1251635200"...these ones,)
<br/>	>Although we strongly recommend against it, if you don’t want your computer to automatically download the updated 		version list from Microsoft, run the following command from a command prompt:
<br/>		reg add "HKCU\Software\Microsoft\Internet Explorer\VersionManager" /v DownloadVersionList /t REG_DWORD /d 0 /f
<br/>
<br/>
<br/><h2>Caveats:</h2>
<br/>	- IE needs to be restarted to take effect. GPO/etc distro methods can be delayed as well [well, unless you WMI exec 		GPUPdate force or whatever you do]
<br/>	- intranet and trusted sites are not blocked by default (good depending on your level of paranoia. mostly good )
<br/>	- it's a whitelist, you need to ammend it or dl ms's one (ignore it says ie11, it's 8-11). 
<br/>		Read more about the supported versions here https://technet.microsoft.com/en-us/library/dn761713.aspx 
<br/>	- by overriding the list [and if you don't it will get overwritten] and disabling MS list updates you will miss out on 	the new blocklists and with managed PCs offsite depending on the distribution method and enforcement styles that 			might suddenly start to matter
<br/>	- no one guarantees the format of the XML won't change. 
<br/>	- you can probably play with the timestamps and time windows at the top instead of disabling auto-update [effectively 	setting the XML to expire after a few days and to resume MS updates for it] 
<br/>	- you should have your trusted sites list ready and preferably controlled via GPO (IEAK is not good but could work 		too)
<br/>		- if you don't have trusted sites managed, you can use >Turn off blocking of outdated ActiveX controls for 			Internet Explorer on specific domains
<br/>	- you need to distribute it - note the debug settings go into the USER Registry hive, and the XML file goes into the %localdata%\
<br/>		- probably means user gpo or computer + looback user GPO [? I haven't tried GPO deploying it myself yet]
<br/>	- or startup script or SCCM package.
<br/>	- manage user settings for this feature
<br/>		e.g. update buttons[because everyone's users' should go and d/l install things, 
<br/>		you're free to point the url in the unsafe version to whereever too instead of adobe], 
<br/>		allow to run anyway buttons [in case you are paranoid that users will click run if a nice picture above tells 		them to like with that windword macro vector [sigh]],  http://go.microsoft.com/fwlink/?LinkId=444484 (GPO templates)
<br/>	- you should test reverting the XML update setting and distributing it along with testing this just in case. 
<br/>	- classid (<blocklistentry key=') may change in the future [e.g. flash 17]
<br/>
<br/>
<br/>
<br/>
<br/>The guide here is really quite good: 
<br/>https://technet.microsoft.com/en-us/library/dn761713.aspx
<br/>
<br/>
<br/><h2>How do I add my own components here?</h2>
<br/>Here's how you can get the ocx and active-x dll file properties for file and product versions for the XML if you need to yourself
<br/>
<br/>1) enable extra loggign for active-x components:
<br/>For testing and enumerating current controls - you can do it via 
<br/>	HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Policies\Ext
<br/>
<br/>	AuditModeEnabled 
<br/>	dword32=1
<br/>
<br/>or better - get the IE ADMx/l templates and test via the GPO or LGPO )
<br/>	http://www.microsoft.com/en-us/download/details.aspx?id=40905  - NB that template for some reason hasn't been updated 	for the Advanced_EnableSSL3Fallback bits 
<br/>
<br/>
<br/>The loaded Active-X csv in %LOCALAPPDATA%\Microsoft\Internet Explorer\AuditMode\VersionAuditLog.CSV
<br/>[you could probably leave it on for future reference]
<br/>
<br/>2) Find out what you want to add:
<br/>IE's add-on properties window gives you a bit more info:
<br/>
<br/>	Name:                   Shockwave Flash Object
<br/>	Publisher:              Microsoft Windows Third Party Application Component
<br/>	Type:                   ActiveX Control
<br/>	Architecture:           32-bit and 64-bit
<br/>	Version:                16.0.0.257
<br/>	File date:              ?Friday, ?16 ?January ?2015, ??12:51 AM
<br/>	Date last accessed:     ?Today, ?24 ?January ?2015, ??2 minutes ago
<br/>	Class ID:               {D27CDB6E-AE6D-11CF-96B8-444553540000}
<br/>	Use count:              21
<br/>	Block count:            0
<br/>	File:                   Flash.ocx
<br/>	Folder:                 C:\Windows\System32\Macromed\Flash
<br/>
<br/>Watch  %LOCALAPPDATA%\Microsoft\Internet Explorer\AuditMode\VersionAuditLog.CSV
<br/>
<br/>c) Add a new group as with the flash one at the top 
<br/>e.g     <blocklistentry key="{d27cdb6e-ae6d-11cf-96b8-444553540000}" entrytype="2" /> (that's by class ID)
<br/>
<br/>d) Add group entries for updated and out of date versions:
<br/>
<br/>  <groupentries>
<br/>  	<!-- flash below -->
<br/>	<groupentry groupname="Shockwave Flash AX 16" fwdlink="http://" latestgroup="1" /><!-- latest flash 16 group -->
<br/>	<groupentry groupname="Shockwave Flash 16" fwdlink="http://" latestgroup="1" /><!-- latest flash 16 ax pluging group (pretty sure that DLL is never used by IE but anyway -->
<br/>    <groupentry groupname="Shockwave Flash" fwdlink="https://get.adobe.com/flashplayer/" /><!-- blanket out of date flash rule - these  get blocekd -->
<br/>	<!-- flash above -->
<br/>	
<br/>e) finally, map the versions to the group entries [latest and blocked]
<br/>
<br/><blocklistfullentries>
<br/>    <!-- flash below -->
<br/>	<!-- check the ocx and active x dll file properties for file and product versions -->
<br/>	<!-- check loaded add-ons registry (for the filenames) for the classids -->
<br/>	<!-- check https://technet.microsoft.com/en-us/library/dn761713.aspx for the GPO and registry flags and settings disable overriding the click to play for out of date prompts, as well at disabling XML auto-update from the MS site -->
<br/>	<!-- as for how the blocklist works, it's actually an alowlist, groupname="Shockwave Flash 16" and AX 16 are marked as latest above and specify the yet unreleased safe versions, while groupname="Shockwave Flash" blank catches all the unsafe ones -->
<br/>	<blocklistentry key="{d27cdb6e-ae6d-11cf-96b8-444553540000}" entrytype="2">
<br/>      <versionentries numberofelements="3">
<br/>
<br/>		 <!-- not sure i think the active-x dll is the non browser one - e.g. if you want to make a screensaver using it in the exe you compile - so up to you to import and test it, it's possibly also used by firefox and older opera -->
<br/>		 <!-- also note - the version numbers are inclusive e.g. -->
<br/>		 <versionentry groupname="Shockwave Flash 16" filename="Flash.ocx" productversion="16.0.0.288-65535.65535.65535.65535" fileversion="16.0.0.288-65535.65535.65535.65535" />
<br/>		  <versionentry groupname="Shockwave Flash AX 16" filename="FlashUtil_ActiveX.dll" productversion="16.0.0.288-65535.65535.65535.65535" fileversion="16.0.0.288-65535.65535.65535.65535" />
<br/>		 <versionentry groupname="Shockwave Flash" filename="*" productversion="*" fileversion="*" />
<br/>		
<br/>      </versionentries>
<br/>    </blocklistentry>
<br/>
<br/><b>What to do if it's not working?</b>
<br/>
<br/>Watch  %LOCALAPPDATA%\Microsoft\Internet Explorer\AuditMode\VersionAuditLog.CSV
<br/>(you need to enable outputting loaded active-x to it first as above)
<br/>
<br/>When testing if you get it correctly it will go from
<br/>
<br/>	https://www.adobe.com/software/flash/about/,C:\Windows\SysWOW64\Macromed\Flash\Flash.ocx,16.0.0.257,16.0.0.257,Allowed,Not in blocklist,EPM 	Compatible
<br/>
<br/>to the following if your version bits are incorrect:
<br/>	https://www.adobe.com/software/flash/about/,C:\Windows\SysWOW64\Macromed\Flash\Flash.ocx,16.0.0.257,16.0.0.257,Allowed,Version not in blocklist,EPM 	Compatible
<br/>
<br/>to 
<br/>	https://www.adobe.com/software/flash/about/,C:\Windows\SysWOW64\Macromed\Flash\Flash.ocx,16.0.0.257,16.0.0.257,Blocked,Out of date,EPM Compatible
<br/>
<br/>	
<br/>with the final one, you will be rewarded with the GUI message about it being blocked. 
<br/>	
<br/>

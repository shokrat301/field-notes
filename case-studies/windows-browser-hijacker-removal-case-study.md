# Case Study: Removing a Persistent Chrome Browser Hijacker from Windows

Background
This incident started as a recurring browser hijack. When searching from Chrome, the browser redirected traffic through suspicious domains such as:

nextgeeker.com
directsearchapp.com

At first, it looked like a simple Chrome search-engine hijack. But after cleanup attempts, the infection kept returning. That showed this was not only a browser setting problem. It had Windows-level persistence.
The hijacker modified Chrome search settings, forced suspicious behavior through browser policy, created fake Chrome-related folders, and added Microsoft Defender exclusions so its folders would be ignored during scans.

Main Symptoms
The visible symptoms were:
Chrome search redirected to nextgeeker.com
Chrome search engine URL was modified to directsearchapp.com
Chrome showed suspicious enterprise/policy behavior
Chrome launched with a fake profile path instead of the normal one
Suspicious scheduled task existed: NetOneUpdater
Microsoft Defender had exclusions for suspicious paths

The fake Chrome profile path was:
C:\Users\hamut\AppData\Local\DGoogle\DChrome\DUser Data

A normal Chrome profile should not use DGoogle or DChrome.
Confirmed Malicious/Suspicious Items
We found and removed these paths:

C:\Users\hamut\AppData\Local\DGoogle
C:\Users\hamut\AppData\Local\DDapps
C:\Users\hamut\AppData\Local\EMicrosoft
C:\ProgramData\Chromnius
C:\ProgramData\ChromniusEdge
C:\Program Files (x86)\GeneralAISupport2Solutions

The scheduled task NetOneUpdater pointed to:
C:\Program Files (x86)\GeneralAISupport2Solutions\GeneralAISupport2\NetSixUpdater.exe

That was treated as the reinfection mechanism.
Microsoft Defender exclusions were also compromised. Suspicious exclusions included fake browser paths such as Chromnius, ChromniusEdge, DDapps, EMicrosoft, and browser install directories. This strongly suggested malware persistence, not a normal Chrome issue.

What We Did
1. Removed malicious scheduled task
We disabled and deleted:
Disable-ScheduledTask -TaskName "NetOneUpdater"
Unregister-ScheduledTask -TaskName "NetOneUpdater" -Confirm:$false
2. Killed suspicious updater processes
   
Stop-Process -Name "NetSixUpdater" -Force -ErrorAction SilentlyContinue
Stop-Process -Name "GeneralAISupport2" -Force -ErrorAction SilentlyContinue

3. Removed malicious folders
We deleted:
GeneralAISupport2Solutions
DGoogle
DDapps
EMicrosoft
Chromnius
ChromniusEdge
Some folders required ownership and permission repair before deletion.

4. Removed Microsoft Defender exclusions
The malware had added Defender exclusions. We removed them with:

$exclusions = Get-MpPreference | Select-Object -ExpandProperty ExclusionPath
foreach ($x in $exclusions) {
  Remove-MpPreference -ExclusionPath $x -ErrorAction SilentlyContinue
}

After cleanup, Defender exclusion output was empty.

5. Removed Chrome policy hijack
Chrome policy had previously forced extension behavior. We removed the policy keys and confirmed:

chrome://policy
No policies set

6. Fully removed Chrome
Normal uninstall was not enough. Chrome files remained locked and produced many Access is denied errors during deletion, even after service cleanup.
We booted into Safe Mode and manually deleted:

C:\Program Files\Google
C:\Program Files (x86)\Google

We used:
takeown /f Google /a /r /d y
icacls Google /reset /t /c /q
icacls Google /grant *S-1-5-32-544:F *S-1-5-18:F /t /c /q
del /f /s /q Google\*.*
rmdir /s /q Google

After that, both folders returned:
File Not Found

7. Reinstalled Chrome cleanly
After rebooting normally, we verified:
Test-Path "C:\Program Files\Google"
Test-Path "C:\Program Files (x86)\Google"
Test-Path "$env:LOCALAPPDATA\DGoogle"
Test-Path "$env:LOCALAPPDATA\Google\Chrome"
Get-MpPreference | Select-Object -ExpandProperty ExclusionPath

Expected clean result:
False
False
False
False
and no Defender exclusions.
Then Chrome was reinstalled fresh.

8. Verified clean Chrome launch
After reinstall, Chrome launched from:
C:\Program Files\Google\Chrome\Application\chrome.exe
There was no:
DGoogle
DChrome
--user-data-dir
That confirmed the fake profile hijack was gone.
Final Status
The system was cleaned to the point where:

Chrome installed fresh
Chrome launched from the normal path
Fake DGoogle profile did not return
Defender exclusions were empty
Chrome policy showed no forced policies
Search engine no longer showed directsearchapp or nextgeeker
Suspicious folders were removed
NetOneUpdater was removed
Lessons Learned

This was not just a Chrome extension problem. The hijacker used multiple persistence layers:

Browser search-engine modification
Fake Chrome profile path
Scheduled task persistence
Defender exclusions
Locked Chrome installation leftovers
Fake browser-like folder names to look legitimate

The key lesson: when a browser hijacker keeps returning, do not only reset the browser. Check:

Scheduled Tasks
Defender Exclusions
Chrome Policies
Chrome launch command line
Fake profile paths
Startup entries
ProgramData and AppData folders

The final fix required removing both the browser-level hijack and the Windows-level persistence.

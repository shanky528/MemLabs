# Memory Forensics Challenge Walkthrough  
**Challenge: Never Too Late Mister**  
My friend John is an "environmental" activist and a humanitarian. He hated the ideology of Thanos from the Avengers: Infinity War. He sucks at programming. He used too many variables while writing any program. One day, John gave me a memory dump and asked me to find out what he was doing while he took the dump. Can you figure it out for me?

**Challenge file:** https://drive.google.com/file/d/1MjMGRiPzweCOdikO3DTaVfbdBK5kyynT/view

**Challenge Source:**  
This challenge is solved from stuxnet999 Github page.

---

## Tools Used  
- Volatility Framework 2.6  
- Python 2 (Volatility requires Python 2.x)  
- Linux environment (Kali Linux recommended)

---

## Step 1: Identify the profile (Operating System)  
The first step in any memory forensics task is identifying the right profile, which tells us the OS version the memory dump was taken from. Volatility has a plugin for this:

Run the following command:

```bash
python2 vol.py -f /home/shanky/Documents/Case-1/Challenge.raw imageinfo
```
This gives multiple suggestions. While kdbgscan can help further refine the choice, it wasn’t necessary for this challenge.

---

## Step 2: Identify Key Artifacts
As a forensic analyst, here’s what I typically want to uncover:
- Active processes
- Commands executed via terminal
- Hidden or terminated processes
- (If needed) Browser history

```bash
python2 vol.py -f /home/shanky/Documents/Case-1/Challenge.raw --profile=Win7SP1x86 pslist
```
Some processes stood out:
- cmd.exe – command prompt
- DumpIt.exe – likely used to grab this memory dump
- explorer.exe – Windows Explorer handler

## Step 3: Investigating Shell Activity
Since cmd.exe was active, I wanted to check what commands were executed. For that, I used cmdscan:

```bash
python2 vol.py -f /home/shanky/Documents/Case-1/Challenge.raw --profile=Win7SP1x86 cmdscan
```
It revealed the execution of a Python script:
```bash
C:\Python27\python.exe C:\Users\hello\Desktop\demon.py.txt
```
To check the script’s output, I used the consoles plugin:
```bash
python2 vol.py -f /home/shanky/Documents/Case-1/Challenge.raw --profile=Win7SP1x86 consoles
```
This showed a suspicious hex string:
```bash
335d366f5d6031767631707f
```

## Step 4: Step 4: Finding Clues in Environment Variables
The challenge hinted at the word "environment". I checked system environment variables using:

```bash
python2 vol.py -f /home/shanky/Documents/Case-1/Challenge.raw --profile=Win7SP1x86 envars
```
One variable caught my attention:
```bash
Name: Thanos
Value: xor, password
```
So I had:

- A hex string
- A hint that it’s XOR encoded
- A clue involving a password

## Step 5: XOR Decoding
Using Python, I decoded the hex string with XOR:

```bash
import binascii

a = binascii.unhexlify("335d366f5d6031767631707f")

for i in range(0, 256):
    b = ""
    for j in a:
        b += chr(j ^ i)
    print(f"[{i}] {b}")
```
Among the outputs, one result looked like part of a flag:
```bash
1_4m_b3tt3r}
```

## Step 6: Step 6: Extracting Password Hashes
The final part mentioned a password. So I used hashdump to pull NTLM hashes:

```bash
python2 vol.py -f /home/shanky/Documents/Case-1/Challenge.raw --profile=Win7SP1x86 hashdump
```
I got this NTLM hash:
```bash
101da33f44e92c27835e64322d72e8b7
```

Using an online NTLM cracker, I retrieved the actual password. Putting everything together:
```bash
flag{find_out_for_yourself}
```
#### That’s all for now! I hope you’re not upset that I didn’t include the flags here. As a fellow cybersecurity student, I’ve found that when an article reveals all the flags upfront, it can take away the motivation to try the challenge independently. My goal is for you to learn from these labs, replicate the steps on your own system, and enjoy the rewarding experience of discovering the flag yourself.

#### Happy hacking!

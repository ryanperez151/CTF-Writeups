HTB Cyber Apocalypse - 3/9-13/2024 - Writeups by Ryan Perez
===============================================================

Intro -


Challenge start! (after party example)
![image](https://github.com/ryanperez151/CTF-Writeups/assets/50554328/f06187b1-c766-4e50-9d26-d7bf49416b0f)


For the forensics challenges I used several VMs with operating system packages such as a Windows 10, SIFT workstation, Kali, and remnux. Some of the main tools I found myself using were wireshark, cyberchef, python, volatility, and MFTEcmd among others.


Challenges
===============================================================

It Has Begun (very easy) - “The Fray is upon us, and the very first challenge has been released! Are you ready factions!? Considering this is just the beginning, if you cannot muster the teamwork needed this early, then your doom is likely inevitable.”

Summary: Analyze and decode a UNIX shell script using some basic reversing and decoding with tools like a text editor and cyberchef.

Downloaded and unzipped the challenge file to find a ‘script.sh’ file which I decided to open in notepad++ for analysis:
![image](https://github.com/ryanperez151/CTF-Writeups/assets/50554328/16ebc20f-3639-4376-9a60-3c965df73c4b)


I noticed the UNIX shell script contained what appeared to be material to store SSH authentication material, DNS resolution for a suspicious looking host, perform some host and process discovery, kill off processes, then finally get a payload and execute with bash encoded command into /etc/crontab (execution and persistence).
This being a CTF challenge I knew there may be some parts of the script I could pick out that may be a flag (or parts of a flag). The highlighted parts stood out to me based on the content and encoding:
![image](https://github.com/ryanperez151/CTF-Writeups/assets/50554328/ae4b685d-cfe3-4041-b6db-89e9fa7d7b68)


The username stored into /root/.ssh/authorized_keys stood out because I noticed the curly bracket which is typically indicative of a flag following the HTB format like: HTB{1337-5P34K}
I grabbed this and threw it into CyberChef to reverse the string: 
![image](https://github.com/ryanperez151/CTF-Writeups/assets/50554328/576efe9d-a577-481a-8d14-70facaf4a6f1)


Now that we had the first part of the flag string I thought to work on the second. Leveraging some experience, I noticed the final bash command appeared base64 encoded so I decoded that as well for the second part of the flag:
![image](https://github.com/ryanperez151/CTF-Writeups/assets/50554328/c1694ea1-e58c-4508-a0f4-30623100292d)


Now putting the flags together (HTB{w1ll_y0u_St4nd_y0uR_Gr0uNd!!}) we’re able to submit them in the CTF portal and earn some points!
![image](https://github.com/ryanperez151/CTF-Writeups/assets/50554328/a7d13e04-b5c6-423b-afc3-b78659cf60fa)


Flag: HTB{w1ll_y0u_St4nd_y0uR_Gr0uNd!!}


Urgent (very easy) 
===============================================================

“In the midst of Cybercity's "Fray," a phishing attack targets its factions, sparking chaos. As they decode the email, cyber sleuths race to trace its source, under a tight deadline. Their mission: unmask the attacker and restore order to the city. In the neon-lit streets, the battle for cyber justice unfolds, determining the factions' destiny.” 

Summary: Analyze and decode a phishing email with an .html attachment containing encoded JavaScript payload to uncover the flag.

We’re provided an .eml file within the challenge .zip that appears to be a phishing lure containing a .html attachment file:
![image](https://github.com/ryanperez151/CTF-Writeups/assets/50554328/5a63a718-6b19-4edf-98f9-8c8615229cd9)


Saving the attachment we’re able to open it in a text editor and review the contents which appear to have some apparent URL encoded JavaScript:
![image](https://github.com/ryanperez151/CTF-Writeups/assets/50554328/430efe9c-e93c-4bc8-a32b-90037cae816c)


Grabbing this encoded content we can begin decoding in a tool like CyberChef and find an output with a full flag value:
![image](https://github.com/ryanperez151/CTF-Writeups/assets/50554328/4c055ff0-8802-4087-b997-cfd9f89722e1)


The phishing payload itself was a fake 404 not found page that contained a powershell download cradle for a ‘form1.exe’ file next to where we find our flag value.


Flag: HTB{4n0th3r_d4y_4n0th3r_ph1shi1ng_4tt3mpT}


Fake Boost (easy)
===============================================================

“In the shadow of The Fray, a new test called ""Fake Boost"" whispers promises of free Discord Nitro perks. It's a trap, set in a world where nothing comes without a cost. As factions clash and alliances shift, the truth behind Fake Boost could be the key to survival or downfall. Will your faction see through the deception? KORP™ challenges you to discern reality from illusion in this cunning trial.”

Summary: This challenge resembles analyzing malware and the corresponding exfiltration. Analyze a packet capture file containing a powershell script advertised as free Discord Nitro but turns out to be a credential stealer with exfiltration capabilities. Reversing the script will allow uncovering the secrets contained in the exfiltrated data observed in a POST request.

Analysis of the pcap file revealed some interesting HTTP traffic. The first bit of interesting traffic was a response containing 'discordnitro.ps1' file:
![image](https://github.com/ryanperez151/CTF-Writeups/assets/50554328/4feb8a5f-c239-49e9-80ea-7f47f09cead2)


This .ps1 file required a bit of decoding (reverse, then base64) to make sense of it and resulted in part1 of the encoded flag:
![image](https://github.com/ryanperez151/CTF-Writeups/assets/50554328/ac23bf56-6591-4ffd-ae48-588ae3f00db1)
![image](https://github.com/ryanperez151/CTF-Writeups/assets/50554328/54a31b04-e09f-40b7-abae-1737effb3d58)


The script itself appears to actually steal Discord tokens then send this off using AES encryption via a POST request: 
![image](https://github.com/ryanperez151/CTF-Writeups/assets/50554328/57010df1-cc72-4dc8-98ff-a1fa4ac38284)


We can see the AES key used in that script and it appears we can also find the POST request in that same pcap provided by the challenge:
![image](https://github.com/ryanperez151/CTF-Writeups/assets/50554328/4cafb39b-e15f-4406-b56d-63700fc9b909)


Now we try to decode and decrypt the exfil payload. It appeared the IV (initialization vector) ought to be contained in the first part of that POST with base64 encoding, but I struggled to work out this decryption piece until I was able to talk through it with Robert:
![image](https://github.com/ryanperez151/CTF-Writeups/assets/50554328/bbcf325f-e015-4d0d-88b7-8a0307c224a0)
![image](https://github.com/ryanperez151/CTF-Writeups/assets/50554328/c31d6542-6121-4646-9c4f-aac79612fe5c)


After some trial and error, closer review/scrutiny of the .ps1 script, and teamwork I was able to correct the decryption method of the data in the POST request by first decoding the payload from base64, then correcting the initialization vector by properly using the first 16 bytes of the payload in UTF-8 encoding and omitting the IV from the payload to gather the exfiltrated Discord tokens in the POST request: 
![image](https://github.com/ryanperez151/CTF-Writeups/assets/50554328/00115ee5-08a7-405b-a72e-ae901594a1b7)


Noticing that the email appeared as base64 encoded data instead of an actual email, I base64 decoded this as well as the 1st part from the .ps1 script and found the full flag:
![image](https://github.com/ryanperez151/CTF-Writeups/assets/50554328/da2f34b2-419f-4f70-bd48-66e3d602dd08)


Flag: HTB{fr33_N17r0G3n_3xp053d!_b3W4r3_0f_T00_g00d_2_b3_7ru3_0ff3r5}


Pursue The Tracks (easy)
===============================================================

“Luxx, leader of The Phreaks, immerses himself in the depths of his computer, tirelessly pursuing the secrets of a file he obtained accessing an opposing faction member workstation. With unwavering determination, he scours through data, putting together fragments of information trying to take some advantage on other factions. To get the flag, you need to answer the questions from the docker instance.”

Summary: This resembles a file system artifact forensic challenge. We were provided a docker instance to answer some questions about the MFT file provided. Use tools like MFTEcmd and volatility to perform analysis on the MFT record and answer the corresponding questions in the docker instance to earn the flag: 
![image](https://github.com/ryanperez151/CTF-Writeups/assets/50554328/77c8ccb4-4804-4365-b335-42d0dd9a27bd)


Unzipped the challenge file and found ‘z.mft’. Initially used the Eric Zimmerman tool, MFTEcmd, to parse the MFT record from NTFS filesystem into a more human-readable output in .csv format:
![image](https://github.com/ryanperez151/CTF-Writeups/assets/50554328/6e3f6fa5-4e6e-4ef1-9ef4-1f3778d55753)


Reviewed this output using TimelineExplorer paying close attention to file attributes including Windows MACB timestamps (modified, accessed, changed, and born):
![image](https://github.com/ryanperez151/CTF-Writeups/assets/50554328/8d8cd2ce-f011-4e92-9a30-8705ba350228)


As MFTEcmd didn’t appear to parse the hidden file attributes, I used volatility with a generic Windows profile and ‘mftparser’ module to also parse the MFT record in more detail:
![image](https://github.com/ryanperez151/CTF-Writeups/assets/50554328/55014c49-08b4-4111-bcd1-ea6dfea196aa)


Reviewed the output in a text editor and found the hidden file:
![image](https://github.com/ryanperez151/CTF-Writeups/assets/50554328/52f13860-dd8c-4336-a44e-33b88d80cbe2)


Flag: HTB{p4rs1ng_mft_1s_v3ry_1mp0rt4nt_s0m3t1m3s}


Phreaky (medium)
===============================================================


“In the shadowed realm where the Phreaks hold sway,
A mole lurks within, leading them astray.
Sending keys to the Talents, so sly and so slick,
A network packet capture must reveal the trick.
Through data and bytes, the sleuth seeks the sign,
Decrypting messages, crossing the line.
The traitor unveiled, with nowhere to hide,
Betrayal confirmed, they'd no longer abide.”

Summary: This challenge seemed to resemble another exfiltration scenario. We’re provided with a packet capture file which observes 15 emails. Within these email files we find password protected .zip files and a corresponding password in the body of the email. After unzipping the attachments we’re left with 15 pieces of an apparent .pdf file. When reassembled the full PDF reveals the flag value in plaintext. I took this challenge as an opportunity to practice python scripting and learned a lot.


Upon initial analysis of the .pcap file contained in the challenge .zip I noticed some SMTP traffic stick out: 
![image](https://github.com/ryanperez151/CTF-Writeups/assets/50554328/d8ba8024-33b9-4612-b3e8-d16825589a89)


Looking further into this and I noticed 15 exportable IMF (.eml) files available:
![image](https://github.com/ryanperez151/CTF-Writeups/assets/50554328/361ff307-5fab-4f2c-8e33-2ca43692a533)


From here I inspected some of the emails by following their TCP streams and noticed some interesting content in the body and attachment of each email indicating a password string and zip file respectively:
![image](https://github.com/ryanperez151/CTF-Writeups/assets/50554328/c1901d4c-8e4d-4b91-b342-f6b904f0fda6)


Each email followed this format so I decided to automate extraction of the email password and attachments using a python script after some trial and error:
![image](https://github.com/ryanperez151/CTF-Writeups/assets/50554328/65714f31-e0ed-4bc9-b711-dafa202ae1f6)
![image](https://github.com/ryanperez151/CTF-Writeups/assets/50554328/a0b05ba7-339f-4478-9120-93f1e9e39993)


I spent some time trying to script decoding then unzipping each attached file with the respective password but ended up performing that step manually after significant trial and error:
![image](https://github.com/ryanperez151/CTF-Writeups/assets/50554328/aa90f379-0f9b-431d-970c-edf35306631c)


The result was 15 separate parts of what appeared to be a PDF file:
![image](https://github.com/ryanperez151/CTF-Writeups/assets/50554328/a20b6fb5-2f36-4193-bb09-a684be443c77)


I tried to challenge myself again to automate combining these file parts into a whole and came up with another python script (this was…. challenging for me LOL):
![image](https://github.com/ryanperez151/CTF-Writeups/assets/50554328/72991e77-3380-4f95-8000-608cee3c2095)


I finally got it!
![image](https://github.com/ryanperez151/CTF-Writeups/assets/50554328/9f59c492-a966-4a8a-b467-e7a0cc7c076e)


After opening that PDF file I read through all their ‘plans’ and found the flag value in plaintext at the bottom:
![image](https://github.com/ryanperez151/CTF-Writeups/assets/50554328/d91f39a2-ac31-479f-8675-c5536151f7a0)


Flag: HTB{Th3Phr3aksReadyT0Att4ck} 


Note: following the conclusion of the event I found a writeup from another competitor that detailed how the full automation of extracting the pieces and combining could be done. I was impressed to learn how it could be accomplished with some proper code.
![image](https://github.com/ryanperez151/CTF-Writeups/assets/50554328/e1cbaae3-7218-49f9-9ae7-0bfceb4fc8c2)
(source: github/pspspsps-ctf/writeups…)


Closing Thoughts
===============================================================

I ended up trying another medium and hard difficulty challenge during the rest of the remaining time of the CTF, but was unable to gather the full flag values for either.
Overall, the challenges and camaraderie were fantastic and I look forward to more opportunities to compete with the Dumpster Fire Banshees in the future!

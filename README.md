# AIS-challenges - C4n y0u H4ck 1t ?
AIS Hacking Challenge Site workarounds

I currently have **1055** points.

## { Client Side Protections }

### [Super Admin] (10 points)

 - Click submit - error pops out: `Javascript validation failed. You are not super admin. is_super_admin = false`
 - Open console, type: `is_super_admin = true`
 - Click submit again

_Voila!_


### [Timer] (50 points)
 - Inspect Submit button with dev tools;
 - Right click on "<button type="button" class="btn btn-success">SUBMIT</button>" - select `Break on > Subtree Modification`;
 - Go to sources and Pause script execution;
 - `HackerChallenge.js` will open and `  display.text(seconds);` will be highlighted;
 - In console, type `seconds = 1` (DO NOT set to 0 or you will have to wait for... -1 seconds when you resume!);
 - Resume script execution, go to page and click **Submit**.

_Voila!_


### [Paid Content] (100 points)
 - Open dev tools > Network tab;
 - Click submit;
 - You should see a `400 Status` for a `POST method`; 
 - Go to console, expand the `XHR POST https://hack.ainfosec.com/challenge/submit-answer/ [HTTP/2.0 400 Bad Request` and select `Stack Trace` so you can find the file where the function is;
 - Look for `HackerChallenge.submitAnswer` and click on the coresponding file next to it `hackerchallenge.js`;
 - Add a breakpoint at `return $.ajax({ ` (line 53) - this will stop execution before the AJAX call;
 - Go on page and click **Submit** again - since the execution is stopped, you can alter the answer;
 - In console type: `answer = answer.replace('"paid":false', '"paid":true')`;
 - Now, go back to Sources tab, and _Resume script execution_;

_Voila!_


## { Networking }

### [HTTP Basic] (15 points)

 - Download `http-auth.cap` file (~60MB);
 - A `.cap` file is a file that contains packets captured using a sniffing tool;
 - Install Wireshark if you don't have it;
 - Open and load `.cap` file into it;
 - in 'Apply a display filter' type: `http.request.method == POST` so you can see all the forms submitted;
 - Check for: 
 ```
 HTML Form URL Encoded: application/x-www-form-urlencoded
 Form item: "name" = "xxxxxxx"
 Form item: "pass" = "XXXXXXXXXXX"
 ```
_Voila!_

### [WPA2 Deauth] (30 points)

 - Download `de-auth.cap` file (~18kb);
 - It is a file that contains the EAPOL handshake (google it if you don't know what it is);
 - We are going to use HashCat, so first we have to convert `.cap` file to `.hccapx`:
 - Convert it to `.hccapx` here: https://hashcat.net/cap2hccapx/ ;
 - Install hashcat: `brew install hashcat` (on macOS);
 - Download dictionary file - a file that contains **LOTS** of passwords: `http://downloads.skullsecurity.org/passwords/rockyou.txt.bz2` ;
 - Unzip it: `bunzip2 rockyou.txt.bz2` ;
 - Run: `hashcat -m 2500 capture.hccapx rockyou.txt` 
 (_-m 2500_ is the parameter for setting the hash type to WPA-EAPOL-PBKDF2);
 - You will get something like that: `SOME_LONG_SEQUENCE_OF_CHARACTERS:ROUTER_MAC:CLIENT_MAC:NETWORK_SSID:PASSWORD`.

_Voila!_

## { Input Validation } 

### [SQL Login] (50 Points)

 - We are facing a basic login form, so we test by entering `'`;
 - An error message will pop: `'SELECT username, password FROM users WHERE username='admin' AND password='''' - unrecognized token: "'''"`;
 - Easy peasy! Just enter this: `' or '1'='1`.
 
_Voila!_

### [Cross Site Scripting] (75 Points)

 - By checking the stored cookies, we see for `admin_sess_id` the following value: `flag_should_be_here%20%F0%9F%A4%94`
 - `%20%F0%9F%A4%94` is this emoticon: 🤔
 - Let's test the input for URL with a basic XSS payload: 
 `<img src="http://url.to.file.which/not.exist" onerror=alert(document.cookie);>` and click **Submit**
 - TA-DA! Here's our flag...I mean...

_Voila!_

### [SQL Credit Cards] (100 Points)
 - As usual, test form by entering `'`;
 - An error message will pop:`'SELECT username FROM credit_cards WHERE username=''' COLLATE NOCASE' - unrecognized token: "''' COLLATE NOCASE"`;
 - So we have to forge the query. Try to enter this: `  ' and FALSE union SELECT card FROM credit_cards WHERE username='xxxxxx  `
 - Card number will be displayed in page: `XXXXXXXXXXXXXXXXX`
 
_Voila!_


## { Crypto } 

### [Encoded] (25 points)

```python
import codecs
import bz2
import textwrap
```

- We receive a string that is base64 encoded, so the first step is to decode it; `codecs.decode(STRING, 'base64')`
- Looking at the result we see the first characters are different from the rest: `BZh91AY&SY`; 
- This is a sign that we are dealing with a bz2 encoded string;
- So we decompress it, and... we get a long binary sequence; `bz2.decompress(RESULT)`
- We split it `DECOMPRESSED.split()`, and transform each sequence into an int (base 2) and get the corresponding character;
```python
ANOTHER_RESULT = []
for item in SPLITTED:
    ANOTHER_RESULT.append(chr(int(item,2)))
```
- We join all the characters `''.join(ANOTHER_RESULT)` and... we are facing a hexadecimal sequence;
- So we split the sequence into pairs of 2 `textwrap.wrap(HEX_SEQUENCE, 2)`, we transform each pair into an int (base 16), and into a character;
- And we join the string: `flag{XXXXXXXXXXXX}`
```python
print (''.join( chr(int(a, 16) ) for a in SPLITTED_HEX_SEQUENCE))
```
_Voila!_

### [Base64] (75 points)
 - This time, we receive a larger string sequence.
 - I inspected the content in Burp, under Decode tab - playing with different encodings - , and I saw a mention at the end of the file: `Did you know docx files are basically just zip files?` and some filenames (`flag.txt`, `xor_key.txt` and other xml files); 
 - So we might be dealing with a file. Let's create one. Or better make it two!
 - If using terminal, type: `touch raw out` - this will create two empty files: `raw` and `out`
 - Put the content inside the raw file, and using python, import base64 package and let's decode the content:
 ```python
>>> import base64
>>> input = open('raw', 'rb')
>>> output = open('out', 'wb')
>>> base64.decode(input, output)
>>> input.close()
>>> output.close()
```
- Now, it's time to play with `out` file; 
- Out of curiosity, I opened `out` file as a docx file: `NOTHING TO SEE HERE MOVE ALONG` it said.\
- Indeed, nothing to see, so I changed the file extension to zip and extracted it.
- Inside the folder created, there are a bunch of files but only two are interesting: `flag.txt` and `xor_key.txt`, both containing some long strings;
- Filenames are self explanatory, so our next step is to decrypt flag sequence using XOR key. But how?
- XOR Decryption is identical to encryption because the XOR operation is symmetrical;
- So all we need to do is to XOR encrypt the flag using the XOR key provided;
- We can do that using python:
```python
import textwrap

key = 'OUR_LONG_SEQUENCE_OF_CHARACTERS_FROM_XOR_KEY_DOT_TXT'
flag = 'OUR_EVEN_LONGER_SEQUENCE_OF_CHARACTERS_FROM_OUR_FILE_NAMED_FLAG_DOT_TXT'
# we split the flag sequence into 2pairs, because hex
flag_list = textwrap.wrap(flag,2)
# and the key sequence into characters
key_list = textwrap.wrap(key,1)
# using XOR magic and base 16 conversion of hex into int combined with ASCII magic from int into char, and
# by the power of join
print (''.join( chr(int(a, 16) ^ ord(b) ) for a, b in zip(flag_list, key_list) ))
# we get the flag:flag{XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX}
```
- PS - Beware, it's an ugly one!

_Voila!_


### [Enigma] (100 points) [----WORKING ON----]
 - Currently working on it!

### [XOR] (300 points)
 - Completed, but ain't gonna post it!
 - You have to score 720 points in order to submit your e-mail address! _:)_
 
 ## { Exploitation } 
 
 ### [Stack Overflow] (75 points)
  - Completed, the code is self explanatory. Hints: buffer overflow, arc injection, and most important: ASCII !
 

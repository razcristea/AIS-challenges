# AIS-challenges - C4n y0u H4ck 1t ?
AIS Hacking Challenge Site workarounds

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

 - We are facing a basic login form, so we test by entering '
 - An error message will pop: `'SELECT username, password FROM users WHERE username='admin' AND password='''' - unrecognized token: "'''"`
 - Easy peasy! Just enter this: `' or '1'='1`
 
_Voila!_

### [Cross Site Scripting] (75 Points)

 - By checking the stored cookies, we see for `admin_sess_id` the following value: `flag_should_be_here%20%F0%9F%A4%94`
 - `%20%F0%9F%A4%94` is this emoticon: 🤔
 - Let's test the input for URL with a basic XSS payload: 
 `<img src="http://url.to.file.which/not.exist" onerror=alert(document.cookie);>` and click **Submit**
 - TA-DA! Here's our flag...I mean...

_Voila!_

\## 🧩 Task 1 – Brief



\*\*Summary\*\*:  

This room introduces various methods attackers use to \*\*bypass website authentication\*\*. These vulnerabilities are critical because successful exploitation can lead to \*\*data leaks\*\* and unauthorized access.



\*\*Key Points\*\*:

\- Focus: How attackers defeat or bypass login mechanisms

\- Risk: High — often leads to exposure of sensitive user data

\- Goal: Understand real-world attack vectors and how to mitigate them



\*\*Action Taken\*\*:

\- ✅ Started the machine



-------------------------------------------------------------------------------------------



\## 🔍 Task 2 – Username Enumeration



\### 🧠 Concept:

Username enumeration occurs when a web app leaks whether a username exists via different responses. Here, the signup form shows the error "username already exists" for valid users, allowing enumeration.



\### 🔧 Tool Used:

\- \*\*ffuf\*\* (Fuzz Faster U Fool)



\### 💻 Command Used:

```bash

ffuf -w /usr/share/wordlists/SecLists/Usernames/Names/names.txt -X POST -d "username=FUZZ\&email=x\&password=x\&cpassword=x" -H "Content-Type: application/x-www-form-urlencoded" -u http://10.10.139.92/customers/signup -mr "username already exists"



-------------------------------------------------------------------------------------

🧩 Task 3 – Brute Force Login

Use the valid\_usernames.txt file created from the username enumeration task.



Perform a brute force attack on the login page at http://10.10.48.164/customers/login.



Use the ffuf tool with two wordlists: one for usernames (valid\_usernames.txt) and one for passwords (10-million-password-list-top-100.txt).



Command:



bash

Copy

Edit

ffuf -w valid\_usernames.txt:W1,/usr/share/wordlists/SecLists/Passwords/Common-Credentials/10-million-password-list-top-100.txt:W2 -X POST -d "username=W1\&password=W2" -H "Content-Type: application/x-www-form-urlencoded" -u http://10.10.48.164/customers/login -fc 200

Explanation:



-w specifies multiple wordlists separated by a comma.



W1 and W2 are placeholders for usernames and passwords respectively.



-fc 200 filters out HTTP 200 responses, so only successful logins (non-200) show up.



Run the command in the same directory as the valid\_usernames.txt file.



The output will reveal the valid username and password combination in the format username/password.

-----------------------------------------------------------------------------------------



🧩 Task 4 – Password Reset Logic Flaw

The password reset form uses both GET (query string) and POST parameters.



The email address is passed via the URL query string (GET), e.g., ?email=robert@acmeitsupport.thm.



The username is sent in the POST form data.



The server uses PHP’s $\_REQUEST to process inputs, which prioritizes POST data over GET when keys are duplicated.



This allows an attacker to manipulate the email in the POST data, overriding the GET email parameter.



Attack Method:



Create your own account on the Acme IT Support site, giving you an email like {your\_username}@customer.acmeitsupport.thm.



Use curl to send a password reset request for Robert’s email, but override the email POST parameter with your own email.



The password reset email gets sent to your email address (the one you control).



This lets you receive a login link for Robert’s account.



Example curl command:



bash

Copy

Edit

curl 'http://10.10.48.164/customers/reset?email=robert@acmeitsupport.thm' \\

-H 'Content-Type: application/x-www-form-urlencoded' \\

-d 'username=robert\&email={your\_username}@customer.acmeitsupport.thm'

Using this method, you gain access to Robert’s account and can view his support tickets.



Flag found in Robert’s ticket:

THM{AUTH\_BYPASS\_COMPLETE}



-------------------------------------------------------------------------------------------



🧩 Task 5 – Cookie Tampering

Concept:

Cookies control user session state and permissions.



Some cookies are in plain text (e.g., logged\_in=true; admin=false).



Changing cookie values can manipulate user privileges (e.g., escalate to admin).



Some cookies are hashed or encoded (e.g., Base64), which can be reversed or modified if you know the encoding.



Practical Steps:

Test accessing the page without cookies — get "Not Logged In".



Set logged\_in=true; admin=false — access as a regular logged-in user.



Set logged\_in=true; admin=true — access as admin and retrieve the flag.



Important Terms:

Plain Text Cookies: Easily editable values, vulnerable to tampering.



Hashed Cookies: Irreversible but consistent. Usually safe.



Encoded Cookies: Base64, base32 etc. reversible encoding, can be decoded, edited, and re-encoded.



Commands Example:

bash

Copy

Edit

curl -H "Cookie: logged\_in=true; admin=false" http://10.10.48.164/cookie-test

curl -H "Cookie: logged\_in=true; admin=true" http://10.10.48.164/cookie-test

Hashing Example:

MD5 hash 3b2a1053e3270077456a79192070aa78 corresponds to a specific string you can lookup.



Base64 Encoding/Decoding:

Base64 decode VEhNe0JBU0U2NF9FTkNPRElOR30= gives the string: THM{BASE64\_ENCODING}



To get admin access, base64 encode {"id":1,"admin":true} to set the session cookie.








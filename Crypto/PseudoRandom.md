# PseudoRandom

## Category
Crypto

## Description

We were given the following code and output. Find our mistake and tell us the flag.

```python
import random
import time
import datetime  
import base64

from Crypto.Cipher import AES
flag = b"find_me"
iv = b"\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"

for i in range(0, 16-(len(flag) % 16)):
    flag += b"\0"

ts = time.time()

print("Flag Encrypted on %s" % datetime.datetime.fromtimestamp(ts).strftime('%Y-%m-%d %H:%M'))
seed = round(ts*1000)

random.seed(seed)

key = []
for i in range(0,16):
    key.append(random.randint(0,255))

key = bytearray(key)


cipher = AES.new(key, AES.MODE_CBC, iv) 
ciphertext = cipher.encrypt(flag)

print(base64.b64encode(ciphertext).decode('utf-8'))
```

Output:
```
Flag Encrypted on 2023-08-02 10:27
lQbbaZbwTCzzy73Q+0sRVViU27WrwvGoOzPv66lpqOWQLSXF9M8n24PE5y4K2T6Y
```

## Solution

- The encryption uses a random seed which is based on the current unix timestamp at the time of the encryption, including milliseconds.
- Reverse the encryption function and try all timestamps which
    - are within that minute
    - includes all possible timezone offsets

Example code:

```python
import random
import base64
from Crypto.Cipher import AES

def decrypt(ts):
    encrypted_text = "lQbbaZbwTCzzy73Q+0sRVViU27WrwvGoOzPv66lpqOWQLSXF9M8n24PE5y4K2T6Y"
    ciphertext = base64.b64decode(encrypted_text.encode('utf-8'))

    iv = b"\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"

    seed = round(ts*1000)

    random.seed(seed)

    key = []
    for i in range(0, 16):
        key.append(random.randint(0, 255))

    key = bytearray(key)

    cipher = AES.new(key, AES.MODE_CBC, iv)
    plaintext = cipher.decrypt(ciphertext)

    # Remove padding
    flag = plaintext.rstrip(b"\0")

    return flag.decode('utf-8', 'ignore')

original_ts = 1690972020 # 2023-08-02 10:27 GMT

for i in range(-12, 13):
    ts = original_ts + (3600 * i)
    limit = ts + 60
    while ts < limit:
        candidate = decrypt(ts)
        if candidate.startswith("flag{"):
            print(candidate)
            print("Timestamp: " + str(ts) + " - " + str(i) + " hours offset")
        ts+=0.001
```

Output:
```
flag{r3411y_R4nd0m_15_R3ally_iMp0r7ant}
Timestamp: 1690986434.4389534 - 4 hours offset
```
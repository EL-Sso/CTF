# Crack me

일단 코드를 보면
```
#!/usr/bin/python3
import hashlib

flag = 'hspace{}'
salt = int(flag.encode().hex(), 16)
number = int(input('Number: '))
print('Your Number:', number)
print('Hash:', hashlib.sha256(str(number + salt).encode()).hexdigest())
```
flag는 "hspace{}"라는 문자열이고 salt는 flag를 바이트로 인코딩한 후, 그 결과를 16진수로 변환하여 정수로 변환한 값이다.
사용자가 정수를 입력하면 salt를 더한 값을 문자열로 변환한 후, 이를 SHA-256 해시로 변환하고 최종적으로 이 해시 값을 16진수 문자열로 출력한다.
salt 값은 
```
flag = 'hspace{}'
salt = int(flag.encode().hex(), 16)
print(salt)
```
![스크린샷 2024-09-28 130113](https://github.com/user-attachments/assets/9fc4bb9a-34a1-4296-bc2f-6fba0db71164)

이걸 10진수->16진수로 바꿔봤다.
6873706163657b7d 맞고 이걸 아스키로 바꾸면 

![image](https://github.com/user-attachments/assets/05240fd0-24d1-4e80-bdbe-27c40431b14d)

hspace{}인데 여기서 무슨 숫자를 입력해야 한다는건지를 잘 모르겠다.
여기서 뭐 준게 아무것도 없는데 도커파일이 있길래 한번 docker-compose로 올려봤다.

sudo docker-compose up --build
로 빌드하고

![image](https://github.com/user-attachments/assets/388c880f-c4df-4304-9033-aac8dff0de25)

sudo docker exec -it crackme /bin/bash
로 컨테이너 내부로 들어가봤는데

![image](https://github.com/user-attachments/assets/695f85ef-9a23-4ee9-ba90-29bdc4c83f1b)

별로 특별한게 뜨는건 없다.

![image](https://github.com/user-attachments/assets/371dd8fd-73b1-4682-ac2f-f6e683b57d8a)

웹에서도 당연히 예상했지만 안열린다.
# mnnm

```
from Crypto.Util.number import *

flag = "hspace{}"
m = bytes_to_long(flag.encode())

while True:
    n = int(input("n = "))
    print(pow(m, n, n) == m)
```

flag 문자열을 바이트로 변환한 후 정수로 변환한다. m = bytes_to_long(flag.encode())에서 m은 hspace{} 문자열의 정수 표현이다.
pow(x, y, z=None) 형태로 사용된다.
x = 밑, y = 지수, z = option(나머지 계산에 이용)
pow(x, y, z) 로 사용할 경우, 결과는 (x ** y) % z 와 동일하다.
사용자가 입력한 n이 m의 거듭제곱 모듈러 연산 결과가 m과 같은지를 확인하는거다. pow(m, n, n)가 m과 같으면 True를 반환하고, 그렇지 않으면 False를 반환하는데 이는 즉,
m^n mod n=m이면 True인 것이다.

```
from Crypto.Util.number import bytes_to_long

flag = "hspace{}"
m = bytes_to_long(flag.encode())
print(m)
```

하면 

![image](https://github.com/user-attachments/assets/460354cc-6f0a-47d5-84c4-9b26943e8a21)

라고 한다.

일단 m값을 알았으니 n을 m의 약수로 대충 잡아보고 코드를 짜서 한번 보겠다.

```
from Crypto.Util.number import *

flag = "hspace{}"
m = bytes_to_long(flag.encode())

# n 값을 찾기
def find_n(m):
    valid_n = []
    
    # m의 약수
    for n in range(1, m + 1):
        if m % n == 0:  # n이 m의 약수인지 확인
            if pow(m, n, n) == m:
                valid_n.append(n)
    
    return valid_n

n_values = find_n(m)
print("Valid n values:", n_values)

```

아니면 
![image](https://github.com/user-attachments/assets/1f8238a2-fe90-418d-a06a-d0c923a21baf)

이 성질을 이용해볼까?

```
from Crypto.Util.number import bytes_to_long

flag = "hspace{}"
m = bytes_to_long(flag.encode())

def modu(m):
    results = []
    for n in range(1, m + 1):
        m_mod_n = m % n
        if pow(m_mod_n, n, n) == m:  # (m mod n)^n mod n == m
            results.append(n)
    return results

# 
n_values = modu(m)
print("Valid n values:", n_values)

```

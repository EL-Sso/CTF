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

여기서 뭐 준게 아무것도 없는데 도커파일이 있길래 한번 docker-compose로 올려봤다.

![image](https://github.com/user-attachments/assets/388c880f-c4df-4304-9033-aac8dff0de25)

아무것도 안뜬다. 컨테이너 내부로 들어가봤는데

![image](https://github.com/user-attachments/assets/695f85ef-9a23-4ee9-ba90-29bdc4c83f1b)

별로 특별한게 뜨는건 없다.

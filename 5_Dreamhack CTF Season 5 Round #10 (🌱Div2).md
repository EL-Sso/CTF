![스크린샷 2024-05-31 194048](https://github.com/EL-Sso/CTF/assets/128667862/d5b50fdc-0a01-41b5-8fa3-995159958a41)


일단 제공된 코드를 열어보면



![스크린샷 2024-05-31 191513](https://github.com/EL-Sso/CTF/assets/128667862/e82ac974-43d9-4b24-88ba-49ae6ad1c31c)



이렇게 뜬다. Vigenere 클래스를 먼저 보면



![image](https://github.com/EL-Sso/CTF/assets/128667862/280621ff-12bf-48a0-8296-8b1f79f0f284)



shift는 주어진 문자 a를 키값인 'd' 만큼 이동시키고 a가 words에 없으면 그대로 리턴한다..
encrypt는 평문의 각 문자에 대해 키 값을 사용하여 shift 함수를 호출하여 암호화 한다.
decrypt는 암호문의 각 문자에 대해 키 값을 사용하여 shift 함수를 호출하여 암호문을 복호화한다.



![image](https://github.com/EL-Sso/CTF/assets/128667862/f43d06e1-a2a7-4113-af31-49abe98f0c85)




key는 길이가 16인 무작위 키 리스트를 생성하 각 값은 words 문자열의 길이 내에서 랜덤하게 선택된다. 여기서 words는 대문자, 소문자, 숫자를 합친 문자열이다.
그리고 cipher로 Vigenere 인스턴스를 생성햐고 암호화한다. 보니까 decrypt 함수를 이용해서 
그리고 output.txt를 열어보면



![image](https://github.com/EL-Sso/CTF/assets/128667862/9abb3dad-c1bd-47e2-a519-20f41e3e820d)


위처럼 암호화된 문장이 뜬다. 
일단 Vigenere클래스의 decrypt 함수를 이용해 복호화하는 코드를 짜면 될 것 같은데 


![image](https://github.com/EL-Sso/CTF/assets/128667862/6f47aec0-2027-497e-b432-8244da5318a4)



key=[random.randint(0, len(words)) for _ in range(16)]

cipher = Vigenere(key)

encrypted_message = "39 2YAx k5LgCy iP Aj9geVQy nEvXnd3Je c kX9P 8z uZ7dbErYRAyegw Ona3Js eRcKyO u7 ffFTl70 TH9ScB – M0HbNuV HDd 4yk TE c3uVU 8Y D1Q ZDNhBpRc 9WY27fjiF – dVWNBP D1Q NAZsM fW bPlBI N3e p8 6i97Nc. FmP HjIjVi 7Zu 4Zth9 bL kYD Gg0yZjc0 dBrYVQ, f8GQ0PD, Bu kGue 8BUlT9FmE0 UDCNp2vQi xd jkfm97 9uDfndFF egcc9CZ 27 mTE 2Y7u A8Zi fM N4Ks3 UVK Dg0. LTAabG 2RSDTqDu GP7QbLq. qy ZE2Xy GUpG kYD eYvEXf DJdMc fE Ep2DTjX4Zt dlk uObyx f DJq7ckZM0 \"w8gse0WtBie\" (L 4yk) FT LyZkB1 a29Tjc FmIjRSDDd yFQwj QfMvVi. 9I, Tjc0 dHoVj DSc zXfR. Ek{sreSsZxMq8OPf37BwUdvpZKzQ8oNg7Z8rwASqbxQsBl0g935rwDLQaNxNCnxK44g}"

decrypted_message = cipher.decrypt(encrypted_message)

print(f"{decrypted_message}")

이렇게 짜봤는데 생각해보니 키값이 뭔질 모른다.. Vigenere가 알파벳 하나 당 일대일로 대응되는 암호화 기법도 아니라 키값을 찾지 못했다.


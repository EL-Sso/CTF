![image](https://github.com/EL-Sso/CTF/assets/128667862/c58c03a8-6900-44dd-8116-e055af8461ce)



문제를 보니 address bar라는게 보인다. 이걸 보니까 url 복호화를 해야하는 것 같다. 그래서 url decode를 진행했다.


![스크린샷 2024-05-25 002243](https://github.com/EL-Sso/CTF/assets/128667862/fff8517b-d60a-4e9a-b5ae-d53561ea31d9)


그랬더니



![스크린샷 2024-05-25 002254](https://github.com/EL-Sso/CTF/assets/128667862/bbd50845-af06-4ce4-b152-6d1c9bcf5e66)


암호화가 풀리지 않았다. 그래서 이게 아닌가 생각하고 있었는데 문제로 돌아와서 보니 it happened twice란다. 암호화가 두번 적용되어있었나보다. 그 말은 복호화를 한번 더해야한다는 것 같아서 한번 더했다.


![스크린샷 2024-05-25 002301](https://github.com/EL-Sso/CTF/assets/128667862/0add91aa-c7bd-4670-a7bd-17d344764d3b)


답이 나온 것을 볼 수 있다.





다음은 twine 문제를 풀어보겠다.


![스크린샷 2024-05-25 224206](https://github.com/EL-Sso/CTF/assets/128667862/76e6c247-f77f-41ea-89a3-704fd4446a03)


문제에는 twine의 뜻과 jpg 파일이 첨부되어 있다. 사진을 보면


![twine](https://github.com/EL-Sso/CTF/assets/128667862/46c8055c-c13d-48d8-a1ae-92582969ce1f)


문제에 나온 것처럼 뭐 여러개를 꼬아놓은 줄같은 것이 있다. 
그래서 일단 스테가노 그래피 해독 사이트에 넣어 봤는데


![image](https://github.com/EL-Sso/CTF/assets/128667862/d19ab4ce-2826-477d-8b62-f4d12010313c)


아무것도 없는걸 보니 아닌 것 같았고 헥스디로 돌려봤다.

일단 헤더랑 푸터는 

![image](https://github.com/EL-Sso/CTF/assets/128667862/868ba45d-0d44-4555-9a16-fa0677b1a000)

![image](https://github.com/EL-Sso/CTF/assets/128667862/bdf69678-1740-4fb4-8b12-d70598447f78)

정상적으로 들어가있었고 다른 파일들이 들어있지는 않은 것 같았다. 그래서 설마하고 flag 문자열을 찾아봤는데 



![스크린샷 2024-05-25 012201](https://github.com/EL-Sso/CTF/assets/128667862/877caf60-0686-4eab-8627-27d446d91610)


이거였다.


# Docker-02(네트워크)

![스크린샷 2024-04-17 오후 1.28.15.png](https://github.com/s-u-n-h-o/TIL/blob/main/docker-02/images/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2024-04-17_%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE_1.28.15.png)


---

<aside>
💡 도커에서 여러개의 컨테이너를 생성하고 컨테이너끼리 통신하는법
가장많이 공부하는 워드프레스 사이트 구축을 해보려고 한다.

</aside>

---

## 1. 도커 네트워크 생성/삭제

- **워드프레스 = 워드프레스 컨테이너 + MySQL컨테이너**
- 각각의 컨테이너를 생성한다고 하더라도 연결이 되어있지않으면 사용이 불가
- `네트워크` 에 두개의 컨테이너를 소속하여 연결한다.

**1.네트워크 생성**

`docker network create 네트워크이름` 

![Untitled](https://github.com/s-u-n-h-o/TIL/blob/main/docker-02/images/Untitled.png)

**2.워드프레스 + MySQL 컨테이너 연동**

<aside>
✏️ 연동전 옵션들을 알고가면 좋은 옵션사항들

</aside>

| 항목 | 옵션 |
| --- | --- |
| 네트워크 이름 | —net |
| 실행 옵션 | -dit |
| MySQL 루트 패스워드 | -e MYSQL_ROOT_PASSWORD |
| MySQL 패스워드 | -e MYSQL_PASSWORD |
| MySQL 데이터 베이스 이름 | -e MYSQL_DATABASE |
| 포트번호 설정 | -p |

**2-1. MySQL 컨테이너 생성 및 실행**

```
docker run \
--name mysql -dit \
--net=docker_network1 \
-e MYSQL_ROOT_PASSWORD=myrootpass \
-e MYSQL_DATABASE=mysqlDB \
-e MYSQL_USER=sunho \
-e MYSQL_PASSWORD=sunhopass \
mysql --character-set-server=utf8mb4 \
--collation-server=utf8mb4_unicode_ci \
```

![Untitled](https://github.com/s-u-n-h-o/TIL/blob/main/docker-02/images/Untitled%201.png)

![Untitled](https://github.com/s-u-n-h-o/TIL/blob/main/docker-02/images/Untitled%202.png)

<aside>
❗ 명령어를 입력하다가  mysql 컨테이너가 생성된후 2초뒤에 자동으로 종료가 되는데 명령어 오타도 아니고 한참찾았는데 
`--default-authentication-plugi` 이 옵션은 MySQL 8.0 이상에서는 **`default-`** 옵션이 더 이상 사용되지 않고, MySQL 8.0에서는 기본적으로 'caching_sha2_password'라는 새로운 인증 플러그인이 사용되는것을 알게되었다! 이 옵션을 빼고 실행하니까 잘 된다

</aside>

**2-2. 워드프레스 컨테이너 생성 및 실행**

```
docker run \
--name wordpress -dit \
--net=docker_network1 -p 8082:80 \
-e WORDPRESS_DB_HOST=mysql \
-e WORDPRESS_DB_NAME=mysqlDB \
-e WORDPRESS_DB_USER=sunho \
-e WORDPRESS_DB_PASSWORD=sunhopass wordpress
```

![Untitled](https://github.com/s-u-n-h-o/TIL/blob/main/docker-02/images/Untitled%203.png)

![Untitled](https://github.com/s-u-n-h-o/TIL/blob/main/docker-02/images/Untitled%204.png)

**2-3. 웹브라우저로 워드프레스에 접근하여 연결되었는지 확인**

- 8082 포트에 porting했기때문에 **[localhost:8082](http://localhost:8082) 에 접근해서 보면 워드프레스의 초기화면을 확인할수있다.**
    
    ![스크린샷 2024-05-11 오후 6.59.25.png](https://github.com/s-u-n-h-o/TIL/blob/main/docker-02/images/%E1%84%82%E1%85%A6%E1%84%90%E1%85%B3%E1%84%8B%E1%85%AF%E1%84%8F%E1%85%B3%2043283eb227ed413f80112c8740c9fe15/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2024-05-11_%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE_6.59.25.png)
    

---

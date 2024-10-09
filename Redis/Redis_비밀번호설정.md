1. Redis 컨테이너 생성
    
    ```bash
    docker run -d --name redis-container \
    -p 6379:6379 \
    -v redis-data:/data \
    redis redis-server --appendonly yes
    ```
    
2. docker 컨테이너에 접속
    
    ```bash
    docker exec -it redis-container bash
    ```
    
3. redis-cli 접속
    
    ```bash
    root@285df54fcffe:/data# redis-cli
    ```
    
4. redis 비밀번호 실행 테스트
    
    ```bash
    127.0.0.1:6379> AUTH 'test'
    (error) ERR AUTH <password> called without any password configured for the default user. Are you sure your configuration is correct?
    ```
    
    ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/016eec1c-2eaf-4365-9453-c1cd98dc23a6/6cead2d0-6a70-45b9-8f90-4109fc8de773/image.png)
    
    ⇒ 비밀번호가 설정되어 있지 않아 error가 발생한다.
    
5. 비밀번호 설정 정보 확인
    
    ```bash
    127.0.0.1:6379> config get requirepass
    1) "requirepass"
    2) ""
    ```
    
    ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/016eec1c-2eaf-4365-9453-c1cd98dc23a6/38d61a21-b99f-4cd3-b25a-529d563cfcc1/image.png)
    
    ⇒ 비밀번호 정보가 없다.
    
6. 비밀번호 설정
    
    ```bash
    127.0.0.1:6379> config set requirepass your_password
    ```
    
    ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/016eec1c-2eaf-4365-9453-c1cd98dc23a6/127f1bcd-91d2-48ff-99f8-4dbe16bfae10/image.png)
    
7. 테스트를 위해 redis-cli 재접속
8. 테스트를 위한 정보 세팅
    
    ```bash
    127.0.0.1:6379> set foo "1234"
    ```
    
    ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/016eec1c-2eaf-4365-9453-c1cd98dc23a6/2f8742a7-d386-4ecd-b605-bea9abf846e6/image.png)
    
9. 비밀번호 인증없이 조회 시 에러 발생 확인
    
    ```bash
    127.0.0.1:6379> get foo
    ```
    
    ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/016eec1c-2eaf-4365-9453-c1cd98dc23a6/9cb48ecf-80d8-4a60-964a-142cb1536389/image.png)
    
10. 비밀번호 인증
    
    ```bash
    127.0.0.1:6379> AUTH j11d109semonemo!
    ```
    
    ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/016eec1c-2eaf-4365-9453-c1cd98dc23a6/15d64aab-7cf2-4f2f-ae7e-974b52e52082/image.png)
    
11. 인증 후 조회 테스트
    
    ```bash
    127.0.0.1:6379> get foo
    ```
    
    ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/016eec1c-2eaf-4365-9453-c1cd98dc23a6/b871deef-56bd-462d-9cbb-281ba5cda1a4/image.png)
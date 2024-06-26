# Authentication : 인증

> 웹 애플리케이션에 접근하려는 유저를 식별하는 행위
> 
- 아이디, 비밀번호와 같은 인증 객체(식별자)를 통해 사용자를 확인하는 행위
- 항상 인가보다 먼저 등장한다.
- 에러 코드로 401을 반환

# Authorization : 인가

> 인증이 완료된 사용자가 어떠한 권한, 특권 혹은 역할을 가지고 있는지 확인하는 행위
> 
- 인증이 성공적으로 완료된 이후에 등장 ⇒ 따라서, 사용자의 자격 증명에 대해 걱정할 필요가 없다.
- 사용자, 관리자 … 와 같은 역할을 두는 것
- 에러 코드로 403을 반환
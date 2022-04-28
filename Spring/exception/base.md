### 예외 계층
- Object : 모든 객체의 최상위 부모는 Object 이므로 예외의 최상위 부모도 Object 이다.
- Throwable : 최상위 예외로 하위에 Exception과 Error가 있다.
- Error : 메모리 부족이나 심각한 시스템 오류와 같이 애플리케이션에서 복구 불가능한 시스템 예외이다. 애플리케이션 개발자는 이 예외를 잡으려고 해서는 안된다.
- Exception : 체크 예외
    - 애플리케이션 로직에서 사용할 수 있는 실질적인 최상위 예외이다.
    - Exception 과 그 하위 예외는 모두 컴파일러가 체크하는 체크 예외이다. 하지만 RuntimeException은 예외이다.
- RuntimeException : 언체크 예외, 런타임 예외
    - 컴파일러가 체크하지 않는 언체크 예외이다.

### 체크 예외 
- Exception 과 그 하위 예외는 모두 컴파일러가 체크하는 체크 예외이다. 단 RuntimeException은 예외이다.
- 체크 에외는 잡아서 처리하거나, 밖으로 던지도록 선언해야한다. 그렇지 않으면 컴파일 오류가 발생한다.
- 장점
    - 개발자가 실수로 예외를 누락하지 않도록 컴파일러를 통해 문제를 잡아주는 안전장치이다.
- 단점
    - 개발자가 모든 체크 예외를 반드시 잡거나 던지도록 처리해야 하기 때문에 너무 번거로운 일이 된다. 크게 신경쓰고 싶지 않은 예외까지 모두 챙겨야한다. 의존관계에 따른 단점도 있다.

### 언체크 예외
- RuntimeException 과 그 하위 예외이다.
- 컴파일러가 예외를 체크하지 않는다는 뜻이다.
- 예외를 던지는 throws를 선언하지 않고 생략할 수 있다. 이 경우 자동으로 예외를 던진다.
- 장점
    - 신경쓰고 싶지 않은 언체크 예외를 무시할 수 있다. 체크 예외의 경우 처리할 수 없는 예외를 밖으로 던지려면 항상 'throws 예외' 를 선언해야 하지만 언체크 예외는 이 부분을 생략할 수 있다.
- 단점
    - 개발자가 실수로 예외를 누락할 수 있다.

### 체크 예외 활용
기본원칙
- 기본적으로는 언체크(런타임) 예외를 사용하자.
- 체크 예외는 비즈니스 로직상 의도적으로 던지는 예외에만 사용하자.
    - 해당 예외를 잡아서 반드시 처리해야 하는 문제일 때만 사용하자. ex) 계좌이체 실패, 결제시 포인트 부족, 로그인 ID,PW 불일치
    - 개발자가 놓치면 매우 심각한 문제를 체크 예외로 만들어 두면 컴파일러를 통해 놓친 예외를 인지할 수 있다.

체크 예외의 문제점
- repository 에서 SQLException, NetworkException 과 같은 예외를 service 계층으로 넘겨도 service 계층은 이를 해결할 수 있는 방법이 없다. 하지만 throws 예외 구문은 필수적으로 적어줘야한다.
- 복구 불가능한 예외 
- 의존 관계에 대한 문제
    - controller, service, repository 모든 계층에서 SQLException을 의존하고 있는 경우 리포지토리를 JDBC 기술이 아닌 다른기술로 변경한 다면 모든 계층을 수정해야하는 일이 벌어진다.

### 런타임 예외 활용
리포지토리에서 체크 예외인 SQLException이 발생하면 런타임 예외인 RuntimeSQLException으로 전환하여 예외를 던진다. 이 때 기존 예외를 포함해주어야 예외 출력시 스택 트레이스에서 기존 예외도 함께 확인할 수 있다.
- 대부분 복구 불가능한 예외 : 런타임 예외를 사용하면 서비스나 컨트롤러가 복구 불가능한 예외를 신경쓰지 않아도 된다. 이렇게 복구 불가능한 예외는 일관성 있게 공통으로 처리해야한다.
- 의존관계에 대한 문제 : 해당 객체가 처리할 수 없는 예외는 무시하여 예외를 강제로 의존하지 않아도 된다.

### 예외 포함과 스택 트레이스
예외를 전환할 때는 반드시 기존 예외를 포함해야한다. 그렇지 않으면 스택 트레이스를 확인할 때 심각한 문제가 발생한다.
```
    static class Repository {
        public void call() {
            try {
                runSQL();
            } catch (SQLException e) {
                // 예외를 포함해야한다.
                throw new RuntimeSQLException(e);
            }
        }

        private void runSQL() throws SQLException {
            throw new SQLException("ex");
        }
    }

    static class RuntimeConnectException extends RuntimeException {
        public RuntimeConnectException(String message) {
            super(message);
        }
    }

    static class RuntimeSQLException extends RuntimeException {

        public RuntimeSQLException(Throwable cause) {
            super(cause);
        }
    }
```
기존 예외를 포함하지 않는다면 기존에 발생한 SQLException과 스택 트레이스를 확인할 수 없다.
# 1강 - JDBC 이해

# 개요

## 과거

![Untitled](https://user-images.githubusercontent.com/61227459/184537361-0df3241f-4477-42ff-b400-f4ee0443da46.png)

1. 커넥션 연결 : TCP/IP 연결
2. SQL 전달 : DB가 이해할 수 있는 SQL을 연결된 커넥션을 통해 DB에 전달
3. 결과 응답

### 문제

1. DB 종류에 따라 코드 변경
2. 개발자가 커넥션 연결, SQL, 결과 응답 방법 DB마다 새로 학습

## 현재

<aside>
💡 JDBC(Java Database Connectivity)는 자바에서 데이터베이스에 접속할 수 있도록 하는 자바 API다. JDBC는 데이터베이스에서 자료를 쿼리하거나 업데이트하는 방법을 제공한다. - 위키백과

</aside>

![Untitled 1](https://user-images.githubusercontent.com/61227459/184537356-11901441-02c9-44d0-8920-2345b549b500.png)

→ 애플리케이션 로직은 JDBC 표준 인터페이스에만 의존

# 최신 데이터 접근 기술

## SQL Mapper

![Untitled 2](https://user-images.githubusercontent.com/61227459/184537357-be91815d-78b5-4995-80de-e9a0435bbfbe.png)

### 장점

- 반복 코드 제거
- 응답을 객체로 반환

### 단점

- SQL 작성

## ORM

![Untitled 3](https://user-images.githubusercontent.com/61227459/184537358-15dda8bb-01f4-4dd9-99e1-5469480eb40e.png)

- 객체를 관계형 데이터베이스 테이블과 매핑
- SQL을 직접 구현하지 않아도 됨

# JDBC 커넥션 인터페이스와 구현

## DriverManager

> DB 드라이버들을 관리하고 커넥션을 획득
>


![Untitled 4](https://user-images.githubusercontent.com/61227459/184537359-fd87693f-74da-466b-aee2-2d40f9cc9a16.png)

1. DriverManger.getConnection() 호출
2. 라이브러리에 등록된 드라이버 목록을 자동으로 인식 이후 정보들을 넘겨 커넥션 확인
    1. URL, 이름, 비번

# 코드

## 저장

```java
@Slf4j
public class MemberRepositoryV0 {
 public Member save(Member member) throws SQLException {
 String sql = "insert into member(member_id, money) values(?, ?)";
 Connection con = null;
 PreparedStatement pstmt = null;
 try {
 con = getConnection();
 pstmt = con.prepareStatement(sql);
 pstmt.setString(1, member.getMemberId());
 pstmt.setInt(2, member.getMoney());
 pstmt.executeUpdate();
 return member;
 } catch (SQLException e) {
 log.error("db error", e);
 throw e;
 } finally {
 close(con, pstmt, null);
 }
 }
 private void close(Connection con, Statement stmt, ResultSet rs) {
 if (rs != null) {
 try {
 rs.close();
 } catch (SQLException e) {
 log.info("error", e);
 }
 }
 if (stmt != null) {
 try {
 stmt.close();
 } catch (SQLException e) {
 log.info("error", e);
 }
 }
 if (con != null) {
 try {
 con.close();
 } catch (SQLException e) {
 log.info("error", e);
 }
 }
 }
 private Connection getConnection() {
 return DBConnectionUtil.getConnection();
 }
}
```

<aside>
💡 리소스 정리 : try-catch 구문 마지막에 finally 활용 → 에러가 나든 말든 항상 수행해야 하기 때문

</aside>

> `PreparedStatement` 는 `Statement` 의 자식 타입인데, ? 를 통한 파라미터 바인딩을 가능하게 해준다.
> 
> 
> → 참고로 SQL Injection 공격을 예방하려면 `PreparedStatement` 를 통한 파라미터 바인딩 방식을 사용해야 한다
> 

## 조회

```java
public Member findById(String memberId) throws SQLException {
 String sql = "select * from member where member_id = ?";
 Connection con = null;
 PreparedStatement pstmt = null;
 ResultSet rs = null;
 try {
 con = getConnection();
 pstmt = con.prepareStatement(sql);
 pstmt.setString(1, memberId);
 rs = pstmt.executeQuery();
 if (rs.next()) {
 Member member = new Member();
 member.setMemberId(rs.getString("member_id"));
 member.setMoney(rs.getInt("money"));
 return member;
 } else {
 throw new NoSuchElementException("member not found memberId=" +
memberId);
 }
 } catch (SQLException e) {
 log.error("db error", e);
 throw e;
 } finally {
 close(con, pstmt, rs);
 }
}
```

### ResultSet


![Untitled 5](https://user-images.githubusercontent.com/61227459/184537360-dd3c938d-4615-4d3d-9ec5-e330f4c8baea.png)

`rs.next()` 로 cursor 이동하면서 데이터를 가르킴

## 수정, 삭제

### 삭제 test code

→ 삭제를 하게 되면 데이터가 없어지기 때문에 조회가 안됨

→ `NoSuchElementException` 이 발생하게 됨

→ 위 exception이 발생됐다는 것을 assert

→ 즉, 예외가 발생하는 것이 정상이다
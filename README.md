# mybatis-spring-tutorial

Mybatis를 Spring에서 사용하는 방법을 다룹니다.

## 의존성

[Mybatis](https://github.com/mybatis/mybatis-3)와 [Mybatis-Spring](https://github.com/mybatis/spring) 라이브러리를 사용합니다.

## 공식 문서 참고

[Mybatis 3 한글문서](http://www.mybatis.org/mybatis-3/ko/index.html)

[Mybatis-Spring 한글문서](http://www.mybatis.org/spring/ko/index.html)

## 목차

* [Mybatis란](#Mybatis란)
    * [Mybatis의 설정 방법](#Mybatis의%20설정)
        * [기본 XML 정의](#기본%20XML%20정의)
        * [기본이 되는 XML 설정 예시](#기본이%20되는%20XML%20설정%20예시)
* [Mybatis-Spring이란](#Mybatis-Spring이란)
    * [Mybatis-Spring의 설정 방법]()
    * [Mybatis-Spring을 썼을 때의 장단점]()
* [공통으로 적용되는 내용](#공통으로%20적용되는%20내용)
    * [Mapper 정의하기](#Mapper%20정의하기)
    * [Mapper에서 SQL을 정의하는 방법](#Mapper에서%20SQL을%20정의하는%20방법)
    * [Mybatis에서 사용되는 객체에 대하여]()
    
## Mybatis란

Mybatis는 질의 후 수행해야 하는 객체 매핑을 대신 수행해줌으로써 프로그래머가 SQL 작성에만 신경쓸 수 있게 하는 Persistance Framework이다. `Primitive(or Wrapper)Type`, `Map`, `POJO`를 매핑할 수 있고, 일부 설정은 (Spring을 사용하지 않아도) 어노테이션을 사용할 수 있다.

### Mybatis 설치

- 최신버전: 3.4.7-SNAPSHOT (2018 3월 12일 배포)

```xml
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis</artifactId>
  <version>3.4.7-SNAPSHOT</version>
</dependency>
```

### Mybatis의 설정

Mybatis **자체**의 설정은 (Spring을 사용하지 않는 경우, 당연하게도) XML 설정만을 지원한다. 이번 챕터는 Mybatis의 설정만을 다룬다.

#### 기본 XML 정의

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
</configuration>
```

#### 기본이 되는 XML 설정 예시

- Mybatis의 XML 설정 옵션은 정말 많지만 이번에는 기본적이고 필수적인 설정만 다룬다. 아래 설정만 익혀도 Mybatis의 사용에는 문제가 없다.

- Mybatis-Spring은 Bean으로 요소들을 등록하기 때문에 아래 방식으로 dataSource와 transactionManager를 사용할 수 없다. Spring과의 연동을 원하는 경우 참고만 하고 넘어가면 된다.

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
<!-- 어떤 environment를 사용할지 기본값 설정 -->
<environments default="development">
    <!-- environment 의 정의 - 'development' -->
    <environment id="development">
    <!-- transactionManager 정의 -->
    <transactionManager type="JDBC"/>
    <!-- dataSource 정의 -->
    <dataSource type="POOLED">
        <property name="driver" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
    </dataSource>
    </environment>
</environments>
<!-- Mapper들을 등록할 태그 -->
<mappers>
    <!-- Mapper XML 등록 -->
    <!-- classpath 상대주소 방식 -->
    <mapper resource="org/mybatis/example/BlogMapper.xml"/>
    <mapper resource="org/mybatis/builder/PostMapper.xml"/>

    <!-- URL 접근 방식 -->
    <!-- /var 폴더 하에 있는 XML 파일에 접근 -->
    <mapper url="file:///var/sqlmaps/AuthorMapper.xml"/>

    <!-- Mapper 인터페이스 등록 -->
    <mapper class="org.mybatis.builder.AuthorMapper" />

    <!-- 해당 Package 이하의 모든 인터페이스가 등록됨 -->
    <package name="org.mybatis.builder" />
</mappers>
</configuration>
```

1. `environtments`는 환경을 구분하는 데 사용되는 Tag이다. `environments` 태그에선 반드시 `default` 값이 필요하다.

2. `environment`는 각 환경을 정의하며, `id`를 필수적으로 정의해야 한다.

3. `transcationManager`는 Mybatis가 제공하는 TransactionManager를 설정하는 Tag로 `type`을 필수적으로 지정해야 한다. Mybatis는 `type`으로 `JDBC`와 `MANAGED`를 지원한다. `JDBC`는 JDBC의 트랜잭션을 Mybatis가 처리하게 되고, `MANAGED`는 Container에게 맡기고 (`CMT(Container-Managed Transaction`) connection을 close하는 것 이외에 아무것도 하지 않는다. - `MANAGED` 사용 시 공식 문서 참고

4. `dataSource`는 JDBC Connection 객체를 생성하는 방법을 정의하는 `type`을 필수적으로 정의해야 한다. `POOLED`와 `UNPOOLED`, `JNDI`가 올 수 있다. `UNPOOLED`, `JNDI` 사용 시 공식문서를 참고하기 바란다. `POOLED`의 경우 pooling을 수행하는 것으로, 옵션이 굉장히 많다. 

    - `poolMaximumActiveConnections = 10` – 주어진 시간에 존재할 수 있는 활성화된(사용중인) 커넥션의 수.
    - `poolMaximumIdleConnections` – 주어진 시간에 존재할 수 있는 유휴 커넥션의 수

    - `poolMaximumCheckoutTime = 20000(ms, 20초)` – 강제로 리턴되기 전에 풀에서 “체크아웃” 될 수 있는 커넥션의 시간.

    - `poolTimeToWait = 20000(ms, 20초)` – 풀이 로그 상태를 출력하고 비정상적으로 긴 경우 커넥션을 다시 얻을려고 시도하는 로우 레벨 설정.

    - `poolPingQuery` – 커넥션이 작업하기 좋은 상태이고 요청을 받아서 처리할 준비가 되었는지 체크하기 위해 데이터베이스에 던지는 핑쿼리(Ping Query). 디폴트는 “핑 쿼리가 없음” 이다. 이 설정은 대부분의 데이터베이스로 하여금 에러메시지를 보게 할수도 있다.

    - `poolPingEnabled = false` – 핑쿼리를 사용할지 말지를 결정. 사용한다면, 오류가 없는(그리고 빠른) SQL을 사용하여     - `poolPingQuery` 프로퍼티를 설정해야 한다.

    - `poolPingConnectionsNotUsedFor = 0` – `poolPingQuery`가 얼마나 자주 사용될지 설정한다. 필요이상의 핑을 피하기 위해 데이터베이스의 타임아웃 값과 같을 수 있다. 디폴트 값은 `poolPingEnabled`가 `true`일 경우에만, 모든 커넥션이 매번 핑을 던지는 값이다. 

5. `mappers`는 Mapper를 감싸는 Tag 이다.

6. `mapper`는 `resource`값을 필수적으로 지정해야 한다. 해당 값은 XML의 위치를 나타낸다. classpath 상대주소로 XML에 접근하거나, URL에 접근하는 방식, Mapper 인터페이스를 명시하는 방식, 패키지 명시 방식이 있다.

## Mybatis-Spring이란


## 공통으로 적용되는 내용

Mybatis-spring를 사용하더라도 동일한 작업을 해야 하는 것들에 대한 내용입니다. Spring에 비의존적인 부분을 다룹니다.

### Mapper 정의하기

* Mybatis는 SQL 작성과 리턴타입(or Map), 캐시 여부만 지정한다면 질의 이후 결과 객체를 받는 것까지의 기능이 구현된다. Mapper를 정의하는 것은 XML과 어노테이션(그냥 Mybatis에서도 가능) 모두 사용할 수 있다. 

* XML로 Mapper를 정의하는 방법. 한 개의 매퍼 XML 파일에는 많은 수의 매핑 구문을 정의할 수 있다. 아래는 `org.mybatis.example.BlogMapper` 네임스페이스에서 selectBlog라는 매핑 구문을 정의한 것이다. 아래 Select는 `org.mybatis.example.BlogMapper.selectBlog` 형태로 실행할 수 있게 된다.

    ```xml
    <mapper namespace="org.mybatis.example.BlogMapper">
        <select id="selectBlog" resultType="Blog">
            SELECT * FROM blog WHERE id = #{id}
        </select>
    </mapper>
    ```

* 어노테이션으로 Mapper를 정의하는 방법. 인터페이스로 메소드를 정의하고, 각 메소드 위에 어노테이션으로 SQL을 정의하는 방법이다. 매핑된 구문은 XML 에 전혀 매핑될 필요가 없다.

    ```java
    package org.mybatis.example;

    public interface BlogMapper {
        @Select("SELECT * FROM blog WHERE id = #{id}")
        Blog selectBlog(int id);
    }
    ```

* Select 실행 방법
    
    첫 번째 방법

    ```java
    Blog blog = (Blog) session.selectOne("org.mybatis.example.BlogMapper.selectBlog", 101);
    ```
    
    두 번째 방법

    ```java
    BlogMapper mapper = session.getMapper(BlogMapper.class);
    Blog blog = mapper.selectBlog(101);
    ```

    두번째 방법은 Magic Constant(namespace 문자열)에 의존하지 않아 더 클린 코드에 가깝고,
     리턴타입에 대해 타입 캐스팅을 하지 않아도 되는 등의 장점을 가진다. 

* XML vs Annotation

    매핑된 구문을 일관된 방식으로 정의하는 것이 중요하다. 어노테이션은 XML로 쉽게 변환할 수 있고 그 반대의 형태 역시 쉽게 처리할 수 있다.

### Mapper에서 SQL을 정의하는 방법

1. 간단한 POJO 객체를 CRUD 하는 방법
    * SQL에 필요한 Parameter 설정 - #{propName} 의 원리
        1. Primitive Type의 경우 - 값 그대로 사용
        2. Object의 경우 - 프로퍼티를 객체 내에서 찾아 그 값을 사용
    * SQL에 필요한 Parameter 설정 - $ vs #
        1. $는 SQL문을 값을 replace해서 그대로 execute한다. (이 때 escape은 자동으로 처리하지 않음) 따라서 $는 권장되는 옵션이 아니다.
        2. #은 PreparedStatement의 값으로 전달한다.
    * `<insert>`,`<select>`,`<update>`,`<delete>` 등의 예제와 설정할 수 있는 프로퍼티 소개
    * `parameterType`, `resultType`에 사용할 수 있는 Mybatis의 별칭 소개

2. 좀 더 생산성을 높이는 방법
    * `<sql>`
    * 타이핑을 줄이는 방법 - typeAlias 정의

        ```xml
        <!-- Mybatis XML 설정파일에서 -->
        <typeAlias type="com.someapp.model.User" alias="User"/>

        <!-- Mapper XML파일에서 -->
        <select id="selectUsers" resultType="User">
            SELECT id, username, hashedPassword
            FROM some_table
            WHERE id = #{id}
        </select>
        ```

        적용 전/후

        ```xml
        <select id="selectUsers" resultType="com.someapp.model.User" ... />
        <select id="selectUsers" resultType="User" ... />
        ```

3. 매핑 객체가 1-depth POJO가 아닌 경우
    * `<resultMap>` 정의 및 사용

        ResultMap은 ResultSet을 Mapping하는 방법을 정의한 것이다. <br />
        놀랍게도(?) ResultMap은 `resultType`으로 등록된 객체를 기준으로 자동 생성된다. <br />
        그러나 이 좋은 기능을 걷어차버리고 프로그래머가 직접 정의할 수도 있다. <br />
        직접 정의하는 경우 `resultMap` 속성을 사용하게 된다.

        ```xml
        <resultMap id="userResultMap" type="User">
            <id property="id" column="user_id" />
            <result property="username" column="username"/>
            <result property="password" column="password"/>
        </resultMap>
        ```

        그러나 이 정도로 간단한 객체는 자동 생성에 당연히 맡길 것이다. <br />
        실제로는 JOIN 등의 사용 시 정의하게 된다.

        ```xml
        <!-- resultMap 정의: Blog 객체에 매핑 -->
        <resultMap id="blogResultMap" type="Blog">

            <!-- 호출할 constructor 및 매개변수 명시 -->
            <constructor>
                <!-- 아래 2가지 방법으로 정의할 수 있다. -->
                <!-- id 속성의 경우 'idArg' 태그를 사용하면 전체 성능을 향상시킬 수 있다고 한다 -->
                <arg column="blog_id" javaType="int" />
                <idArg column="blog_id" javaType="int"/>
            </constructor>
            
            <!-- result 태그는 필드를 의미한다. -->
            <!-- property = POJO의 속성명, column = SELECT된 결과의 속성명 -->
            <result property="title" column="blog_title" />

            <!-- association 태그는 중첩된 ResultMap을 의미한다. -->
            <!-- 이미 정의된 ResultMap의 ID를 참조시킬 수도 있다. -->
            <!-- ResultMap이므로 동일한 태그를 사용하게 된다. -->
            <!-- 중첩된 객체를 처리하는 데 사용된다. -->
            <association property="author" javaType="Author">
                <!-- id 태그는 result와 동일하나 ID임을 명시한다. 성능 향상이 있다고 한다. -->
                <id property="id" column="author_id" />
                <result property="username" column="author_username" />
                <result property="password" column="author_password" />
                <result property="email" column="author_email" />
                <result property="bio" column="author_bio" />
                <result property="favouriteSection" column="author_favourite_section" />
            </association>

            <!-- collection 태그는 Collection을 의미한다. -->
            <!-- collection 태그 또한 중첩된 ResultMap을 의미한다. -->
            <!-- 마찬가지로 이미 정의된 ResultMap의 ID를 참조시킬 수 있다. -->
            <collection property="posts" ofType="Post">
                <id property="id" column="post_id" />
                <result property="subject" column="post_subject" />
                <association property="author" javaType="Author" />
                <collection property="comments" ofType="Comment">
                    <id property="id" column="comment_id" />
                </collection>
                <collection property="tags" ofType="Tag" >
                    <id property="id" column="tag_id" />
                </collection>
                <discriminator javaType="int" column="draft">
                    <case value="1" resultType="DraftPost" />
                </discriminator>
            </collection>
        </resultMap>
        ```

        Association Tag에서 resultMap의 ID를 사용하는 방법

        ```sql
        /* 예제로 사용되는 SELECT문 */
        SELECT
            B.id            AS blog_id,
            B.title         AS blog_title,
            B.author_id     AS blog_author_id,
            A.id            AS author_id,
            A.username      AS author_username,
            A.password      AS author_password,
            A.email         AS author_email,
            A.bio           AS author_bio
        FROM Blog B LEFT OUTER JOIN Author A ON B.author_id = A.id
        WHERE B.id = #{id}
        ```

        ```xml
        <!-- author 속성에 대응하는 resultMap 사용 -->
        <resultMap id="blogResult" type="Blog">
            <id property="id" column="blog_id" />
            <result property="title" column="blog_title"/>
            <association property="author" resultMap="authorResultMap" />
        </resultMap>

        <!-- Author 객체에 대응되는 resultMap 정의 -->
        <resultMap id="authorResultMap" type="Author">
            <id property="id" column="author_id"/>
            <result property="username" column="author_username"/>
            <result property="password" column="author_password"/>
            <result property="email" column="author_email"/>
            <result property="bio" column="author_bio"/>
        </resultMap>
        ```

4. Mybatis 수준에서 캐시하는 방법
    * `<cache>`,`<cache-ref>`

### Mybatis에서 사용되는 객체에 대하여


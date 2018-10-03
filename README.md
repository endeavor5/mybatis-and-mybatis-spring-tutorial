# mybatis-and-mybatis-spring-tutorial

이 문서는 Mybatis를 Spring에서 사용하는 방법을 자세히 다룹니다. 실제로 사용하는 입장에서 즉각적으로 도움이 되기 위해서 설정, Mapper 정의, 생산성 향상 부분 등의 목차와 링크를 제공합니다. 이 문서의 모든 부분을 제대로 익힌다면 코딩의 즐거움과 편안함을 충분히 누릴 수 있을 것이라고 생각합니다 :smile:

## 의존성

[Mybatis](https://github.com/mybatis/mybatis-3)와 [Mybatis-Spring](https://github.com/mybatis/spring) 라이브러리를 사용합니다.

## 공식 문서 참고

한글 문서는 영문 독해가 어느 정도 되신다면 추천하지 않습니다. 최신 버전의 반영이 되지 않았기 때문입니다. 그러나 한글 문서더라도 참고하지 않는 것보다 100배 정도는 좋습니다. 이 문서는 설정 부분을 제외한 모든 부분을 영문 문서와 Mybatis github repository의 issue에 있는 QnA를 참고하여 만들었습니다.

[Mybatis 3 한글문서](http://www.mybatis.org/mybatis-3/ko/index.html)

[Mybatis-Spring 한글문서](http://www.mybatis.org/spring/ko/index.html)

## 사용 가능한 플러그인/라이브러리

[Mybaptise - 이클립스 Mybatis 플러그인](https://github.com/mybatis/mybatipse)

[Mybatis Generator - 라이브러리](https://github.com/mybatis/generator)

## 목차

* [Mybatis란](#mybatis란)
    * [Mybatis 설치](#mybatis-설치)
    * [Mybatis의 설정 방법](#mybatis의-설정)
        * [기본 XML 정의](#기본-xml-정의)
        * [기본이 되는 XML 설정 예시](#기본이-되는-xml-설정-예시)
* [Mybatis-Spring이란](#mybatis-spring이란)
    * [Mybatis-spring 설치](#mybatis-spring-설치)
    * [Mybatis-Spring의 설정 방법](#mybatis-spring의-설정-방법)
    * [Mybatis-Spring을 썼을 때의 장단점 (작성중)](#mybatis-spring을-썼을-때의-장단점)
* [공통으로 적용되는 내용](#공통으로-적용되는-내용)
    * [Mapper 정의하기](#mapper-정의하기)
    * [Mapper에서 SQL을 정의하는 방법](#mapper에서-SQL을-정의하는-방법)
        * [간단한 POJO 객체를 CRUD 하는 방법](#간단한-pojo-객체를-crud-하는-방법)
            * [SELECT, INSERT, UPDATE, DELETE 태그](#select-insert-update-delete-태그)
            * [SELECT 시 각 필드를 매핑하는 데 사용되는 태그](#select-시-각-필드를-매핑하는-데-사용되는-태그)
        * [좀 더 생산성을 높이는 방법](#좀-더-생산성을-높이는-방법)
        * [매핑 객체가 1-depth POJO가 아닌 경우](#매핑-객체가-1-depth-pojo가-아닌-경우)
    * [Mybatis 수준에서 캐시하는 방법 (작성중)](#mybatis-수준에서-캐시하는-방법)
    * [Mybatis의 동적 SQL: 조건문 (작성중)](#mybatis의-동적-sql-조건문)
    * [Mybatis에서 사용되는 객체에 대하여 (작성중)](#mybatis에서-사용되는-객체에-대하여)
* [Mybatis 처음부터 끝까지 직접 설정하기 (작성중)](#mybatis-처음부터-끝까지-직접-설정하기)
    
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

Mybatis **자체**의 설정은 XML 설정만을 지원한다. 이번 챕터는 Mybatis의 설정만을 다룬다.

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

- Mybatis-Spring은 Bean으로 요소들을 등록하기 때문에 아래 방식으로 environment, dataSource,transactionManager를 사용할 수 없다. (무시된다. 공식 문서 참고) Spring과의 연동을 원하는 경우 참고만 하고 넘어가면 된다.

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

    * `environtments`는 환경을 구분하는 데 사용되는 Tag이다. `environments` 태그에선 반드시 `default` 값이 필요하다.

    * `environment`는 각 환경을 정의하며, `id`를 필수적으로 정의해야 한다.

    * `transcationManager`는 Mybatis가 제공하는 TransactionManager를 설정하는 Tag로 `type`을 필수적으로 지정해야 한다. Mybatis는 `type`으로 `JDBC`와 `MANAGED`를 지원한다. `JDBC`는 JDBC의 트랜잭션을 Mybatis가 처리하게 되고, `MANAGED`는 Container에게 맡기고 (`CMT(Container-Managed Transaction`) connection을 close하는 것 이외에 아무것도 하지 않는다. - `MANAGED` 사용 시 공식 문서 참고

    * `dataSource`는 JDBC Connection 객체를 생성하는 방법을 정의하는 `type`을 필수적으로 정의해야 한다. `POOLED`와 `UNPOOLED`, `JNDI`가 올 수 있다. `UNPOOLED`, `JNDI` 사용 시 공식문서를 참고하기 바란다. `POOLED`의 경우 pooling을 수행하는 것으로, 옵션이 굉장히 많다. 

        - `poolMaximumActiveConnections = 10` – 주어진 시간에 존재할 수 있는 활성화된(사용중인) 커넥션의 수.
        - `poolMaximumIdleConnections` – 주어진 시간에 존재할 수 있는 유휴 커넥션의 수

        - `poolMaximumCheckoutTime = 20000(ms, 20초)` – 강제로 리턴되기 전에 풀에서 “체크아웃” 될 수 있는 커넥션의 시간.

        - `poolTimeToWait = 20000(ms, 20초)` – 풀이 로그 상태를 출력하고 비정상적으로 긴 경우 커넥션을 다시 얻을려고 시도하는 로우 레벨 설정.

        - `poolPingQuery` – 커넥션이 작업하기 좋은 상태이고 요청을 받아서 처리할 준비가 되었는지 체크하기 위해 데이터베이스에 던지는 핑쿼리(Ping Query). 디폴트는 “핑 쿼리가 없음” 이다. 이 설정은 대부분의 데이터베이스로 하여금 에러메시지를 보게 할수도 있다.

        - `poolPingEnabled = false` – 핑쿼리를 사용할지 말지를 결정. 사용한다면, 오류가 없는(그리고 빠른) SQL을 사용하여     - `poolPingQuery` 프로퍼티를 설정해야 한다.

        - `poolPingConnectionsNotUsedFor = 0` – `poolPingQuery`가 얼마나 자주 사용될지 설정한다. 필요이상의 핑을 피하기 위해 데이터베이스의 타임아웃 값과 같을 수 있다. 디폴트 값은 `poolPingEnabled`가 `true`일 경우에만, 모든 커넥션이 매번 핑을 던지는 값이다. 

    * `mappers`는 Mapper를 감싸는 Tag 이다.

    * `mapper`는 `resource`값을 필수적으로 지정해야 한다. 해당 값은 XML의 위치를 나타낸다. classpath 상대주소로 XML에 접근하거나, URL에 접근하는 방식, Mapper 인터페이스를 명시하는 방식, 패키지 명시 방식이 있다.

## Mybatis-Spring이란

Mybatis-Spring은 Mybatis를 Spring에 매끈하게 통합하는 라이브러리입니다. 이 라이브러리는 Mybatis가 Spring Transanction에 참여할 수 있게하고, Mybatis Mapper과 SqlSession 객체의 생성과 이들을 다른 객체에 주입하는 일, Mybatis의 Exception을 Spring의 Exception으로 변환하는 등의 일을 수행합니다. 

### Mybatis-spring 설치

```xml
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>1.3.2</version>
</dependency>
```

### Mybatis-Spring의 설정 방법

Spring 프레임워크 상에서 Mybatis를 사용하려면, Spring Application Context에 `SqlSessionFactory` 빈과 최소 하나의 `Mapper` 인터페이스가 존재해야합니다. (Mapper 인터페이스는 Annotation, XML 방식 어느 것을 사용하더라도 존재해야 함.) 

1. SqlSessionFactory 빈 등록

    Mybatis에선 `sqlSessionFactory` 객체를 `SqlSessionFactoryBuilder` 객체로 생성하지만, Mybatis-Spring에서는 `SqlSessionFactory` 객체를 Spring의 FactoryBean인 `SqlSessionFactoryBean` 객체를 Bean으로 등록하게 되므로 Factory의 빈은 아래와 같이 XML 설정을 해야 합니다. (참고: FactoryBean은 `getObject()` 메소드로 반환한 객체가 Spring에서 빈으로 최종 생성됨)

    ```xml
    <!-- class를 SqlSessionFactoryBean을 사용함 -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!-- sqlSessionFactory에 dataSource 주입 -->
        <property name="dataSource" ref="dataSource" />
        
        <!-- Mybatis XML config 파일이 전달될 수 있음 -->
        <!-- 이 때 config 파일 내용은 완전한 Mybatis 설정일 필요가 없음 -->
        <!-- 오히려, environtment, dataSource, transactionManager는 무시됨 -->
        <!-- 해당 config에선 settings, typeAliases, mappers 등을 정의할 수 있음 -->
        <property name="configLocation" value="..." />

        <!-- Mybatis XML mapper 파일들의 경로를 지정함 (Ant 패턴 사용) -->
        <!-- 이 방식을 사용하면, SqlSessionTemplate을 사용해서 실행하게 된다. -->
        <!-- 이 방식을 사용하는 경우, Mapper를 직접 @Autowired 받을 수 없다. -->
        <property name="mapperLocations" value="classpath*:sample/cfg/mappers/**/*.xml" />
    </bean>

    <!-- 1.3.0 버전부터 configuration 프로퍼티가 추가되었음. -->
    <!-- Configuration 객체를 지정하는 것으로, Mybatis Config 파일을 대체할 수도 있음. -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource" />
        <property name="configuration">
            <!-- Configuration 구현체 등록 -->
            <bean class="org.apache.ibatis.session.Configuration">
                <property name="mapUnderscoreToCamelCase" value="true"/>
            </bean>
        </property>
    </bean>
    ```

2. Mapper 빈 등록

    * Mybatis-spring 라이브러리 사용 시에는 직접 SqlSession을 사용하지 않고도, Mapper 객체를 Spring에서 `@Autowired`로 의존성 주입을 받을 수 있다. (이 객체는 `org.apache.ibatis.binding.MapperProxy` 프록시 객체이다.)
    
    * 위에서 빈으로 등록한 `sqlSessionFactory`에 `mapperLocations`으로 mapper XML 설정을 반드시 완료해야 한다. `MapperProxy` 객체에서 `sqlSession` 객체에 대한 참조를 갖고, `sqlSession` 객체는 `configuration` 객체에서 mapper 설정을 보유하기 때문이다.

    * 당연하게도 Mapper 객체를 주입받으려면, Spring Context에 빈으로 등록해야 한다. Mybatis-Spring에서는 한꺼번에 등록하는 방식을 지원한다. `MapperFactoryBean`을 통해서 1개씩 등록할 수도 있는데, 권장하진 않는다.

    * 한꺼번에 등록하기 위해, Scan 대상의 basePackage를 지정하게 된다. package 값에는 Ant 패턴을 사용할 수 있다.

    * 사용할 수 있는 옵션이 3개가 있다.

        - `basePackage`는 패키지 아래에 있는 모든 클래스를 등록하며, comma, semicolon으로 구분한다.

        - `annotation`은 해당 어노테이션이 붙어있는 클래스만 등록한다.

        - `markerInterface`는 해당 마커 인터페이스를 상속한 인터페이스만 등록한다. annotation 옵션과 동시에 사용가능하며 OR 조건으로 기능한다.

    1. XML방식: `<mybatis:scan base-package="" />` 사용

        ```xml
        <beans xmlns="http://www.springframework.org/schema/beans"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:mybatis="http://mybatis.org/schema/mybatis-spring"
            xsi:schemaLocation="
            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
            http://mybatis.org/schema/mybatis-spring http://mybatis.org/schema/mybatis-spring.xsd">
        
            <!-- 실제 인터페이스가 존재하는 패키지이다. XML 설정 파일의 경로가 아니다. -->
            <mybatis:scan base-package="org.mybatis.spring.sample.mapper" />

        </beans>
        ```

    2. 어노테이션 방식: `@MapperScan("")` 사용

        `@Configuration` 어노테이션이 붙은 Config 목적의 클래스에 붙여야 한다.

        ```java
        @Configuration
        @MapperScan("org.mybatis.spring.sample.mapper")
        public class AppConfig {
            // 이 Config 빈에선 주로 JavaConfig로 dataSource와 transactionManager를 정의한다.
        }
        ```

    3. `MapperScannerConfiguerer` 빈을 Spring Application Context에 등록해 사용

        ```xml
        <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
            <property name="basePackage" value="org.mybatis.spring.sample.mapper" />
            <property name="annotationClass" value="ex.annotation.Mapper"/>
            <property name="markerInterface" value="ex.marker.Mapper"/>
        </bean>
        ```

    4. Mapper를 하나씩 등록하는 `MapperFactoryBean` 정의하는 방법

        ```xml
        <!-- 상속용으로 사용하기 위해 abstract=true로 설정했다. -->
        <bean id="baseMapper" class="....MapperFactoryBean" abstract="true" lazy-init="true">
            <property name="sqlSessionFactory" ref="sqlSessionFactory" />
        </bean>

        <!-- Spring Bean 정의 시 parent 속성을 이용하여 속성을 상속받을 수 있다. -->
        <!-- parent 속성 사용 시 property 모두를 상속받고 따라서 sqlSesionFactory를 주입받는다. -->
        <!-- parent 속성 사용 시 class를 정의하지 않은 경우 parent의 class를 사용하게 된다. -->
        <!-- class 속성 말고도, constructor, init-method, destroy-method도 마찬가지이다. -->
        <!-- lazy-init, depends-on, autowire-mode, dependency-check, scope는 상속하지 않는다. -->
        <bean id="oneMapper" parent="baseMapper">
            <property name="mapperInterface" value="my.package.MyMapperInterface" />
        </bean>

        <bean id="anotherMapper" parent="baseMapper">
            <property name="mapperInterface" value="my.package.MyAnotherMapperInterface" />
        </bean>
        ```

### Mybatis-Spring을 썼을 때의 장단점



## 공통으로 적용되는 내용

Mybatis-spring를 사용하더라도 동일한 작업을 해야 하는 것들에 대한 내용입니다. Spring에 비의존적인 부분을 다룹니다.

### Mapper 정의하기

* Mybatis는 SQL 작성과 리턴타입(직접 Mapping을 정의할 수도 있음. 명시하지 않을 시 자동 매핑), 캐시 여부만 지정한다면 질의 이후 결과 객체를 받는 것까지의 기능이 구현된다. Mapper를 정의하는 것은 XML과 어노테이션(그냥 Mybatis에서도 가능) 모두 사용할 수 있다. 

* XML로 Mapper를 정의하는 방법
    
    한 개의 매퍼 XML 파일에는 많은 수의 매핑 구문을 정의할 수 있다. 아래는 `org.mybatis.example.BlogMapper` 네임스페이스에서 selectBlog라는 매핑 구문을 정의한 것이다. 아래 Select는 `org.mybatis.example.BlogMapper.selectBlog` 형태로 실행할 수 있게 된다.

    ```xml
    <mapper namespace="org.mybatis.example.BlogMapper">
        <select id="selectBlog" resultType="Blog">
            SELECT * FROM blog WHERE id = #{id}
        </select>
    </mapper>
    ```

* 어노테이션으로 Mapper를 정의하는 방법

    인터페이스로 메소드를 정의하고, 각 메소드 위에 어노테이션으로 SQL을 정의하는 방법이다. (추가 설정 없음)

    ```java
    package org.mybatis.example;

    public interface BlogMapper {
        @Select("SELECT * FROM blog WHERE id = #{id}")
        Blog selectBlog(int id);
    }
    ```

* Mapper에 정의된 질의 실행 방법
    
    첫 번째 방법

    ```java
    // select 예시이다.
    Blog blog = (Blog) session.selectOne("org.mybatis.example.BlogMapper.selectBlog", 101);
    ```
    
    두 번째 방법

    ```java
    BlogMapper mapper = session.getMapper(BlogMapper.class);
    Blog blog = mapper.selectBlog(101);
    ```

    두 번째 방법은 Magic Constant(namespace 문자열)에 의존하지 않아 더 클린 코드이며
     리턴타입에 대해 타입 캐스팅을 하지 않는 장점을 가진다. 

* XML vs Annotation

    둘 중 특정 방식이 낫다고 할 순 없다. 매핑된 구문을 일관된 방식으로 정의하는 것이 중요하다. 어노테이션은 XML로 쉽게 변환할 수 있고 그 반대의 형태 역시 쉽게 처리할 수 있다.

### Mapper에서 SQL을 정의하는 방법

#### 간단한 POJO 객체를 CRUD 하는 방법

##### SELECT, INSERT, UPDATE, DELETE 태그

* `<select>` 태그

    아래와 같은 XML 설정을 `int(or Integer)`를 파라미터로 받아 `HashMap`을 반환하는 `selectPerson` 문이라고 한다. 이 때 key는 column명이고, value는 각 column에 대응되는 값이 된다. 당연하게도, hashmap이므로 해당 SELECT의 결과는 1 row 여야 한다. 객체 내에 Primitive(or Wrapper) Type만 있는 경우 아래 정도의 SELECT문을 사용하기만 해도, 클래스의 필드명과 결과 테이블의 컬럼명만 일치한다면 자동 매핑된다. ([camelCase의 경우 이 곳 참고](#camelCase))

    ```xml
    <select id="selectPerson" parameterType="int" resultType="hashmap">
        SELECT * FROM person WHERE id = #{id}
    </select>
    ```

    위의 SELECT 쿼리문에서 사용된 `#{id}`는 `PreparedStatement`를 사용하라고 지정하는 것이다. 즉, 아래와 같은 구현 코드가 Mybatis 내부적으로 사용되게 된다.

    ```java
    // JDBC로 따지면 아래와 비슷한 코드일 것이다. (실제 구현 코드는 아님.)
    String selectPerson = "SELECT * FROM person WHERE id=?";
    PreparedStatement ps = conn.prepareStatement(selectPerson);
    ps.setInt(1,id);
    // ...
    ```

    * SELECT문은 이후 `sqlSession`에서 호출하게 된다. 이 때 파라미터는 하나만 전달할 수 있는데, 그 때문에 `#{propName}` 사용 시 전달한 타입의 클래스 여부에 따라 값이 바인딩되는 전략이 다르다.
        1. 클래스가 아닌 경우 - 값 그대로 사용 - 이 때 `propName`은 아무 값이나 가능하다.
        2. 클래스의 경우 - 값를 객체 내에서 찾아 사용 - `propName`은 필드명과 동일해야 한다.

    * Parameter 설정 시 사용할 수 있는 기호는 2가지가 있다. `$`과 `#`이다.
        1. `$`는 SQL문에서 값을 `replace`하는 방식으로 질의한다. 이 때 escape은 자동으로 처리되지 않아 SQL Injection에 대한 방어가 수행되지 않기 때문에 권장되는 옵션이 아니다. 
        2. `#`은 PreparedStatement의 값으로 전달한다.

    `<select>`태그에 사용될 수 있는 속성은 아래와 같이 많다.

    ```xml
    <select
        id="selectPerson"
        parameterType="int"
        parameterMap="deprecated"
        resultType="hashmap"
        resultMap="personResultMap"
        flushCache="false"
        useCache="true"
        timeout="10000"
        fetchSize="256"
        statementType="PREPARED"
        resultSetType="FORWARD_ONLY">
    ```

    `id` - 같은 namespace 내(ex: `org.mybatis.example.BlogMapper`)에서 해당 구문(`<select>` 등의 구문을 정의한 것)을 식별할 때 사용되는 문자열.

    `parameterType`, `parameterMap` - 사용하지 않는다.

    `resultType` - 패키지명까지 명시된 클래스명, 혹은 클래스에 대한 별칭(`alias`)을 값으로 사용한다. 해당 클래스로 SELECT문의 결과가 [자동으로 매핑](#auto-mapping)되게 된다. 만약 1 row가 아닌 여러 row의 결과가 나오는 경우, `<select>`가 아닌 `<collection>`을 사용해야 한다.

    `resultMap` - `resultMap`의 ID값을 지정하여 결괏값이 해당 resultMap에서 정의한대로 생성되게 한다. 복잡한 객체를 SELECT를 통해 만드는 경우, (특히 JOIN 사용 시) resultMap을 사용하는 것이 권장된다.

    `flushCache`, `useCache` - 캐시에 관련된 내용은 공식 문서 참고 바람.

    `timeout`, `fetchSize`, `statementType`, `resultSetType`, `databaseId`, `resultSets`, `resultOrdered`의 경우 자주 사용하지 않으므로 공식 문서 참고 바람.

* `<insert>`, `<update>`, `<delete>` 태그

    `INSERT`, `UPDATE`는 서로 동일한 속성을, `DELETE`는 `SELECT`보다도 다소 적은 속성을 갖는다. 아래는 XML로 각 구문을 정의하는 예제이다.

    ```xml
    <insert id="insertAuthor">
        INSERT INTO author (id,username,password,email,bio)
        VALUES (#{id},#{username},#{password},#{email},#{bio})
    </insert>

    <update id="updateAuthor">
        UPDATE author SET
            username = #{username},
            password = #{password},
            email = #{email},
            bio = #{bio}
        WHERE id = #{id}
    </update>

    <delete id="deleteAuthor">
        DELETE FROM author WHERE id = #{id}
    </delete>
    ```

    이 태그들이 가질 수 있는 속성은 아래와 같다. `id`, `parameterType`, `flushCache`, `statementType`, `timeout`, `databaseId` 등의 `<select>`와 동일하다. INSERT와 UPDATE에는 `useGeneratedKeys`, `keyProperty`, `keyColumn` 등의 추가적인 속성이 있다. 전체적인 속성의 수는 SELECT보다 꽤 적다.

    ```xml
    <insert
        id="insertAuthor"
        parameterType="domain.blog.Author"
        flushCache="true"
        statementType="PREPARED"
        keyProperty=""
        keyColumn=""
        useGeneratedKeys=""
        timeout="20">

    <update
        id="updateAuthor"
        parameterType="domain.blog.Author"
        flushCache="true"
        statementType="PREPARED"
        timeout="20">

    <delete
        id="deleteAuthor"
        parameterType="domain.blog.Author"
        flushCache="true"
        statementType="PREPARED"
        timeout="20">
    ```

    `useGeneratedKeys` - `INSERT`, `UPDATE`에서만 사용할 수 있는 속성으로, JDBC의 `getGeneratedKeys` 메소드를 사용하여 Database(My or MSSQL)에서 생성된 key를 가져와 사용한다.
    
    `keyProperty` - `INSERT`, `UPDATE`에서만 사용할 수 있는 속성으로, `getGeneratedKeys` 옵션 사용시 반환되는 Key 값을 value로 사용할 column명을 지정한다.

    `keyColumn` - `INSERT`, `UPDATE`에서만 사용할 수 있는 속성으로, id로 사용되는 column명을 지정하는 데 사용된다. PostgreSQL과 같은 일부 DBMS에서만 필요하다. 복합키인 경우 ,로 구분한다.

    INSERT를 여러번 해야 할 때, 아래와 같이 `<foreach>` 태그를 사용할 수 있다.

    ```xml
    <insert id="insertAuthor" useGeneratedKeys="true" keyProperty="id">
        INSERT INTO author (username, password, email, bio) VALUES
        <foreach item="item" collection="list" separator=",">
            (#{item.username}, #{item.password}, #{item.email}, #{item.bio})
        </foreach>
    </insert>
    ```

    `<selectKey>` - `<insert>`내에서 사용하는 태그로, Key를 데이터베이스에서 조회하여 가져와야 할 경우 사용한다. 공식 문서를 참고바란다.
    
* `3.2.4` 버전부터의 `parameterType`의 생략에 대하여

    Mybatis 3.2.4 버전부터 parameter를 자동으로 추론하기 때문에 해당 속성이 무시된다. (한글 사용자 가이드에서는 소개돼있지 않음. 영문 가이드 참고.)

##### SELECT 시 각 필드를 매핑하는 데 사용되는 태그

* `<id>`, `<result>` 태그

    이 태그들은 하나의 **단순 타입** (Date, String, int, ... 등) 속성을 객체의 속성으로 매핑한다. `id`와 `result`의 차이는 두 객체를 비교할 때 `id`로 선언된 속성이 사용된다는 점이다. `id` 속성을 명시함으로써 캐시와 JOIN 성능이 향상된다고 한다.

    `property`는 자바 객체의 필드명을 의미한다.

    `column`은 결과 테이블의 속성명을 의미한다.

    `javaType`은 생략해도 되지만, HashMap일 경우 명시해야 한다.

* `<constructor>` 태그

    어떤 생성자를 선택할지 표현하는 태그이다. 아래의 XML 설정 예제를 참고하라.

    ```xml
    <constructor>
        <idArg column="id" javaType="int"/>
        <arg column="username" javaType="String"/>
        <arg column="age" javaType="_int"/>
    </constructor>
    ```

    위 XML 설정에 의해 아래의 생성자가 선택된다.

    ```java
    public class User {
        // 해당 생성자가 선택됨
        public User(Integer id, String username, int age) {
            //...
        }
        //...
    }
    ```

    생성자에 많은 매개변수가 전달되는 경우, 생성자 선택 측면에서 오류 발생 가능성이 높기 때문에, 3.4.3 버전부터는 각 매개변수 변수의 이름을 명시하면 매개변수 순서를 무시할 수 있다. 단 이 때는 각 매개변수에 `@Param` 어노테이션을 붙이거나 compile 옵션에 `-parameters`를 추가해야 한다.

#### 좀 더 생산성을 높이는 방법
    
* 재사용 가능한 SQL 코드 조각을 정의 - `<sql>` 태그 사용

    ```xml
    <!-- User 객체를 매핑하는 SQL -->
    <!-- alias라는 매개변수를 <property> 태그로 전달받는다. -->
    <sql id="userColumns"> ${alias}.id,${alias}.username,${alias}.password </sql>

    <!-- User 객체를 실제로 사용하는 SELECT문 -->
    <!-- 2명의 유저 객체를 조회한다 -->
    <select id="selectUsers" resultType="map">
        SELECT
            <!-- <sql>을 사용할 때는 <include refid="id">와 같이 사용한다. -->
            <!-- <property>로 "alias"의 value로 t1, t2를 각각 전달한다. -->
            <include refid="userColumns"><property name="alias" value="t1"/></include>,
            <include refid="userColumns"><property name="alias" value="t2"/></include>
        FROM some_table t1
        CROSS JOIN some_table t2
    </select>
    ```

* 타이핑을 줄이는 방법 - `typeAlias` 정의

    ```xml
    <!-- Mybatis XML 설정파일에서 -->
    <typeAliases>
        <typeAlias type="com.someapp.model.User" alias="User"/>
    </typeAliases>

    <!-- Mapper XML파일에서 -->
    <select id="selectUsers" resultType="User">
        SELECT id, username, hashedPassword
        FROM some_table
        WHERE id = #{id}
    </select>
    ```

    적용 전/후

    ```xml
    <!-- 적용 전: Fully Qualified Name를 사용한다. -->
    <select id="selectUsers" resultType="com.someapp.model.User" ... />
    
    <!-- 적용 후: 별칭을 사용한다. -->
    <select id="selectUsers" resultType="User" ... />
    ```

* <a name="auto-mapping">자동 매핑 설정</a>

    Mybatis에서 resultMap을 명시하지 않아도 resultType을 명시하면 해당 객체의 속성을 참고하여 필드와 속성명이 같은 필드에 대해 자동으로 값을 전달하게 된다. <a name="camelCase"></a>이 때 `lowercase with underscore -> camelCase`를 감지하여 매핑되기하려면, `mapUnderscoreToCamelCase` 속성을 `true`로 지정하면 된다. (기본으로 적용되지 않으므로 명시적으로 지정해야 함!)

    참고로 자동 매핑은 resultMap을 정의했을 때에도 작동하는데, `<result>` 등의 태그로 매핑되지 않은 속성 중 필드와 일치하는 컬럼의 경우 매핑된다. 이는 `auto-mapping=PARTIAL`(이게 기본값이다.)의 특징인데, `NONE`의 경우 아예 자동 매핑을 사용하지 못하니 문제이고, `FULL`의 경우, JOIN 등의 경우에 필드 이름이 겹치는 경우 자동 매핑 때문에 잘못된 값을 매핑할 수 있어 주의하여 사용해야 하는 옵션이다. 더 나은 방법으로, `<resultMap>`에 `autoMapping="false"` 속성을 주는 것이 낫다.

#### 매핑 객체가 `1-depth` POJO가 아닌 경우

* `<associaton>` 태그

    1:1 관계의 객체를 불러올 때 사용한다. 동시에 (이후 나오는) `resultMap`이다. 
    
    `property` 속성으로 대응하는 필드명을 받고, `javaType` 속성으로 타입을 받는다(생략가능).

    `columnPrefix` 속성으로 prefix 값을 명시할 수 있다. 결과 테이블에서 필드로 매핑 시 해당 값이 붙어있는 속성을 사용하게 된다. 주로 JOIN 시 속성명이 곂치고, 따로 resultMap을 정의해서 재사용할 때 사용하게 된다. 권장하는 옵션이다.

    아래는 `columnPrefix` 사용 시의 SQL 예시이다.

    ```sql
    <select id="selectBlog" resultMap="blogResult">
        SELECT
            B.id AS blog_id,
            B.title AS blog_title,
            B.author_id AS blog_author_id,
            P.id AS post_id,
            P.subject AS post_subject,
            P.body AS post_body,
        FROM Blog B
        LEFT OUTER JOIN Post P ON B.id = P.blog_id
        WHERE B.id = #{id}
    </select>
    ```

    아래는 위의 SQL에 대응하는 resultMap 정의이다.

    ```xml
    <resultMap id="blogResult" type="Blog">
        <id property="id" column="blog_id" />
        <result property="title" column="blog_title"/>
        <collection property="posts" ofType="Post" resultMap="blogPostResult" columnPrefix="post_"/>
    </resultMap>

    <resultMap id="blogPostResult" type="Post">
        <id property="id" column="id"/>
        <result property="subject" column="subject"/>
        <result property="body" column="body"/>
    </resultMap>
    ```

    `resultMap` 속성으로 이미 정의된 resultMap을 참조할 수도 있다.

    `fetchType`이라는 속성을 명시하여 `lazy`, `eager` 로딩 방식을 선택할 수 있다. 해당 속성에 넘겨진 값은 전역 속성인 `lazyLoadingEnabled`보다 우선한다. `lazy` 방식 선택시에 ORM에서 겪게 되는 `OSIV` 이슈가 없다 :smile:

    `select` 속성으로 정의된 `<select>`를 재활용할 수 있다. 만약 key가 다중 컬럼인 경우 `column="{prop1=col1, prop2=col2}"`와 같이 명시할 수 있다. 해당 속성 사용 시 객체 매핑 시에 Select가 각각 실행된다. 

    ```xml
    <!-- association의 select 속성 사용 예시 -->
    <resultMap id="blogResult" type="Blog">
        <association property="author" column="author_id" javaType="Author" select="selectAuthor"/>
    </resultMap>

    <!-- association 대상 객체를 갖는 Blog 객체의 SELECT 문 -->
    <select id="selectBlog" resultMap="blogResult">
        SELECT * FROM BLOG WHERE ID = #{id}
    </select>

    <!-- association 대상 객체의 SELECT 문 -->
    <select id="selectAuthor" resultType="Author">
        SELECT * FROM AUTHOR WHERE ID = #{id}
    </select>
    ```

    `notNullColumn` 속성은 값으로 전달된 속성이(여러 개라면 ,를 사용) `null`인 경우 대상 객체를 생성하지 않는다.

* `<collection>` 태그

    collection 태그는 association 태그와 거의 동일한 동작과 속성을 가진다.

    collection 사용 시 `ofType`으로 타입을 명시해야한다.

    아래는 `select` 쿼리를 명시한 collection 예시이다. (미사용 시는 association과 너무 동일하여 기재하지 않음.)

    ```xml
    <!-- javaType 속성은 추론이 가능하므로 필요 없다. 명시 하지말 것! -->
    <collection property="posts" javaType="ArrayList" column="id" ofType="Post" select="selectPostsForBlog"/>
    ```
    
    `<collection>` 사용 시 `select` 속성을 사용하면 SQL이 굉장히 간단해지는 대신, 1:N인 관계를 표현하는 이상, N개의 대상이 존재하게 되며, (예: `List<Author> authors`) 조회가 각각의 N번 더 실행되는 `1+N(혹은 N+1) 문제`를 겪게 된다. 따라서 `collection`의 경우는 JOIN을 사용하는 것을 추천한다. (`association`의 경우는 `select` 옵션을 사용하더라도 최대 1회만 더 호출되므로 덜 영향을 받는다.)

* `<discriminator>` 태그

    설명은 생략한다.

    아래는 예시이다.

    ```xml
    <!-- 예시 1 -->
    <discriminator javaType="int" column="draft">
        <case value="1" resultType="DraftPost"/>
    </discriminator>

    <!-- 예시 2 -->
    <!-- Vehicle 객체를 조회하는 SELECT -->
    <!-- discriminator의 case를 모두 만족하지 않는 경우 Vehicle 객체가 반환된다. -->
    <!-- vehicle_type 값에 따라 discriminator가 다른 resultMap을 선택하게 된다. -->
    <!-- 다른 resultMap이 선택된 경우 Vehicle 필드의 매핑은 모두 취소된다. -->
    <resultMap id="vehicleResult" type="Vehicle">
        <id property="id" column="id" />
        <result property="vin" column="vin"/>
        <result property="year" column="year"/>
        <result property="make" column="make"/>
        <result property="model" column="model"/>
        <result property="color" column="color"/>
        <discriminator javaType="int" column="vehicle_type">
            <case value="1" resultMap="carResult"/>
            <case value="2" resultMap="truckResult"/>
            <case value="3" resultMap="vanResult"/>
            <case value="4" resultMap="suvResult"/>
        </discriminator>
    </resultMap>

    <!-- discriminator에 의해 결정된 타입의 resultMap 정의 -->
    <!-- Car를 반환하는 경우, doorCount 필드 외에 다른 필드는 매핑되지 않는다. -->
    <resultMap id="carResult" type="Car">
        <result property="doorCount" column="door_count" />
    </resultMap>

    <!-- 만약 Car가 선택되더라도 Vehicle의 필드까지 매핑되게 하려면 아래와 같이 extends 옵션으로 resultMap을 상속해야 한다. -->
    <resultMap id="carResult" type="Car" extends="vehicleResult">
        <result property="doorCount" column="door_count" />
    </resultMap>

    <!-- 위와 같이 resultMap을 선언하기에 정말 적은 수의 추가 속성만 갖는 경우에는 resultType을 사용하면 되는데, 이게 가능한 이유는 resultType을 명시하는 경우 resultMap이 자동 생성되어 모든 필드에 대응되기 때문이다. (참고: 기본 값이 자동 생성이다.) -->
    ```xml
    <resultMap id="vehicleResult" type="Vehicle">
        <id property="id" column="id" />
        <result property="vin" column="vin"/>
        <result property="year" column="year"/>
        <result property="make" column="make"/>
        <result property="model" column="model"/>
        <result property="color" column="color"/>
        <discriminator javaType="int" column="vehicle_type">
            <case value="1" resultType="carResult">
                <result property="doorCount" column="door_count" />
            </case>
            <case value="2" resultType="truckResult">
                <result property="boxSize" column="box_size" />
                <result property="extendedCab" column="extended_cab" />
            </case>
                <case value="3" resultType="vanResult">
            <result property="powerSlidingDoor" column="power_sliding_door" />
            </case>
            <case value="4" resultType="suvResult">
                <result property="allWheelDrive" column="all_wheel_drive" />
            </case>
        </discriminator>
    </resultMap>
    ```

* `<resultMap>` 정의 및 사용

    `ResultMap`은 `ResultSet`을 Mapping하는 방법을 정의한 것이다. <br />
    놀랍게도(?) `ResultMap`은 `resultType`으로 등록된 객체를 기준으로 자동 생성된다. <br />
    그러나 이 좋은 기능을 걷어차버리고 프로그래머가 직접 정의할 수도 있다. <br />
    직접 정의하는 경우 `resultMap` 속성을 사용하게 된다.

    ```xml
    <resultMap id="userResultMap" type="User">
        <id property="id" column="user_id" />
        <result property="username" column="username"/>
        <result property="password" column="password"/>
    </resultMap>
    ```

    그러나 이 정도로 간단한 객체는 자동 생성에 당연히 맡길 것이다. 따라서 실제로는 JOIN 등의 사용 시에나 resultMap 정의하게 된다. 아래는 그 예시이다.

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

        <!-- association 태그는 1:1 관계를 표현한다. -->
        <!-- association 태그는 중첩된 ResultMap이다. -->
        <!-- 이미 정의된 ResultMap의 ID를 참조시킬 수도 있다. -->
        <!-- ResultMap이므로 동일한 태그를 사용하게 된다. -->
        <!-- 중첩된 객체를 처리하는 데 사용된다. -->
        <association property="author" javaType="Author">
            <!-- id 태그는 result와 동일하나 ID임을 명시한다. 캐시와 JOIN 시 성능 향상이 있다고 한다. -->
            <id property="id" column="author_id" />
            <result property="username" column="author_username" />
            <result property="password" column="author_password" />
            <result property="email" column="author_email" />
            <result property="bio" column="author_bio" />
            <result property="favouriteSection" column="author_favourite_section" />
        </association>

        <!-- collection 태그는 1:N 관계를 표현한다. -->
        <!-- collection 태그는 Collection을 의미한다. -->
        <!-- collection 태그 또한 중첩된 ResultMap이다. -->
        <!-- 마찬가지로 이미 정의된 ResultMap의 ID를 참조시킬 수 있다. -->
        <collection property="posts" ofType="Post">
            <id property="id" column="post_id" />
            <result property="subject" column="post_subject" />
            
            <!-- 두 번째 사용 시엔 그냥 막 써도 되는건가? -->
            <association property="author" javaType="Author" />

            <collection property="comments" ofType="Comment">
                <id property="id" column="comment_id" />
            </collection>

            <collection property="tags" ofType="Tag" >
                <id property="id" column="tag_id" />
            </collection>

            <!-- column의 값을 읽어서 값에 따라 사용할 resultMap(Type)을 달리한다. -->
            <!-- 값에 따라 다른 객체를 선택할 수 있도록 case 문을 사용할 수 있다. -->
            <!-- 아래는 draft=1일 때 resultType을 DraftPost로 사용한다는 얘기이다. -->
            <discriminator javaType="int" column="draft">
                <case value="1" resultType="DraftPost" />
            </discriminator>
        </collection>
    </resultMap>
    ```

    아래는 Association Tag에서 resultMap의 ID를 사용하는 방법이다.

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

### Mybatis 수준에서 캐시하는 방법

* `<cache>`, `<cache-ref>`

    [Mybatis 3 English Doc](http://www.mybatis.org/mybatis-3/sqlmap-xml.html)의 최하단에 있는 `cache` 탭을 참고하기 바란다.

### Mybatis의 동적 SQL: 조건문



### Mybatis에서 사용되는 객체에 대하여


## Mybatis 처음부터 끝까지 직접 설정하기
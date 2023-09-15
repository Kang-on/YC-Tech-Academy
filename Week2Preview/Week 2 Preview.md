# Week 2 Preview

# Annotation?

코드 사이에 주석처럼 쓰이면서 특별한 의미나 기능을 수행하도록 하는 기술

데이터를 위한 데이터(실제 데이터 X) → Called `meta data`

용도

- 컴파일러에게 코드 작성 문법 에러 체크하도록 정보 제공
- 빌드나 배치 시 코드를 자동으로 생성할 수 있도록 정보 제공
- 실행 시(런타임 등) 특정 기능 실행하도록 정보 제공

## @ComponentScan

@Component, @Service, @Repository, @Controller, @Configuration Annotation이 붙은 클래스 Bean들을 찾아서 Context에 bean등록을 해 주는 Annotation

## @Component

개발자가 직접 작성한 Class를 Bean으로 등록하기 위한 Annotation

@ComponentScan에 의해 스캔될 때, 주어진 패키지 내에서 @Component가 적용된 클래스 식별 → 클래스의 Bean을 생성하여 ApplicationContext에 등록함

![Untitled](Week%202%20Preview%209323551c4f914f0aa58fb7350392b3ad/Untitled.png)

@Controller, @Service, @Repository Annotations → @Component 어노테이션의 구체화도니 형태

## @Bean?

개발자가 직접 제어가  불가능한 외부 라이브러리 등을 Bean으로 만들려고 할 떄 사

= Spring IoC 컨테이너에 의해 생성되고 관리되는 Java 객체

@Bean과 같이 알아야 할 @Configuration

@Configuration : 해당 클래스에서 1개 이상의 Bean을 생성하고 있음을 명시

→ @Bean 어노테이션을 사용하는 클래스는 반드시 @Configuration과 함께 사용되어야 

## Bean 주입

1. 생성자 기반 주입

```java
public calss Simple1{
	private Simple2 simple2;

	public Simple1(Simple2 simple2){
		this.simple2 = simple2;
	}
}
```

## @Bean vs @Component

@Bean 

- 개발자의 컨트롤이 불가능한 외부 라이브러리들을 Bean으로 등록하고 싶은 경우
- Method 레벨에서 선언
- Setter나 Builder등을 통해 사용자가 프로퍼티를 변경해서 생성한 인스턴스를 스프링에게 관리를 맞기는 것(개발자가 수동으로)

```java
**@Configuration**
public class AppConfig {
   **@Bean**
   public MemberService memberService() {
      return new MemberServiceImpl();
   }
}
```

@Component

- 개발자의 컨트롤이 가능한 Calss들의 경우
- Class레벨에서 선언
- 클래스를 스프링더러 알아서 인스턴스 생성 후 등록(bean으로 하라고) 맡기는 것(componentscan으로 알아서)

```java
**@Component**
public class Utility {
   // ...
}
```

Then, 개발자가 생성한 Calss에서 @Bean 선언이 가능한가??

NO

→ @Bean과 @Component는 각자 선언할 수 있는 타입이 정해져 있음

## Field Injection vs Constructor Injection

Spring Framework에서 의존성을 주입하는 방법

1. 생성자 주입 (Constructor Injection)
    
    ```java
    **@Service**
    public class StudentServiceImpl implements StudentService {
    
        private final CourseService courseService;
    
        **@Autowired**
        public StudentServiceImpl(CourseService courseService) {
            this.courseService = courseService;
        }
    
        **@Override**
        public void studentMethod() {
            courseService.courseMethod();
        }
    }
    ```
    
2. 필드 주입 (Field Injection)
    
    ```java
    **@Service**
    public class StudentServiceImpl implements StudentService {
    
        **@Autowired**
        private CourseService courseService;
    
        **@Override**
        public void studentMethod() {
            courseService.courseMethod();
        }
    
    }
    ```
    
3. 수정자 주입(Setter Injection)
    
    ```java
    **@Service**
    public class StudentServiceImpl implements StudentService {
    
        private CourseService courseService;
    
        **@Autowired**
        public void setCourseService(CourseService courseService) {
            this.courseService = courseService;
        }
    
        @Override
        public void studentMethod() {
            courseService.courseMethod();
        }
    }
    ```
    

Constructor Injection을 권장하는 이유

1. 순환 참조 방지
    1. A→B를 참조하면서 B→A를 참조하는 경우 발생
        
        *객체생성시점에서 순환참조가 일어나는 것 ≠ 객체생성 후 비즈니스 로직 상에서 순환참조가 일어나는 것
        
2. final 선언 가능
    1. field, setter Injection은 필드를 final로 선언할 수 없음
    
    → 나중에 변경될 수도 있다는 뜻
    
    But, Constructor Injection은 필드를 final로 선언할 수 있음
    
    **→ 런타임에 객체 불변성을 보장**
    
3. test code작성 용이
    1. 그냥 원하는 객체 생성해서, 생성자에 넣어주면 됨
    

## @Primary, @Qualifier

Spring Boot에서 어노테이션을 통해 자동으로 빈 생성 시, 같은 인터페이스의 구현체 클래스 두 개 이상이 빈으로 등록

→ 

<aside>
💡 **NoUniqueBeanDefinitionException: No qualifying bean of type 'test.a' available: expected single matching bean but found 2**

</aside>

: 1개의 Bean만 매칭되어야하는데 2개의 Bean이 존재한다며 오류 발생

→ 어떤 Bean을 주입해야 하는 지 알려줘야 함

```java
public interface Animal {
	String sound();
}

@Component
public class Dog implements Animal {
	public String sound() {
		return "왈왈";
	}
}
@Component
public class Cat implements Animal {
	public String sound() {
		return "야옹";
	}
}

@Service
public class AnimalService {
	private final Animal animal;
	@Autowired
	public AnimalService(Animal animal) {
			this.animal = animal;
	}
}
//위의 오류 발
```

1. Autowired된 field의 이름을 빈 이름으로 설정
    
    Spring Boot가 dog 이름과 같은 빈이 있는지 알아서 찾아 주
    
    ```java
    @Service
    public class AnimalService {
    	private final Animal dog;
    	@Autowired
    	public AnimalService(Animal animal) {
    			this.animal = animal;
    	}
    }
    ```
    
2. @Qualifier Annotation
    
    빈 이름은 그대로 있고, 추가적으로 구분자가 생기는 것
    
    ```java
    @Component
    @Qualifier("dogdog")
    public class Dog implements Animal {
    	public String sound() {
    		return "왈왈";
    	}
    }
    @Component
    @Qualifier("catcat")
    public class Cat implements Animal {
    	public String sound() {
    		return "야옹";
    	}
    }
    
    @Service
    public class AnimalService {
    	private final Animal animal;
    	@Autowired
    	public AnimalService(@Qualifier("dogdog") Animal animal) {
    			this.animal = animal;
    	}
    }
    ```
    
3. @Primary Annotation
    
    Annotation붙어잇는거 알아서 선택. Qualifier은 모든 코드에 부착. 더 간단하지
    
    ```java
    @Component
    @Primary
    public class Dog implements Animal {
    	public String sound() {
    		return "왈왈";
    	}
    }
    @Component
    public class Cat implements Animal {
    	public String sound() {
    		return "야옹";
    	}
    }
    
    @Service
    public class AnimalService {
    	private final Animal animal;
    	@Autowired
    	public AnimalService(Animal animal) {
    			this.animal = animal;
    	}
    }
    
    //source : https://bestinu.tistory.com/58
    ```
    
    Primary랑 Qualifier동시에 사용해도 됨
    
    메인 기능으로 사용하는 빈에 @Primary, 서브 기능으로 사용하는 빈에 @Qualifier사용
    
    @Primary, @Qualifier의 우선순위
    
    Spring Boot : 자동보다 수동으로 지정한 것에 높은 우선 순위 가짐
    
    → 명시적으로 지정하는 @Qualifier Annotation이 우선 순위를 가짐
    
    ## H2 Database
    
    setttteing
    
    src>main>resources>application.properties
    
    ```java
    //h2 DB생성 설정
    spring.datasource.driver-class-name=**org.h2.Driver**
    spring.datasource.url=jdbc:h2:mem:todo
    spring.datasource.username=codepresso
    spring.datasource.password=asdf1234
    
    //웹 콘솔 설정
    spring.h2.console.enabled=true
    spring.h2.console.path=/h2-console
    ```
    
    ## @Profile
    
    - Bean을 특정 profile에 매핑
    - 개발 중에만 컨테이너에 존재
    
    ```java
    @Component
    @Profile("dev")
    //@profile("!dev")로 제외도 가
    public class DevDatasourceConfig
    ```
    
    WebApplicationInitializer이용
    
    ```java
    @Configuration
    public class MyWebApplicationInitializer 
      implements WebApplicationInitializer {
    
        @Override
        public void onStartup(ServletContext servletContext) throws ServletException {
     
            servletContext.setInitParameter(
              "spring.profiles.active", "dev");
        }
    }
    ```
    
    예시
    
    ```java
    public interface DatasourceConfig {
        public void setup();
    }
    ```
    
    ```java
    //개발 환경 구성
    @Component
    @Profile("dev")
    public class DevDatasourceConfig implements DatasourceConfig {
        @Override
        public void setup() {
            System.out.println("Setting up datasource for DEV environment. ");
        }
    }
    ```
    
    ```java
    //프로덕션 환경에 대한 구성
    @Component
    @Profile("production")
    public class ProductionDatasourceConfig implements DatasourceConfig {
        @Override
        public void setup() {
           System.out.println("Setting up datasource for PRODUCTION environment. ");
        }
    }
    ```
    
    ```java
    public class SpringProfilesWithMavenPropertiesIntegrationTest {
        @Autowired
        DatasourceConfig datasourceConfig;
    
        public void setupDatasource() {
            datasourceConfig.setup();
        }
    }
    ```
    
    ```java
    Setting up datasource for DEV environment.
    ```

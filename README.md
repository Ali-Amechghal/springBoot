# Spring Boot
Spring Boot Notes , Tips &amp; Tricks



### Introduction 

	Spring comes to deal with heavyweight EJB, but spring still heavy in term of configuration and dependences management
	But Springboot eliminate theses heavy things

	Spring Boot Offre
		Automatic configuraiton : configure automaticly datasouce , template , security if the deps its present
		Starters :  starter bring all dependences you need with a correct versions
		CLI : You should install it
		The Actuator : monitoring SP Boot app

	The must important SPB annotation is @SpringBootApplication
	    @SpringBootApplicaion contains theses three annotations
		@Configuration : you can add beans using java configuration
		@ComponentScan : detect components
		@@EnableAutoConfiguration: create automaticly configuration for datasource , ...

###Testing
		@RunWith(SpringRunner.class) or @RunWith(SpringJunit4ClassRunner.class) : [SpringRunner extends SpringJunit4ClassRunner ]
		@SpringBootTest OR @SpringApplicationConfiguraiton(ConfigClass.class)
		@WebAppConfiguration

		@SpringBootTest will search for a main configuration class and load ALL application configuration

		--> Spring boot provide a maven plugin to assiste in building
    	--> to list all dependences tree : $ mvn dependences:tree

### Deps
    	You can exclude transitive deps and add your specific versions
    		<dependency>
    			<groupId></>
    			<artifactId></>
    			<exclusions>
    				<exclusion>
    					<groupId>...</>
    				</exclison>
    			</exclusions>
    		</dependency>

### AutoConfiguration
    	Springboot autoconfiguration its a runtime process (in the application startup)
    	Springboot provide a jar with all configuration classes (springboot-autoconfigure.jar)
    	The configuration are conditional it runs only if there is a deps

    	==> Conditionl
    		Creating a condition class:
```java	
    	public class JdbcTemplateCondition implements Condition {
    		@Override
    		public boolean matches (ConditionContext ctxt, AnnotatedTypeMetaData m){
    			try{
    				ctxt.getClassLoader().loadClass("...JdbcTemplate");
    				return true;
    			}catch(Exception e){
    				return false;
    			}
    		}

    	}
```
    		Using a condition :
              Create MySerice only if JdbcTemplate its present in the classpath
```java

    	@Conditional(JdbcTemplateCondition.class)
    	public MyService myService(){
    		...
    	}
```

    	Spring Provide a ready to use conditional Annotation
    		@ConditionalOnBean : if bean exists
    		@ConditionalOnMissingBean ...
    		@...........Class
    		@...........Java
    		@...........Resource
    		@...........WebApplication

For the most part, the @ConditionalOnMissingBean annotation described in
is what makes it possible to override auto-configuration. The JdbcTemplate
bean defined in Spring Boot’s DataSourceAutoConfiguration is a very simple example
of how @ConditionalOnMissingBean works:
	@Bean
	@ConditionalOnMissingBean(JdbcOperations.class)
	public JdbcTemplate jdbcTemplate() {
	return new JdbcTemplate(this.dataSource);
	}

	(the
interface that JdbcTemplate implements).

Spring Boot is designed to load application-level configuration before considering its
auto-configuration classes

 ### Externalizing configuration with properties

	When you need to adjust the settings, you can specify
these properties via environment variables, Java system properties, JNDI, commandline
arguments, or property files.


	$ java -jar readinglist-0.0.1-SNAPSHOT.jar --spring.main.show-banner=false
	Another way is to create a file named application.properties that includes the following
	line:
		spring.main.show-banner=false
	Or, if you’d prefer, create a YAML file named application.yml that looks like this:
	spring:
		main:
			show-banner: false

	$ export spring_main_show_banner=false
		NB : Note the use of underscores instead of periods and dashes, as required for environment
variable names.

	Order when to lock for configuration:

	1 Command-line arguments
	2 JNDI attributes from java:comp/env
	3 JVM system properties
	4 Operating system environment variables
	5 Randomly generated values for properties prefixed with random.* (referenced
	when setting other properties, such as `${random.long})
	6 An application.properties or application.yml file outside of the application
	7 An application.properties or application.yml file packaged inside of the application
	8 Property sources specified by @PropertySource
	9 Default properties


	As for the application.properties and application.yml files, they can reside in any
of four locations:
	1 Externally, in a /config subdirectory of the directory from which the application is run
	2 Externally, in the directory from which the application is run
	3 Internally, in a package named “config”
	4 Internally, at the root of the classpath


	if you have both application.properties and application.yml
side by side at the same level of precedence, properties in application.yml will override
those in application.properties.

	=> CONFIGURING THE EMBEDDED SERVER

	server:
	   port: 8000

	enable the server to serve securely over HTTPS. The first thing you’ll need to do is create a keystore using
the JDK’s keytool utility:
	$ keytool -keystore mykeys.jks -genkey -alias tomcat -keyalg RSA

	server:
	port: 8443
	ssl:
		key-store: file:///path/to/mykeys.jks //if you package it within the application JAR file, you should use a
classpath: URL to reference it.
		key-store-password: letmein
		key-password: letmein

	### CONFIGURING LOGGING

	By default, Spring Boot configures logging via Logback
	if you decide that you’d rather use Log4j or Log4j2, you’ll need to change your dependencies to include the appropriate starter
for the logging implementation you want to use and to exclude Logback.

	<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter</artifactId>
		<exclusions>
			<exclusion>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-starter-logging</artifactId>
			</exclusion>
		</exclusions>
	</dependency>

	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-log4j</artifactId>
	</dependency>


	in src/main/resources

	<configuration>
		<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
		<encoder>
		<pattern>
		%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n
		</pattern>
		</encoder>
		</appender>
		<logger name="root" level="INFO"/>
		<root level="INFO">
			<appender-ref ref="STDOUT" />
		</root>
		</configuration>

properties can be set in application.properties like this:
	logging.path=/var/logs/
	logging.file=BookWorm.log
	logging.level.root=WARN
	logging.level.root.org.springframework.security=DEBUG
	config.classpath=logging-config.xml

	### CONFIGURING A DATA SOURCE

	spring:
		datasource:
			url: jdbc:mysql://localhost/readinglist
			username: dbuser
			password: dbpass
			driver-class-name: com.mysql.jdbc.Driver  // optional springboot can detect that from URL

	Spring Boot will use this connection data when auto-configuring the DataSource
bean. The DataSource bean will be pooled, using Tomcat’s pooling DataSource if it’s available on the classpath. 
If not, it will look for and use one of these other connection

pool implementations on the classpath:

	■ HikariCP
	■ Commons DBCP
	■ Commons DBCP 2

You may also choose to look up the DataSource from JNDI by setting the spring.datasource.jndi-name 
	property:
	spring:
		datasource:
			jndi-name: java:/comp/env/jdbc/readingListDS

	NB : If you set the spring.datasource.jndi-name property, the other datasource connection
properties (if set) will be ignored.

		### 3.2.2 Externally configuring application beans

		@ConfigurationProperties(prefix="amazon") 
		//sepecify that the bean properties should be injected from config prperties file
public class ReadingListController {...}

	Instead you should use a separate bean for domain configuration properties, and inject this bean in ReadingListController

		@Component
	@ConfigurationProperties("amazon")
	public class AmazonProperties {...

		@Autowired
	public ReadingListController(
	ReadingListRepository readingListRepository,
	AmazonProperties amazonProperties) { ... }

	property in application.properties:
		amazon.associateId=habuma-20

		NB : amazon.associateId is equivalent to both amazon.associate_id and amazon.associate-id
		NB : if you dont use any autoconfig you should add @EnableConfigurationProperties to use @ConfigurationProperties

	==> 3.2.3 Configuring with profiles

	Profilesare a type of conditional configuration where different beans or configuration classes
are used or ignored based on what profiles are active at runtime. For instance, suppose that the security configuration we created

		@Profile("production") // securityconfig will be created if a profile is production
		@Configuration
		@EnableWebSecurity
		public class SecurityConfig extends WebSecurityConfigurerAdapter {

		spring.profiles.active=production

		==> WORKING WITH PROFILE-SPECIFIC PROPERTIES FILES

		pattern “application-{profile}.properties”. OR “application-{profile}.yml” , but un yaml you can use profile config in one file

		spring:
			profiles: development
				logging:
				level:
				root: DEBUG
			---
		spring:
			profiles: production
				logging:
					path: /tmp/
					file: BookWorm.log
					level:
					root: WARN

					========================== TESTING WITH SP-B ========================


	Spring isn’t necessarily involved in those unit tests

	Integration tests, on the other hand, require some help from Spring. If Spring is
responsible for configuring and wiring up the components in your production application,
then Spring should also be responsible for configuring and wiring up those
components in your tests.

		@RunWith(SpringJUnit4ClassRunner.class)
		@ContextConfiguration(
		classes=AddressBookConfiguration.class)
		public class AddressServiceTests { ... }


	Spring’s SpringJUnit4ClassRunner helps load a Spring application context in
JUnit-based application tests (a JUnit class runner that loads a Spring application context
for use in a JUnit test and enables autowiring of beans into the test class.) , 
	you can add @ContextConfiguration(
classes=AddressBookConfiguration.class) to specify context to load

@ContextConfiguration does a great job of loading the Spring application
context, it doesn’t load it with the full Spring Boot treatment. Spring Boot applications
are ultimately loaded by SpringApplication, either explicitly (as in listing 2.1) or using
SpringBootServletInitializer 

SpringApplication not only loads the application context, but also enables logging, the loading of external
properties (application.properties or application.yml), and other features of Spring
Boot. If you’re using @ContextConfiguration, you won’t get those features.
To get those features back in your integration tests, you can swap out @Context-
Configuration for Spring Boot’s @SpringApplicationConfiguration

		@RunWith(SpringJUnit4ClassRunner.class)
		@SpringApplicationConfiguration(
		classes=AddressBookConfiguration.class)
		public class AddressServiceTests {
		...
		}

		NB : @SpringApplicationConfiguration
loads the Spring application context using SpringApplication the same way
and with the same treatment it would get if it was being loaded in a production application.
This includes the loading of external properties and Spring Boot logging.

	==> 4.2 Testing web applications


	■ Spring Mock MVC enables controllers to be tested in a mocked approximation
	of a servlet container without actually starting an application server


	■ Web integration tests—Actually starts the application in an embedded servlet container
	(such as Tomcat or Jetty), enabling tests that exercise the application in a
	real application server
		==> starting a server : will result in a slower test than mocking a servlet container.
		==> server-based tests are closer to the real-world environment that they’ll be running in when deployed to production


			==> 4.2.1 Mocking Spring MVC


			mockMvc.perform(post("/readingList")
				.contentType(MediaType.APPLICATION_FORM_URLENCODED)
				.param("title", "BOOK TITLE")
				.param("author", "BOOK AUTHOR")
				.param("isbn", "1234567890")
				.param("description", "DESCRIPTION"))
				.andExpect(status().is3xxRedirection())
				.andExpect(header().string("Location", "/readingList"));

		==> 4.3 Testing a running application
				
		@WebIntegrationTest : start embedded server and run tests against it
		@WebIntegrationTest(value={"server.port=0"})
		@WebIntegrationTest(randomPort=true)

		First we’ll need to inject the chosen port as an instance variable. To make this convenient,
Spring Boot sets a property with the name local.server.port to the value of
the chosen port.


				@RunWith(SpringJUnit4ClassRunner.class)
				@SpringApplicationConfiguration(
				classes=ReadingListApplication.class)
				@WebIntegrationTest
				public class SimpleWebTest {
				@Value("${local.server.port}")
				private int port;


					@Test(expected=HttpClientErrorException.class)
					public void pageNotFound() {
						try {
						RestTemplate rest = new RestTemplate();
						rest.getForObject("http://localhost:{port}/bogusPage", String.class, port);
						fail("Should result in HTTP 404");
						} catch (HttpClientErrorException e) {
						assertEquals(HttpStatus.NOT_FOUND, e.getStatusCode());
						throw e;
						}
					}
				}
		==> 4.3.2 Testing HTML pages with Selenium
		Add dependecy : testCompile("org.seleniumhq.selenium:selenium-java:2.45.0")

		@RunWith(SpringJUnit4ClassRunner.class)
		@SpringApplicationConfiguration(
		classes=ReadingListApplication.class)
		@WebIntegrationTest(randomPort=true)
		public class ServerWebTests {
			private static FirefoxDriver browser;
			@Value("${local.server.port}")
			private int port;
			@BeforeClass
			public static void openBrowser() {
			browser = new FirefoxDriver();
				browser.manage().timeouts()
				.implicitlyWait(10, TimeUnit.SECONDS);
			}
			@AfterClass
			public static void closeBrowser() {
				browser.quit();
			}
			@Test
			public void addBookToEmptyList() {
				String baseUrl = "http://localhost:" + port;
				browser.get(baseUrl);
				assertEquals("You have no books in your book list",
				browser.findElementByTagName("div").getText());
				browser.findElementByName("title")
				.sendKeys("BOOK TITLE");
				browser.findElementByName("author")
				.sendKeys("BOOK AUTHOR");
				browser.findElementByName("isbn")
				.sendKeys("1234567890");
				browser.findElementByName("description")
				.sendKeys("DESCRIPTION");
				browser.findElementByTagName("form")
				.submit();
				WebElement dl =
				browser.findElementByCssSelector("dt.bookHeadline");
				assertEquals("BOOK TITLE by BOOK AUTHOR (ISBN: 1234567890)",
				dl.getText());
				WebElement dt =
				browser.findElementByCssSelector("dd.bookDescription");
				assertEquals("DESCRIPTION", dt.getText());
			}..}

	==> Spring Boot Developer Tools

	■ Automatic restart—Restarts a running application when files are changed in
		the classpath

			With the developer tools active, any changes to files on the classpath will trigger an
			application restart. To make the restart as fast as possible, classes that won’t change
			(such as those in third-party JAR files) will be loaded into a base classloader, whereas
			application code that is being worked on will be loaded into a separate restart classloader.
			When changes are detected, only the restart classloader is restarted.

			spring:
			  devtools:
				restart:
					exclude: /static/**,/templates/**

			spring:
				devtools:
					restart:
						enabled: false
			Enable when a specific file changed:
			
			spring:
				devtools:
					restart:
						trigger-file: .trigger

	■ LiveReload support—Changes to resources trigger a browser refresh
		automatically
		Spring Boot’s developer tools integrate with LiveReload
		
			spring.devtools.livereload.enabled=true/false

	■ Remote development—Supports automatic restart and LiveReload when
		deployed remotely

		spring:
			devtools:
				remote:
					secret: myappsecret

		1 Select the Run > Run Configurations menu item.
		2 Create a new Java Application launch configuration.
		3 Select the Reading List project in the Project field (either by typing the project
			name or clicking the Browse button and finding it). See figure A.1.
		4 Enter org.springframework.boot.devtools.RemoteSpringApplication into
			the Main Class field. See figure A.1.
		5 On the Arguments tab, enter https://readinglist.cfapps.io into the Program
			Arguments field.

		As changes are detected, they’ll be pushed to the remote server and applied

		To enabled remote debugging you can set JAVA_OPTS in your application’s manifest.yml file like this:
		---
		env:
			JAVA_OPTS: "-Xdebug -Xrunjdwp:server=y,transport=dt_socket,suspend=n"

	■ Development property defaults—Provides sensible development defaults for
		some configuration properties

		Add Deps : 

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
		</dependency>

		==> when the developer tools are active, the following properties are set to false:
			■ spring.thymeleaf.cache
			■ spring.freemarker.cache
			■ spring.velocity.cache
			■ spring.mustache.cache
			■ spring.groovy.template.cache

		==> Globally configuring developer tools

	Create a file named .spring-boot-devtools.properties in your home directory and add to it setting ...
			spring.devtools.restart.trigger-file=.trigger
			spring.devtools.livereload.enabled=false

	==> Taking a peek inside with the Actuator

	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-actuator</artifactId>
	</dependency>
     From Shell : ssh user@localhost -p 2000
     you nedd : 
     <dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-remote-shell</artifactId>
	</dependency>

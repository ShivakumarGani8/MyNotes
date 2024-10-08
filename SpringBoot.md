What is springboot?
    Spring Boot is a framework from the Spring team that makes it easy to create stand-alone, production-grade Spring applications. It provides defaults for code and annotation configuration to get you started quickly.

SpringBoot modules-

1. Spring Boot Starter:

    Starters: A set of convenient dependency descriptors you can include in your application. For example, spring-boot-starter-web for  web applications, spring-boot-starter-data-jpa for JPA-based data access, etc.
    Starter Parent: Provides default configurations, like default versions of dependencies and common settings.

2. Spring Boot Auto-Configuration:

    Automatically configures Spring application based on the included dependencies. For example, if you have spring-boot-starter-data-jpa in your classpath, Spring Boot automatically configures a DataSource, an EntityManagerFactory, etc.
    
3. Spring Boot Actuator:

    Provides production-ready features to help you monitor and manage your application, such as health checks, metrics, and auditing.

4. Spring Boot CLI (Command Line Interface):

    Allows you to quickly run and test Spring Boot applications using Groovy scripts. It’s a lightweight way to develop Spring Boot applications without needing a full Java IDE.
    
5. Spring Boot DevTools:

    Provides tools that enhance the development experience, including automatic restarts, live reload, and configurations specifically for development environments.

6. Spring Boot Security:

    Integrates with Spring Security to provide out-of-the-box security features like authentication, authorization, and protection against common attacks.

7. Spring Boot Testing:

    Provides testing support, including integration with JUnit, TestNG, and Mocking frameworks. It simplifies writing tests for Spring Boot applications.

8. Spring Boot Configuration:

    Supports externalized configuration via properties files, YAML files, environment variables, and command-line arguments.

9. Spring Boot Logging:

    Provides built-in logging support with sensible defaults and integrations with logging frameworks like Logback and Log4j.
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Boot Internals-> Application class(Provided by spring boot)
@SpringBootApplication
public class Application{
    public static void main(String[] args){
        SpringApplication.run(Application.class,args);
    }
} 

-> run() in springBoot:
    -run() is a special static method in SpringApplication class which is take care of boot application execution(bootstraping Spring application).
        SpringApplication.run(Application.class,args);
    -returns ConfigurableApplicationContext

Where @SpringApplication = @Configuration + @EnableAutoConfiguration + @ComponentScan

1. @Configuration: 
    -It helps in spring annotation based configuration.
    -The Configuration annotation indicates that a class declares one or more @Bean methods
    and may be processed by spring container to generate bean defination and service requests
    for those beans at runtime.

2. @EnableAutoConfiguration:
    -It tells spring boot to guess how you want to configure spring, based on the jar dependencies we add.
    -It auto-configures the beans that are present in classpath.
    -For EX: If H2 is on your classpath, and you have have not manually configured any database connection beans, then spring will auto-configure an in-memory database.

3. @ComponentScan:
    -It tells spring to look for other componenets,configurations and services in the specified package.
    -Spring is able to scan, detect and register your beans or componenets from pre-defined project package.
    -If no package is specified current classpath is taken as the root package.

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
-> Creating first springboot WebApplication-
spring-boot-starter-web:
    Step1: Create a project of type spring starter project.
    Step2: Select web type -> Spring web
    Step3: Create a folder webapp under src folder(Where we'll keep all the pages related to web).

Mapping requests to webpages:
    Springboot does not supports jsp requests by default To map we have to create java file and map it using @Controller & @RequestMapping annotations.
    Step1: Create a java file in java package.
    Step2: Mark class with @Controller Annotation.
        Ex:
            @Controller
            public class PageController {
            }

    Step3: To map particular requests use @RequestMapping to any java methods inside Controller page.
        @RequestMapping("home")
	    public String home() {
	    	return "home.jsp";
	    }
    
    ->Finally our java file looks like.
            @Controller
            public class PageController {
            	@RequestMapping("home")
            	public String home() {
            		return "home.jsp";
            	}
            }
    Step4: Include tomcat jasper to send requests to jsp in pom.xml

    Whenever we make requests to home we'll be redirected to home.jsp
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
-> Reading data from page to java
1. Using HttpServletRequest:
    ->By default spring will provide us object for HttpServletRequest & HttpServletResponse objects.
    ->We can setAttribute & getAttribute/using spEL(${name}) for requests & then use it in java/jsp.

Ex: 
    @RequestMapping("home")
	public String home(HttpServletRequest req,HttpServletResponse res) {
		System.out.println("Request: "+req);
		System.out.println("Response: "+res);
		System.out.println("Home page");
		String name=req.getParameter("name");
		System.out.println("Name: "+name);
		req.setAttribute("name", name);
		return "home.jsp";
	}

2. Using HttpSession:
    ->By default spring will provide us object for HttpSession aswell.
    ->We can read attribute directly to our requestMapping by passing parameter with same name as that of the url.
    ->setAttribute to session and read in jsp.

Ex:
    @RequestMapping("home")
	public String home(String name, HttpSession session) {
		System.out.println("Name: "+name);
		session.setAttribute("name", name);
		return "home.jsp";
	}

3. Using @RequestParam:
    -> If we want to give different parameter name from the url we have to make use of @RequestParam annotation to read parameter as a parameter to requests.

Ex: 
    //Using @RequestParam
	@RequestMapping("home")
	public String home(@RequestParam("name") String myName, HttpSession session) {
		System.out.println("Name: "+myName);
		session.setAttribute("name", myName);
		return "home.jsp";
	}

4. Using ModelAndView:
    -> It is a class provided by spring which reads and sends two parameters to the request page using addObject & setViewName methods. 
    -> Where addObject() adds data to the request & setViewName() adds page required for the request.

Ex: 
    @RequestMapping("home")
	public ModelAndView home(@RequestParam("name") String myName) {
		System.out.println("Name: "+myName);
		ModelAndView mv=new ModelAndView();
		mv.addObject("name",myName);
		mv.setViewName("home.jsp");
		return mv;
	}

    -> We can even read & set attributes for whole object directly as a parameter to the request.
Ex: 
    //Using ModelAndView class & setting whole object
	@RequestMapping("home")
	public ModelAndView home(Person person) {
		System.out.println("Name: "+person.getName());
		System.out.println("Age: "+person.getAge());
		System.out.println("Id: "+person.getId());
		ModelAndView mv=new ModelAndView();
		mv.addObject("person",person);
		mv.setViewName("home.jsp");
		return mv;
	}
    Note: Where person object will be automatically assigned by spring based on the name of the attribute of class and name in the url.
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
Rest API->
    -RESTful API acts as an interface between two applications to exchange information securely over the internet.
    -Most business applications have to communicate with other internal and third-party applications to perform various task.

Creating FirstRestAPI applications:
    Step1: Create a spring-starter-project with Maven dependencies.
    Step2: Create a class named anything to test inside eclipse(Say Product class in SpringRestAPI project).
    Step3: Create a RestController class which is used to control API requests(ProductRestController in SpringRestAPI) with @RestController annotation.
        Ex-
            @RestController
            class ProductRestController{
            }
    
    Step4: We can create methods to save/return data/data's to the caller using @PostMapping/@GetMapping to a method.
        EX-
            @PostMapping("/product")  //Handle post type requests
	        public String saveProduct(@RequestBody Product p) {
	        	System.out.println(p);
	        	//Logic to persists data

	        	return "Product saved";
	        }

            //Where pId acts as variable from url
	        @GetMapping("/product/{pId}") //Handles get type requests
	        public Product getProduct(@PathVariable Integer pId) {
	        	System.out.println("Get product");
	        	Product product=null;
	        	if(pId==100) product= new Product(100,"Mouse", 999.00);
	        	else if(pId==101) product= new Product(101,"HDD", 4995.00);

	        	return product;
	        }

            //Method returns list of objects back to the caller
	        @GetMapping("/products")
	        public List<Product> getProducts(){
	        	System.out.println("Get products");
	        	Product product1=new Product(100, "Mouse", 999.00);
	        	Product product2=new Product(102, "SSD", 7809.00);

	        	List<Product> list = Arrays.asList(product1,product2);
	        	return list;
	        }
        
        Step5: We can test the request using tools like postman
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++


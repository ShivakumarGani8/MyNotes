// Creating first Hello world API(The below class has to be created in controller package under our com.spring.springbootdemo) Where our SpringBootApplication class exists.

// SpringBootDemoApplication
@SpringBootApplication
public class SpringBootDemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringBootDemoApplication.class, args);
	}

}

// HomeController
@Controller
@ResponseBody

public class HomeController {

    @RequestMapping("/")
    public String home(){
        return "Hello World";
    }
}

-> @Controller = We can annotate classic controllers with the @Controller annotation. This is simply a specialization of the @Component class, which allows us to auto-detect implementation classes through the classpath scanning.

-> @ResponseBody = The @ResponseBody annotation tells a controller that the object returned is automatically serialized into JSON and passed back into the HttpResponse object.

// Enhancing Controller class to return Object
1. Create a User class below com.spring.springbootdemo.model package with following parameters with necessary getters and setters.
    private String id;
    private String name;
    private String email;

2. Refactor HomeController class with @RestController

@RestController
public class HomeController {
}

-> RestController = @RestController is a specialized version of the controller. It includes the @Controller and @ResponseBody annotations, and as a result, simplifies the controller implementation:

3. Create a method getUser with @GetMapping annotation under HomeController class

@RestController
public class HomeController {

    @RequestMapping("/")
    public String home(){
        return "Hello World";
    }

    @GetMapping("/user")
    public User getUser(){
        User user = new User();
        user.setId("1");
        user.setName("Shivakuamr");
        user.setEmail("shivakumar@gmail.com");

        return user;
    }
}

-> GetMapping = The @GetMapping annotation in Spring is used to map HTTP GET requests to specific handler methods in Spring controllers.


// PathVariables
@GetMapping("{id}/{name}")
public String pathVariables(@PathVariable String id, @PathVariable String name){
    return "The path variable id is : "+ id +" Name is : "+ name;
}

-> PathVariable = @PathVariable annotation can be used to handle template variables in the request URI mapping, and set them as method parameters.
- We can use @PathVarialbe when the parameters exists in URI and are mandatory fields for our request

- A simple URI for the above mapping is = http://localhost:8080/1/shiva

//RequestParam
@GetMapping("/")
public String requestParameters(@RequestParam String id, @RequestParam String email){
    return "The request parameter variable id is : "+id+ " Email : "+email;
}

-> RequestParam = @RequestParam to extract query parameters, form parameters, and even files from the request.
- A URI for follwing above mapping is = http://localhost:8080/?id=1&email=shiva@gani

-> To mark certain parameter as not mandatory we can use required field for parameter & To keep default value when parameter not exists we can use defaultValue field.

@GetMapping("/")
public String requestParameters(@RequestParam String id, @RequestParam(required=false,defaultValue = "") String email){
    return "The request parameter variable id is : "+id+ " Email : "+email;
}


********************************************************************************************
                            EmployeeService API
step 1:-> create Employee class under model package with following parameters
    class Employee{
        private String employeeId;
        private String firstName;
        private String lastName;
        private String emailId;
        private String department;
    }

step 2:-> Create service interface for employee under com.spring.springbootdemo.service package
    public interface IEmployeeService {
        public Employee save(Employee employee);
    }

step 3:-> Create implementation for service class.
    @Service
    public class EmployeeServiceImpl implements IEmployeeService{

        List<Employee> employees=new ArrayList<>();

        public Employee save(Employee employee) {
            if(employee.getEmployeeId()==null || employee.getEmployeeId().isEmpty())
                employee.setEmployeeId(UUID.randomUUID().toString());
            employees.add(employee);
            return employee;
        }
    }

Step 4:-> Create RestController class for EmployeeService
    @RestController
    @RequestMapping("/employees")
    public class EmployeeController {
        @Autowired
        private IEmployeeService employeeService;
    }

-> Implementing Post method to create Employee resourses
    @PostMapping
    public Employee save(@RequestBody Employee employee){
        return employeeService.save(employee);

    }

-> Implementing Get method to getAll employees
    // Controller class
    @GetMapping
    public List<Employee> getAllEmployee(){
        return  employeeService.getAllEmployees();
    }

    // Service class
    @Override
    public List<Employee> getAllEmployees() {
        return employees;
    }

-> Implementing Get method to get Employee by Id
    //Controller class
    @GetMapping
    public List<Employee> getAllEmployee(){
        return  employeeService.getAllEmployees();
    }
    ///Service class
    @Override
    public Employee geEmployeeById(String id) {
        return employees.stream().filter(employee -> employee.getEmployeeId().equalsIgnoreCase(id)).findFirst().get() ;
    }

-> Implementing delete method to remove an employee from the list
    //Controller class
    @DeleteMapping("/{id}")
    public String deleteEmployeeById(@PathVariable String id){
        return employeeService.deletEmployeeById(id);
    }
    //Service class
    @Override
    public String deletEmployeeById(String id) {
        Employee employee= employees.stream().filter(e-> e.getEmployeeId().equalsIgnoreCase(id)).findFirst().get();
        employees.remove(employee);
        return "Employee with Id : "+id+" Has been removed from the list";
    }

********************************************************************************************
Implementing Exception Handling
For Ex: If there is no employee with particular Id while getting an employee by id it will throw some Exception instead we will define our own Exception handling class for Employee named EmployeeNotFoundExeption in com.spring.springbootdemo.error package

    //EmployeeNotFoundException.class
    public class EmployeeNotFoundExceptioon extends RuntimeException{
        public EmployeeNotFoundExceptioon(String message){
            super(message);
        }
    }

Step 2:-> Create an advisor class to handle the all encorporated exceptions.(RestResponseEntityExceptionHandler) under com.spring.springbootdemo.error package.

    @ControllerAdvice
    public class RestResponseEntityExceptionHandler extends ResponseEntityExceptionHandler {
    }

Step 3:-> Create a ErrorMessage class to send response back an object to the request in com.spring.springbootdemo.controller package with following fields

    public class ErrorMessage {
        private HttpStatus status;
        private String message;

        //Getters and setters for the above fileds

        public ErrorMessage() {
        }
    
        public ErrorMessage(HttpStatus status, String message) {
            this.status = status;
            this.message = message;
        }
    }

Step 4:-> Create a method which handles the EmployeeNotFoundException in RestResponseEntityExceptionHandler.class

    @ExceptionHandler(EmployeeNotFoundExceptioon.class)
    @ResponseBody
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorMessage employeeNotFoundHandler(EmployeeNotFoundExceptioon exceptioon){
        ErrorMessage errorMessage=new ErrorMessage(HttpStatus.NOT_FOUND,exceptioon.getMessage());
        return errorMessage;
    }

Step 5:-> Refactor getEmployeeId method in service clas to throw EmployeeNotFoundException when employee with particular id is not available.

    @Override
    public Employee geEmployeeById(String id) {
        return employees.stream().filter(employee -> employee.getEmployeeId().equalsIgnoreCase (id)).findFirst().orElseThrow(()-> new EmployeeNotFoundExceptioon("Employee Not Found With Id : "+id));
    }

Now the follwing message will be returned back when an employee with particular id is not present in the class.
        {
            "status": "NOT_FOUND",
            "message": "Employee Not Found With Id : 6bed35e4-c5e6-41fd-81c1-96aea0591fa"
        }

=>Handling Generic Exceptions
    @ExceptionHandler(RuntimeException.class)
    @ResponseBody
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorMessage genericExceptionHandler(Exception exceptioon){
        ErrorMessage errorMessage=new ErrorMessage(HttpStatus.INTERNAL_SERVER_ERROR,exceptioon.getMessage());
        return errorMessage;
    }


********************************************************************************************
                     Spring Data JPA with SpringBoot
Step 1:-> Add jpa dependencies to the pom.xml file
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
        <version>3.1.6</version>
    </dependency>

Step 2:-> Create EmployeeEnitity class with following attributes in com.spring.springbootdemo.entity package.
 
        @Entity
        public class EmployeeEntity {
            @Id
            private String employeeId;
            private String firstName;
            private String lastName;
            private String emailId;
            private String department;
        }
    
Step 3:-> Create a repository interface extending JPARepository interface in com.spring.springbootdemo.repository package.
    public interface EmployeeRepository extends JpaRepository<EmployeeEntity, String> {
    }


-------------------------------------------------
EmployeeV2Controller class to control the requests for DB

@RestController
@RequestMapping("/v2/employees")
public class EmployeeV2Controller {

    @Qualifier("employeeV2ServiceImpl")
    @Autowired
    private IEmployeeService employeeService;

    @PostMapping
    public Employee save(@RequestBody Employee employee){
        return employeeService.save(employee);
    }

    @GetMapping
    public List<Employee> getAllEmployees(){
        return employeeService.getAllEmployees();
    }

    @GetMapping("/{id}")
    public Employee getEmployeeById(@PathVariable String id){
        return employeeService.getEmployeeById(id);
    }

    @DeleteMapping("/{id}")
    public String deleteEmployeeById(@PathVariable String id){
        return employeeService.deletEmployeeById(id);
    }
}

------------------------------------------------------------
Service class implementation for data persist to DB

@Service
public class EmployeeV2ServiceImpl implements IEmployeeService{

    @Autowired
    private EmployeeRepository employeeRepo;

    @Override
    public Employee save(Employee employee) {
        if(employee.getEmployeeId()==null || employee.getEmployeeId().isEmpty()){
            employee.setEmployeeId(UUID.randomUUID().toString());
        }
        EmployeeEntity employeeEntity = new EmployeeEntity();
        BeanUtils.copyProperties(employee,employeeEntity);
        employeeRepo.save(employeeEntity);
        return employee;
    }

    @Override
    public List<Employee> getAllEmployees() {
        List<EmployeeEntity> employeeEntities= employeeRepo.findAll();
        return employeeEntities.stream().map(employeeEntity -> {
            Employee employee=new Employee();
            BeanUtils.copyProperties(employeeEntity,employee);
            return employee;
        }).collect(Collectors.toList());
    }

    @Override
    public Employee getEmployeeById(String id) {
        EmployeeEntity employeeEntity=employeeRepo.findById(id).get();
        Employee employee=new Employee();
        BeanUtils.copyProperties(employeeEntity,employee);
        return employee;
    }

    @Override
    public String deletEmployeeById(String id) {
        employeeRepo.deleteById(id);
        return "Employee deleted with ID: "+id;
    }
}
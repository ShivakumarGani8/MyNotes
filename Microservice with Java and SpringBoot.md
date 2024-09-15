=> In our project we are creating a rest ecommerce application which talk to other services.
    - There will be 3 services OrderService, ProductService and PaymentServies.
    - We are creating ServiceRegistery which acts as server to make integration for each services where they can interact each other.

********************************************************************************************
=> ProductService service
    - Which runs on port:8081
    - Consists 2 services addProducts and getProducts.
    - ProuductServiceCustomExceptionHandler class to handle all the exceptions occurs in the application


****************************Controller layer for ProductService*****************************
    @RestController
    @RequestMapping("/product")
    public class ProductController {
    
        @Autowired
        private IProductServie productServie;
    
        @PostMapping
        public ResponseEntity<Long> addProduct(@RequestBody ProductRequest productRequest){
            long productId= productServie.addProduct(productRequest);
            return new ResponseEntity<>(productId, HttpStatus.CREATED);
        }
    
        @GetMapping("/{id}")
        public ResponseEntity<ProductResponse> getProduct(@PathVariable(name = "id") String productId){
            ProductResponse productResponse= productServie.getProduct(productId);
            return new ResponseEntity<>(productResponse,HttpStatus.OK);
        }
    }

*******************************Entity class for Product*************************************

    @Entity
    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    @Builder
    public class Product {

        @Id
        @Column(name = "PRODUCT_ID")
        @GeneratedValue(strategy = GenerationType.AUTO)
        private long productId;

        @Column(name = "PRODUCT_NAME")
        private String productName;

        @Column(name = "PRODUCT_QUANTITY")
        private long productQuantity;

        @Column(name = "PRODUCT_PRICE")
        private double productPrice;
    }

******************************ServiceLayer for ProductService**************************

    @Log4j2
    @Service
    public class ProductServiceImpl implements IProductServie{
    
        @Autowired
        private ProductRepository productRepository;
    
        @Override
        public long addProduct(ProductRequest productRequest) {
            log.info("Adding product to the table...");
            Product product= Product.builder().productName(productRequest.getName())
                    .productQuantity(productRequest.getQuantity())
                    .productPrice(productRequest.getPrice()).build();
    
            productRepository.save(product);
            return product.getProductId();
        }
    
        @Override
        public ProductResponse getProduct(String productId) {
            log.info("Get product with the productId : {}", productId);
            Product product= productRepository
                    .findById(productId).orElseThrow(() ->
                            new ProductServiceCustomException("Product not found with ID: "+productId,"PRODUCT_NOT_FOUND"));
    
            ProductResponse productResponse=new ProductResponse();
            BeanUtils.copyProperties(product,productResponse);
    
            return productResponse;
        }
    }

*****************************************************************************************************************************************
Handling Exceptions for ProductService
    Step1:-> Create a class ProductServiceCustomException extending RuntimeException marking with necessary Annotations.
    @Data
    public class ProductServiceCustomException extends RuntimeException {
        private ErrorCode errorCode;

        public ProductServiceCustomException(String message, ErrorCode errorCode){
            super(message);
            this.errorCode=errorCode;
        }
    }s

    Step2:-> Create a class RestResponseEntityExceptionHandler with @ControllerAdviser annotation which will handle all the rest entity exceptions extending ResponseEntityExceptionHandler.
    Step3:-> Create a method which handles the ProductServiceCustomException's
    Step4:-> Mark method with @ExceptionHandler annotation(ClassName.class)
    Step5:-> Create a model class ErrorResponse which will be returned back to the caller
        @Data
        @Builder
        @NoArgsConstructor
        @AllArgsConstructor
        public class ErrorResponse {
            private String errorMessage;
            private ErrorCode errorCode;
        }

    - Now our ControllerAdviser class looks like which will handle our exceptions and returned to the caller.

    @ControllerAdvice
    public class RestResponseEntityExceptionHandler extends ResponseEntityExceptionHandler {

        @ExceptionHandler(ProductServiceCustomException.class)
        public ResponseEntity<ErrorResponse> handleProductNotFoundException(ProductServiceCustomException exception){

            ErrorResponse errorResponse= ErrorResponse.builder()
                    .errorMessage(exception.getMessage())
                    .errorCode(exception.getErrorCode())
                    .build();

            return new ResponseEntity<>(errorResponse, HttpStatus.NOT_FOUND);
        }
    }



******************************************************************************************************
=> ServiceRegistery for application
    - Runs on port 8761
    Step1:-> Create a ServiceRegitery spring application
    Step2:-> Add necessary dependencies -> Eureka server and cloud BootStrap
    Step3:-> Mark application class with @EnableEurekaServer annotation to make the service as a server

    @SpringBootApplication
    @EnableEurekaServer
    public class ServiceRegisteryApplication {

    	public static void main(String[] args) {
    		SpringApplication.run(ServiceRegisteryApplication.class, args);
    	}

    }

    Step4:-> Set necessary properties to the application.yaml file to configure properties to the application

        server:
          port: 8761

        eureka:
          hostname: localhost
          client:
            fetch-registry: false
            register-with-eureka: false

        Now our eureka server is ready to connect clients to the server need to add Eureka  Discovery Client dependencies to our client servers.

    Step5:-> Add Eureka Client Discovery dependencies to client servers
    Step6:-> Set necessary client application properties to connect to server(If necessary add @EnableDiscoveryClient annotation to SpringApplication class of client).

        eureka:
            instance:
              prefer-ip-address: true
            client:
              fetch-registry: true
              register-with-eureka: true
              service-url:
                defaultZone: ${EUREKA_SERVER_ADDRESS:http://localhost:8761/eureka/}


********************************************************************************************
=> OrderService service
    - Which runs on port:8082
    - Which connects to ProductService and PaymentService to check products details and do payment in PaymentService.
    - Contains placeOrder service to place orders.
        1. Takes the OrderRequest.
        2. Reduces the quantity from ProductService.
        3. Updates order details into the order table.

******************************ControllerLayer for OrderService******************************
    public class OrderController {

        @Autowired
        private IOrderService orderService;

        @PostMapping("/placeOrder")
        public ResponseEntity<Long> placeOrder(@RequestBody OrderRequest orderRequest){
            long orderId= orderService.placeOrder(orderRequest);
            return new ResponseEntity<>(orderId, HttpStatus.CREATED);
        }
    }

******************************Entity class for for OrderService*****************************
    @Entity
    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    @Table(name = "ORDER_TABLE")
    @Builder
    public class Order {

        @Id
        @GeneratedValue(strategy = GenerationType.AUTO)
        @Column(name= "ORDER_ID")
        private long orderId;

        @Column(name = "PRODUCT_ID")
        private long productId;

        @Column(name = "QUANTITY")
        private long quantity;

        @Column(name = "ORDER_DATE")
        private Instant orderDate;

        @Column(name = "ORDER_STATUS")
        @Enumerated(EnumType.STRING)
        private OrderStatus orderStatus;

        @Column(name = "AMOUNT")
        private double amount;

        @Column(name = "PAYMENT_MODE")
        @Enumerated(EnumType.STRING)
        private PaymentMode paymentMode;
    }

******************************ServiceLayer for OrderService******************************
    @Service
    public class OrderServiceImpl implements IOrderService{

        @Autowired
        private IOrderRepository orderRepository;

        @Override
        public long placeOrder(OrderRequest orderRequest) {
            Order order= Order.builder()
                    .productId(orderRequest.getProductId())
                    .quantity(orderRequest.getQuantity())
                    .amount(orderRequest.getTotalAmount())
                    .orderStatus(OrderStatus.valueOf(OrderStatus.CREATED.name()))
                    .paymentMode(PaymentMode.valueOf(PaymentMode.CASH.name()))
                    .orderDate(Instant.now()).build();

            orderRepository.save(order);

            return order.getOrderId();
        }
    }

NOTE: placeOrder service need to be imporved further to update data into Product tables

*****************************************************************************************
=> CONFIG-SERVER
    - It manages all the common properties of all the client applications.
    Step1:-> Create a spring project
    Step2:-> Add necessary dependencies
        = Config Server
    Step3:-> Create a repo for common properties and create a property file add common properties
    Step4:-> Add properties of repo to the ConfigServer application file to connect to repository

        server:
          port: 9296
        
        spring:
          application:
            name: CONFIG-SERVER
          cloud:
            config:
              server:
                git:
                  uri: https://github.com/ShivakumarGani8/ecommerce-app-config
                  clone-on-start: true
        
        eureka:
          instance:
            prefer-ip-address: true
          client:
            fetch-registry: true
            register-with-eureka: true
            service-url:
              defaultZone: ${EUREKA_SERVER_ADDRESS:http://localhost:8761/eureka/}

    Step5:-> Mark SpringApplication class of ConfigServer with @EnableConfigServer annotation
    
    Step6:-> Add config client dependency to client server and properties to client servers to import properties from repo

          config:
            import: configserver:http://localhost:9296
    

Now we can connect to the configserver for common configuration files

*******************************************************************************************
=> Implementing reduceQuantity API in ProductService.
    -> Controller layer

        @GetMapping("/{id}")
        public ResponseEntity<ProductResponse> getProduct(@PathVariable(name = "id") String productId){
            ProductResponse productResponse= productServie.getProduct(productId);
            return new ResponseEntity<>(productResponse,HttpStatus.OK);
        }

    -> Service layer

        public long reduceQuantity(Long productId, Long quantity) {
            log.info("Reducing product quantity for product with id : {}",productId);
            Product product= productRepository.findById(productId.toString()).orElseThrow(()->
                    new ProductServiceCustomException("Product not found with ID: "+productId, ProductErrorCode.PRODUCT_NOT_FOUND));
            if(product.getProductQuantity()>=quantity){
                product.setProductQuantity(product.getProductQuantity()-quantity);
                productRepository.save(product);
                log.info("Products quantity reduced successfully..");
            }else{
                log.info("Insufficient quantity of products..");
                throw new ProductServiceCustomException("Insufficient quantity of products",ProductErrorCode.INSUFFICIENT_QUANTITY);
            }
            return product.getProductId();
        }

********************************************************************************************
=> Implementing Feign client to order-service

Step1:-> Include OpenFeign dependencies in order-service pom.
Step2:-> Mark OrderServiceApplication class with @EnableFeignClients annotation.
Step3:-> Create a package external to keep all the external files seperately.
Step4:-> Within the external create a package client where client requests can be handled.
Step5:-> Create an interface with same name as that in ProductService and mark it with @FeignClient annotation and provide name of the project with name property.
            
            @FeignClient(name = "PRODUCT-SERVICE/product")

Step6:-> Now include the API methods from clients which we want to call in our order-service.
            @PutMapping("/reduceQuantity/{id}")
            ResponseEntity<Void> reduceQuantity(
                @PathVariable(name = "id") long productId,
                @RequestParam("quantity") long quantity);
            
Our Final view of ProductService interface looks like 

            @FeignClient(name = "PRODUCT-SERVICE/product")
            public interface IProductService {
                @PutMapping("/reduceQuantity/{id}")
                ResponseEntity<Void> reduceQuantity(
                        @PathVariable(name = "id") long productId,
                        @RequestParam("quantity") long quantity);
            }

Step7:-> Now we can autowire this interface in our application anywhere and call this method to make an API call to reduceQuantity in ProductService.

********************************************************************************************
=> Implementing error decoder for Insufficient Quantity of products
Step1:-> Create a class CustomeErrorDecoder under package com.ecommerce.OrderService.external.decoder by implementing the ErrorDecoder interface.

Step2:-> Provide implementation to the decode method of ErrorDecoder interface.

        @Override
        public Exception decode(String s, Response response) {
            ObjectMapper objectMapper=new ObjectMapper();
            log.info("::{}",response.request().url());
            log.info("::{}",response.request().headers());

            try {
                ErrorResponse errorResponse= objectMapper.readValue(response.body().asInputStream(),
                        ErrorResponse.class);

                return new CustomException(errorResponse.getErrorMessage(),
                        errorResponse.getErrorCode(),
                        response.status());
            } catch (IOException e) {
                throw new CustomException("Internal Server Error", ErrorCode.INTERNAL_SERVER_ERROR,500);
            }
        }

Step3:-> Create ErrorResponse class under external.response package.

        @Data
        @Builder
        @NoArgsConstructor
        @AllArgsConstructor
        public class ErrorResponse {
            private String errorMessage;
            private ErrorCode errorCode;
        }
    
Step4:-> Create a class FeignConfig which handles all the configurations of FeignClients and mark it with @Configuration annotation.

        @Configuration
        public class FeignConfig {
        
            @Bean
            ErrorDecoder errorDecoder(){
                return new CustomeErrorDecoder();
            }
        }
    
Step5:-> Create a Controller advicer class RestResponseEntityHandler by extending ResponseEntityExceptionHandler class and mark it with @ControllerAdvice annotation

        @ControllerAdvice
        public class RestResponseEntityExceptionHandler extends ResponseEntityExceptionHandler {

            @ExceptionHandler(CustomException.class)
            public ResponseEntity<ErrorResponse> handleCustomException(CustomException exception){
                ErrorResponse.ErrorResponseBuilder builder = ErrorResponse.builder();
                builder.errorMessage(exception.getMessage());
                builder.errorCode(exception.getErrorCode());
                ErrorResponse    errorResponse= builder.build();

                return new ResponseEntity<>(errorResponse, HttpStatus.valueOf(exception.getStatus()));
            }
        }   

Now we are all set where we can successfully handles the exceptions that are coming from the Product-service. 

*********************************************************************************************************
=> PaymentService-
    This service will handle all the payment requests comming.

Step1:-> Create a spring project
    Step2:-> Add necessary dependencies
    Step3:-> Create a repo for common properties and create a property file add common properties
    Step4:-> Add properties of repo to the ConfigServer application file to connect to repository

        server:
          port: 8081
        
        spring:
          application:
            name: PAYMENT-SERVICE
          cloud:
            config:
              server:
                git:
                  uri: https://github.com/ShivakumarGani8/ecommerce-app-config
                  clone-on-start: true
        
        eureka:
          instance:
            prefer-ip-address: true
          client:
            fetch-registry: true
            register-with-eureka: true
            service-url:
              defaultZone: ${EUREKA_SERVER_ADDRESS:http://localhost:8761/eureka/}
    
Now we can connect to the configserver for common configuration files

********************************Controller layer for PaymentService**************************************
@RestController
@RequestMapping("/payment")
public class PaymentController {

    @Inject
    private IPaymentService paymentService;

    @PostMapping("/doPayment")
    public ResponseEntity<Long> doPayment(@RequestBody PaymentRequest paymentRequest){
        return new ResponseEntity<>
                (paymentService.makePayment(paymentRequest), HttpStatus.OK);
    }
}

************************************Entity class*********************************************************
@Table(name = "PAYMENT_TABLE")
public class TransactionDetails {

    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name = "PAYMENT_ID")
    @Id
    private long id;

    @Column(name = "ORDER_ID")
    private long orderId;

    @Column(name = "PAYMENT_MODE")
    @Enumerated(EnumType.STRING)
    private PaymentMode paymentMode;

    @Column(name = "REFERENCE_NUMBER")
    private String referenceNumber;

    @Column(name = "PAYMENT_DATE")
    private Instant paymentDate;

    @Column(name = "PAYMENT_STATUS")
    @Enumerated(EnumType.STRING)
    private PaymentStatus paymentStatus;

    @Column(name = "AMOUNT")
    private double amount;

}

***************************************Service Layer*****************************************************
@Log4j2
@Service
public class PaymentServiceImpl implements IPaymentService {

    @Inject
    private ITransactionDetailsRepository transactionDetailsRepository;

    @Override
    public long makePayment(PaymentRequest paymentRequest) {
        log.info("Making payment for order {} ", paymentRequest.getOrderId());
        TransactionDetails transactionDetails= TransactionDetails.builder()
                .orderId(paymentRequest.getOrderId())
                .paymentMode(paymentRequest.getPaymentMode())
                .paymentStatus(PaymentStatus.SUCCESS)
                .paymentDate(Instant.now())
                .amount(paymentRequest.getAmount())
                .build();
        transactionDetailsRepository.save(transactionDetails);

        return transactionDetails.getId();
    }
}

*********************************************************************************************************
=> Calling doPayment API from order service
    Step1:-> Create an interface in external folder
    Step2:-> Mark the interface with @FeighnClient annotation
    Step3:-> Give declaration of API which we want to call with same path and arguments
    
    @FeignClient(name= "PAYMENT-SERVICE/payment")
    public interface IPaymentService {
        @PostMapping("/doPayment")
        ResponseEntity<Long> doPayment(@RequestBody PaymentRequest paymentRequest);
    }
    
    Step4:-> Now we can call this API in orderservice anywhere.

*********************************************************************************************************
=>Calling services using RestTemplate

    
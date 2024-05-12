=> In our project we are creating a rest ecommerce application which talk to other services.
    - There will be 3 services OrderService, ProductService and PaymentServies.
    - We are creating ServiceRegistery which acts as server to make integration for each services where they can interact each other.

********************************************************************************************
=> ProductService service
    - Which runs on port 8080
    - Consists 2 services addProducts and getProducts.
    - ProuductServiceCustomExceptionHandler class to handle all the exceptions occurs in the application


*************Controller layer for ProductService********************
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

****************************Entity class for Product**********************************

    public class Product {
    
        @Id
        @Column(name = "PRODUCT_ID")
        @GeneratedValue(strategy = GenerationType.SEQUENCE)
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


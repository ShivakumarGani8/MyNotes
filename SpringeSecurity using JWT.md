=> Spring Security
    Spring Security is a powerful and customizable authentication and access control framework for Java applications, particularly Spring-based applications. It handles security concerns such as authentication, authorization, protection against attacks like session fixation, clickjacking, cross-site request forgery (CSRF), and more.

- Key Features of Spring Security:
1. Authentication: Verifies the identity of users using methods such as username/password, OAuth2, JWT, etc.
2. Authorization: Controls access to certain resources based on roles or permissions.
3. Security Context: Manages the security context of the application, ensuring that security information (such as the logged-in user) is available throughout the application.
4. CSRF Protection: Prevents CSRF attacks by adding tokens to forms and verifying their legitimacy.
5. Method Security: Allows annotation-based security at the method level (e.g., @PreAuthorize and @Secured).
6. Session Management: Manages user sessions, ensuring that session-based attacks such as session fixation are prevented.
7. Integration with OAuth2 and JWT: Supports modern security mechanisms like OAuth2, allowing integration with identity providers like Google, Facebook, or custom OAuth2 servers.

Basic Configuration in Spring Boot:
To add Spring Security to a Spring Boot application, include the following dependency in your pom.xml:
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>

-> Storing user details in User table
- To store user details we'll user H2 Inmemory database.
- For this purpose we'll create a method which run with @PostConstructor annotation runs and store data in the database.
    
    Example: method in SpringStarterApplication class

    @PostConstruct
	public void initUser(){
		List<User> user= Stream.of(
				new User(101,"user1@email.com","user1","password1"),
				new User(101,"user1@email.com","user1","password1"),
				new User(101,"user1@email.com","user1","password1"),
				new User(101,"user1@email.com","user1","password1")
				).toList();
		userDetailsRepository.saveAll(user);
	}

Note: We'll create User entity and UserDetailsRepository as a pre requisite to handle repo.
- Configuration in application.yaml file for H2 database.
        spring:
          h2:
            console:
              enabled: true

=> Config class to validate username and password.

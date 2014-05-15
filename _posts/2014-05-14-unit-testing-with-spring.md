---
layout: post
heading: Unit Testing Camel TypeConverters
author: ando
---
At Sandbox, we use the Apache Camel framework extensively for its swiss army knife approach to integration, and Spring for its dependency injection capabilities. Testing is a crucial part of our development process, and unit-testing in particular helps ensure our components are robustly designed. Howerver, getting JUnit, Spring and Camel to all play nice when unit-testing is fiddly. We favour Java annotations for Spring configuration and test class examples are few and far between so here\'s one way to do it.

To keep it simple, our unit under test will be a Camel TypeConverter which is essentially a POJO to marshal data between types.

```java
@Converter
@Component
public final class ResponseConverter {
    
    @Autowired
    Foo foo;

    @Converter
    public Response toResponse() {
        Response response = new Response();
        if (foo.getBar()) {
            response.setResult(true);
        } else {
            response.setResult(false);
        }
        return response;
    }
    // ... getters, setters etc.
}
```


We\'ve decided on using instance methods in our converters to help ease the unit testing process. Spring is responsible for injecting the `foo` property and we can ignore the `@Converter` annotations for now. The objective is to test the sole method `toResponse`.

To control the dependency injection, our test class needs to use the Spring JUnit4.x test runner and provide its own Spring Context Configuration. The skeleton of our unit-test looks like this:

~~~ java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  classes = {ResponseConverterTest.ContextConfiguration.class},
  loader = AnnotationConfigContextLoader.class
)
public class ResponseConverterTest {

    @Configuration
    static class ContextConfiguration {
        @Bean
        public Foo createFoo() {
          return new Foo();
        }
    }

    @Test
    public void toResponse_fromRequest_shouldReturnOk() {
        Response response = new Response();
        assertTrue("result should be true", response.getResult());
    }

    @Test
    public void toResponse_fromRequest_shouldReturnFailed() {
        Response response = new Response();
        assertFalse("result should be false", response.getResult());
    }
}
~~~

We favour Java annotations to configure Spring. The `@ContextConfiguration` annotation instructs Spring to use our static class `ContextConfiguration` to instantiate a `Foo` instance for later injection into components.

So we've loaded a Spring contexct but we haven't yet instructed Spring to scan for components. We could use `@ComponentScsan` but we prefer having more explicit control over the instantiation of our unit and ensure we have a new copy of it for each test. We can do this quickl by getting the `ApplicationContext` and use it to wire up a new copy of our unit before each test is run.

~~~ java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  classes = {ResponseConverterTest.ContextConfiguration.class},
  loader = AnnotationConfigContextLoader.class
)
public class ResponseConverterTest {

    @Autowired
    public ApplicationContext ctx;

    private ResponseConverter responseConverter;

    @Configuration
    static class ContextConfiguration {
        @Bean
        public Foo createFoo() {
          return new Foo();
        }
    }

    @Before
    public void setup() {
        responseConverter = new ResponseConverter();
        ctx.getAutowireCapableBeanFactory().autowireBean(responseConverter);
    }

    @Test
    public void toResponse_fromRequest_shouldReturnOk() {
        Response response = new Response();
        assertTrue("result should be true", response.getResult());
    }

    @Test
    public void toResponse_fromRequest_shouldReturnFailed() {
        Response response = new Response();
        assertFalse("result should be false", response.getResult());
    }
}
~~~

Almost there, only remaining task is to mock the `Foo` dependency. We use the excellent Mockito library to mock classes and their methods. Use Spring to wire the `Foo` component into our test class so we can mock its methods. What we discovered during testing is that we needed a new Spring context for each test otherwise mock behaviours we had set would carry over (and we wanted to avoid using `Mockito.reset(...)`). `@DirtiesContext` can be used at the class level or method level which will force a reload of the `ApplicationContext` after any such annotated method as well as after the entire class. Providing the `AFTER_EACH_TEST_MEHOD` forces it to reload after each test. Here's our completed test class.

~~~ java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  classes = {ResponseConverterTest.ContextConfiguration.class},
  loader = AnnotationConfigContextLoader.class
)
@DirtiesContext(classMode = DirtiesContext.ClassMode.AFTER_EACH_TEST_METHOD)
public class ResponseConverterTest {

    @Autowired
    private ApplicationContext ctx;

    @Autowired
    private Foo foo;

    private ResponseConverter responseConverter;

    @Configuration
    static class ContextConfiguration {
        @Bean
        public Foo createFoo() {
          return new Foo();
        }
    }

    @Before
    public void setup() {
        responseConverter = new ResponseConverter();
        ctx.getAutowireCapableBeanFactory().autowireBean(responseConverter);
    }

    @Test
    public void toResponse_fromRequest_shouldReturnOk() {
        Response response = responseConverter.toResponse();
        // mock foo.getBar() to return true
        Mockito.mock(foo.getBar()).thenReturn(true);

        assertTrue("result should be true", response.getResult());
    }

    @Test
    public void toResponse_fromRequest_shouldReturnOk() {
        Response response = responseConverter.toResponse();
        // mock foo.getBar() to return true
        Mockito.mock(foo.getBar()).thenReturn(false);

        assertTrue("result should be true", response.getResult());
    }
}
~~~

This is just a brief introduction to getting started with unit-testing Camel Components in Spring. In the coming weeks we'll build on this and demonstrate how we test complex components with routing.

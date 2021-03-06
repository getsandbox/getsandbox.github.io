---
layout: post
heading: Unit Testing Camel TypeConverters
description: Testing Camel components can be tricky, there are so many options. Here we have laid out our approach to testing the Camel Type Converters with Spring 3 using annotations and Java Config.
author: ando
---
At Sandbox, we use the Apache Camel framework extensively for its swiss army knife approach to integration, and Spring for its dependency injection capabilities. Testing is a crucial part of our development process and, unit-testing in particular, helps ensure our components are robustly designed. Howerver, getting JUnit, Spring and Camel to all play nice when unit-testing is fiddly. We favour Java annotations for Spring configuration and test class examples are few and far between so here\'s one way to do it.

To keep it simple, our unit under test will be a Camel TypeConverter which is essentially a POJO to marshal data between types.


{% highlight java %}
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
{% endhighlight %}


We\'ve decided on using instance methods in our converters to help ease the unit testing process. Spring is responsible for injecting the `foo` property and we can ignore the `@Converter` annotations for now. The objective is to test the sole method `toResponse()`.

To control the dependency injection, our test class needs to use the Spring JUnit4.x test runner and provide its own Spring Context Configuration. The skeleton of our unit-test looks like this:

{% highlight java %}
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
        Response response = new ResponseConverter().toResponse();
        assertTrue("result should be true", response.getResult());
    }

    @Test
    public void toResponse_fromRequest_shouldReturnFailed() {
        Response response = new ResponseConverter().toResponse();
        assertFalse("result should be false", response.getResult());
    }
}
{% endhighlight %}

We favour Java annotations to configure Spring. The `@ContextConfiguration` annotation instructs Spring to use our static class `ContextConfiguration` to instantiate a `Foo` instance for later injection into components.

So we\'ve loaded a Spring contexct but we haven't yet instructed Spring to scan for components. We could use `@ComponentScsan` but we prefer having explicit control over the instantiation of our unit ensuring a new instance is created for each test. We can do this quickly by getting the `ApplicationContext` and use it to wire up a new instance of `ResponseConverter` before each test is run.

{% highlight java %}
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
        Response response = responseConverter.toResponse();
        assertTrue("result should be true", response.getResult());
    }

    @Test
    public void toResponse_fromRequest_shouldReturnFailed() {
        Response response = responseConverter.toResponse();
        assertFalse("result should be false", response.getResult());
    }
}
{% endhighlight %}

Almost there, only remaining task is to mock the `Foo` dependency so we can alter the behaviour to satisfy the second test. We use the excellent Mockito library to mock classes and their methods. Use Spring to wire the `Foo` component into our test class so we can mock its methods. What we discovered during testing is that we needed a new Spring context for each test otherwise mock behaviours we had set would carry over (and we wanted to avoid using `Mockito.reset(...)`). `@DirtiesContext` can be used at the class level or method level which will force a reload of the `ApplicationContext` after any such annotated method as well as after the entire class. Providing the `AFTER_EACH_TEST_MEHOD` forces it to reload after each test. Here's our completed test class.

{% highlight java %}
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
{% endhighlight %}

This is just a brief introduction to getting started with unit-testing Camel Components in Spring. In the coming weeks we'll build on this and demonstrate how we test complex components with routing.
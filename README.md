# Integration and Acceptance Testing in Spring

## Learning Goals

- Define and create acceptance testing.

## Introduction

The last type of testing we will be talking about is acceptance testing.
**Acceptance testing** is the last phase of functional testing. This type of
testing will make sure that all the layers of the API are tested together and
that the functionality that is expected by the end users of the API functions
properly.

We will continue to use our `spring-testing-demo` project to demonstrate how
to implement an acceptance test.

## Code-Along: Acceptance Testing in Spring

For acceptance testing, we want to validate that our functionality works in just
the same way as our users will end up using it. This means different things for
different types of functionality and applications, but in the case of our API
endpoints, it means we want to initialize the entire Spring Framework.

Let's start by creating a new test class for the `DemoController`. This time, we
will add `Acceptance` to the name of the test class to differentiate it from
our previous test classes. We can generate this new class just as we did before
by right-clicking in the blank space within the `DemoController` class -->
Choosing "Generate..." --> Then "Test...":

```java
package com.example.springtestingdemo.controller;

import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class DemoControllerAcceptanceTest {

    @Test
    void hello() {
    }

    @Test
    void getCatFact() {
    }
}
```

Now for this test, we want to imitate a real-world scenario. To do that, we want
to initialize the entire Spring Framework. To do this, we'll add the annotation,
`@SpringBootTest`, to the class definition. We'll also include the
`@AutoConfigureMockMvc` annotation to enable and configure auto-configuration of
`MockMvc`.

```java
package com.example.springtestingdemo.controller;

import org.junit.jupiter.api.Test;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;

import static org.junit.jupiter.api.Assertions.*;

@SpringBootTest
@AutoConfigureMockMvc
class DemoControllerAcceptanceTest { }
```

Like before, we'll autowire a `MockMvc` bean in our test class:

```java
package com.example.springtestingdemo.controller;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.web.servlet.MockMvc;

import static org.junit.jupiter.api.Assertions.*;

@SpringBootTest
@AutoConfigureMockMvc
class DemoControllerAcceptanceTest {

    @Autowired
    private MockMvc mockMvc;

    // other methods

}
```

And now we'll put our `MockMvc` instance to work by implementing a test for both
our endpoints.

```java
package com.example.springtestingdemo.controller;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.MvcResult;

import static org.hamcrest.Matchers.equalTo;
import static org.junit.jupiter.api.Assertions.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@SpringBootTest
@AutoConfigureMockMvc
class DemoControllerAcceptanceTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void hello() throws Exception {
        mockMvc.perform(get("/hello"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(content().string(equalTo("Hello World")));
    }

    @Test
    void getCatFact() throws Exception{

        // Get one cat fact from the external API
        MvcResult response = mockMvc.perform(get("/cat-fact"))
                .andDo(print())
                .andExpect(status().isOk())
                .andReturn();
        String firstFact = response.getResponse().getContentAsString();
        assertNotNull(firstFact);

        // Get a second cat fact from the external API
        response = mockMvc.perform(get("/cat-fact"))
                .andDo(print())
                .andExpect(status().isOk())
                .andReturn();
        String secondFact  = response.getResponse().getContentAsString();
        assertNotNull(secondFact);
        assertNotEquals(firstFact, secondFact);
    }
}
```

The main differences between this "Acceptance" test and our earlier
"Integration" test are:

1. We are now initializing the entire Spring Framework with the annotation
   `@SpringBootTest`, which means the request we make through these test methods
   will actually go through all the layers of the framework, as if they were
   coming from an actual external client.
2. We do not mock the actual service, and instead will be using the real service
   that Spring initializes for the controller.
3. We'll test the responses in the `getCatFact()` test method in a similar way
   to how we tested it in the `CatFactServiceIntegrationTest`. Since the cat
   facts that are being returned are "random", we'll need to check that they
   are not null and that consecutive calls return a different fact.

## Compare run times

Now that we have a set of Unit, Integration and Acceptance tests, let's run all
our tests together and compare run times. In my environment, the results are as
follows:

Now that we have a set of unit, integration, and acceptance tests, let's run all
our tests together and compare run times. To run all the JUnit tests, click on
the configurations bar at the top of the toolbar next to the "Run" (or play)
button and the "Build Project" (or build/hammer) button. Add the following
configuration:

![junit-configuration](https://curriculum-content.s3.amazonaws.com/spring-mod-2/testing/intellij-junit-configuration.png)

Name the configuration "Run All Tests" and make sure to choose "All in package"
from the drop-down menu. Once that has been done, click "Apply" and then "OK".

You should be able to run with that configuration to execute all the unit tests.
When you do so, notice the times it takes to execute each test.

Typically, we'll see that the unit tests are the "cheapest" to run (in terms of
time that it takes for them to run), then integration tests are second, and then
acceptance tests.

As we move up on the testing pyramid, tests are more and more costly to run.
General rules of thumb for tests that are higher up on the pyramid should:

- Run less frequently because they cost the developers more time.
- Be fewer tests. The more tests there are, the longer the whole test suite will
  need in order to run.
- Cover less scenarios/permutations of input for the same reasons.

This is why we try to cover as much functionality as possible with our unit
tests and focus our integration and acceptance tests on the specific integration
points that our unit tests could not (and should not) cover.

## Conclusion

We have fully covered how to test our small, MVC application in a fully
initialized Spring Framework with acceptance testing. In the next lesson, we'll
add Spring Data into the mix to see how that changes things.

## References

- [SpringBootTest Annotation Documentation](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/context/SpringBootTest.html)
- [AutoConfigureMockMvc Annotation Documentation](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/autoconfigure/web/servlet/AutoConfigureMockMvc.html)

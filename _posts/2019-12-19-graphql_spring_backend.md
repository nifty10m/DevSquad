---
layout: post
title: Write your first GraphQL Resolver / Endpoint using graphql-spring-boot
author: sja
author-jobtitle: GraphQL Pioneer
---

For an internal Project we wanted to try out a GraphQL-API to experiment with the current State of Technology and to evaluate the 
Developer Experience in either Front- and Backend.

There is a great Project available at [github/graphql-java-kickstart/graphql-spring-boot](https://github.com/graphql-java-kickstart/graphql-spring-boot) which literally kickstarts your first GraphQL-Steps.

# Dependencies
We included the following Dependencies:

```groovy
  implementation 'org.springframework:spring-websocket:5.2.2.RELEASE'
  implementation 'com.graphql-java-kickstart:graphql-spring-boot-starter:5.11.1'
  testImplementation 'com.graphql-java-kickstart:graphql-spring-boot-starter-test:5.11.1'
```

which is most of the necessary work before you're able to start writing your GraphQL-Schema.

We also included the following Dependencies to add some more Support:

```groovy
  //Add GraphiQL which adds a Playground-UI where you can write Queries and Mutations to test them  
  implementation 'com.graphql-java-kickstart:playground-spring-boot-starter:5.11.1'
  //Adds some java.time Scalare which you can use in your API    
  implementation 'com.zhokhov.graphql:graphql-datetime-spring-boot-starter:1.5.1'
```

# Schema
Next step is to write your schema in a `schema.graphqls` at the root of your project (default, can be changed):

``` 
scalar LocalDate
scalar UUID

type Query {
    employee(
        email: String!,
    ): Employee!
}

type Employee {
    id: ID!,
    firstName: String!,
    lastName: String!,
    email: String!,
    team: String,
    weeklyWorkload: Float!,
    projects(start: LocalDate, end: LocalDate): [Project]!,
}

type Project {
    id: ID!,
    title: String!,
    manager: Employee!
}
```

On Application start, the default GraphQL Schema Parser will look for the following Methods within all spring-beans Implementing the `com.coxautodev.graphql.tools.GraphQLQueryResolver`-Interface
* employee(String email)
* getEmployee(String email)

which fails at this point.

# Query Resolver
In order to implement the mentioned Query we need to provide a Bean which implements the QueryResolver and provides a method where the name equals the schema.query

```java
import de.xm.sample.graphql.domain.entities.Employee;
import de.xm.sample.graphql.entrypoints.web.graphql.resources.EmployeeResource;

import com.coxautodev.graphql.tools.GraphQLQueryResolver;
import lombok.RequiredArgsConstructor;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.stereotype.Service;

@RequiredArgsConstructor
@Service
public class EmployeeQueryResolver implements GraphQLQueryResolver {

    private final EmployeeRepository employeeRepository;

    @PreAuthorize("!isAnonymous()") 
    public EmployeeResource employee(String email) {

        return new EmployeeResource(employeeRepository.findByEmail(email)
            .orElseThrow(() -> Problems.notFound(Employee.class, email))
        );
    }
}
```

This Query Resolver will take the email and query the database for the employee. If an employee could be found, it will be mapped to a pojo matching the graphQL schema. If the Employee could not be found an Exception will be thrown.
At this point, we are able to answer the employee Query with Employee-Information. The good thing about graphQL is, that the Client-Developer 
is able to decide, which Information is relevant for his UseCase. 

We could now provide ALL the Information we defined in the Schema within this Query or we can use something called a `GraphQLResolver` to load this information when a client asks for it. 
In our example this would be the `projects(start: LocalDate, end: LocalDate): [Project]!` within the Employee Type.

This will greatly increase the GrapQL-Query Execution Time when the Client is not interested in Project-Information.

# GraphQLResolver
So we defined a some sort of subquery within the Employee Type called `projects(start: LocalDate, end: LocalDate): [Project]!`. 
Therefore we need to define this Query and `attach` it to the returning Resource of our QueryResolver

```java
@RequiredArgsConstructor
@Service
public class EmployeeResolver implements GraphQLResolver<EmployeeResource> {

    private final ProjectRepository projectRepository;

    public List<ProjectResource> projects(EmployeeResource employeeResource, LocalDate startDate, LocalDate endDate) {
        return projectRepositry.findActiveWithin(startDate, endDate)
            .stream()
            .map(ProjectResource::new)
            .collect(Collectors.toList());
    }
}
```

This Query will only be executed, when the Client asks for project-information and will have no impact otherwise.

# Custom Scalar
You are able to register Custom Scalre which you can use in your Model (e.g. UUID Scalar). You need to Provide a 
`graphql.schema.GraphQLScalarType` describing the Mapping from String to the designated Class

```java
@Configuration
public class GraphQlConfiguration {

    @Bean
    public GraphQLScalarType addScalarUuid() {
        return GraphQLScalarType
            .newScalar()
            .name("UUID")
            .description("java.util.UUID")
            .coercing(new Coercing<UUID, String>() {
                @Override
                public String serialize(Object dataFetcherResult) throws CoercingSerializeException {
                    if (dataFetcherResult instanceof UUID) {
                        return dataFetcherResult.toString();
                    } else {
                        throw new IllegalArgumentException("Unable to serialize " + dataFetcherResult
                            + " as UUID to String. Wrong Type. Expected UUID.class, Got " + dataFetcherResult.getClass()
                        );
                    }
                }

                @Override
                public UUID parseValue(Object input) throws CoercingParseValueException {
                    if (input instanceof String) {
                        return UUID.fromString((String) input);
                    } else {
                        throw new IllegalArgumentException("Unable to serialize " + input
                            + " as UUID to String. Wrong Type. Expected String.class, Got " + input.getClass()
                        );
                    }
                }

                @Override
                public UUID parseLiteral(Object input) throws CoercingParseLiteralException {
                    if (input instanceof StringValue) {
                        return UUID.fromString(((StringValue) input).getValue());
                    } else {
                        throw new IllegalArgumentException("Unable to serialize " + input
                            + " as UUID to String. Wrong Type. Expected StringValue.class, Got " + input.getClass()
                        );
                    }
                }
            })
            .build();

    }

}
```

# Wrap up
It is fairly easy to start with a GraphQL Backend. Like most API's the real work lays within defining the Schema. 
With the [github/graphql-java-kickstart/graphql-spring-boot](https://github.com/graphql-java-kickstart/graphql-spring-boot) 
you are able to remove most of the complexity of GraphQL for your Team and start working on the actual Business Logic.  

{% include twitter.html %}

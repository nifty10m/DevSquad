---
layout: post
title: Flyway only once per testrun
---

We use [flyway](https://flywaydb.org/) in our setup for creating the database. When running our test suite with `@SpringBootTest` Spring Boot try reusing of the application test. This is not possible if a test suite is annotaded with a dedicated port. In this case Spring Boot is starting a new application context.

Within out startup we are resetting the database by dropping all tables. This requires, due database constraints, the dropping and reinstallation of the flyway extension.
``` java
log.warn("Clearing database contents before executing flyway migrations");
// Postgres extensions have to be droped before flyway clean otherwise clean will fail
try (Connection connection = flyway.getDataSource().getConnection()) {
    try (Statement dropPostgis = connection.createStatement()) {
	dropPostgis.execute("DROP EXTENSION IF EXISTS postgis CASCADE");
    }
    try (Statement dropFuzzystrmatch = connection.createStatement()) {
	dropFuzzystrmatch.execute("DROP EXTENSION IF EXISTS fuzzystrmatch CASCADE");
    }
} catch (SQLException e) {
    throw new RuntimeException("Fatal error on database migration, unable to drop extensions", e);
}

flyway.clean();
try (Connection connection = flyway.getDataSource().getConnection()) {
    try (Statement createPostgis = connection.createStatement()) {
	createPostgis.execute("CREATE EXTENSION IF NOT EXISTS postgis");
    }
} catch (SQLException e) {
    throw new RuntimeException("Fatal error on database migration", e);
}
flyway.baseline();
flyway.migrate();
```
While this works fine when running once this seems to errorprone when running multiple times in a test suite and we get a `lookup cache type` error from the postgres database.

After investigating and searching a lot about this (rare) error, I just created a very simple solution using a static variable:
``` java
// Ensure database migration is done only once within JVM (fly fails with postgis otherwise)
static boolean migrated = false;

/**
* Custom flyway strategy that clears the database contents before executing migrations.
*/
@Bean
@ConditionalOnProperty(value = "flyway.clean-on-startup", havingValue = "true")
public FlywayMigrationStrategy cleanMigrationStrategy() {
return flyway -> {
    if (!migrated) {

	log.warn("Clearing database contents before executing flyway migrations");
	// Postgres extensions have to be droped before flyway clean otherwise clean will fail
	try (Connection connection = flyway.getDataSource().getConnection()) {
	    try (Statement dropPostgis = connection.createStatement()) {
		dropPostgis.execute("DROP EXTENSION IF EXISTS postgis CASCADE");
	    }
	    //...
	} catch (SQLException e) {
	    throw new RuntimeException("Fatal error on database migration, unable to drop extensions", e);
	}

	flyway.clean();
	try (Connection connection = flyway.getDataSource().getConnection()) {
	    try (Statement createPostgis = connection.createStatement()) {
		createPostgis.execute("CREATE EXTENSION IF NOT EXISTS postgis");
	    }
	} catch (SQLException e) {
	    throw new RuntimeException("Fatal error on database migration", e);
	}
	flyway.baseline();
	flyway.migrate();
	migrated = true;
    }
};
}
``` 

Static variables are only evaluated once within the JVM lifeteam. So flyway migration is not restarted (and tables not resetted) even if spring is restarting the application context.

{% include twitter.html %}

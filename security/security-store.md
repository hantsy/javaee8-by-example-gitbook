# IdentityStore

There are two built-in `IdentityStore` implementations provided in Glassfish v5, Database or Ldap.

An example of using built-in `@DatabaseIdentityStoreDefinition` to setup database based `IdentityStore`.

```java
@DatabaseIdentityStoreDefinition(
    dataSourceLookup = "${'java:global/MyDS'}",
    callerQuery = "#{'select password from caller where name = ?'}",
    groupsQuery = "select group_name from caller_groups where caller_name = ?",
    hashAlgorithm = Pbkdf2PasswordHash.class,
    priorityExpression = "#{100}",
    hashAlgorithmParameters = {
        "Pbkdf2PasswordHash.Iterations=3072",
        "${applicationConfig.dyna}"
    } // just for test / example
)
@ApplicationScoped
@Named
public class ApplicationConfig {

    public String[] getDyna() {
        return new String[]{"Pbkdf2PasswordHash.Algorithm=PBKDF2WithHmacSHA512", "Pbkdf2PasswordHash.SaltSizeBytes=64"};
    }

}
```

Initializes database with the initial users.

```java
@DataSourceDefinition(
        // global to circumvent https://java.net/jira/browse/GLASSFISH-21447
        name = "java:global/MyDS",
        className = "org.h2.jdbcx.JdbcDataSource",
        // :mem:test would be better, but TomEE insists on this being a file
        url = "jdbc:h2:mem:test;DB_CLOSE_ON_EXIT=FALSE"
)
@Singleton
@Startup
public class DatabaseSetup {

    @Resource(lookup = "java:global/MyDS")
    private DataSource dataSource;

    @Inject
    private Pbkdf2PasswordHash passwordHash;

    @Inject
    Logger LOG;

    @PostConstruct
    public void init() {

        LOG.info("initializing database...");

        Map<String, String> parameters = new HashMap<>();
        parameters.put("Pbkdf2PasswordHash.Iterations", "3072");
        parameters.put("Pbkdf2PasswordHash.Algorithm", "PBKDF2WithHmacSHA512");
        parameters.put("Pbkdf2PasswordHash.SaltSizeBytes", "64");
        passwordHash.initialize(parameters);

        executeUpdate(dataSource, "DROP TABLE IF EXISTS caller");
        executeUpdate(dataSource, "DROP TABLE IF EXISTS caller_groups");

        executeUpdate(dataSource, "CREATE TABLE IF NOT EXISTS caller(name VARCHAR(64) PRIMARY KEY, password VARCHAR(255))");
        executeUpdate(dataSource, "CREATE TABLE IF NOT EXISTS caller_groups(caller_name VARCHAR(64), group_name VARCHAR(64))");

        executeUpdate(dataSource, "INSERT INTO caller VALUES('user', '" + passwordHash.generate("password".toCharArray()) + "')");
        executeUpdate(dataSource, "INSERT INTO caller VALUES('reza', '" + passwordHash.generate("secret1".toCharArray()) + "')");
        executeUpdate(dataSource, "INSERT INTO caller VALUES('alex', '" + passwordHash.generate("secret2".toCharArray()) + "')");
        executeUpdate(dataSource, "INSERT INTO caller VALUES('arjan', '" + passwordHash.generate("secret2".toCharArray()) + "')");
        executeUpdate(dataSource, "INSERT INTO caller VALUES('werner', '" + passwordHash.generate("secret2".toCharArray()) + "')");

        executeUpdate(dataSource, "INSERT INTO caller_groups VALUES('user', 'foo')");
        executeUpdate(dataSource, "INSERT INTO caller_groups VALUES('user', 'bar')");

        executeUpdate(dataSource, "INSERT INTO caller_groups VALUES('reza', 'foo')");
        executeUpdate(dataSource, "INSERT INTO caller_groups VALUES('reza', 'bar')");

        executeUpdate(dataSource, "INSERT INTO caller_groups VALUES('alex', 'foo')");
        executeUpdate(dataSource, "INSERT INTO caller_groups VALUES('alex', 'bar')");

        executeUpdate(dataSource, "INSERT INTO caller_groups VALUES('arjan', 'foo')");
        executeUpdate(dataSource, "INSERT INTO caller_groups VALUES('werner', 'foo')");
    }

    @PreDestroy
    public void destroy() {
        try {
            executeUpdate(dataSource, "DROP TABLE IF EXISTS caller");
            executeUpdate(dataSource, "DROP TABLE IF EXISTS caller_groups");
        } catch (Exception e) {
            LOG.log(Level.SEVERE, "error drop tables:." + e.getMessage());

        }
    }

    private void executeUpdate(DataSource dataSource, String query) {
        try (Connection connection = dataSource.getConnection()) {
            try (PreparedStatement statement = connection.prepareStatement(query)) {
                statement.executeUpdate();
            }
        } catch (SQLException e) {
            LOG.log(Level.SEVERE, "error executed query:." + e.getMessage());
        }
    }

}
```

**Note**, we configure a `Pbkdf2PasswordHash` bean which is used to hash password.

Similar with `@DatabaseIdentityStoreDefinition`, there is a `@LdapIdentityStoreDefinition` for configuring users and groups in Ldap servers, such Microsoft Active Directory, [Apache Directory](http://directory.apache.org/).

You can also customize `IdentityStore` by implementing the `IdentityStore` interface.

```java
@ApplicationScoped
public class TestIdentityStore implements IdentityStore {

    public CredentialValidationResult validate(UsernamePasswordCredential usernamePasswordCredential) {

        if (usernamePasswordCredential.compareTo("user", "password")) {
            return new CredentialValidationResult("user", new HashSet<>(asList("foo", "bar")));
        }

        return INVALID_RESULT;
    }

}
```

Grab the [source codes](https://github.com/hantsy/ee8-sandbox) from my GitHub account, and have a try.


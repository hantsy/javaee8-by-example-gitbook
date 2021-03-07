# Bean Validation 2.0

Bean Validation 2.0 added more alignments with Java 8 and CDI, and bring a series of new annotations.

* Supports Java 8 Date time and `Optional`.
* Supports annotations applied on parameterized type, such as `List<@Email String> emails`.
* Add a series of new annotations, eg. `@Email`,`@NotEmpty`, `@NotBlank`, `@Positive`, `@PositiveOrZero`, `@Negative`, `@NegativeOrZero`, `@PastOrPresent` and `@FutureOrPresent`.

Let's create a simple example to experience it.

```text
public class Todo implements Serializable {

    @NotNull
    @NotBlank
    private String name;

    private String description;

    @PastOrPresent
    private LocalDateTime creationDate = LocalDateTime.now();

    @Future
    private LocalDateTime dueDate = LocalDateTime.now().plusDays(2);

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public LocalDateTime getCreationDate() {
        return creationDate;
    }

    public void setCreationDate(LocalDateTime creationDate) {
        this.creationDate = creationDate;
    }

    public LocalDateTime getDueDate() {
        return dueDate;
    }

    public void setDueDate(LocalDateTime dueDate) {
        this.dueDate = dueDate;
    }


}
```

Create a test to verify it.

```java
@RunWith(Arquillian.class)
public class TodoTest {

    @Deployment(name = "test")
    public static Archive<?> createDeployment() {

        JavaArchive archive = ShrinkWrap.create(JavaArchive.class)
                .addPackage(Todo.class.getPackage())
                //.addAsManifestResource("META-INF/test-persistence.xml", "persistence.xml")
                //.addAsResource("persons.json", "persons.json")
                .addAsManifestResource(EmptyAsset.INSTANCE, "beans.xml");
        // System.out.println(archive.toString(true));
        return archive;
    }

    @Inject
    Logger LOG;

    @Inject
    ValidatorFactory validatorFactory;

    @Inject
    Validator validator;

    @Test()
    public void testInvalidTodo() {
        Todo todo = new Todo();
        assertNull(todo.getName());
        Set<ConstraintViolation<Todo>> violations = validatorFactory.getValidator().validate(todo);
        LOG.log(Level.INFO, "violations.size ::" + violations.size());
        assertTrue(violations.size() > 0);
    }

    @Test()
    public void testInvalidTodo2() {
        Todo todo = new Todo();
        Set<ConstraintViolation<Todo>> violations = validator.validate(todo, Default.class);
        LOG.log(Level.INFO, "violations.size ::" + violations.size());
        assertTrue(violations.size() > 0);
    }

    @Test()
    public void testInvalidDueDate() {
        Todo todo = new Todo();
        todo.setName("test");
        todo.setDueDate(LocalDateTime.now());
        Set<ConstraintViolation<Todo>> violations = validator.validate(todo, Default.class);
        LOG.log(Level.INFO, "violations.size ::" + violations.size());
        assertTrue(violations.size() == 1);
    }

}
```

`Validator` or `ValidatorFactory` is available for injection, you can inject them into your beans as other CDI beans. `validate` method will return a collection of `ConstraintViolation`.

Grab the [source codes](https://github.com/hantsy/ee8-sandbox) from my GitHub account, and have a try.


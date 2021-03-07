# More CDI Alignments

In JPA 2.1, `EntityListener` supports `@Inject`\(some persistence providers are still problematic\). JPA 2.2 adds injection support in `@AttributeConverter`.

Let's reuse the [sample codes](https://github.com/hantsy/ee7-sandbox/wiki/jpa-converter) in my Java EE 7 sample.

The following is the original version for JPA 2.1, which is used to convert a tag list to string in database.

```java
@Converter
public class ListToStringConveter implements AttributeConverter<List<String>, String> {

    @Override
    public String convertToDatabaseColumn(List<String> attribute) {
        if (attribute == null || attribute.isEmpty()) {
            return "";
        }
        return StringUtils.join(attribute, ",");
    }

    @Override
    public List<String> convertToEntityAttribute(String dbData) {
        if (dbData == null || dbData.trim().length() == 0) {
            return new ArrayList<String>();
        }

        String[] data = dbData.split(",");
        return Arrays.asList(data);
    }
}
```

Extract the processing to a standalone CDI bean.

```java
@ApplicationScoped
public class ConverterUtils {

    public String listToString(List<String> tags) {
        return join(",", tags);
    }

    public List stringToList(String str) {
        return Arrays.asList(str.split(","));
    }
}
```

The new tag converter looks like.

```java
@Converter(autoApply = false)
public class TagsConverter implements AttributeConverter<List<String>, String> {

    @Inject
    Logger LOG;

    @Inject
    ConverterUtils utils;

    @PostConstruct
    public void postConstruct(){
        LOG.log(Level.INFO, "calling @PostConstruct");
    }

    @PreDestroy
    public void preDestroy(){
        LOG.log(Level.INFO, "calling @PreDestroy");
    }

    @Override
    public String convertToDatabaseColumn(List<String> attribute) {
        LOG.log(Level.FINEST, "utils injected: {0}", utils != null);
        if (attribute == null || attribute.isEmpty()) {
            return "";
        }
        return utils.listToString(attribute);
    }

    @Override
    public List<String> convertToEntityAttribute(String dbData) {
        if (dbData == null || dbData.trim().length() == 0) {
            return Collections.<String>emptyList();
        }
        return utils.stringToList(dbData);
    }

}
```

Apply it in the entity class.

```java
@Entity
public class Post implements Serializable {

    static enum Status {
        DRAFT, PUBLISHED;
    }

    @Id
    @GeneratedValue(strategy = AUTO)
    private Long id;

    private String title;

    private String content;

    @Enumerated(EnumType.STRING)
    private Status status = Status.DRAFT;

    @Convert(converter = TagsConverter.class)
    private List<String> tags = new ArrayList<>();

    private LocalDateTime publishedAt;

    private LocalDateTime createdAt;

    private LocalDateTime updatedAt;

    @Version
    private Long version;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }

    public Status getStatus() {
        return status;
    }

    public void setStatus(Status status) {
        this.status = status;
    }

    public List<String> getTags() {
        return tags;
    }

    public void setTags(List<String> tags) {
        this.tags = tags;
    }


    public LocalDateTime getPublishedAt() {
        return publishedAt;
    }

    public void setPublishedAt(LocalDateTime publishedAt) {
        this.publishedAt = publishedAt;
    }

    public LocalDateTime getCreatedAt() {
        return createdAt;
    }

    public void setCreatedAt(LocalDateTime createdAt) {
        this.createdAt = createdAt;
    }

    public LocalDateTime getUpdatedAt() {
        return updatedAt;
    }

    public void setUpdatedAt(LocalDateTime updatedAt) {
        this.updatedAt = updatedAt;
    }

    public Long getVersion() {
        return version;
    }

    public void setVersion(Long version) {
        this.version = version;
    }

    @PrePersist
    public void beforeSave() {
        setCreatedAt(LocalDateTime.now());
    }

    @PreUpdate
    public void beforeUpdate() {
        setUpdatedAt(LocalDateTime.now());
        if (PUBLISHED == this.status) {
            setPublishedAt(LocalDateTime.now());
        }
    }

    @Override
    public int hashCode() {
        int hash = 5;
        hash = 19 * hash + Objects.hashCode(this.title);
        hash = 19 * hash + Objects.hashCode(this.content);
        hash = 19 * hash + Objects.hashCode(this.status);
        return hash;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }
        if (obj == null) {
            return false;
        }
        if (getClass() != obj.getClass()) {
            return false;
        }
        final Post other = (Post) obj;
        if (!Objects.equals(this.title, other.title)) {
            return false;
        }
        if (!Objects.equals(this.content, other.content)) {
            return false;
        }
        if (this.status != other.status) {
            return false;
        }
        return true;
    }

    @Override
    public String toString() {
        return "Post{"
                + "id=" + id
                + ", title=" + title
                + ", content=" + content
                + ", status=" + status
                + ", publishedAt=" + publishedAt
                + ", createdAt=" + createdAt
                + ", updatedAt=" + updatedAt
                + '}';
    }

}
```

In the above use `@Convert` to apply it on tags field.

According to JPA 2.2 specification, this should work.

> Attribute converter classes in Java EE environments support dependency injection through the Contexts and Dependency Injection API \(CDI\) \[ 7 \] when CDI is enabled\[51\]. An attribute converter class that makes use of CDI injection may also define lifecycle callback methods annotated with the PostConstruct and PreDestroy annotations. These methods will be invoked after injection has taken place and before the attribute converter instance is destroyed respectively.

**NOTE**, But it does not work at the moment, see [the issue description in details on stackoverflow](https://stackoverflow.com/questions/46832385/inject-issues-in-attributeconverter-in-jpa-2-2-java-ee-8-glassfish-v5).

Grab the [source codes](https://github.com/hantsy/ee8-sandbox) from my GitHub account, and have a try.


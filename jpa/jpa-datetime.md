# Java 8 Datetime support

In the former version, there are some approaches to add Java Date time support by yourself.

* Custom `AttributeConverter`, I described [here](https://github.com/hantsy/ee7-sandbox/wiki/jpa-converter). Spring Data JPA also provides `AttributeConverter` for Java 8 DataTime supports.
* Unwrap `EntityManager` to provider's implementation, such as Hibernates `Session`, which has built-in [Java 8 Datetime support](https://github.com/hantsy/angularjs-ee7-sample/wiki/java8). 

JPA 2.2 adds following Java 8 Datetime APIs support by default.

* java.time.LocalDate
* java.time.LocalTime
* java.time.LocalDateTime
* java.time.OffsetTime
* java.time.OffsetDateTime

Create entity class.

```java
@Entity
public class Post extends AbstractEntity {

    public static enum Status {
        DRAFT, PUBLISHED;
    }

    private String title;

    private String content;

    @Enumerated(EnumType.STRING)
    private Status status = Status.DRAFT;

    private LocalDateTime publishedAt;

    private LocalDateTime createdAt;

    private LocalDateTime updatedAt;


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
                + "id=" + super.getId()
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

`AbstractEntity` is a `@MappedSuperclass` annotated base class, add shared id and version fields for all entities.

```java
@MappedSuperclass
public class AbstractEntity implements Serializable {

    private static final long serialVersionUID = 1L;

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name = "id")
    private Long id;

    @Version
    @Column(name = "version")
    private Long version;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public Long getVersion() {
        return version;
    }

    public void setVersion(Long version) {
        this.version = version;
    }
}
```

**NOTE**, we do not need to add `@Temporal` annotations on these fields

Grab the [source codes](https://github.com/hantsy/ee8-sandbox) from my GitHub account, and have a try.


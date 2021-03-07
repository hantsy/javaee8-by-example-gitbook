# Return Stream based result from Query

Hibernate has already added a lot of [alignments with Java 8](https://github.com/hantsy/angularjs-ee7-sample/wiki/java8), `Stream` and `Optional` are supported.

JPA 2.2 add official support for Stream result of query.

The following codes demonstrates query posts by keyword in JPA 2.1.

```java
public List<Post> findByKeyword(String keyword) {
    CriteriaBuilder cb = this.entityManager.getCriteriaBuilder();

    CriteriaQuery<Post> q = cb.createQuery(Post.class);
    Root<Post> c = q.from(Post.class);

    List<Predicate> predicates = new ArrayList<>();

    if (null != keyword && "".equals(keyword.trim())) {
        predicates.add(
                cb.or(
                        cb.like(c.get(Post_.title), '%' + keyword + '%'),
                        cb.like(c.get(Post_.content), '%' + keyword + '%')
                )
        );
    }

    q.where(predicates.toArray(new Predicate[predicates.size()]));

    TypedQuery query = this.entityManager.createQuery(q);

    return query.getResultList();
}
```

This can be simplified in JPA 2.2. The following is the complete codes of `PostRepository`.

```java
@Stateless
public class PostRepository extends AbstractRepository<Post, Long> {

    @PersistenceContext
    private EntityManager entityManager;

    public List<Post> findByKeyword(String keyword) {
        return stream().filter(p -> p.getTitle().contains(keyword) || p.getContent().contains(keyword))
                .collect(toList());
    }

    public long countByKeyword(String keyword) {
        return stream().filter(p -> p.getTitle().contains(keyword) || p.getContent().contains(keyword))
                .count();
    }

    @Override
    protected EntityManager entityManager() {
        return this.entityManager;
    }

}
```

And `AbstractRepository` is a generic Repository class can be reused for all entities.

```java
public abstract class AbstractRepository<T extends AbstractEntity, ID> {

    protected abstract EntityManager entityManager();

    private Class<T> entityClass() {
        ParameterizedType parameterizedType = (ParameterizedType) this.getClass().getGenericSuperclass();
        return (Class<T>) parameterizedType.getActualTypeArguments()[0];
    }

    public List<T> findAll() {

        CriteriaBuilder cb = this.entityManager().getCriteriaBuilder();

        CriteriaQuery<T> q = cb.createQuery(entityClass());
        Root<T> c = q.from(entityClass());

        return entityManager().createQuery(q).getResultList();
    }

    public T save(T entity) {
        if (entity.getId() == null) {
            entityManager().persist(entity);

            return entity;
        } else {
            return entityManager().merge(entity);
        }
    }

    public T findById(Long id) {
        return entityManager().find(entityClass(), id);
    }

    public void delete(T entity) {
        T _entity = entityManager().merge(entity);
        entityManager().remove(_entity);
    }

    public Stream<T> stream() {
        CriteriaBuilder cb = this.entityManager().getCriteriaBuilder();
        CriteriaQuery<T> q = cb.createQuery(entityClass());
        Root<T> c = q.from(entityClass());

        return entityManager().createQuery(q).getResultStream();
    }

    public Optional<T> findOptionalById(Long id) {
       return Optional.ofNullable(findById(id));
    }

}
```

Grab the [source codes](https://github.com/hantsy/ee8-sandbox) from my GitHub account, and have a try.


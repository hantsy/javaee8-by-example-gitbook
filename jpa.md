# JPA 2.2 

JPA 2.2 added more alignments with Java 8, such as Java 8 DateTime APIs as well as CDI improvments. 

Changed against Java 8 language improvements, a few annotaions in JPA supports `@Repeatable`.

* AssociationOverride
* AttributeOverride
* Convert
* JoinColumn
* MapKeyJoinColumn
* NamedEntityGraph
* NamedNativeQuery
* NamedQuery
* NamedStoredProcedureQuery
* PersistenceContext
* PersistenceUnit
* PrimaryKeyJoinColumn
* SecondaryTable
* SqlResultSetMapping

You do not need a collection concept annotaion to wrap these.

* [Java 8 Datetime](jpa-datetime.md)
* [Stream Result](jpa-streamresult.md)
* [CDI Alignments](jpa-cdi.md)
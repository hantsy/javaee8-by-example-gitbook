# JPA 2.2 

JPA 2.2 added more alignments with Java 8, such as Java 8 DateTime APIs as well as CDI improvments. 

Changed against Java 8 language improvements, a few annotations in JPA supports `@Repeatable`.

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

You do not need a collection concept annotation to wrap these.

Other updates including:

* [Java 8 Datetime support](jpa-datetime.md)
* [Return Stream based result from Query](jpa-streamresult.md)
* [More CDI Alignments](jpa-cdi.md)

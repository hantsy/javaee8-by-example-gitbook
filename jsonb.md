# JSON-B 1.0

JSON-P provides the lower JSON node operations, JSON Binding API provides advanced operations on serialization of Java objects to JSON string and deserialization of JSON string to Java objects.

## Serialization

An example to serilize a Java object to JSON string.

```java
Person duke = new Person("Duke", LocalDate.of(1995, 5, 23));
duke.setPhoneNumbers(
		Arrays.asList(
				new PhoneNumber(HOME, "100000"),
				new PhoneNumber(OFFICE, "200000")
		)
);

Jsonb jsonMapper = JsonbBuilder.create();
String json = jsonMapper.toJson(duke);

LOG.log(Level.INFO, "converted json result: {0}", json);
```		
You can read json value via `JsonPath`.

```java
String name = JsonPath.parse(json).read("$.name");
assertEquals("Duke", name);
```

`JsonPath` provides configuration APIs to configure the parse provider, eg. using `Gson` as backend json provider.

```java
Configuration config = Configuration.defaultConfiguration()
		.jsonProvider(new GsonJsonProvider())
		.mappingProvider(new GsonMappingProvider());
TypeRef<List<String>> typeRef = new TypeRef<List<String>>() {
};

List<String> numbers = JsonPath.using(config).parse(json).read("$.phoneNumbers[*].number", typeRef);

assertEquals(Arrays.asList("100000", "200000"), numbers);
```			

## Deserialization

Read the json string and map to an object directly.

```java
Jsonb jsonMapper = JsonbBuilder.create();

Person person = jsonMapper.fromJson(JsonbTest.class.getResourceAsStream("/person.json"), Person.class);

assertEquals("Duke", person.getName());

Type type = new ArrayList<Person>() {}
		.getClass()
		.getGenericSuperclass();

List<Person> persons = jsonMapper.fromJson(JsonbTest.class.getResourceAsStream("/persons.json"), type);

assertTrue(persons.size() == 2);
```
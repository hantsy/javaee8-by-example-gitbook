# JSON-P 1.1

JSON processing APIs is updated and aligned with Java 8, and provides Stream support for JSON reader.

Let's create an example to demonstrate it.

```java
public class Person implements Serializable {

    private String name;
    private LocalDate birthDate;
    private List<PhoneNumber> phoneNumbers = new ArrayList<>();

    // setters and getters
}

public class PhoneNumber implements Serializable {

    public static enum Type {
        HOME, OFFICE;
    }

    private Type type;
    private String number;

    // setters and getters
}
```

Assume we have a contact list in JSON file, we will read it and convert into `Person`.

```javascript
[
    {
        "name": "Duke",
        "birthDate": "1995-05-23",
        "phoneNumbers": [
            {
                "number": "100000",
                "type": "HOME"
            }, {
                "number": "200000",
                "type": "OFFICE"
            }
        ]
    }, 

    {
        "name": "Hantsy",
        "birthDate": "1978-01-01",
        "phoneNumbers": [
            {
                "number": "13812345678",
                "type": "HOME"
            }
        ]
    }
]
```

Convert `JsonArray` to `Stream` via `stream` method.

```java
@Test
public void testJsonStream() {
    JsonReader reader = Json.createReader(JsonpTest.class.getResourceAsStream("/persons.json"));
    List<String> nameList = reader.readArray().stream()
            .map(o -> o.asJsonObject().getJsonString("name").getString())
            .collect(toList());

    assertEquals(Arrays.asList("Duke", "Hantsy"), nameList);

}
```

JSON-P 1.1 also added [JSON Pointer](https://tools.ietf.org/html/rfc6901), [JSON Patch](https://tools.ietf.org/html/rfc6902), [JSON Merge Patch](https://tools.ietf.org/html/rfc7386) support.

An example of using JSON Pointer to query JSON node.

```java
JsonReader reader = Json.createReader(JsonpTest.class.getResourceAsStream("/persons.json"));

JsonArray arrays = reader.readArray();

JsonPointer p = Json.createPointer("/0/name");
JsonValue name = p.getValue(arrays);

System.out.println("json value ::" + name);
```

An example of using JSON Patch to update some JSON nodes.

```java
JsonReader reader = Json.createReader(JsonpTest.class.getResourceAsStream("/persons.json"));
JsonArray jsonaArray = reader.readArray();

JsonPatch patch = Json.createPatchBuilder()        
        .replace("/0/name", "Duke Oracle")
        .remove("/1")
        .build();

JsonArray result = patch.apply(jsonaArray);
System.out.println(result.toString());

Type type = new ArrayList<Person>() {}.getClass().getGenericSuperclass();

List<Person> person = JsonbBuilder.create().fromJson(result.toString(), type);
assertEquals("Duke Oracle", person.get(0).getName());
```

They are very useful to patch the existing entity when add HTTP Patch method support in RESTful APIs, we will demonstrate this later in JAX-RS.

Grab the [source codes](https://github.com/hantsy/ee8-sandbox) from my GitHub account, and have a try.


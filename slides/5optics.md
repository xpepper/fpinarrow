## Optics

<img style="height:500px" src="slides/images/optics.jpg">

---slide---

### Immutablity

* Kotlin data class

---slide---

### Some data classes

<pre>
<code class="hljs kotlin" style="max-height: 100%;font-size:150%">data class Person(val name: String, val city: City)

data class City(val name: String, val street : Street)

data class Street(val name : String)

</code>
</pre>

---slide---

### Changing a single value

<pre>
<code class="hljs kotlin" style="max-height: 100%;font-size:150%">val person = Person("MyName", City("MyCity", Street("MyStreet")))

val newPerson = person.copy(name = "MyNewName")

</code>
</pre>

---slide---

### Changing nested values
<pre>
<code class="hljs kotlin" style="max-height: 100%;font-size:150%">val updatedStreetPerson = person.copy(
  city = person.city.copy(
    street = person.city.street.copy(name = "MyNewStreet")
  )
)

</code>
</pre>

---slide---

### Have my cake and eat it too

<img style="height:500px;float:right;" src="slides/images/cake3.jpg">

* Immutablity
* Easy state transitions

---slide---

### Arrow Optics

* Lens
* Traversal

---slide---

#### Lens

* Looking at a propery of an object

<div>
<img style="height:500px" src="slides/images/lens.jpg">
</div>

---slide---

#### Creating a Lens

<pre>
<code class="hljs kotlin" style="max-height: 100%;font-size:150%" data-line-numbers="1-6|3|4">//Person > City
val cityLens: Lens&lt;Person, City> = Lens(
  get = { it.city },
  set = { person : Person, city : City -> person.copy(city = city) }
)

</code>
</pre>

---slide---

#### Using a Lens

<pre>
<code class="hljs kotlin" style="max-height: 100%;font-size:150%">val person = Person("MyName", City("MyCity", Street("MyStreet")))


val city: City = cityLens.get(person) //Get City property


val modifyCity = cityLens.modify(person) { //Modify City property

  c : City -> c.copy(name = "MyNewCity") 
	
}

</code>
</pre>

---slide---

#### More Lenses
<pre>
<code class="hljs kotlin" style="max-height: 100%;font-size:130%" data-line-numbers="1-5|6-10">//City > Street
val streetLens: Lens&lt;City, Street> = Lens(
  get = { it.street },
  set = { city: City, street: Street -> city.copy(street = street) }
)
//Street > name
val streetNameLens: Lens&lt;Street, String> = Lens(
  get = { it.name },
  set = { street: Street, name: String -> street.copy(name = name) }
)

</code>
</pre>	
---slide---
#### Composing Lenses

<pre>
<code class="hljs kotlin" style="max-height: 100%;font-size:130%">//Person > City > Street > name
val personStreetNameLens = cityLens
  .compose(streetLens)
  .compose(streetNameLens)


	
val updatedStreetPerson = personStreetNameLens.modify(person){ "MyNewStreet" }
//Person(name=MyName, city=City(name=MyCity, street=Street(name=MyNewStreet)))

</code>
</pre>	

---slide---

#### Generated Lens
<pre>
<code class="hljs kotlin" style="max-height: 100%;font-size:130%">@optics data class Person(val name: String, val city: City) {companion object}
@optics data class City(val name: String, val street: Street) {companion object}
@optics data class Street(val name: String) {companion object}

</code>
</pre>

---slide---

#### Generated Lens code

<pre>
<code class="hljs kotlin" style="max-height: 100%;font-size:130%" data-line-numbers="2">val person = Person("MyName", City("MyCity", Street("MyStreet")))
val updatedStreetPerson = Person.city.street.name.modify(person){"MyNewStreet"}

</code>
</pre>

---slide---

#### Traversal

<img style="height:500px" src="slides/images/traverse.jpg">

---slide---

#### Have an object with a list

<pre>
<code class="hljs kotlin" style="max-height: 100%;font-size:130%">data class Person(val name : String, val siblings : List&lt;Person>)

val person = Person("Bob",
  listOf(
    Person("bro", listOf()),
    Person("sis", listOf())
  )
)

</code>
</pre>

---slide---

#### Making lenses

<pre>
<code class="hljs kotlin" style="max-height: 100%;font-size:130%">//Person > siblings
val siblingsLens: Lens&lt;Person, List&lt;Person>> = Lens(
  get = { it.siblings },
  set = { s : Person, v : List&lt;Person> -> s.copy(siblings = v) }
)

//Person > name
val personNameLens: Lens&lt;Person, String> = Lens(
  get = { it.name },
  set = { p : Person, v : String -> p.copy(name = v) }
)

</code>
</pre>

---slide---

#### Update all siblings names

<pre>
<code class="hljs kotlin" style="max-height: 100%;font-size:150%">//Person > siblings > Traversal&lt;List&lt;Siblings> > name

val updateAllSiblingNames = siblingsLens

  .compose(Traversal.list())
	
  .compose(personNameLens)
	
  .modify(person) { "newSiblingName" }
	
//Person(name=test, siblings=[
//	Person(name=newSiblingName, siblings=[]), 
//	Person(name=newSiblingName, siblings=[])]
//)

</code>
</pre>

---slide---

#### Or on a specific index

<pre>
<code class="hljs kotlin" style="max-height: 100%;font-size:150%">//Person > siblings > Index&lt;List&lt;Siblings> > name

val updateFirstSiblinName = siblingsLens

  .compose(Index.list&lt;Person>().index(0))
	
  .compose(personNameLens)
	
  .modify(person) { "newSiblingName" }
	
//Person(name=test, siblings=[
//	Person(name=newSiblingName, siblings=[]), 
//	Person(name=sis, siblings=[])]
//)

</code>
</pre>

---slide---

#### Or using a filter

<pre>
<code class="hljs kotlin" style="max-height: 100%;font-size:150%">//Person > siblings > FilterIndex&lt;List&lt;Siblings> > name

val updateFirstSiblinName = siblingsLens

  .compose(FilterIndex.list&lt;Person>().filter { it == 0 })
	
  .compose(personNameLens)
	
  .modify(person) { "newSiblingName" }
	
//Person(name=test, siblings=[
//	Person(name=newSiblingName, siblings=[]), 
//	Person(name=sis, siblings=[])]
//)

</code>
</pre>

---slide---

## What have we learned

* Lenses allow us to see and modify an attribute
* Lenses compose
* Arrow generates them for you

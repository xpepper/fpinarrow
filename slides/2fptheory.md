## Just In Time FP Theory

<img style="height:500px" src="slides/images/jitfp.jpg">

---slide---

### What is FP

<ul>
<li class="fragment">Loving your compiler</li>
<li class="fragment">Loving your teammembers by being explicit</li>
<li class="fragment">Loving your cake</li>
</ul>

---slide---

### Function signatures

fun foo(a : String) : Int

String &rightarrow; Int

A &rightarrow; B

---slide---

### Error handling

<img style="height:500px" src="slides/images/errorhandling.jpg">

---slide---

### What is an error
* Invalid state
* Short circuit
* Recover or crash program/thread

---slide---

### Error handling in Kotlin

* Exceptions
* Nullability

---slide---

#### Exceptions

<pre>
<code class="hljs kotlin" style="max-height: 100%;font-size:150%">/**
 * @throws: ArithmeticException when b is 0.0
 */
fun divide(a : Double, b : Double) : Double = if(b == 0.0) {

  throw ArithmeticException("Cannot divide by 0")
	
} else {

  a/b 
	
}
</code>
</pre>

Double, Double &rightarrow; Double (or crash)

---slide---

#### Nullability

<pre>
<code class="hljs kotlin" style="max-height: 100%;font-size:130%">fun divide(a : Double, b : Double) : Double? = if(b == 0.0) null else a/b
</code>
</pre>


Double, Double &rightarrow; Double?

---slide---

#### Chaining the happy flow

<pre>
<code class="hljs kotlin" style="max-height: 100%;font-size:150%" data-line-numbers="1|3|5|7">val result : Double = divide(1.0, 0.0)

  ?.let { it * 2 }
		
  ?.let{ divide(it, 2.0) } 
		
  ?: 0.0
</code>
</pre>

---slide---

### Have my cake and eat it too

<img style="height:500px;float:right;" src="slides/images/cake1.jpg">

* Having an error context
* Error as a return type

---slide---

## Arrow

* FP library for Kotlin
* 1.0.0 release on September 20, 2021
* Has monads, improvements to coroutines, optics and metaprogramming
## The Either monad

---slide---

### Monads

<img style="height:500px" src="slides/images/wrapper.jpg">

---slide---

### Left vs Right

Either<Left, Right>

* Right is right
* Left is context

---slide---

### Implementation

<pre>
<code class="hljs kotlin" style="max-height: 100%;font-size:150%">sealed class Either&lt;out A, out B>

data class Left&lt;out A> : Either&lt;A, Nothing>()

data class Right&lt;out B> : Either&lt;Nothing, B>()

</code>
</pre>

---slide---

### Creating an either

<pre>
<code class="hljs kotlin" style="max-height: 100%;font-size:150%">fun divide(a: Double, b: Double): Either&lt;String, Double> = 
  if(b == 0.0) {

    Either.Left("Cannot divide by zero")
		
  } else {

    Either.Right(a / b)
		
  }

</code>
</pre>

---slide---

### Creating an either with extension functions

<pre>
<code class="hljs kotlin" style="max-height: 100%;font-size:150%">fun divide(a: Double, b: Double): Either&lt;String, Double> = 
  if(b == 0.0) {

    "Cannot divide by zero".left()
		
  } else {

    (a / b).right()
		
  }

</code>
</pre>

---slide---

### Creating an either with catch

<pre>
<code class="hljs kotlin" style="max-height: 100%;font-size:150%">fun divide(a : Double, b : Double): Either&lt;Throwable, Double> = 

  Either.catch { a / b }

</code>
</pre>

Double, Double &rightarrow; Either<Throwable, Double>

---slide---

### Transforming the left

<pre>
<code class="hljs kotlin" style="max-height: 100%;font-size:140%" data-line-numbers="5">fun divide(a : Double, b : Double): Either&lt;String, Double> = 

  Either.catch { a / b }
	
    .mapLeft { leftValue : Throwable -> "Cannot divide by zero" }

</code>
</pre>


Double, Double &rightarrow; Either<String, Double>

---slide---

### Transforming the right

<pre>
<code class="hljs kotlin" style="max-height: 100%;font-size:140%" data-line-numbers="5">fun divide(a : Double, b : Double): Either&lt;Throwable, String> = 
	
  Either.catch { a / b }
	
    .map { rightValue : Double -> rightValue.toString() }

</code>
</pre>

Double, Double &rightarrow; Either<Throwable, String>

---slide---

### Back to a value

<pre>
<code class="hljs kotlin" style="max-height: 100%;font-size:150%" data-line-numbers="1-16|1|3-4|5|7-11|12-16">val myEither: Either&lt;String, Double> = divide(10.0, 2.0)

//Ignore left
val orNull: Double? = myEither.orNull()
val orElse: Double = myEither.getOrElse { 0.0 }

//Or handle left
when(myEither){
  is Either.Left -> println(myEither.value) //String
  is Either.Right -> println(myEither.value * 2) //Double
}	
val orHandle: Double = myEither.getOrHandle { leftValue: String -> { 
    logger.error { leftValue }
    return 0.0
  } 
}

</code>
</pre>

---slide---

## Chaining the happy flow

<img style="height:500px" src="slides/images/chaining.jpg">

---slide---

### Composing / chaining
<pre>
<code class="hljs kotlin" style="max-height: 100%; font-size:150%" data-line-numbers="1-11|2,8|3,9|4,10|5,11">//Compose with Either
val result : Double = divide(6.0, 3.0) //Right(2.0)
  .map { right : Double -> right * 2 } //Right(4.0)
  .flatMap { right : Double -> divide(right, 2.0) } //Right(2.0)
  .getOrElse { 0.0 } // 2.0

//Compose with Nullability
val result : Double = divide(6.0, 2.0) // 2.0
  ?.let { it * 2 } // 4.0
  ?.let{ divide(it, 2.0) } // 2.0
  ?: 0.0 // 2.0

</code>
</pre>
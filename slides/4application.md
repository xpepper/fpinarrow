## Real world eithers

Let's build an application

---slide---

## The Repositories

<pre>
<code class="hljs kotlin" style="max-height: 100%;font-size:150%">class SpeakerRepository {
  fun getSpeaker(name : String) : Speaker = TODO()
}

class ConferenceRepository {
  fun getConference(name: String): Conference = TODO()
}

class BookingRepository {
  fun store(booking : Booking) : Long = TODO()
}

</code>
</pre>

---slide---

## The service

<pre>
<code class="hljs kotlin" style="max-height: 100%;font-size:150%">fun createBooking(speakerName: String, conferenceName: String): Long {

  val speaker = speakerRepository.getSpeaker(speakerName)
	
  val conference = conferenceRepository.getConference(conferenceName)
	
  val b = Booking(speaker, conference)
	
  return bookingRepository.store(b)
}

</code>
</pre>

---slide---

## The Controller

<pre>
<code class="hljs kotlin" style="max-height: 100%;font-size:150%">fun createBooking(@RequestBody booking : Booking): ResponseEntity&lt;Any> {

  val createBooking: Long = myService.createBooking(
	
    speakerName = booking.speakerName,
		
    conferenceName = booking.conferenceName
		
  )
	
  return ResponseEntity.ok(createBooking)
}

</code>
</pre>

---slide---

## but what about errors

<img style="height:500px" src="slides/images/errors.jpg">

---slide---

## Exceptions

<pre>
<code class="hljs kotlin" style="max-height: 100%;font-size:140%" data-line-numbers="2,8-12">fun createBooking(@RequestBody booking : Booking): ResponseEntity&lt;Any> {
  try {
    val createBooking: Long = myService.createBooking(
      speakerName = booking.speakerName,
      conferenceName = booking.conferenceName
    )
    return ResponseEntity.ok(createBooking)
  } catch ( e : NotFoundException){
    return ResponseEntity.status(404).body(e.message)
  } catch ( e : Throwable){
    return ResponseEntity.internalServerError().build()
  }
}

</code>
</pre>

---slide---

## Introducing Eithers

---slide---

## Creating the left

<pre>
<code class="hljs kotlin" style="max-height: 100%;font-size:140%">sealed class BadState {

  data class NotFound(val reason : String) : BadState()
	
  data class BadStateWithException(val throwable : Throwable) : BadState()
}

</code>
</pre>

---slide---

## Return the Either
<pre>
<code class="hljs kotlin" style="max-height: 100%;font-size:140%">class SpeakerRepository {
  fun getSpeaker(name : String) : Either&lt;BadState, Speaker> = TODO()
}

class ConferenceRepository {
  fun getConference(name : String) : Either&lt;BadState, Conference> = TODO()
}

class BookingRepository {
  fun store(booking : Booking) : Either&lt;BadState, Long> = TODO()
}

</code>
</pre>

---slide---

## Bringing the either back to a value
<pre>
<code class="hljs kotlin" style="max-height: 100%; font-size:150%" data-line-numbers="1-18|2-5|6-19|7|8-18|10-12|13-16|1-20">fun booking(@RequestBody booking : Booking): ResponseEntity&lt;Any> {
  val createBooking: Either&lt;BadState, Long> = createBooking(
    speakerName = booking.speakerName,
    conferenceName = booking.conferenceName
  )
  return when(createBooking){
    is Either.Right -> ResponseEntity.ok(createBooking.value)
    is Either.Left -> {
      when (val badState = createBooking.value) {
        is BadState.NotFound -> {
          ResponseEntity.status(404).body(badState.reason)
        }
        is BadState.BadStateWithException -> {
          logger.error(badState.throwable) { "Something went wrong" }
          ResponseEntity.internalServerError().build()
        }
      }
    }
  }
}

</code>
</pre>

---slide---

## Composing the either

<pre>
<code class="hljs kotlin" style="max-height: 100%; font-size:140%" data-line-numbers="1-12|3|4|5-7|8-9|1-12">fun createBooking(speakerName: String, 
				  conferenceName: String): Either&lt;BadState, Long> {
  val speakerE = speakerRepository.getSpeaker(speakerName)
  val conferenceE = conferenceRepository.getConference(conferenceName)
  return speakerE
    .flatMap { speaker ->
      conferenceE.flatMap { conference ->
        val booking = Booking(speaker, conference)
        return bookingRepository.store(booking)
      }
    }
}

</code>
</pre>

---slide---

### Have my cake and eat it too

<img style="height:500px;float:right;" src="slides/images/cake2.jpg">

* Compiler checked error handling
* Readable code

---slide---

## Monad Comprehension

Making monads easy

---slide---

## Either Comprehension

<pre>
<code class="hljs kotlin" style="max-height: 100%;font-size:150%" data-line-numbers="2,10|3|5|7|9|1">suspend fun foo(): Either&lt;BadState, String> = 
  either {
    val myEither: Either&lt;BadState, String> = "".right()
		
    val rightValue: String = myEither.bind()
		
    println(rightValue)
		
    rightValue //return
  }

</code>
</pre>

---slide---

## Either Comprehension without suspend

<pre>
<code class="hljs kotlin" style="max-height: 100%;font-size:150%" data-line-numbers="">fun foo(): Either&lt;BadState, String> = 
  either.eager {
    ...
  }

</code>
</pre>


---slide---

## Either Comprehension

<pre>
<code class="hljs kotlin" style="max-height: 100%;font-size:120%" data-line-numbers="1-8|3|4|5|6">fun createBooking(speakerName: String, conferenceName: String): Either&lt;BadState, Long> {
  return either.eager {
    val speaker = speakerRepository.getSpeaker(speakerName).bind()
    val conference = conferenceRepository.getConference(conferenceName).bind()
    val b = Booking(speaker, conference)
    bookingRepository.store(b).bind() //return
  }
}

</code>
</pre>

---slide---

## What have we learned

* Compile time safety using Eithers
* Imperative programming using Either comprehension
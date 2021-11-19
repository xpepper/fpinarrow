#### Prism

---slide---

#### Prism

<pre>
<code class="hljs kotlin" style="max-height: 100%;font-size:150%">sealed class Result
data class Success(val value : String) : Result()
data class Failure(val value : Throwable) : Result()
</code>
</pre>

---slide---

#### Creating a prism

<pre>
<code class="hljs kotlin" style="max-height: 100%;font-size:150%" data-line-numbers="1-8|2-6|7">val prism : Prism&lt;Result, Success> = Prism(
	getOrModify = { when (it) {
			is Success -> it.right()
			else -> it.left()
		}
	},
	reverseGet = { it }
)
</code>
</pre>

---slide---

#### Composing a prism

<pre>
<code class="hljs kotlin" style="max-height: 100%;font-size:130%" data-line-numbers="1-12|1-4|6|8-9|11-12">val lens : Lens&lt;Success, String> = Lens(
	get = { it.value },
	set = { success : Success, value : String -> success.copy(value=value) }
)

val prismLens = prism.compose(lens)

val success : Result = Success("test")
val failure : Result = Failure(RuntimeException())

val updatedSuccess: Result = prismLens.modify(success) { "test2" }
val updatedFailure = prismLens.modify(failure) { "" } //Does nothing
</code>
</pre>

---slide---

#### Generated Prism

<pre>
<code class="hljs kotlin" style="max-height: 100%;font-size:130%">@optics sealed class Result { companion object }
@optics data class Success(val value : String) : Result() { companion object }
@optics data class Failure(val value : Throwable) : Result() { companion object }
</code>
</pre>

---slide---

#### Generated Prism usage

<pre>
<code class="hljs kotlin" style="max-height: 100%;font-size:150%">val success : Result = Success("test")
Result.success.value.modify(success){ "test2" }
</code>
</pre>

---slide---
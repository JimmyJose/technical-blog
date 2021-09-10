---
layout: post
title:  "Tiny URL - Base62 Encoding"
date:   2021-09-09 20:14:00 -0700
category: SystemDesign
---
In this blog, we will be discussing one of the approaches used for generating a unique tiny url.

Essentially given a long URL, we need to generate a unique tiny url. For generating such a unique tiny URL, base62 encoding can be used.

**Base 62:**
It is comprised of the following characters: a-z (26), A-Z (26), 0-9 (10), in total 62 characters.

If we decide to use upto 7 characters for representing a tiny url using base62, then we can generate upto ~3.6 (62^7) trillion unique urls, which is quite realistically speaking more than enough for most cases.

#### Tiny URL Generation
1. Request for tiny url generation contains the long URL
2. Create a new record in DB and store the long URL
3. Base 62 encode the id of the new DB record and store the encoded value in the database
4. Return the tinyURL in the response

##### DB Table for registering the tiny URL
<table class="table table-dark">
  
  <thead>
    <tr>
      <th scope="col">ID</th>
      <th scope="col">LONG_URL</th>
      <th scope="col">HASH</th>
      <th scope="col">CREATED_AT</th>
      <th scope="col">EXPIRES_ON</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td scope="row">111118899</td>
      <td scope="row">https://stackoverflow.com/nullpointerexception-in-java-with-no-stacktrace</td>
      <td scope="row">hGpgt</td>
      <td scope="row">123445555555</td>
      <td scope="row">123445555555</td>
    </tr>
  </tbody>
</table>

**Generated Short URL:** https://my.tinyurl.com/hGpgt

<br>

#### Java Code
{% highlight java %}
package com.jj.enterprise.problems;

public class Base62Encoder {
	
	private static final String BASE62_CHARACTER_SET = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
	private static final char[] SUPPORTED_CHARACTERS = BASE62_CHARACTER_SET.toCharArray();
	private static final int BASE = SUPPORTED_CHARACTERS.length;
	
	/**
	 * Encode the Id to base62
	 */
	public String encode(long id) {
		if (id <= 0) throw new IllegalArgumentException("Id cannot be a 0 or negative number!");
		
		StringBuilder encoded = new StringBuilder();
		while (id > 0) {
			encoded.append(SUPPORTED_CHARACTERS[(int)(id % BASE)]);
			id = id / BASE;
		}
		return encoded.reverse().toString();
	}

	/**
	 * Decode the base62 encoded string
	 */
	public long decode(String encoded) {
		if (encoded == null || encoded.isBlank()) throw new IllegalArgumentException("null or empty string as input is invalid.");
		
		int index = encoded.length() - 1;
		int position = 0;
		long decoded = 0;
		while (index >= 0) {
			char character = encoded.charAt(index--);
			decoded += BASE62_CHARACTER_SET.indexOf(character) * Math.pow(BASE, position++);
		}
		return decoded;
	}
	
	private static final Base62Encoder INSTANCE = new Base62Encoder();
	private Base62Encoder() {}
	public static Base62Encoder getInstance() {
		return INSTANCE;
	}
}
{% endhighlight %}

#### Unit Test
{% highlight java %}
package com.jj.enterprise.problems;

import static org.hamcrest.CoreMatchers.is;
import static org.hamcrest.MatcherAssert.assertThat;

import java.util.stream.Stream;

import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.Arguments;
import org.junit.jupiter.params.provider.MethodSource;

public class Base62EncoderTest {

	@Test
	public void givenZeroForEncoding_ThenItIsAnUnsupportedInput() {
		Assertions.assertThrows(IllegalArgumentException.class, () -> {
			Base62Encoder.getInstance().encode(0);
		});
	}
	
	@Test
	public void givenNegativeValueForEncoding_ThenItIsAnUnsupportedInput() {
		Assertions.assertThrows(IllegalArgumentException.class, () -> {
			Base62Encoder.getInstance().encode(-123);
		});
	}
	
	@Test
	public void givenNullForDecoding_ThenItIsAnUnsupportedInput() {
		Assertions.assertThrows(IllegalArgumentException.class, () -> {
			Base62Encoder.getInstance().decode(null);
		});
	}
	
	@Test
	public void givenEmptyStringForEncoding_ThenItIsAnUnsupportedInput() {
		Assertions.assertThrows(IllegalArgumentException.class, () -> {
			Base62Encoder.getInstance().decode("");
		});
	}
	
	@Test
	public void givenBlankStringForEncoding_ThenItIsAnUnsupportedInput() {
		Assertions.assertThrows(IllegalArgumentException.class, () -> {
			Base62Encoder.getInstance().decode("    ");
		});
	}
	
	@ParameterizedTest
	@MethodSource("base64_encoding_mapping")
	public void givenValidInputsForEncoding_ThenTheyAreEncodedRight(
			long id, 
			String expected) {
		assertThat(Base62Encoder.getInstance().encode(id), is(expected));
	}
	
	@ParameterizedTest
	@MethodSource("base64_encoding_mapping")
	public void givenValidInputsForDncoding_ThenTheyAreDecodedRight(
			long expected, 
			String decoded) {
		assertThat(Base62Encoder.getInstance().decode(decoded), is(expected));
	}
	
	private static Stream<Arguments> base64_encoding_mapping() {
		return Stream.of(
				Arguments.arguments(1, "b"),
				Arguments.arguments(10, "k"),
				Arguments.arguments(100, "bM"),
				Arguments.arguments(123456, "Gho"),
				Arguments.arguments(10345432000L, "lsixCC"),
				Arguments.arguments(3500000000000L, "9MzntHo")
		);
	}
}
{% endhighlight %}
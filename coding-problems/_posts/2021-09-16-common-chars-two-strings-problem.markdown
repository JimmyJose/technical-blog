---
layout: post
title:  "Common Characters Between Two Strings Problem"
date:   2021-09-16 14:02:00 -0700
category: CodingProblem
---
Given two strings, determine characters common between them and return a comma separated string containing those characters.

If there are no characters common between the given strings then return an empty string

**Note:** The characters in the output string can be in any order.

**Example 1:**
{% highlight plaintext %}
Input:
	a = "afgeegkffpqrt"
	b = "zxfgeefgkil"

output: "f,g,e,k"
{% endhighlight %}

**Example 2:**
{% highlight plaintext %}
Input:
	a = "abcdef"
	b = "ghijklm"

output: ""
{% endhighlight %}
  

### Solution
Lets look at a few options for solving this problem.

#### Brute Force
Brute force approach for solving this problem would essentially be iterate through one of the strings one character at a time and in the second inner loop search for this character. Not the most efficient approach but works. 

Lets look at the implementation.
{% highlight java %}
	public String findUniqueCommonCharacters_bruteForce(String candidate1, String candidate2) {
		validateInput(candidate1, candidate2);
		Set<Character> uniqueCommonCharacters = new HashSet<>();
		for (char c1 : candidate2.toCharArray()) {
			for (char c2 : candidate1.toCharArray()) {
				if (c1 == c2) {
					uniqueCommonCharacters.add(c1);
					break;
				}
			}
		}
		return build(uniqueCommonCharacters);
	}

	private String build(Set<Character> set) {
		if (set == null || set.isEmpty()) return "";
		StringJoiner joiner = new StringJoiner(",");
		set.forEach(item -> joiner.add(String.valueOf(item)));
		return joiner.toString();
	}

	private void validateInput(String input1, String input2) {
		if (StringUtils.isBlank(input1) || StringUtils.isBlank(input2)) {
			throw new IllegalArgumentException();
		}
	}
{% endhighlight %}
##### Complexity:
- Time: O(n*m)
- Space: O(n)

Where 'n' is the length of the longer input string and 'm' is the length of the other input string.

#### Optimization using two sets
The brute force approach is not at all optimal. Lets see how we can make the time complexity linear.
The solution essentially stores the characters from each of the strings in individual sets and then
performs an intersection, which essentially gives the common character set. next we just build a string and return.

Lets take a look at the implementation.

{% highlight java %}
	public String findUniqueCommonCharactersUsingSets(String candidate1, String candidate2) {
		validateInput(candidate1, candidate2);
		Set<Character> set1 = convert(candidate1);
		Set<Character> set2 = convert(candidate2);
		set1.retainAll(set2);
		return build(set1);
	}

	private Set<Character> convert(String input) {
		final Set<Character> set = new HashSet<>();
		for (char c : input.toCharArray()) {
			set.add(c);
		}
		return set;
	}
{% endhighlight %}

##### Complexity:
- Time: O(n)
- Space: O(n)

Where 'n' is the length of the longer input string.

#### More optimal approach?
Since we need to scan both the input strings, we cannot get more optimal then linear time.
However, we can definitely optimize from the space perspective!

For the following solution, we are assuming the following things to keep things simple:
- Input string is comprized of just lower case letters ranging from a-z
- We do not consider the output string as additional space

Lets look at the implementation:
{% highlight java %}
public String findUniqueCommonCharacters(String candidate1, String candidate2) {
		validateInput(candidate1, candidate2);
		
		// Initialized with 0's by default in Java
		int[] characters = new int[26]; 
		for (char c : candidate1.toCharArray()) {
			characters[c - 'a']++;
		}
		
		StringBuilder builder = new StringBuilder();
		for (char c : candidate2.toCharArray()) {
			if (characters[c - 'a'] > 0) {
				// So that we skip duplicates
				characters[c - 'a'] = 0; 
				builder.append(c).append(",");
			}
		}
		return builder.isEmpty()? "" : builder.toString().substring(0, builder.length() - 1);
	}
{% endhighlight %}

##### Complexity:
- Time: O(n)
- Space: O(1)

Where 'n' is the length of the longer input string and 'm' is the length of the other input string.

**Note:** The characters array is of a fixed size (in this example 26) so that is constant space
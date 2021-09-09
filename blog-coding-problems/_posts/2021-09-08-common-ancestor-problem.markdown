---
layout: post
title:  "Common Ancestor Problem"
date:   2021-09-08 14:01:53 -0700
category: CodingProblem
---
Given an array containing parent and child relationship, determine if two numbers have a common ancestor.

**Example:**
{% highlight plaintext %}
     1     2
    / \   / \
   /   \ /   \
  3     4     9
   \   / \
    \ /   \
     5     6
      \   /
       \ /
        7

relationship =
    [
      [1, 3], 
      [1, 4], 
      [2, 4], 
      [2, 9], 
      [3, 5], 
      [4, 5], 
      [4, 6], 
      [5, 7], 
      [6, 7]
    ]

Looking at the input we can see that:
    3 and 6 have a common ancestor (1)
    4 and 9 have a common ancestor (2)
    3 and 9 do not have a common ancestor
  {% endhighlight %}
  

### Solution
**Approach 1:**
1. Build a child to parent lookup map using the relationship array for easy lookup.
2. Get the set of all the ancestors of the first candidate (a)
3. Get the set of all the ancestors of the second candidate (b)
4. Get an intersection of the above two sets:
    - If the intersection exists, then they have a common ancestor
    - Otherwise, there is no common ancestor

**Note:** To determine all the ancestors for a given individual, we essentially perform a BFS (breadth first search).
We use a stack datastructure.

### Java code
{% highlight java %}
package com.jj.enterprise.problems;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.HashMap;
import java.util.HashSet;
import java.util.List;
import java.util.Map;
import java.util.Set;
import java.util.Stack;

public class AncestryProblem {
	/**
	 * Method returns true if a common ancestor exists, false otherwise
	 */
	public boolean hasCommonAncestor(int[][] parentChildPairs, int individualA, int individualB) {
		if (parentChildPairs == null || parentChildPairs.length == 0) return false;
		
		Map<Integer, List<Integer>> parentsLookupMap = buildParentsLookupMap(parentChildPairs);
		Set<Integer> aAncestors = getAllAncestors(individualA, parentsLookupMap);
		Set<Integer> bAncestors = getAllAncestors(individualB, parentsLookupMap);	
		aAncestors.retainAll(bAncestors);
		return !aAncestors.isEmpty();
	}
	
	private Map<Integer, List<Integer>> buildParentsLookupMap(int[][] relationship) {
		final Map<Integer, List<Integer>> pairsMap = new HashMap<>();
		
		Arrays.stream(relationship).forEach(pair -> {
			int parent = pair[0];
			int child = pair[1];
			
			List<Integer> parents = pairsMap.get(child);
			if (parents == null) {
				parents = new ArrayList<>();
				pairsMap.put(child, parents);
			}
			parents.add(parent);
			pairsMap.put(child, parents);
		});
		
		return pairsMap;
	}
	
	private Set<Integer> getAllAncestors(int individual, Map<Integer, List<Integer>> mapping) {
		Set<Integer> allAncestors = new HashSet<>();
		Stack<Integer> stack = new Stack<>();
		// Add the immediate parents of the individual to the stack
		stack.addAll(mapping.get(individual));
		
		while (!stack.isEmpty()) {
			int size = stack.size();
			for (int i = 0; i < size; i++) {
				int val = stack.pop();
				allAncestors.add(val);
				List<Integer> ancestors = mapping.get(val);
				
				if (ancestors != null && !ancestors.isEmpty()) {
					// Add the parent's parent of they exist to the stack
					stack.addAll(ancestors);
				}
			}
		}
		
		return allAncestors;
	}
	
	/** Private constructor for enforcing singleton */
	private AncestryProblem() {}
	private static final AncestryProblem INSTANCE = new AncestryProblem();
	public static AncestryProblem getInstance() {
		return INSTANCE;
	}
}
{% endhighlight %}

### Unit Test
{% highlight java %}
package com.jj.enterprise.problems;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.Arguments;
import org.junit.jupiter.params.provider.MethodSource;

import static org.hamcrest.CoreMatchers.is;
import static org.hamcrest.MatcherAssert.assertThat;

import java.util.stream.Stream;

public class AncestryProblemTest {
	
	@Test
	public void givenNoRelationship_ThenThereIsNoCommonAncestor() {
		assertThat(AncestryProblem.getInstance().hasCommonAncestor(null, 1, 3), is(false));
	}
	
	@ParameterizedTest
	@MethodSource("common_ancestor_inputs")
	public void givenValidRelationShips_ThenCommonAncestorDeterminationIsAccurate(
			int[][] relationship, 
			int individualA, 
			int individualB, 
			boolean expected) {
		assertThat(AncestryProblem.getInstance().hasCommonAncestor(relationship, individualA, individualB), is(expected));
	}
	
	private static Stream<Arguments> common_ancestor_inputs() {
		final int[][] relationship = {
				{1, 3},
				{1, 4},
				{2, 4},
				{2, 9},
				{3, 5},
				{4, 5},
				{4, 6},
				{5, 7},
				{6, 7}
		};
		
		return Stream.of(
				Arguments.arguments(relationship, 3, 6, true),
				Arguments.arguments(relationship, 4, 9, true),
				Arguments.arguments(relationship, 7, 9, true),
				
				Arguments.arguments(relationship, 3, 9, false)
		);
	}
}
{% endhighlight %}

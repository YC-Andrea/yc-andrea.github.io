---
layout: post
title: "Building an Element Combination Game"
subtitle: "Little Alchemy Clone Game in Python"
date:       2022-10-20 12:00:00
author: "Yingqian Cao"
header-style: text
catalog:      true
tags:
  - CMU
  - Python
---

## Introduction

In this blog post, I'll walk through my implementation of a Little Alchemy-style element combination game developed for [Carnegie Mellon University's 15112 course (Fundamentals of Programming)](https://www.cs.cmu.edu/~112/syllabus.html). This project involved creating a game where players combine basic elements to discover new ones, complete with visual feedback and strategic gameplay elements.
![](/img/CombinationGame_gif.mov)


## Project Overview

Game features includes:  
1. 435 combinable elements loaded from CSV files  
2. Drag-and-drop interface for combining elements  
3. Visual feedback with element icons  
4. "Next possible elements" hint system  
5. Dark mode toggle  
6. Workspace management  

## Key Implementation Details  

### 1. Data Loading System  
Implemented functions to load element and reaction data from CSV files:
```ts
PROCEDURE LoadElementProperties(fileContent):
    // Initialize empty element database
    elementData ← CREATE key-value storage
    // Process each data record
    FOR EACH line IN fileContent:
        // Split into identifier and properties
        PARSE line INTO (elementName, propertyValue) USING comma delimiter
        // Skip header row
        IF elementName ≠ "Name":
            // Store element characteristics
            elementData[elementName] ← propertyValue
    RETURN elementData
END PROCEDURE
```
```ts
PROCEDURE LoadInteractionRules(fileContent):
    // Initialize reaction registry
    reactionMap ← CREATE compound-key storage
    // Process each interaction rule
    FOR EACH line IN fileContent:
        // Split into components
        PARSE line INTO (elementA, elementB, reactionOutcome) USING comma
        // Skip header row
        IF elementA ≠ "Element1":
            // Register bidirectional interaction
            reactionMap[(elementA, elementB)] ← reactionOutcome
            reactionMap[(elementB, elementA)] ← reactionOutcome
    RETURN reactionMap
END PROCEDURE
```

### 2. Core Game Logic  
Checked if two elements can react:
```ts
PROCEDURE DetermineElementInteraction(elementA, elementB, reactionDatabase):
    // Check primary combination order
    IF (elementA + elementB) combination EXISTS IN reactionDatabase:
        RETURN reactionDatabase[elementA][elementB]
    // Check reverse combination order
    ELSE IF (elementB + elementA) combination EXISTS IN reactionDatabase:
        RETURN reactionDatabase[elementB][elementA]
    // No valid interaction found
    ELSE:
        RETURN "No Reaction"
END PROCEDURE
```

### 3. Strategic Hint System  
Implemented functions to help players discover new elements:
```ts
PROCEDURE FindPossibleSynthesis(currentElements, reactionRules):
    // Generate all possible new elements from current inventory
    potentialElements ← EMPTY SET
    FOR EACH pair (elementA, elementB) IN currentElements:
        reactionResult ← CHECK interaction BETWEEN elementA AND elementB
        IF valid reaction EXISTS:
            ADD reaction product TO potentialElements
    RETURN potentialElements
END PROCEDURE
```
```ts
PROCEDURE DetermineOptimalSynthesis(currentElements, reactionRules):
    // Phase 1: Potential candidates identification
    candidateElements ← FindPossibleSynthesis(currentElements, reactionRules)
    IF candidateElements IS EMPTY: RETURN NO_OPTION
    // Phase 2: Future potential evaluation
    maxFuturePotential ← 0
    bestCandidates ← EMPTY LIST
    FOR EACH candidate IN candidateElements:
        TEMPORARILY ADD candidate TO currentElements
        futureOptions ← FindPossibleSynthesis(updatedElements, reactionRules)
        futureCount ← NUMBER OF UNIQUE ITEMS IN futureOptions
        IF futureCount > maxFuturePotential:
            RESET maxFuturePotential TO futureCount
            RESET bestCandidates TO [candidate]
        ELSE IF futureCount == maxFuturePotential:
            ADD candidate TO bestCandidates
        REMOVE candidate FROM currentElements
    // Phase 3: Final selection logic
    IF bestCandidates NOT EMPTY:
        SORT bestCandidates ALPHABETICALLY
        RETURN FIRST candidate IN sorted list
    ELSE:
        RETURN NO_OPTION
END PROCEDURE
```


## Technical Challenges & Solutions  

### Challenge 1: Efficient Element Combination Checking
**Problem:** Needed to quickly check if any two elements could combine, considering both orderings.  
**Solution:** Implemented a dictionary with tuple keys and optimized lookup:
```ts
DEFINE REACTION_RULES:
    // Element Pair → Resulting Compound
    REACTION earth + fire → lava
    REACTION air + air → pressure
    REACTION air + fire → energy
END DEFINITION
```

### Challenge 2: Dynamic Hint Generation
**Problem:** Calculating possible next elements became computationally expensive with large sets.  
**Solution:**   
Using sets for O(1) lookups  
Implementing memoization  
Only calculating hints when explicitly requested  

### Challenge 3: Visual Feedback System
**Problem:** Needed to show which elements were "terminal" (couldn't be combined further).  
**Solution:** Implemented terminal element detection and visually distinguished them with underlined text in the UI:  
```ts
PROCEDURE IdentifyTerminalElement(targetElement, reactionDatabase):
    // Compile all elements that can initiate reactions
    activeElements ← EMPTY SET
    // Process all known chemical interactions
    FOR EACH interaction IN reactionDatabase:
        // Extract components needed for reaction
        primaryComponent ← interaction[0]
        secondaryComponent ← interaction[1]
        // Record elements with reactive potential
        ADD primaryComponent TO activeElements
        ADD secondaryComponent TO activeElements
    // Terminal determination logic
    IF targetElement NOT FOUND IN activeElements:
        RETURN TRUE  // Element cannot participate in any reactions
    ELSE:
        RETURN FALSE // Element has reactive capabilities
END PROCEDURE
```


## Lessons Learned
This project gave me valuable experience in game architecture, user interface design, and data processing. The complete implementation demonstrates how relatively simple systems can combine to create an engaging gameplay experience.


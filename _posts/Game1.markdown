---
layout: post
title: "Building an Element Combination Game"
subtitle: "Little Alchemy Clone"
date:       2022-09-20 12:00:00
author: "Yingqian Cao"
header-style: text
catalog:      true
tags:
  - CMU
  - Python
---

## Introduction

In this blog post, I'll walk through my implementation of a Little Alchemy-style element combination game developed for [Carnegie Mellon University's 15112 course (Fundamentals of Programming)](https://www.cs.cmu.edu/~112/syllabus.html). This project involved creating a game where players combine basic elements to discover new ones, complete with visual feedback and strategic gameplay elements.
![](/img/Game1.png)


## Project Overview

Game features:  
435 combinable elements loaded from CSV files  
Drag-and-drop interface for combining elements  
Visual feedback with element icons  
"Next possible elements" hint system  
Dark mode toggle  
Workspace management  

## Key Implementation Details  

### 1. Data Loading System  
Implemented functions to load element and reaction data from CSV files:  
```ts
def loadElementData(filename):
    d = {}
    for line in fileString.splitlines():
        key,value = line.split(",")
        if key != "Name":
            d[key] = value
    return d
def loadReactionData(filename):
    d = {}
    for line in fileString.splitlines():
        elem1,elem2,value = line.split(",")
        if elem1 != "Element1":
            d[elem1,elem2] = value
    return d
```

### 2. Core Game Logic  
Checked if two elements can react:  
```ts
def tryReaction(elem1, elem2, reactionsDict):
    if (elem1,elem2) in reactionsDict:
        return reactionsDict[elem1, elem2]
    elif (elem2,elem1) in reactionsDict:
        return reactionsDict[elem2, elem1]
    else:
        return None
```

### 3. Strategic Hint System  
Implemented functions to help players discover new elements:  
```ts
def allNextElements(elementSet, reactionsDict):
    myset = set()
    for elem1 in elementSet:
        for elem2 in elementSet:
            if tryReaction(elem1, elem2, reactionsDict) != None:
                myset.add(tryReaction(elem1, elem2, reactionsDict))
    return myset
def bestNextElement(elementSet, reactionsDict):
    maxLen = 0
    mylist = []
    myset = allNextElements(elementSet, reactionsDict)
    # Find element that enables most future combinations
    for factor in myset:
        elementSet.add(factor)
        if len(allNextElements(elementSet, reactionsDict)) > maxLen:
            maxLen = len(allNextElements(elementSet, reactionsDict))
        elementSet.remove(factor)
    # Collect all elements that achieve this maximum
    for factor in myset:
        elementSet.add(factor)
        if len(allNextElements(elementSet, reactionsDict)) == maxLen:
            mylist.append(factor)
        elementSet.remove(factor)
    return sorted(mylist)[0] if mylist else None
```


## Technical Challenges & Solutions  

### Challenge 1: Efficient Element Combination Checking  
Problem: Needed to quickly check if any two elements could combine, considering both orderings.  
Solution: Implemented a dictionary with tuple keys and optimized lookup:  
```ts
reactionsDict = {
    ("earth","fire"):"lava",
    ("air","air"):"pressure",
    ("air","fire"):"energy"
}
```

### Challenge 2: Dynamic Hint Generation  
Problem: Calculating possible next elements became computationally expensive with large sets.  
Solution:   
Using sets for O(1) lookups  
Implementing memoization  
Only calculating hints when explicitly requested  

### Challenge 3: Visual Feedback System  
Problem: Needed to show which elements were "terminal" (couldn't be combined further).  
Solution: Implemented terminal element detection and visually distinguished them with underlined text in the UI:  
```ts
def isTerminalElement(elem, reactionsDict):
    possibleElements = set()
    for key in reactionsDict:
        possibleElements.update(key)
    return elem not in possibleElements
```


## Lessons Learned  
Data-Driven Design: Storing game data in CSVs made it easy to modify and expand the element set.  
Performance Matters: Even simple games need optimization (e.g., for the hint system).  
Good UI Makes Games Fun: Small touches like drag effects and visual feedback significantly improve playability.  
State Management is Key: Tracking element positions, selections, and the toolbox scroll position required careful state management.  
This project gave me valuable experience in game architecture, user interface design, and data processing. The complete implementation demonstrates how relatively simple systems can combine to create an engaging gameplay experience.


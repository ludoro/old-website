# Coding interview preparation

## Table of Contents
1. [Array and Hashing](#arrayhashing)


## Array and Hashing {#arrayhashing}

### 1. Contains duplicate
Given an integer array nums, return true if any value appears at least twice in the array, and return false if every element is distinct.

```python
def containsDuplicate(nums: List[int]) -> bool:
    # O(n) time, O(n) space
    mySet = set()
    for num in nums:
        if num in mySet:
            return True
        else:
            mySet.add(num)
    return False
```

### 2. Valid anagram
Given two strings s and t, return true if t is an anagram of s, and false otherwise.
An Anagram is a word or phrase formed by rearranging the letters of a different word or phrase, typically using all the original letters exactly once.

```python
def isAnagram(s: str, t: str) -> bool:
    lettersInS = {}
    for letter in s:
        if letter in lettersInS:
            lettersInS[letter] += 1
        else:
            lettersInS[letter] = 1
    
    # Removing letters
    for letter in t:
        if letter not in lettersInS:
            return False
        else:
            lettersInS[letter] -= 1
    
    # Checking if letters are all 0.
    for letter in lettersInS:
        if lettersInS[letter] != 0:
            return False
    return True
```

### 3. Replace elements with greatest Element on right side
Given an array arr, replace every element in that array with the greatest element among the elements to its right, and replace the last element with -1.

After doing so, return the array.

```python
def replaceElements(arr: List[int]) -> List[int]:
    if len(arr) <= 1:
        return [-1]
    if len(arr) == 2:
        return [arr[-1],-1]

    currentMax = arr[-1]
    arr[-1] = -1
    i = len(arr) - 2
    while i >= 0:
        currentVal = arr[i]
        arr[i] = currentMax
        currentMax = max(currentMax, currentVal)
        i -= 1
    return arr
```

### 4. Is subsequence
Given two strings s and t, return true if s is a subsequence of t, or false otherwise.

A subsequence of a string is a new string that is formed from the original string by deleting some (can be none) of the characters without disturbing the relative positions of the remaining characters. (i.e., "ace" is a subsequence of "abcde" while "aec" is not).

```python
def isSubsequence(s: string, t: str) -> List[int]:
    if len(s) > len(t):
        return False
    if not s or not t:
        return False
    if s == "" and t:
        return True
    indexFirst = 0
    indexSecond = 0
    lengthFirst = len(s)
    lengthSecond = len(t)
    while indexFirst < lengthFirst and indexSecond < lengthSecond:
        if s[indexFirst] == t[indexSecond]:
            indexFirst += 1
            indexSecond += 1
        else:
            indexSecond += 1
    return indexFirst == len(s)

```

### 5. Length of last word
Given a string s consisting of words and spaces, return the length of the last word in the string.
A word is a maximal substring consisting of non-space characters only.

```python
def lengthOfLastWord(s: string) -> List[int]:
    N = len(s)
    i = N - 1
    letterCounter = 0
    while i >= 0:
        if (s[i] != ' ' and letterCounter == 0):
            pass
        if (s[i] != ' '):
            letterCounter += 1
        if (s[i] == ' '):
            return letterCounter
        i -= 1
    return letterCounter
```

### 6. Two sum
Given an array of integers nums and an integer target, return indices of the two numbers such that they add up to target.
You may assume that each input would have exactly one solution, and you may not use the same element twice.
You can return the answer in any order.

```python
def twoSum(nums: List[int], target: int) -> List[int]:
    mapDiffIndex = {}
    for i, num in enumerate(nums):
        if target - num in savingResults:
            return [mapDiffIndex[target - num], i]
        mapDiffIndex[num] = i
    return [-1]
```
### 7. Longest Common Prefix
Write a function to find the longest common prefix string amongst an array of strings.

If there is no common prefix, return an empty string "".

```python
def longestCommonPrefix(strs: List[str]) -> str:
    # Only care about two least words -> sort
    strs.sort()
    firstWord = strs[0]
    lastWord = strs[-1]
    index = 0
    while index < min(len(firstWord),len(lastWord)):
        if firstWord[index] != lastWord[index]:
            break
        index += 1
    return firstWord[index:]
```

### 7. Group Anagrams
Given an array of strings strs, group the anagrams together. You can return the answer in any order.

An Anagram is a word or phrase formed by rearranging the letters of a different word or phrase, typically using all the original letters exactly once.

```python
def longestCommonPrefix(strs: List[str]) -> dict[str]:


```

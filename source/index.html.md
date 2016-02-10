---
title: Boostbot API Reference

language_tabs:
  - C#

toc_footers:
  - <a href='https://boostbot.org'>For use with Boostbot</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the Boostbot API documentation page.  blah blah blah some stuff here

# Project Setup

Create a new project of type <code>Class Library</code> (dll)

## References

You will need to reference the boostbot application (.exe), Core.dll, and Shared.dll.  All of these can be found in the bots main folder.

# Addon Attributes

## Addon

In order for your addon to be picked up by the bot, you must include the Addon attribute on your namespace

### Contructor

`assembly: Addon(string addOnName, string description, string author)`

```c#
[assembly: Addon("Nameof Addon", "This is a sample addon", "BoostBotTeam")]
```
### Parameters
Parameter | Description
--------- | ------- 
addOnName | Name of your addon 
description | A short description of your addon
author | You!

<aside class="notice">
Remember — this is NOT a description of your algorithm, just your addon
</aside>


## AttackAlgorithm

For each algorithm, you must in include the AttackAlgorithm Attribute on your class

### Constructor

`AttackAlgorithm(string name, string description)`

```c#
[AttackAlgorithm("MyDeploy", "Deploys units in a smiley face pattern.")]
```

### Parameters

Parameter | Description
--------- | ------- 
name | Name of your attack algorithm
description | A short description of your algorithm

<aside class="notice">
Your algorithm class must implement <code>BaseAttack</code>
</aside>

# BaseAttack implementation

When implementing BaseAttack, add the following methods

## Initial Constructor


Your BaseAttack class will accept a <code>BaseStats</code> parameter.

```c#
public class MyAlgorithm(BaseStats baseStats): base(baseStats){}
```

## ToString

Override ToString() for debugging purposes

```c#
public override string ToString()
{
    return "My deploy algorithm"
}
```

## ShouldAccept

<code>public override double ShouldAccept()</code>

The <code>ShouldAccept</code> method returns a double, which is the score of how successful your algorithm believes it will be, ranging from 0 to 1

## AttackRoutine

<code>public override IEnumerable&lt;int&gt; AttackRoutine()</code>

This is your actual attack / deploy phase. It returns an int which is the number of milliseconds the bot should allow for this routine.

## Sample initial Setup
```c#
[assembly: Addon("My Addon", "This is a sample addon", "BoostBotTeam")]
namespace MyDeploy
{
    [AttackAlgorithm("My Algorithm", "Deploys units in a smiley face pattern.")]
    public class MyAlgorithm : BaseAttack
    {
        public MyAlgorithm(BaseStats baseStats) : base(baseStats)
        {
        }

        public override string ToString()
        {
            return "My deploy algorithm";
        }

        public override double ShouldAccept()
        {
            if (!MeetsRequirements(BaseStats))
                return 0;
            return .5;
        }

        public override IEnumerable<int> AttackRoutine()
        {
            yield return 100;
        }
    }
}
```

<aside class="warning">Inside HTML code blocks like this one, you can't use Markdown, so use <code>&lt;code&gt;</code> blocks to denote code.</aside>




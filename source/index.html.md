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
Your algorithm class must extend <code>BaseAttack</code>
</aside>

<aside class="warning">Inside HTML code blocks like this one, you can't use Markdown, so use <code>&lt;code&gt;</code> blocks to denote code.</aside>



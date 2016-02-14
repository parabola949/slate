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

This returns the name of your deployment. This is used to display the name that appears in "Attack" tab of the bot.

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
            return 1;
        }

        public override IEnumerable<int> AttackRoutine()
        {
            yield return 100;
        }
    }
}
```

# Basic Methods

Use these methods to get started with an attack.

## MeetsRequirements

<code>protected bool MeetsRequirements(BaseStats baseStats)</code>

This method is used to check if the base the bot is currently on meets the requirements set by the user on the "Attack" tab. Use it in the <code>ShouldAccept</code> method.

Parameter | Description
--------- | -----------
BaseStats | An object with the stats of the current enemy base. This is passed to your algorithm in the constuctor.

```c#
public override double ShouldAccept()
{
    if (!MeetsRequirements(BaseStats))
        return 0; \\skip this base
    return 1; \\attack this base
}
```

## GetAvailableDeployElements

<code>public static List&lt;DeployElement&gt; GetAvailableDeployElements()</code>

This method returns a list of the troops and spells currently available to deploy using the DeployElement object. Use it in the <code>AttackRoutine</code> method.

```c#
public override IEnumerable<int> AttackRoutine()
{
	//get the deploy elements that have unit data
	var deployElements = AttackHelper.GetAvailableDeployElements().Where(x => x.UnitData != null);

	//get only the tank units from the deploy elements
    var tankUnits = deployElements.Where(x => x.ElementType == DeployElementType.NormalUnit && x.UnitData.AttackType == AttackType.Tank).ToArray();

	//get only the attacking units from the deploy elements
    var attackUnits = deployElements.Where(x => x.ElementType == DeployElementType.NormalUnit && x.UnitData.AttackType == AttackType.Damage).ToArray();

	//get only the healing units from the deploy elements
    var healUnits = deployElements.Where(x => x.ElementType == DeployElementType.NormalUnit && x.UnitData.AttackType == AttackType.Heal).ToArray();

	//TODO: deploy units to screen.
}
```

### DeployElement

Property | Type | Description
-------- | ---- | -----------
Name | string | The name used to identify the element
Rect | (object)Rectangle | The rectangle outlining the location of the unit on the screen.
Count | int | The number of available elements
UnitData | (object)Unit | An object with information about the unit
ElementType | (enum)DeployElementType | The type of element; Elements types are: NormalUnit, HeroKing, HeroQueen, HeroWarden, Spell, ClanTroops
IsRanged | bool | Determines if the element is a ranged unit.
IsHero | bool | Determines if the element is a hero.

<code>public void Recount()</code>

This method will recount the element. It's useful after deploying troops to see what is left.

### Unit

Property | Type | Description
-------- | ---- | -----------
AffectedTargets | (enum)AffectedTargets | Flags showing the Units favorite target: Ground, Air, Any, AirGround, Buildings, Walls, BuildingsWalls
AmountPerPulse | int | 
AttackSpeed | double | Unit attack speed
AttackType | (enum)AttackType | Type of unit: Damage, Wallbreak, Tank, Heal
BoostSeconds | int |
BuildingLevelRequired | int |
BuildingType | (enum)BuildingType | Building used to queue: Barracks, DarkBarracks, SpellFactory, DarkSpellFactory
DamageIncreasePercentage | int |
DarkElixirCost | int | Dark elixir cost to queue
DPS | double | Damage per second
EffectType | (enum)EffectType | Damage type: Single, Splash
ElixirCost | int | Elixir cost to queue
GoldCost | int | Gold cost to queue
HousingSpace | int | Number of space used for each unit
HP | double | HP for the unit
LaboratoryLevelRequired | int | Level of the lab required
Level | int | Current unit level
MovementSpeed | double | Movement speed of the unit
Name | string | Full name of the unit
NameSimple | string | Simple name for the unit
NumberOfPulses | int | 
RandomRange | double | 
Range | double | Range of the unit
ResearchCost | int | Cost to research the next level
SecondsBetweenPulses | double |
SpeedIncreasePercentage | int |
SpellDurationSeconds | double |
SpellType | (enum)SpellType | Type of spell: Offensive, Support, Utility
SplashRange | double |
TargetType | (enum)TargetType | Target type of the unit: None, Defense, AirDefense, Loot, Walls
TrainingButton | (object)Point | Location of the training button on the screen
TrainingTime | double | Length of time to train
Type | (enum)ItemType | Type of item: Unit, Spell
UnitType | (enum)UnitType | Type of unit: Ground, Air
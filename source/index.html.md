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

# Getting Started

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
	return 0; //skip this base
return 1; //attack this base
}
```

## GetAvailableDeployElements

<code>public static List&lt;DeployElement&gt; GetAvailableDeployElements()</code>

This method returns a list of the troops and spells currently available to deploy using the DeployElement object. Use it in the <code>AttackRoutine</code> method.

```c#
public override IEnumerable<int> AttackRoutine()
{
//get the deploy elements that have unit data
var deployElements = GetAvailableDeployElements().Where(x => x.UnitData != null);

//get only the tank units from the deploy elements
var tankUnits = deployElements.Where(x => x.ElementType == DeployElementType.NormalUnit && x.UnitData.AttackType == AttackType.Tank).ToArray();

//get only the attacking units from the deploy elements
var attackUnits = deployElements.Where(x => x.ElementType == DeployElementType.NormalUnit && x.UnitData.AttackType == AttackType.Damage).ToArray();

//get only the healing units from the deploy elements
var healUnits = deployElements.Where(x => x.ElementType == DeployElementType.NormalUnit && x.UnitData.AttackType == AttackType.Heal).ToArray();

//TODO: deploy units to screen.
}
```



# PluginBase Class

## PluginBase Constructors

Name | Description
---- | -----------
PluginBase() | Initializes a new instance of the PluginBase class

## PluginBase Properties

Property | Type | Description
-------- | ---- | -----------
AttackOnlyDeadBases | bool | Gets the user's attack setting for attack only dead bases
TrophyPushMode | bool | Gets the user's setting for trophy push mode
TrophyPushThDistanceLimit | int |  Gets the user's setting for trophy push max townhall distance to border
WaveDelay | double | Gets the user's setting for wave delay
WaveSize | int | Gets the user's setting for wave size
WaveTrapSize | int | Gets the user's setting for wave trap size
UseKing | bool | Gets the user's setting for using the Barbarian King
UseQueen | bool | Gets the user's setting for using the Archer Queen
UseWarden | bool | Gets the user's settings for using the Grand Warden
UseClanTroops | bool | Gets the user's settings for using clan troops
SaveAttackAnalysisImage | bool | Gets the user's setting for saving an attack analysis image
DisplayAttackAnalysisImage | bool | Gets the user's setting for displaying an attack analysis image
RedPoints | List&lt;Point&gt; | Gets a list of coordinates for the red border around a base
BottomEnd | int | 

## PluginBase Structures

### ActiveSearch Structure

#### ActiveSearch Properties

Property | Type | Description
-------- | ---- | -----------
NeedOnlyOneRequirementForAttack | bool | Gets the user's setting for meet only one requirement for active bases
MinGold | int | Gets the user's setting for the minimum gold requirement for active bases
MinElixir | int | Gets the user's setting for the minimum elixir requirement for active bases
MinDarkElixir | int | Gets the user's setting for the minimum dark elixir requirement for active bases
MaxThLevel | int | Gets the user's setting for the maximum townhall level requirement for active bases

#### ActiveSearch Methods

Method | Returns | Description
------ | ------- | -----------
MeetsRequirements(BaseStats) | bool | Determines if the current base meets the active base requirements

### DeadSearch Structure

#### DeadSearch Properties

Property | Type | Description
-------- | ---- | -----------
NeedOnlyOneRequirementForAttack | bool | Gets the user's setting for meet only one requirement for dead bases
MinGold | int | Gets the user's setting for the minimum gold requirement for dead bases
MinElixir | int | Gets the user's setting for the minimum elixir requirement for dead bases
MinDarkElixir | int | Gets the user's setting for the minimum dark elixir requirement for dead bases
MaxThLevel | int | Gets the user's setting for the maximum townhall level requirement for dead bases

#### DeadSearch Methods

Method | Returns | Description
------ | ------- | -----------
MeetsRequirements(BaseStats) | bool | Determines if the current base meets the dead base requirements

### DeployElementType Structure

#### DeployElementType Properties

Property | Type | Description
-------- | ---- | -----------
NormalUnit | DeployElementType | Gets the DeployElementType enum for NormalUnit
HeroKing | DeployElementType | Gets the DeployElementType enum for HeroKing
HeroQueen | DeployElementType | Gets the DeployElementType enum for HeroQueen
HeroWarden | DeployElementType | Gets the DeployElementType enum for HeroWarden
Spell | DeployElementType | Gets the DeployElementType enum for Spell
ClanTroops | DeployElementType | Gets the DeployElementType enum for ClanTroops

### DeployPointA Structure

#### DeployPointA Properties

Property | Type | Description
-------- | ---- | -----------
Top | Point | Gets the deploy point for the top of the base
Left | Point | Gets the deploy point for the left of the base
Right | Point | Gets the deploy point for the right of the base
Bottom | Point | Gets the deploy point for the bottom of the base

## PluginBase Methods

Method | Returns | Description
------ | ------- | -----------
MeetsRequirements(BaseStats baseStats) | bool | Determines if the current base meets the requirements based on if it is considered dead or alive
ToUnitString(List&lt;DeployElement&gt; unitCounts) | string | Returns a string with the name and count of each unit
OrderUnitsForDeploy(List&lt;DeployElement&gt; units) | void | Orders units for deployment; Tank > Wallbreaker > Heal > Damage > Heroes
GetPointsForLine(Point p1, Point p2, int count) | List&lt;Point&gt; | Returns a list of points with the specified count along the line created by two specified points
GetRectPoints(int pointsPerSide) | List&lt;Point&gt; | Returns a list of the number of points per side specified along the outside rectangle of the current enemy base
GetStorageAttackPoints(List&lt;Point&gt; redLinePoints) | Point[] | Returns an array of deploy points near storages based on the specified list of red line points
ExtractHeroes(List&lt;DeployElement&gt; units, List&lt;DeployElement&gt; heroes) | void | Extracts heroes from the specified units list and adds them to the specified heroes list
DeployHeroes(List&lt;DeployElement&gt; heroes, IEnumerable&lt;Point&gt; deployPoints, bool activateHeroAbilities) | IEnumerable&lt;int&gt; | Deploys the specified heroes on the specified deploy points.
TryActivateHeroAbilities(List&lt;DeployElement&gt; heroes, bool forceActivate) | void |
SurrenderDeployMethod() | IEnumerable&lt;int&gt; |
Surrender() | void |
SurrenderIfWeHaveAStar() | bool |
HaveAStar() | bool |
GetTrophyPushDeployPoints(TrophyPushOpponentAnalysis trophyPushAnalysis) | Point[] |
DrawAnalysis(List&lt;Point&gt; deployPoints, List&lt;Point&gt; redLinePoints) | void |
GetAvailableDeployElements() | List&lt;DeployElement&gt; |
GenerateDeployLinesFromSettings() | Tuple&lt;Point, Point&gt;[] |
AreTroopSetsDifferent(List&lt;DeployElement&gt; a, List&lt;DeployElement&gt; b) | bool |
ClickAlongLine(Point p1, Point p2, int count, int sleepTime) | void |
CheckForTownhallNearBorder() | TrophyPushOpponentAnalysis |
GenerateDeployPointsFromMines(List&lt;Point&gt; deployPoints, IEnumerable&lt;Point&gt; redLinePoints, List&lt;Rectangle&gt; detectedMines) | IEnumerable&lt;int&gt; |
GenerateDeployPointsFromMinesToMilk(List&lt;Point&gt; deployPoints, List&lt;Point&gt; redLinePoints, List&lt;Rectangle&gt; detectedMines, bool fullGold, bool fullElixir, bool fullDelixir) | IEnumerable&lt;int&gt; |
GetOutsideCollectorCount(List&lt;Point&gt; redLinePoints) | int |
GetResourcesState() | ResourcesFull |
GetAttackResources() | int[] |
DeployUnits(DeployElement[] units, Point[] deployPoints, int clickDelay, int waveCount, int waveDelay, int firstCycleDelay) | IEnumerable&lt;int&gt; |
DeployUnitsPerPoint(DeployElement[] units, Point[] deployPoints, int unitsPerPoint, int clickDelay, int cycleDelay) | IEnumerable&lt;int&gt; |
WaitForNoResourceChange(double seconds) | IEnumerable&lt;int&gt; |

```c#
	bool MeetsRequirements(BaseStats baseStats)
	string ToUnitString(List<DeployElement> unitsCounts)
	void OrderUnitsForDeploy(List<DeployElement> units)
	List<Point> GetPointsForLine(Point p1, Point p2, int count)
	List<Point> GetRectPoints(int pointsPerSide)
	Point[] GetStorageAttackPoints(List<Point> redLinePoints)
	void ExtractHeroes(List<DeployElement> units, List<DeployElement> heroes)
	IEnumerable<int> DeployHeroes(List<DeployElement> heroes, IEnumerable<Point> deployPoints, bool activateHeroAbilities = true)
	void TryActivateHeroAbilities(List<DeployElement> heroes, bool forceActivate = false)
	IEnumerable<int> SurrenderDeployMethod()
	void Surrender()
	bool SurrenderIfWeHaveAStar()
	bool HaveAStar()
	Point[] GetTrophyPushDeployPoints(TrophyPushOpponentAnalysis trophyPushAnalysis)
	void DrawAnalysis(List<Point> deployPoints, List<Point> redLinePoints)
	List<DeployElement> GetAvailableDeployElements()
	Tuple<Point, Point>[] GenerateDeployLinesFromSettings()
	bool AreTroopSetsDifferent(List<DeployElement> a, List<DeployElement> b)
	void ClickAlongLine(Point p1, Point p2, int count, int sleepTime)
	TrophyPushOpponentAnalysis CheckForTownhallNearBorder()
	IEnumerable<int> GenerateDeployPointsFromMines(List<Point> deployPoints,IEnumerable<Point> redLinePoints = null, List<Rectangle> detectedMines = null)
	IEnumerable<int> GenerateDeployPointsFromMinesToMilk(List<Point> deployPoints, List<Point> redLinePoints, List<Rectangle> detectedMines, bool fullGold = false, bool fullElixir = false, bool fullDelixir = false)
	int GetOutsideCollectorCount(List<Point> redLinePoints)
	ResourcesFull GetResourcesState()
	int[] GetAttackResources()
	IEnumerable<int> DeployUnits(DeployElement[] units, Point[] deployPoints, int clickDelay = 0, int waveCount = 1, int waveDelay = 0, int firstCycleDelay = 0)
	IEnumerable<int> DeployUnitsPerPoint(DeployElement[] units, Point[] deployPoints, int unitsPerPoint = 6, int clickDelay = 0, int cycleDelay = 0)
	IEnumerable<int> WaitForNoResourceChange(double seconds = 3)
```

# BaseAttack Class

# BaseStats Class

## BaseStats Properties

# InitialAttack Class

## InitalAttack Methods

# AttackHelper Class

## AttackHelper Methods

# DeployHelper Class

## DeployHelper Properties

## DeployHelper Methods

# DeployElement Class

## DeployElement Properties

Property | Type | Description
-------- | ---- | -----------
Name | string | The name used to identify the element
Rect | Rectangle | The rectangle outlining the location of the unit on the screen.
Count | int | The number of available elements
UnitData | Unit | An object with information about the unit
ElementType | DeployElementType | The type of element; Elements types are: NormalUnit, HeroKing, HeroQueen, HeroWarden, Spell, ClanTroops
IsRanged | bool | Determines if the element is a ranged unit.
IsHero | bool | Determines if the element is a hero.

## DeployElement Methods

Method | Returns | Description
------ | ------- | -----------
Recount() | void | Recounts the Element

# Unit Class

## Unit Properties

Property | Type | Description
-------- | ---- | -----------
AffectedTargets | AffectedTargets | Flags showing the Units favorite target: Ground, Air, Any, AirGround, Buildings, Walls, BuildingsWalls
AmountPerPulse | int | 
AttackSpeed | double | Unit attack speed
AttackType | AttackType | Type of unit: Damage, Wallbreak, Tank, Heal
BoostSeconds | int |
BuildingLevelRequired | int |
BuildingType | BuildingType | Building used to queue: Barracks, DarkBarracks, SpellFactory, DarkSpellFactory
DamageIncreasePercentage | int |
DarkElixirCost | int | Dark elixir cost to queue
DPS | double | Damage per second
EffectType | EffectType | Damage type: Single, Splash
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
SpellType | SpellType | Type of spell: Offensive, Support, Utility
SplashRange | double |
TargetType | TargetType | Target type of the unit: None, Defense, AirDefense, Loot, Walls
TrainingButton | Point | Location of the training button on the screen
TrainingTime | double | Length of time to train
Type | ItemType | Type of item: Unit, Spell
UnitType | UnitType | Type of unit: Ground, Air



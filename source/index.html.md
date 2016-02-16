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
			// name of your algorithm that appears on the attack tab
			return "My deploy algorithm";
		}

		public override double ShouldAccept()
		{
			if (!MeetsRequirements(BaseStats))
				return 0; // skip this base
			return 1; // attack this base
		}

		public override IEnumerable<int> AttackRoutine()
		{
			// routine to deploy troops here
			// [...]
			yield return 100;
		}
	}
}
```

# Getting Started

Use these methods to get started with an attack.

## Checking a Base

### Using MeetsRequirements

```c#
protected bool MeetsRequirements(BaseStats baseStats);
```

This method is used to check if the base the bot is currently on meets the requirements set by the user on the "Attack" tab. Use it in the `ShouldAccept` method.

Parameter | Description
--------- | -----------
BaseStats | An object with the stats of the current target's base. This is passed to your algorithm in the constuctor.

```c#
public override double ShouldAccept()
{
	if (!MeetsRequirements(BaseStats))
		return 0; //skip this base
	return 1; //attack this base
}
```

### Expanded Example

This is the `ShouldAccept` method for the milking algorithm. After checking if the base meets the user's requirements, it performans another check to make sure there are atleast two collectors outside the walls.

```c#
public override double ShouldAccept()
{
	// check if the target's base meets the user's settings
    if (!MeetsRequirements(BaseStats))
        return 0; // skip this base

	// get a count of collectors outside the walls
    var outsideCollectorsCount = GetOutsideCollectorCount(RedPoints);

	// check if there are more than two collectors outside the wall
    if (outsideCollectorsCount < 2)
        return 0; // skip this base

    return 1; // accept this base
}
```

## Getting Deploy Points

Deploy points are needed to figure out where to place your troops.

### Using RedPoints

The `RedPoints` property will get a list with all points the go around the target's base following the red line.

### Example Methods

Here are some examples of how to get deploy points. More methods can be found in the `PluginBase` and `DeployHelper` class documentation.

```c#
public override IEnumerable<int> AttackRoutine()
{
	// get 15 deploy points per side or 60 in total
	var deployPointsRect = GetRectPoints(15);

	// get deploy points near the townhall if it is close to the edge
	TrophyPushOpponentAnalysis trophyPushAnalysis = CheckForTownhallNearBorder();
    Point[] deployPointsNearTownhall = null;

    if (IsTownhallNearBorder(trophyPushAnalysis))
    {
        deployPointsNearTownhall = GetTrophyPushDeployPoints(trophyPushAnalysis);
    }

	// get deploy points near collectors
	var deployPointsNearCollectors = new List<Point>();
	var mineRects = new List<Rectangle>();

	foreach(var t in GenerateDeployPointsFromMines(deployPointsNearCollectors, RedPoints, , mineRects))
		yield return t; // wait until function finishes

	// get troops
	// [...]

	// deploy troops
	// [...]
}
```

## Getting Units

### Using GetAvailableDeployElements

```c#
public static List<DeployElement> GetAvailableDeployElements();
```

This method returns a list of the troops and spells currently available to deploy using the DeployElement object. Use it in the `AttackRoutine` method.

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

	// deploy units to screen.
	// [...]
}
```

## Deploying units



# PluginBase Class

```c#
public abstract class PluginBase
```

## PluginBase Constructors

```c#
protected PluginBase();
```

Name | Description
---- | -----------
PluginBase() | Initializes a new instance of the PluginBase class

## PluginBase Properties

Name | Type | Description
---- | ---- | -----------
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

```c#
protected static bool AttackOnlyDeadBases { get; }
protected static bool TrophyPushMode { get; }
protected static int TrophyPushThDistanceLimit { get; }
protected static double WaveDelay { get; }
protected static int WaveSize { get; }
protected static int WaveTrapSize { get; }
protected static bool UseKing { get; }
protected static bool UseQueen { get; }
protected static bool UseWarden { get; }
protected static bool UseClanTroops { get; }
protected static bool SaveAttackAnalysisImage { get; }
protected static bool DisplayAttackAnalysisImage { get; }
protected List<Point> RedPoints { get; }
protected static int BottomEnd { get; }
```

## PluginBase Structures

### ActiveSearch Structure

```c#
protected struct ActiveSearch
{
	public static bool NeedOnlyOneRequirementForAttack { get; }
	public static int MinGold { get; }
	public static int MinElixir { get; }
	public static int MinDarkElixir { get; }
	public static int MaxThLevel { get; }

	public static bool MeetsRequirements(BaseStats baseStats);
}
```
#### ActiveSearch Properties

Name | Type | Description
---- | ---- | -----------
NeedOnlyOneRequirementForAttack | bool | Gets the user's setting for meet only one requirement for active bases
MinGold | int | Gets the user's setting for the minimum gold requirement for active bases
MinElixir | int | Gets the user's setting for the minimum elixir requirement for active bases
MinDarkElixir | int | Gets the user's setting for the minimum dark elixir requirement for active bases
MaxThLevel | int | Gets the user's setting for the maximum townhall level requirement for active bases

#### ActiveSearch Methods

Name | Returns | Description
---- | ------- | -----------
MeetsRequirements(BaseStats) | bool | Determines if the current base meets the active base requirements

### DeadSearch Structure

```c#
protected struct DeadSearch
```

### DeadSearch Properties

Name | Type | Description
---- | ---- | -----------
NeedOnlyOneRequirementForAttack | bool | Gets the user's setting for meet only one requirement for dead bases
MinGold | int | Gets the user's setting for the minimum gold requirement for dead bases
MinElixir | int | Gets the user's setting for the minimum elixir requirement for dead bases
MinDarkElixir | int | Gets the user's setting for the minimum dark elixir requirement for dead bases
MaxThLevel | int | Gets the user's setting for the maximum townhall level requirement for dead bases

```c#
public static bool NeedOnlyOneRequirementForAttack { get; }
public static int MinGold { get; }
public static int MinElixir { get; }
public static int MinDarkElixir { get; }
public static int MaxThLevel { get; }
```

### DeadSearch Methods

Name | Returns | Description
---- | ------- | -----------
MeetsRequirements(BaseStats) | bool | Determines if the current base meets the dead base requirements

```c#
public static bool MeetsRequirements(BaseStats baseStats);
```

### DeployElementType Structure

```c#
protected struct DeployElementType
```

### DeployElementType Properties

Name | Type | Description
---- | ---- | -----------
NormalUnit | DeployElementType | Gets the DeployElementType enum for NormalUnit
HeroKing | DeployElementType | Gets the DeployElementType enum for HeroKing
HeroQueen | DeployElementType | Gets the DeployElementType enum for HeroQueen
HeroWarden | DeployElementType | Gets the DeployElementType enum for HeroWarden
Spell | DeployElementType | Gets the DeployElementType enum for Spell
ClanTroops | DeployElementType | Gets the DeployElementType enum for ClanTroops

```c#
public static Helpers.DeployElementType NormalUnit { get; }
public static Helpers.DeployElementType HeroKing { get; }
public static Helpers.DeployElementType HeroQueen { get; }
public static Helpers.DeployElementType HeroWarden { get; }
public static Helpers.DeployElementType Spell { get; }
public static Helpers.DeployElementType ClanTroops { get; }
```

### DeployPointA Structure

```c#
protected struct DeployPointA
```

### DeployPointA Properties

Name | Type | Description
---- | ---- | -----------
Top | Point | Gets the deploy point for the top of the base
Left | Point | Gets the deploy point for the left of the base
Right | Point | Gets the deploy point for the right of the base
Bottom | Point | Gets the deploy point for the bottom of the base

```c#
public static Point Top { get; }
public static Point Left { get; }
public static Point Right { get; }
public static Point Bottom { get; }
```

## PluginBase Methods

Name | Returns | Description
---- | ------- | -----------
MeetsRequirements(BaseStats baseStats) | bool | Determines if the current base meets the requirements based on if it is considered dead or alive
ToUnitString(List&lt;DeployElement&gt;) | string | Returns a string with the name and count of each unit
OrderUnitsForDeploy(List&lt;DeployElement&gt;) | void | Orders units for deployment; Tank > Wallbreaker > Heal > Damage > Heroes
GetPointsForLine(Point, Point, int) | List&lt;Point&gt; | Returns a list of points with the specified count along the line created by two specified points
GetRectPoints(int) | List&lt;Point&gt; | Returns a list of the number of points per side specified along the outside rectangle of the current target's base
GetStorageAttackPoints(List&lt;Point&gt;) | Point[] | Returns an array of deploy points near storages based on the specified list of red line points
ExtractHeroes(List&lt;DeployElement&gt;, List&lt;DeployElement&gt;) | void | Extracts heroes from the specified units list and adds them to the specified heroes list
DeployHeroes(List&lt;DeployElement&gt;, IEnumerable&lt;Point&gt;,bool) | IEnumerable&lt;int&gt; | Deploys the specified heroes on the specified deploy points.
TryActivateHeroAbilities(List&lt;DeployElement&gt;, bool) | void |
SurrenderDeployMethod() | IEnumerable&lt;int&gt; |
Surrender() | void |
SurrenderIfWeHaveAStar() | bool |
HaveAStar() | bool |
GetTrophyPushDeployPoints(TrophyPushOpponentAnalysis) | Point[] |
DrawAnalysis(List&lt;Point&gt;, List&lt;Point&gt;) | void |
GetAvailableDeployElements() | List&lt;DeployElement&gt; |
GenerateDeployLinesFromSettings() | Tuple&lt;Point, Point&gt;[] |
AreTroopSetsDifferent(List&lt;DeployElement&gt;, List&lt;DeployElement&gt;) | bool |
ClickAlongLine(Point, Point, int, int) | void |
CheckForTownhallNearBorder() | TrophyPushOpponentAnalysis |
GenerateDeployPointsFromMines(List&lt;Point&gt;, IEnumerable&lt;Point&gt;, List&lt;Rectangle&gt;) | IEnumerable&lt;int&gt; |
GenerateDeployPointsFromMinesToMilk(List&lt;Point&gt;, List&lt;Point&gt;, List&lt;Rectangle&gt;, bool, bool, bool) | IEnumerable&lt;int&gt; |
GetOutsideCollectorCount(List&lt;Point&gt;) | int |
GetResourcesState() | ResourcesFull |
GetAttackResources() | int[] |
DeployUnitsInWaves(DeployElement[], Point[], int, int, int, int) | IEnumerable&lt;int&gt; |
DeployUnitsPerPoint(DeployElement[], Point[], int, int, int) | IEnumerable&lt;int&gt; |
WaitForNoResourceChange(double) | IEnumerable&lt;int&gt; |

```c#
protected bool MeetsRequirements(BaseStats baseStats);
public static string ToUnitString(List<DeployElement> unitsCounts);
public static void OrderUnitsForDeploy(List<DeployElement> units);
protected static List<Point> GetPointsForLine(Point p1, Point p2, int count);
protected static List<Point> GetRectPoints(int pointsPerSide);
protected static Point[] GetStorageAttackPoints(List<Point> redLinePoints);
protected static void ExtractHeroes(List<DeployElement> units, List<DeployElement> heroes);
protected static IEnumerable<int> DeployHeroes(List<DeployElement> heroes, IEnumerable<Point> deployPoints, bool activateHeroAbilities = true);
protected internal static void TryActivateHeroAbilities(List<DeployElement> heroes, bool forceActivate = false);
protected internal static IEnumerable<int> SurrenderDeployMethod();
protected internal static void Surrender();
protected internal static bool SurrenderIfWeHaveAStar();
protected internal static bool HaveAStar();
public static Point[] GetTrophyPushDeployPoints(TrophyPushOpponentAnalysis trophyPushAnalysis);
protected static void DrawAnalysis(List<Point> deployPoints, List<Point> redLinePoints);
protected static List<DeployElement> GetAvailableDeployElements();
protected static Tuple<Point, Point>[] GenerateDeployLinesFromSettings();
protected static bool AreTroopSetsDifferent(List<DeployElement> a, List<DeployElement> b);
protected static void ClickAlongLine(Point p1, Point p2, int count, int sleepTime);
protected static TrophyPushOpponentAnalysis CheckForTownhallNearBorder();
public static IEnumerable<int> GenerateDeployPointsFromMines(List<Point> deployPoints, IEnumerable<Point> redLinePoints = null, List<Rectangle> detectedMines = null);
public static IEnumerable<int> GenerateDeployPointsFromMinesToMilk(List<Point> deployPoints, List<Point> redLinePoints, List<Rectangle> detectedMines, bool fullGold = false, bool fullElixir = false, bool fullDelixir = false);
public static int GetOutsideCollectorCount(List<Point> redLinePoints);
public static ResourcesFull GetResourcesState();
public static int[] GetAttackResources();
public static IEnumerable<int> DeployUnitsInWaves(DeployElement[] units, Point[] deployPoints, int clickDelay = 0, int waveCount = 1, int waveDelay = 0, int firstCycleDelay = 0);
public static IEnumerable<int> DeployUnitsPerPoint(DeployElement[] units, Point[] deployPoints, int unitsPerPoint = 6, int clickDelay = 0, int cycleDelay = 0);
public static IEnumerable<int> WaitForNoResourceChange(double seconds = 3);
```

# BaseAttack Class

Your attack algorithm class must implement the `BaseAttack` class.

```c#
public abstract class BaseAttack : PluginBase, IOpponentSelector, IAttackStrategy
```

# BaseAttack Constructors

Name | Description
---- | -----------
BaseAttack(BaseStats) | 

```c#
protected BaseAttack(BaseStats baseStats);
```

## BaseAttack Fields

Name | Type | Description
---- | ---- | -----------
BaseStats | BaseStats | Holds the stats for the current target's base

```c#
protected BaseStats BaseStats;
```

## BaseAttack Methods

Name | Returns | Description
---- | ------- | -----------
ShouldAccept() | double | Determines if the attack algorithm is a good match for the current target's base using a range from 0 to 1
AttackRoutine() | IEnumerable&lt;int&gt; | Attack routine used on the target's base

```c#
public abstract double ShouldAccept();
public abstract IEnumerable<int> AttackRoutine();
```

# BaseStats Class

The `BaseStats` class is used to hold information about the target's base.

```c#
public class BaseStats
```

## BaseStats Constuctors

Name | Description
---- | -----------
BaseStats(int) | 

```c#
public static BaseStats CreateBaseStats(int baseDisplayCount);
```

## BaseStats Properties

Name | Type | Description
---- | ---- | -----------
BaseDisplayCount | int | 
Gold | int | Gets or sets the amount of gold available in the current target's base
Elixir | int | Gets or sets the amount of elixir available in the current target's base
DarkElixir | int | Gets or sets the amount of dark elixir available in the current target's base
Trophies | int | Gets or sets the trophy count of the current target's base
Th | int | Gets or sets the townhall level of the current target's base
IsDead | bool | Gets or sets if the current target's base is dead
IsStrongBase | bool | Determines if the current target's base is strong based on user settings

```c#
public int BaseDisplayCount { get; private set; }
public int Gold { get; private set; }
public int Elixir { get; private set; }
public int DarkElixir { get; private set; }
public int Trophies { get; private set; }
public int Th { get; private set; }
public bool IsDead { get; private set; }
public bool IsStrongBase {  get; }
```

## BaseStats Methods

Name | Returns | Description
---- | ------- | -----------
CreateBaseStats(int) | BaseStats | Returns base stats for the current target's base
ShouldAcceptCurrentOpponent_StrongBaseCheck() | AttackModule.AcceptOpponentResult | Determines if the current target's base should be accepted based on the user's strong base settings

```c#
public static BaseStats CreateBaseStats(int baseDisplayCount);
static AttackModule.AcceptOpponentResult ShouldAcceptCurrentOpponent_StrongBaseCheck();
```

# InitialAttack Class

```c#
internal class InitialAttack : BaseAttack
```

## InitalAttack Constuctor

Name | Description
---- | -----------
InitialAttack(BaseStats) | 

```c#
public InitialAttack(BaseStats baseStats);
```

## InitalAttack Methods

Name | Returns | Description
---- | ------- | -----------
ShouldAccept() | double | Not implemented
AttackRoutine() | IEnumerable&lt;int&gt; | Deploys an initial attack based on the user's settings

```c#
public override double ShouldAccept();
public override IEnumerable<int> AttackRoutine()
```

## InitialAttack Example

```c#
// your algorithm's attack routine
public override IEnumerable<int> AttackRoutine()
{
	// create an initial attack class with the current target's base stats
	var clearWave = new InitialAttack(BaseStats);

	// deploy the initial attack wave
	foreach(var t in clearWave.AttackRoutine())
		yield return t;

	// continue with your algorithm
	// [...]
}
```

# AttackHelper Class

## AttackHelper Methods

# DeployHelper Class

## DeployHelper Properties

## DeployHelper Methods

# DeployElement Class

## DeployElement Properties

Name | Type | Description
---- | ---- | -----------
Name | string | The name used to identify the element
Rect | Rectangle | The rectangle outlining the location of the unit on the screen.
Count | int | The number of available elements
UnitData | Unit | An object with information about the unit
ElementType | DeployElementType | The type of element; Elements types are: NormalUnit, HeroKing, HeroQueen, HeroWarden, Spell, ClanTroops
IsRanged | bool | Determines if the element is a ranged unit.
IsHero | bool | Determines if the element is a hero.

## DeployElement Methods

Name | Returns | Description
---- | ------- | -----------
Recount() | void | Recounts the Element

# Unit Class

## Unit Properties

Name | Type | Description
---- | ---- | -----------
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



---
title: Boostbot API Reference

language_tabs:
- C#

toc_footers:
- <a href='https://boostbot.org'>For use with Boostbot</a>

includes:

search: true
---

# Introduction

Welcome to the Boostbot API documentation page.
This website is intended for programmers with at least a basic understanding of how to use VisualStudio and C#.
If you never used C# or any programming language before, learn that first and come back later.

# Project Setup

Create a new project of type <code>Class Library</code> (.dll).

You will need to reference the boostbot application (.exe), Core.dll, and Shared.dll.  All of these can be found in the bots main folder.
The two example projects inside the bots <code>addons</code> folder already contains completely projects which are used in the bot already;
so you can use these as a guide as well.

Obviously, it's also possible to just copy-paste one of the existing projects and rename everything.

After you added the references add the needed namespaces to your <code>.cs</code> file.

In order for the bot to be able to load the dll you'll eventually compile, it needs a few special attributes and your
class which describes your addon will have to implement <code>BaseAttack</code>

See the information below for how thats done.


# Addon Attributes

## Addon

In order for your addon to be picked up by the bot, you must include the Addon attribute in the assembly namespace.
The class you define must also inherit from BaseAttack and have a special attribute (AttackAlgorithm)

### Contructor

```c#
[assembly: Addon("Nameof Addon", "This is a sample addon", "BoostBot Team")]
```

`assembly: Addon(string addonName, string description, string author)`

### Parameters
Parameter | Description
--------- | ------- 
addonName | Name of your addon 
description | A short description of your addon
author | You!

<aside class="notice">
Remember — this is NOT a description of your algorithm, just your addon
</aside>

## AttackAlgorithm

For each algorithm, you must in include the AttackAlgorithm Attribute on your class

### Constructor

```c#
[AttackAlgorithm("MyDeploy", "Deploys units in a smiley face pattern.")]
```

`AttackAlgorithm(string name, string description)`

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

```c#
public class MyAlgorithm(BaseStats baseStats): base(baseStats){}
```

Your BaseAttack class will accept a <code>BaseStats</code> parameter.

BaseStats is a class that contains various settings from the user,
as well as the current Gold/Elixir/DarkElixir the base which is being attacked has.

## ToString

```c#
public override string ToString()
{
	return "My deploy algorithm";
}
```

This returns the name of your deployment. This is used to display the name that appears in "Attack" tab of the bot.

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

Use these examples to get started with an attack.

## ShouldAccept

> standard method to check a base against a user's settings

```c#
public override double ShouldAccept()
{
	if (!MeetsRequirements(BaseStats))
		return 0; //skip this base
	return 1; //attack this base
}
```

> example method used for milking

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

`ShouldAccept` is ran on each target base. When a value above 0 is returned. The base is accepted and `AttackRoutine` will be executed.

Use `MeetsRequirements(BaseStats)` to determine if the current base meets the requirements defined by the user's settings.

## AttackRoutine

```c#
public override IEnumerable<int> AttackRoutine()
{
	yield return 1000;
}
```

`AttackRoutine` holds all the logic for your actual attack. The standard pattern to attacking uses these steps.

1. Get deploy points
2. Get available units to deploy
3. Deploy units

`AttackRoutine` returns	`IEnumberable<int>`. When you want the bot to pause for a certain amount of milliseconds, use `yield return` followed by the time.

### Getting Deploy Points

> get deployment points from the outer rectangle—15 points per side

```c#
var deployPointsRect = GetRectPoints(15);
```

> get deploy points near the townhall if it is close to the edge

```c#
TrophyPushOpponentAnalysis trophyPushAnalysis = CheckForTownhallNearBorder();
Point[] deployPointsNearTownhall = null;

if (IsTownhallNearBorder(trophyPushAnalysis))
{
    deployPointsNearTownhall = GetTrophyPushDeployPoints(trophyPushAnalysis);
}
```

> get deploy points near collectors

```c#
var deployPointsNearCollectors = new List<Point>();
var mineRects = new List<Rectangle>();

foreach(var t in GenerateDeployPointsFromMines(deployPointsNearCollectors, RedPoints, , mineRects))
	// wait until function finishes
	// do not return 0 here! always return what the function you're calling returns!
	yield return t; 
```

Deploy points are needed to figure out where to place your troops. In the code section are some examples of how to get deploy points. More methods can be found in the `PluginBase` and `DeployHelper` class documentation.

The `RedPoints` property will get a list with all points the go around the target's base following the red line.

### Getting Units

> get the deploy elements and break them up into groups to use for deployment
	
```c#
//get the deploy elements that have unit data
var deployElements = GetAvailableDeployElements().Where(x => x.UnitData != null);

//get only the tank units from the deploy elements
var tankUnits = deployElements.Where(x => x.ElementType == DeployElementType.NormalUnit && x.UnitData.AttackType == AttackType.Tank).ToArray();

//get only the attacking units from the deploy elements
var attackUnits = deployElements.Where(x => x.ElementType == DeployElementType.NormalUnit && x.UnitData.AttackType == AttackType.Damage).ToArray();

//get only the healing units from the deploy elements
var healUnits = deployElements.Where(x => x.ElementType == DeployElementType.NormalUnit && x.UnitData.AttackType == AttackType.Heal).ToArray();
```

`GetAvailableDeployElements` returns a list of the troops and spells currently available to deploy using the DeployElement object. You can use `linq` to filter this list into certain troops.


## Deploying units

> deploy 5 normal units at each deploy point with a 100ms click delay and 1000ms delay between points then wait 1000ms

```c#
if (attackUnits.Any())
{
    foreach (var t in DeployUnitsPerPoint(attackUnits, _deployPoints, 5, 100, 1000))
        yield return t;
    yield return 1000;
}
```

> deploy 3 waves of normal units with a 100ms click delay and 1000ms between waves then wait 1000ms

```c#
if (attackUnits.Any())
{
    foreach (var t in DeployUnitsInWaves(attackUnits, _deployPoints, 100, 3, 1000))
        yield return t;
    yield return 1000;
}
```

## Surrendering Early

## Using Heroes

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
### ActiveSearch Properties

Name | Type | Description
---- | ---- | -----------
NeedOnlyOneRequirementForAttack | bool | Gets the user's setting for meet only one requirement for active bases
MinGold | int | Gets the user's setting for the minimum gold requirement for active bases
MinElixir | int | Gets the user's setting for the minimum elixir requirement for active bases
MinDarkElixir | int | Gets the user's setting for the minimum dark elixir requirement for active bases
MaxThLevel | int | Gets the user's setting for the maximum townhall level requirement for active bases

### ActiveSearch Methods

Name | Returns | Description
---- | ------- | -----------
MeetsRequirements(BaseStats) | bool | Determines if the current base meets the active base requirements

### DeadSearch Structure

```c#
protected struct DeadSearch
{
	public static bool NeedOnlyOneRequirementForAttack { get; }
	public static int MinGold { get; }
	public static int MinElixir { get; }
	public static int MinDarkElixir { get; }
	public static int MaxThLevel { get; }

	public static bool MeetsRequirements(BaseStats baseStats);
}
```

### DeadSearch Properties

Name | Type | Description
---- | ---- | -----------
NeedOnlyOneRequirementForAttack | bool | Gets the user's setting for meet only one requirement for dead bases
MinGold | int | Gets the user's setting for the minimum gold requirement for dead bases
MinElixir | int | Gets the user's setting for the minimum elixir requirement for dead bases
MinDarkElixir | int | Gets the user's setting for the minimum dark elixir requirement for dead bases
MaxThLevel | int | Gets the user's setting for the maximum townhall level requirement for dead bases

### DeadSearch Methods

Name | Returns | Description
---- | ------- | -----------
MeetsRequirements(BaseStats) | bool | Determines if the current base meets the dead base requirements

### DeployElementType Structure

```c#
protected struct DeployElementType
{
	public static Helpers.DeployElementType NormalUnit { get; }
	public static Helpers.DeployElementType HeroKing { get; }
	public static Helpers.DeployElementType HeroQueen { get; }
	public static Helpers.DeployElementType HeroWarden { get; }
	public static Helpers.DeployElementType Spell { get; }
	public static Helpers.DeployElementType ClanTroops { get; }
}
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

### DeployPointA Structure

```c#
protected struct DeployPointA
{
	public static Point Top { get; }
	public static Point Left { get; }
	public static Point Right { get; }
	public static Point Bottom { get; }
}
```

### DeployPointA Properties

Name | Type | Description
---- | ---- | -----------
Top | Point | Gets the deploy point for the top of the base
Left | Point | Gets the deploy point for the left of the base
Right | Point | Gets the deploy point for the right of the base
Bottom | Point | Gets the deploy point for the bottom of the base

## PluginBase Methods

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

# Base Classes

## BaseAttack Class

```c#
public abstract class BaseAttack : PluginBase, IOpponentSelector, IAttackStrategy
```

Your attack algorithm class must implement the `BaseAttack` class.

### BaseAttack Constructors

```c#
protected BaseAttack(BaseStats baseStats);
```

Name | Description
---- | -----------
BaseAttack(BaseStats) | 

### BaseAttack Fields

```c#
protected BaseStats BaseStats;
```

Name | Type | Description
---- | ---- | -----------
BaseStats | BaseStats | Holds the stats for the current target's base

### BaseAttack Methods

```c#
public abstract double ShouldAccept();
public abstract IEnumerable<int> AttackRoutine();
```

Name | Returns | Description
---- | ------- | -----------
ShouldAccept() | double | Determines if the attack algorithm is a good match for the current target's base using a range from 0 to 1
AttackRoutine() | IEnumerable&lt;int&gt; | Attack routine used on the target's base

## BaseStats Class

```c#
public class BaseStats
```

The `BaseStats` class is used to hold information about the target's base.

### BaseStats Constuctors

```c#
public static BaseStats CreateBaseStats(int baseDisplayCount);
```

Name | Description
---- | -----------
BaseStats(int) | 

### BaseStats Properties

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

### BaseStats Methods

```c#
public static BaseStats CreateBaseStats(int baseDisplayCount);
static AttackModule.AcceptOpponentResult ShouldAcceptCurrentOpponent_StrongBaseCheck();
```

Name | Returns | Description
---- | ------- | -----------
CreateBaseStats(int) | BaseStats | Returns base stats for the current target's base
ShouldAcceptCurrentOpponent_StrongBaseCheck() | AttackModule.AcceptOpponentResult | Determines if the current target's base should be accepted based on the user's strong base settings

# Attack Classes

## InitialAttack Class

```c#
internal class InitialAttack : BaseAttack
```

### InitalAttack Constuctor

```c#
public InitialAttack(BaseStats baseStats);
```

Name | Description
---- | -----------
InitialAttack(BaseStats) | 

### InitalAttack Methods

```c#
public override double ShouldAccept();
public override IEnumerable<int> AttackRoutine()
```

Name | Returns | Description
---- | ------- | -----------
ShouldAccept() | double | Not implemented
AttackRoutine() | IEnumerable&lt;int&gt; | Deploys an initial attack based on the user's settings

### InitialAttack Example

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

## AttackHelper Class

Under Construction

# Deploy Classes

## DeployHelper Class

Under Construction

## DeployElement Class

### DeployElement Fields

```c#
public string Name;
public Rectangle Rect;
public int Count;
public Unit UnitData;
public DeployElementType ElementType;
```

Name | Type | Description
---- | ---- | -----------
Name | string | The name used to identify the element
Rect | Rectangle | The rectangle outlining the location of the unit on the screen.
Count | int | The number of available elements
UnitData | Unit | An object with information about the unit
ElementType | DeployElementType | The type of element; Elements types are: NormalUnit, HeroKing, HeroQueen, HeroWarden, Spell, ClanTroops

### DeployElement Properties

```c#
public bool IsRanged { get; }
public bool IsHero { get; }
```

Name | Type | Description
---- | ---- | -----------
IsRanged | bool | Determines if the element is a ranged unit.
IsHero | bool | Determines if the element is a hero.

### DeployElement Methods

```c#
public void Recount();
```

Name | Returns | Description
---- | ------- | -----------
Recount() | void | Recounts the Element

# Unit Class

## Unit Fields

```c#
public AffectedTargets AffectedTargets;
public int AmountPerPulse;
public double AttackSpeed;
public AttackType AttackType;
public int BoostSeconds;
public int BuildingLevelRequired;
public BuildingType BuildingType;
public int DamageIncreasePercentage;
public int DarkElixirCost;
public double DPS;
public EffectType EffectType;
public int ElixirCost;
public int GoldCost;
public int HousingSpace;
public double HP;
public int LaboratoryLevelRequired;
public int Level;
public double MovementSpeed;
public string Name;
public string NameSimple;
public int NumberOfPulses;
public double RandomRange;
public double Range;
public int ResearchCost;
public double SecondsBetweenPulses;
public int SpeedIncreasePercentage;
public double SpellDurationSeconds;
public SpellType SpellType;
public double SplashRange;
public TargetType TargetType;
public Point TrainingButton;
public double TrainingTime;
public ItemType Type;
public UnitType UnitType;
```

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

# CoC_Bot.Visualize
The Visualize class exists to draw on Bitmaps, helping you to properly visualize what the bot is seeing, or to draw new items on the screen
## Visualize Methods
```c#
public static void Grid(Bitmap bmp, Color? color = null, bool redZone = true)
public static void GridFromPoints(Bitmap bmp, Color? color = null, float size = 2, bool redZone = true)
public static void Axes(Bitmap bmp, Color? color = null)
public static void Target(Bitmap bmp, PointFT location, int size = 50, Color? color = null)
public static void RectangleT(Bitmap bmp, RectangleT rectangle, Pen outline = null, Brush fill = null)
public static void Rectangle(Bitmap bmp, Rectangle rectangle, Pen outline = null, Brush fill = null, bool center = true, string caption = null, Font captionFont = null)
public static void Crosshair(Bitmap bmp, Point point, int size = 41, Color? color = null)
public static void Coverage(Bitmap bmp, RangedBuilding building, Color? color = null)
```

Name | Returns | Description
---- | ------- | -----------
Grid(Bitmap, Color?, bool) | void | Draws a grid (lines) over the entire game area or red zone on the supplied bitmap.
GridFromPoints(Bitmap, Color?, float, bool) | void | Draws a grid (dots) over the entire game area or red zone on the supplied bitmap.
Axes(Bitmap, Color?) | void | Draws a set of axes over the game area, centered on the origin.
Target(Bitmap, PointFT, int, Color?) | void | Draws a target at the specified PointFT.
RectangleT(Bitmap, RectangleT, Pen, Brush) | void | Draws and/or fills the specified RectangleT structure on the bitmap.
Rectangle(Bitmap, Rectangle, Pen, Brush, bool, string, Font) | void | Draws and/or fills a rectangle (in absolute screen coordinates) on the bitmap.
Crosshair(Bitmap, Point, int, Color?) | void | Draws a one pixel crosshair at the specified location.
Coverage(Bitmap, RangedBuilding, Color?) | void | Draws a coverage indicator for the specified RangedBuilding

# CoC_Bot.GameGrid

## GameGrid Properties
```c#
public const int TilesX
public const int TilesY
public static PointFT[] RedPoints
```
Name | Type | Description
---- | ---- | -----------
TilesX | int | "Horizontal" (the / diagonal) size of the game grid, in tiles.
TilesY | int | "Vertical" (the \ diagonal) size of the game grid, in tiles.
RedPoints | PointFT | Accesses the red zone information for the current base.

## GameGrid Methods
```c#
public static void ClearCache()
```
Name | Returns | Description
---- | ------- | -----------
ClearCache() | void | Clears the red zone cache

# CoC_Bot.GameGrid.RedZoneExtents
<aside class="notice">
RedZoneExtents is an advanced class, it is highly recommended to use PointFT instead.
</aside>
## RedZoneExtents Properties
```c#
public const int TilesX
public const int TilesY
public const float T
public const float B
public const float M
public const float L
public const float R
public const float C
```
Name | Type | Description
---- | ---- | -----------
TilesX | int | "Horizontal" (the / diagonal) size of the game grid, in tiles.
TilesY | int | "Vertical" (the \ diagonal) size of the game grid, in tiles.
T | float | Top absolute (pixel) coordinates of the game grid's red zone.
B | float | Bottom absolute (pixel) coordinates of the game grid's red zone.
M | float | Middle (vertical) absolute (pixel) coordinates of the game grid's red zone.
L | float | Left absolute (pixel) coordinates of the game grid's red zone.
R | float | Right absolute (pixel) coordinates of the game grid's red zone.
C | float | Center (horizontal) absolute (pixel) coordinates of the game grid's red zone.

# CoC_Bot.GameGrid.RedZoneI

## RedZoneI Constructor
```c#
public RedZoneI()
```
Name | Description
---- | -----------
RedZoneI() | Constructor. Populates a new RedZone object based on the current screen.

## RedZoneI Properties
```c#
public GridState this[int x, int y]
public GridState this[PointFT pft] 
```
Use these two properties to get the state of the red zone at the given coordinates.
## GridState enum
```c#
Unknown = 0, Red, Green
```
Enumeration indicating whether troops can be deployed on the specified tile.  Unknown signifies that it is uncertain whether troops can be deployed on a given tile.

# CoC_Bot.Buildings

## Buildinds.Building
This is the absolute base Building class, from which all other buildings inherit.
### Properties
```c#
public Rectangle MatchedRectangle
public Definition Definition
public RectangleT Location
public int? Level
public int? MaxHitPoints
```
Name | Type | Description
---- | ---- | -----------
MatchedRectangle | Rectangle | Screen rectangle that matched the definition.
Definition | Definition | Name of the definition that matched this building.
Location | RectangleT | Location of this building.
Level | int? | Level of this building.
MaxHitPoints | int? | Number of hitpoints this building started with.

### Building Methods
```c#
protected static TBuilding[] Find<TBuilding>(CacheBehavior behavior = CacheBehavior.Default, int? minLevel = null, int? maxLevel = null, int? stopAfter = null, byte width = 3, byte height = 3)
public static void ClearCache()
```
Name | Returns | Description
---- | ------- | -----------
Find(CacheBehavior, int?, int?, int?, byte, byte) | TBuilding[] | Convenience method that handles caching, level checking, etc. to make it easy to implement Find and FindAll in subtypes. Logs a warning or debug message and returns an empty array when it encounters a situation that won't return results.
ClearCache() | void | Clears the building Cache.

## Buildings.RangedBuilding
Inherits Building
Parent type for an offensive building that has a ranged attack.  
### Properties
```c#
public int MinRange
public int MaxRange
public virtual bool InRange(PointFT pft)
```
Name | Type | Description
---- | ---- | -----------
MinRange | int | Minimum attack range, in tile units.
MaxRange | int | Maximum attack range, in tile units.
InRange(PointFT) | bool | Determines whether the specified point lies within this building's range.

## Buildings.DirectionalBuilding
Inherits RangedBuilding
Parent type for an offensive building that has a direction associated with it.  
<aside class="alert">The property InRange for Directional Buildings has not been implemented at the time of this writing</aside>
### Properties
```c#
public int? Direction
public int? Sweep
```
Name | Type | Description
---- | ---- | -----------
Direction | int? | Direction this building is facing, in degrees from horizontal on the tile grid. For example, 0 is NE on screen, 45 is N on screen, 225 is S on screen, etc.
Sweep | int? | Total number of degrees covered by this building, with Direction at the center. So a Sweep of 90 covers 45 degrees on either side of Direction.

## Building Classes
Name | Inherits | Additional Properties
---- | -------- | ---------------------
AirSweeper | DirectionalBuilding
ArcherQueen | Building | bool Sleeping - whether or not hero is sleeping
ArcherTower | RangedBuilding | 
BarbarianKing | Building | bool Sleeping - whether or not hero is sleeping
ClanCastle | Building | 
DarkElixirDrill | Building | 
DarkElixirStorage | Building | 
ElixirCollector | Building | 
ElixirStorage | Building | 
GoldMine | Building | 
GoldStorage | Building | 
GrandWarden | Building | bool Sleeping - whether or not hero is sleeping
HiddenTesla | RangedBuilding | 
InfernoTower | RangedBuilding | 
TownHall | Building | 
WizardTower | RangedBuilding | 



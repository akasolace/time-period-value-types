
Time Period Value Types
=========================
[![Build Status](https://dev.azure.com/cmdty/github/_apis/build/status/cmdty.time-period-value-types?branchName=master)](https://dev.azure.com/cmdty/github/_build/latest?definitionId=4&branchName=master)
![Azure DevOps coverage](https://img.shields.io/azure-devops/coverage/cmdty/github/4)
[![NuGet](https://img.shields.io/nuget/v/cmdty.timeperiodvaluetypes.svg)](https://www.nuget.org/packages/Cmdty.TimePeriodValueTypes/)

### Table of Contents
* [Overview](#overview)
* [Installing](#installing)
* [User Guide](#user-guide)
    * [Creating Instances](#Creating-Instances)
    * [Parsing and Formatting](#Parsing-and-Formatting)
    * [Comparing Instances](#Comparing-Instances)
    * [Offsetting Instances](#Offsetting-Instances)
    * [Converting Between Granularities](#Converting-Between-Granularities)
    * [Expanding](#Expanding)
    * [ITimePeriod\<T\> Interface](#ITimePeriodT-Interface)
    * [Range of Valid Values](#Range-of-Valid-Values)
    * [Extension Methods](#Extension-Methods)
* [Interactive Documentation](#Interactive-Documentation)
* [Building](#building)
* [License](#License)


## Overview
The package Cmdty.TimePeriodValueTypes contains a set of types used to represent time periods for specific granularities. Examples of such types include Month, Quarter, HalfHour and Hour, but many others are also present. Implemented as [readonly structs](https://docs.microsoft.com/en-us/dotnet/csharp/write-safe-efficient-code#declare-readonly-structs-for-immutable-value-types) and only 32 bits in size, these types are also suitable for situations where high performance 
is necessary.

These types are used extensively within the Cmdty library to represent the delivery periods for traded commodities. However, there is nothing commodity-specific within the Time Period Value Types API, and hence these types could be used in many other non-commodity business contexts.

## Installing
From the Visual Studio Package Manager console:
```
PM> Install-Package Cmdty.TimePeriodValueTypes
```
Or from the .NET Core CLI:
```
dotnet add package Cmdty.TimePeriodValueTypes
```

## User Guide
### Creating Instances
As well as being able to create instances using constructors, many types have helper static factory methods as shown for the Month and Quarter types below.
All Time Period types also have a FromDateTime static methods for
converting instances of .NET System.DateTime to time period types.
```c#
var mar19 = new Month(2019, 3);
Console.WriteLine(mar19);

var aug19 = Month.CreateAugust(2019);
Console.WriteLine(aug19);
var qu119 = Quarter.CreateQuarter1(2019);
Console.WriteLine(qu119);

Day christmas2020 = Day.FromDateTime(new DateTime(2020, 12, 25));
Console.WriteLine(christmas2020);
```
```
2019-03
2019-08
Q1-2019
2020-12-25
```

### Parsing and Formatting
The ToString method is overridden on all types to provide a human readable text representation.
All types also have a static Parse method which can create an intance 
from this text representation. The ToString and Parse methods can be used in a round-trip fashion.

Some of the types also implement IFormattable which means an additional override of the ToString method is provided which includes parameters for a format string and format provider. This is demonstrated below for the Hour type.
```c#
Month dec19Original = Month.CreateDecember(2019);
string dec19Text = dec19Original.ToString();

Month dec19FromText = Month.Parse(dec19Text);
Console.WriteLine("Dec-19 parsed from text: " + dec19FromText);

var hour = new Hour(2019, 9, 11, 12);
Console.WriteLine(hour.ToString("dd-MMM-yyyy hh:mm:ss", CultureInfo.InvariantCulture));
```

```
Dec-19 parsed from text: 2019-12
11-Sep-2019 12:00:00
```

### Comparing Instances
All time period types implement [IComparable\<T\>](https://docs.microsoft.com/en-us/dotnet/api/system.icomparable-1?view=netstandard-2.0) and so have the CompareTo method for strongly typed comparison between instances of the same type.

[IEquatable\<T\>](https://docs.microsoft.com/en-us/dotnet/api/system.iequatable-1?view=netstandard-2.0) is implemented for a strongly typed Equals method.

For convenience, the comparison and equality operators are also overloaded for all time period types.
```c#
var qu119 = new Quarter(2019, 1);
var qu219 = Quarter.CreateQuarter2(2019);

// IComparable<T>.CompareTo
Console.WriteLine(qu119.CompareTo(qu219));
Console.WriteLine();

// IEquatable<T>.Equals
Console.WriteLine(qu119.Equals(qu219));
Console.WriteLine();

// Comparison operators
Console.WriteLine(qu119 < qu219);
Console.WriteLine(qu119 <= qu219);
Console.WriteLine(qu119 == qu219);
Console.WriteLine(qu119 != qu219);
Console.WriteLine(qu119 > qu219);
Console.WriteLine(qu119 >= qu219);
```
```
-1

False

True
True
False
True
False
False
```

### Offsetting Instances
The Offset method allows the calculation of time period instances a relative number of time periods before or after a specific instance.

The OffsetFrom method performs the inverse operation, calculating the integer number of periods difference between the current instance and another instance.
```c#
var tenAm = new Hour(2019, 8, 30, 10);
Hour midday = tenAm.Offset(2);
Console.WriteLine(midday);

int numHours = midday.OffsetFrom(tenAm);
Console.WriteLine(numHours);
```

```
2019-08-30 12:00
2
```

The same logic as the instance methods Offset and OffsetFrom from can be called using the + and - operators respectively. The increment and decrement operators, ++ and -- are also provide for all time period types.

```c#
var calYear19 = new CalendarYear(2019);

CalendarYear calYear22 = calYear19 + 3;
Console.WriteLine(calYear22);

int yearsDifference = calYear22 - calYear19;
Console.WriteLine(yearsDifference);

// The increment ++, and decrement --, operators can also be used
var halfHour = new HalfHour(2019, 8, 30, 22, 0);

Console.WriteLine();
Console.WriteLine("Incrementing Half Hour");
Console.WriteLine(halfHour);
halfHour++;
Console.WriteLine(halfHour);
halfHour++;
Console.WriteLine(halfHour);

Console.WriteLine();
Console.WriteLine("Decrementing Half Hour");
halfHour--;
Console.WriteLine(halfHour);
halfHour--;
Console.WriteLine(halfHour);
halfHour--;
Console.WriteLine(halfHour);
```

```
2022
3

Incrementing Half Hour
2019-08-30 10:00
2019-08-30 10:30
2019-08-30 11:00

Decrementing Half Hour
2019-08-30 10:30
2019-08-30 10:00
2019-08-30 09:30
```

### Converting Between Granularities
Convertion between different granularity time periods can be achieved using the methods First\<T\> and Last\<T\>.
```c#
var qu119 = Quarter.CreateQuarter1(2019);
Console.WriteLine("The first month in Q1-19 is " + qu119.First<Month>());
Console.WriteLine("The last month in Q1-19 is " + qu119.Last<Month>());
```

```
The first month in Q1-19 is 2019-01
The last month in Q1-19 is 2019-03
```

### Expanding
The Expand\<T\> method can be use to expand a time period into a collection of instances of higher granularity time period type, as is shown in the example below which calculates all the months in Q2-19.
```c#
var qu219 = Quarter.CreateQuarter2(2019);
IEnumerable<Month> allMonthsInQ119 = qu219.Expand<Month>();
Console.WriteLine("All the months in Q2-19:");
foreach (Month month in allMonthsInQ119)
{
    Console.WriteLine(month);
}
```

```
All the months in Q2-19:
2019-04
2019-05
2019-06
```

### ITimePeriod\<T\> Interface
All time period types implement the [ITimePeriod<T>](src/Cmdty.TimePeriodValueTypes/ITimePeriod.cs) interface. One use of this is to write generic code which can be used
on any of the implementing time period types as shown in the example below.

```c#
void PrintFirstAndLastPeriods<T, T2>(T timePeriod)
    where T : ITimePeriod<T>
    where T2 : ITimePeriod<T2>
{
    string periodType = typeof(T).Name;
    string period2Type = typeof(T2).Name;

    T2 firstHour = timePeriod.First<T2>();
    T2 lastHour = timePeriod.Last<T2>();
    Console.WriteLine($"The {periodType} period '{timePeriod}' has first {period2Type} '{firstHour.ToString()}' and last {period2Type} '{lastHour.ToString()}'");
}

var dec20 = new Month(2020, 12);
PrintFirstAndLastPeriods<Month, Hour>(dec20);
```

```
The Month period '2020-12' has first Hour '2020-12-01 00:00' and last Hour '2020-12-31 23:00'
```

### Range of Valid Values
All time period types have static properties give information on the valid range for each time period type.
```c#
Console.WriteLine("Minimum Day: " + Day.MinDay);
Console.WriteLine("Maximum Day: " + Day.MaxDay);
```

```
Minimum Day: 0001-01-01
Maximum Day: 9998-12-31
```

### Extension Methods
Extension method provide extra useful functionality, such as the EnumerateTo and EnumerateWeekdaysDays methods shown in the example below.
```c#
var quarterStart = Quarter.CreateQuarter3(2020);
var quarterEnd = Quarter.CreateQuarter2(2021);
Console.WriteLine($"All the quarters from {quarterStart} to {quarterEnd}");
foreach (Quarter quarter in quarterStart.EnumerateTo(quarterEnd))
{
    Console.WriteLine(quarter);
}

Console.WriteLine();
var dayStart = new Day(2019, 8, 30);
var dayEnd = new Day(2019, 9, 4);
Console.WriteLine($"All week days from {dayStart} to {dayEnd}");
foreach (Day weekday in dayStart.EnumerateWeekdays(dayEnd))
{
    Console.WriteLine(weekday);
}
```

```
All the quarters from Q3-2020 to Q2-2021
Q3-2020
Q4-2020
Q1-2021
Q2-2021

All week days from 2019-08-30 to 2019-09-04
2019-08-30
2019-09-02
2019-09-03
2019-09-04
```

## Interactive Documentation
An interactive version of the above [user guide](#user-guide) has been provided using 
[Try .NET](https://dotnet.microsoft.com/platform/try-dotnet) 
a tool which allows C# code to be run in a browser using Markdown files. See [/samples/README.md](/samples/README.md) for details.

## Building
This section describes how to run a scripted build on a cloned repo, using the [Cake](https://github.com/cake-build/cake) build tool.

#### Build Prerequisites
The following are required on the host machine in order for the build to run.
* The .NET Core SDK. Check the [global.json file](global.json) for the version necessary, taking into account [the matching rules used](https://docs.microsoft.com/en-us/dotnet/core/tools/global-json#matching-rules).

#### Running the Build on Windows
The build is started by running the PowerShell script build.ps1 from a PowerShell console, ISE, or the Visual Studio Package Manager Console.

```
PM> .\build.ps1
```

#### Running the Build on Linux or macOS
Run the build by invoking the [build.sh](build.sh) script.

```
> chmod +x ./build.sh
> ./build.sh
```

Alternatively, if [PowerShell Core](https://github.com/PowerShell/PowerShell) is installed,
the build can be run with the following command:
```
> pwsh ./build.ps1
```
#### Build Artifacts
The result of the build is the NuGet package Cmdty.TimePeriodValueTypes.[version].nupkg, 
which is saved into the artifacts directory (which itelf will be created in the top 
directory of the repo).


## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details

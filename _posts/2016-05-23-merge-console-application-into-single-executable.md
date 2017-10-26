---
layout: post
title: Merge Console Application into Single Executable
tags: [ilmerge, csproj, target, afterbuild]
---

Goal is to have nice and clean output for release build.

At minimum you may:

Turn off vhost.exe by unchecking "Enable the Visual Studio hosting process" under "Debug" tab of your project proerties. Make sure to do this only for release configuration.

You can also do it by hands, by adding `<UseVSHostingProcess>false</UseVSHostingProcess>` to the `<PropertyGroup Condition=" '$(Configuration)|$(Platform)' == 'Release|AnyCPU' ">` of your csproj file.

Next step may be removing pdb and xml files that are copied with referenced libraries, to do so, you may want to add afterbuild target to your cproj file like this:

```xml
<Target Name="AfterBuild" Condition="'$(Configuration)' == 'Release'">
	<Delete Files="$(OutputPath)Elasticsearch.Net.pdb" />
	<Delete Files="$(OutputPath)Nest.pdb" />
	<Delete Files="$(OutputPath)Elasticsearch.Net.xml" />
	<Delete Files="$(OutputPath)Nest.xml" />
</Target>
```

Or if you want to delete everything at once you may do something like this:

```xml
<Target Name="AfterBuild" Condition="'$(Configuration)' == 'Release'">
	<ItemGroup>
		<FilesToDelete Include="$(OutputPath)*.pdb"/>
		<FilesToDelete Include="$(OutputPath)*.xml"/>
	</ItemGroup>
	<Message Text="Delete: @(FilesToDelete, ', ')" Importance="High" />
	<Delete Files="@(FilesToDelete)" />
</Target>
```

`ItemGroup` is a way to list files, that able to use asteriks.

`@(FilesToDelete, ', ')` is equivalent to `string.Join(", ", FilesToDelete)`

By default Visual Studio displays minimal output from MSBuild, so to see your own, do not forget to add `Importance="High"`

There is option to not create any pdb files in advanced compilation options, but it seems that it is works only for your own class libraries


Merge dll into exe
------------------

First of all we need ILMerge tool, you may install it via nuget: `Install-Package ILMerge`

```xml
<Target Name="AfterBuild" Condition="'$(Configuration)' == 'Release'">
	<ConvertToAbsolutePath Paths="$(OutputPath)">
		<Output TaskParameter="AbsolutePaths" PropertyName="OutputFullPath" />
	</ConvertToAbsolutePath>
	<ItemGroup>
		<MergeAssemblies Include="$(OutputPath)$(MSBuildProjectName).exe" />
		<MergeAssemblies Include="$(OutputPath)Newtonsoft.Json.dll" />
	</ItemGroup>
	<PropertyGroup>
		<OutputAssembly>$(OutputFullPath)$(MSBuildProjectName).Standalone.exe</OutputAssembly>
		<Merger>$(SolutionDir)packages\ILMerge.2.14.1208\tools\ILMerge.exe</Merger>
	</PropertyGroup>
	<Message Text="Merge -&gt; $(OutputAssembly)" Importance="High" />
	<Exec Command="&quot;$(Merger)&quot; /target:exe /out:&quot;$(OutputAssembly)&quot; @(MergeAssemblies->'&quot;%(FullPath)&quot;', ' ')" />
	<ItemGroup>
		<FilesToDelete Include="$(OutputPath)$(MSBuildProjectName).exe" />
		<FilesToDelete Include="$(OutputPath)*.pdb" />
		<FilesToDelete Include="$(OutputPath)*.xml" />
		<FilesToDelete Include="$(OutputPath)*.dll" />
	</ItemGroup>
	<Message Text="Cleanup -&gt; @(FilesToDelete, ', ')" Importance="High" />
	<Delete Files="@(FilesToDelete)" />
	<Copy SourceFiles="$(OutputAssembly)" DestinationFiles="$(OutputPath)$(MSBuildProjectName).exe" />
	<Delete Files="$(OutputAssembly)" />
</Target>
```

`ConvertToAbsolutePath` is used to get full output path.

`MergeAssemblies` item group should contain list of referenced libraries (dll) and executable itself

`PropertyGroup` is used just to define some variables for later use

Note that later on, ILMerge package version may change, you should not forget to fix verion number

The main job is going to be in: `Exec` which is going to run something like:

```sh
ILMerge.exe /target:exe /out:App.Standalone.exe App.exe Newtonsoft.Json.dll
```

`/target` may be `exe` or `winexe` depending of project you are building

Notice that we can not write to file we are building, so by default we are mering into something like App.Standalone.exe and then replacing original one with out. It need to be done to not brake ConfigurationManager if one is used.

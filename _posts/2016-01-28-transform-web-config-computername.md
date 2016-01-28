---
layout: post
title: Transform Web.config after build with computername
tags: [asp, web.config, trasnformation, msbuild, target]
---

The goal is to run Web.config transformation after build if there is Web.COMPUTERNAME.config file.

Here is sample web.config file

Web.config
----------

	<?xml version="1.0" encoding="utf-8"?>
	<configuration>
		<appSettings>
			<add key="Sample" value="Hello World"/> <!-- We are going change this -->
		</appSettings>
		<connectionStrings>
			<add name="Default" connectionString="Data Source=localhost;Initial Catalog=Northwind;Integrated Security=True" providerName="System.Data.SqlClient"/> <!-- And this -->
		</connectionStrings>
		<system.web>
			<compilation debug="true" targetFramework="4.5.2"/>
			<httpRuntime targetFramework="4.5.2"/>
		</system.web>
		<system.serviceModel>
			<bindings>
				<basicHttpBinding>
					<binding name="JobsearcherWSSoap" />
				</basicHttpBinding>
				<customBinding>
					<binding name="JobsearcherWSSoap12">
						<textMessageEncoding messageVersion="Soap12" />
						<httpTransport />
					</binding>
				</customBinding>
			</bindings>
			<client>
				<endpoint address="http://example.com/acme.asmx"
				 binding="basicHttpBinding" bindingConfiguration="AcmeWSSoap"
				 contract="ServiceReferenceSample.AcmeWSSoap" name="AcmeWSSoap" /> <!-- And this -->
			</client>
		</system.serviceModel>
	</configuration>


Web.Mac.config
--------------

	<?xml version="1.0" encoding="utf-8"?>
	<configuration xmlns:xdt="http://schemas.microsoft.com/XML-Document-Transform">
		<connectionStrings>
			<add name="Default" connectionString="Data Source=127.0.0.1;Initial Catalog=Northwind;Integrated Security=True" xdt:Transform="SetAttributes" xdt:Locator="Match(name)"/>
		</connectionStrings>
		<appSettings>
			<add key="Sample" value="MAC was here" xdt:Transform="SetAttributes" xdt:Locator="Match(key)"/>
		</appSettings>
		<system.serviceModel>
			<client>
				<endpoint name="AcmeWSSoap" address="http://127.0.0.1/acme.asmx" xdt:Transform="SetAttributes" xdt:Locator="Match(name)" />
			</client>
		</system.serviceModel>
	</configuration>

Transformation rules are simple enought and can be copy pasted as much as needed.

To make your configuration file appear under Web.config like Release and Debug change your csproj like this:

	<ItemGroup>
		...
		<None Include="Web.Debug.config">
			<DependentUpon>Web.config</DependentUpon>
		</None>
		<!--
		Was:
		<None Include="Web.Mac.config" / >
		Now:
		-->
		<None Include="Web.Mac.config">
			<DependentUpon>Web.config</DependentUpon>
		</None>
		<None Include="Web.Release.config">
			<DependentUpon>Web.config</DependentUpon>
		</None>
	</ItemGroup>


Other notes:

If you want to use messages for debuging and do not see them in output window - [increase verbosity of msbuild](http://stackoverflow.com/a/3352309/1168586)

> Tools \ Options \ Projects and Solutions \ Build and Run:
> And set MSBUild verbosity from minimal to normal.

You can use environment variables like so:

	<Message Text="HELLO WORLD $(COMPUTERNAME)" />


It seems that there can be only one before/after build target

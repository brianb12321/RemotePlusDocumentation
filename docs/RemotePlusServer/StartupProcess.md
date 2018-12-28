# Overall Process
Depending on the server core loaded on the server, the process varies, but the general phases includes the following:

  * Building the IoC container
  * Creating WCF Builders
  * Reading configuration file
  * Loading known types
  * Reading resource file
  * Loading scripting engine
  * Scanning and Loading extension libraries
  * Building WCF services
  * Starting post initialization on all extension libraries
  * Show server GUI

## Building the IoC Container
In this phase, the server core will load the implementations of all the services in the system.

|Service      |Description     |
|-------------|:--------------:|
|ServerManager|Manages the life-cycle of all WCF services|
|ResourceManager|Provides APIs used for querying and creating resources|
|ErrorHandler|Catches all errors that occurrs when the a service is running|
|ExtensionContainer|Stores all the extension libraries loaded into the system|
|ServerControlPage|The GUI that appears after initialization|
|ScriptingEngine|Allows the execution of scripts from script files or from the command-line|
|ConfigurationDataAccess|Provides methods for reading and loading config files|
|Authentication|Provides methods for creating and authenticating user accounts|
|PackageManager|When RSPM is loaded, provides methods for installing packages|
|CommandLine|Provides services for the lexer, parser, and manages all the commands in the system|

!!! warning
    If a server core does not bind an implementation of a service, you will **NOT and SHOUT NOT** inject the service into your classes. Make sure that all the necessary services have been binded to the container.
	An Exception will be thrown by NInject if there is no binding for a particular service.

## Creating WCF Builders
In WCF, a `ServiceHost` must be provisioned before communication can occur, but once a `ServiceHost` has been created, no new known-types or initialization settings can be changed.
In RemotePlus, we have created builders used to build `ServiceHost` before creating the actual `ServiceHost`
You will be able to change binding settings, behaviors, etc. before the `Servicehost` has been provisioned.

!!! important
    You must register all known types during this phase of initialization. After this phase, no new known types will be recognized by the WCF runtime.

## Reading Configuration File
RemotePlus reads from a global configuration file for its settings. The file is located in the `Configurations\Server\` directory of your application folder with the name `GlobalServerSettings.config`.
The default configuration loader loads an XML file with the `DataContractSerializer` as its backing serializer.
Your configuration file should look like the following:

``` xml
<?xml version="1.0" encoding="utf-8"?>
<ServerSettings xmlns:i="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.datacontract.org/2004/07/RemotePlusLibrary.Configuration.ServerSettings">
  <BannedIPs xmlns:d2p1="http://schemas.microsoft.com/2003/10/Serialization/Arrays" i:nil="true" />
  <DiscoverySettings>
    <ConnectToBuiltInProxyServer>true</ConnectToBuiltInProxyServer>
    <Connection>
      <ProxyServerURL>net.tcp://localhost:8080/Proxy</ProxyServerURL>
    </Connection>
    <DiscoveryBehavior>Connect</DiscoveryBehavior>
    <Setup>
      <DiscoveryPort>9001</DiscoveryPort>
      <ProxyClientEndpointName>ProxyClient</ProxyClientEndpointName>
      <ProxyEndpointName>Proxy</ProxyEndpointName>
    </Setup>
  </DiscoverySettings>
  <EnableMetadataExchange>false</EnableMetadataExchange>
  <LoggingSettings xmlns:d2p1="http://schemas.datacontract.org/2004/07/RemotePlusLibrary.Configuration">
    <d2p1:CleanLogFolder>false</d2p1:CleanLogFolder>
    <d2p1:DateDelimiter>45</d2p1:DateDelimiter>
    <d2p1:LogFileCountThreashold>10</d2p1:LogFileCountThreashold>
    <d2p1:LogFolder>ServerLogs</d2p1:LogFolder>
    <d2p1:LogOnShutdown>true</d2p1:LogOnShutdown>
    <d2p1:TimeDelimiter>45</d2p1:TimeDelimiter>
  </LoggingSettings>
  <PortNumber>9000</PortNumber>
</ServerSettings>
```

!!! note
    If the config file does not exist, RemotePlus will create one for you.

!!! important
    Currently RemotePlus will not start if your config file has been damaged, or the folder in which the config file should be loaded does not exist.

## Loading Known Types
The WCF runtime uses SOAP as its underlying data structure. When a message gets sent over the wire, the WCF runtime must serialize the whole object graph and transmit it over the communication channel, then
the runtime will need to deserialize the object graph and rebuild it on the other side. During serialization, the `DataContractSerializer` must know ahead of time what types of objects can be serialized.
This is expecially important when dealing with inheritance, because the serializer must know about the sub types for it to support inheritance.
In order to satisfy the serializer's requirement, all extensions that will transmit their own types over the wire, must register their custom types into the `DefaultKnownTypesManager` before the server opens communication.

!!! warning
    If a component fails to register a custom type and tries to serialize it over the wire, the communication channel will transition into the faulted state and must be disposed.

## Loading Resource File
RemotePlus loads all custom resources from the `GlobalResources.rpr` file when starting. Resources are used for storing music, files, strings, etc. for use later or for easy retrieval.
You can give a command a resource as a placeholder for values or even a placeholder for entire files!

!!! warning
    In the current version of RemotePlus: All resources will be loaded into memory on startup. This could be an implication when using `MemoryResources`, because the whole byte array will be loaded
	from memory.

!!! warning
    Any `MemoryResources` will be saved to the file, thus, can lead to large resource files.

!!! hint
    Use the `$` prefix to reference a resource in a command.

## Loading Scripting Engine
The `DefaultServerCore.dll` will bind the scripting service to the default scripting engine, which is IronPython. IronPython is an open-source project that allows .NET developers to integrate Python into
their code. A .NET developer can expose APIs for scripts to consume, thus allowing an application to be extendable.
RemotePlus allows script writers to automate server tasks or create whole new APIs for RemotePlus.

!!! danger
    RemotePlus does not currently facilitate any sandboxing capabilities. **RUN SCRIPTS AT YOUR OWN RISK!!!** Not only could malicious scripts destroy your computer, but it could use your computer as a bot for other
	nafarious purposes. You should examine scripts before executing them on a server. **Just use your brain!**
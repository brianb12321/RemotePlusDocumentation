# What is RemotePlus
RemotePlus is a remote-control solution/api. RemotePlus is designed to be lightweight, yet powerful.

!!! note
    RemotePlus uses the MIT license. Please read the license file or visit the [open source page](https://opensource.org/licenses/MIT)
# Prerequisites
Like any software, RemotePlus needs a few pieces of software in order to function properly. RemotePlus requires both hardware and software in order to work properly.
## Software
You will need to install .NET Framework 4.7.1 on all machines running RemotePlus

!!! warning
    RemotePlus does **NOT** support .NET core right now. Some components in the class libraries uses APIs not supported by .NET Core

RemotePlus was designed with as few external dependencies required for you, the user, to install. You don't need SQL, A web server, SSH, etc--everything is contained into the RemotePlus package.

!!! inportant
    Any extensions you install onto the server may require you to install external dependencies.

## Hardware
The only hardware prerequisites required are the ones that you needed to install Word.
But the difference between Word and RemotePlus is RemotePlus requires a working network card to communicate with other servers and the client.

!!! note
    It is recommended to set a status IP address on the server(s) that computers will connect to. Not setting a static IP may result in the server's IP address being changed by DHCP.
	Please consult your operating system documentation for instructions on configuring a static IP address.

!!! important
    You must configure port forwarding on the router that the server is connecting to, except when the server is connecting to a proxy server. If you are using a proxy server, the proxy server must be
	available and is allowed to accept inbound communication. Please see your router documentation for instructions on configuring port forwarding.
# Server Core
RemotePlus utilizes a design pattern called Dependency Injection (DI) and Inversion of Control (IoC) for providing services to extensions in the system.
Part of DI and IoC is to configure what services to give to the modules, and one of the important tenants of RemotePlus is flexability. RemotePlus loads one library that will configure the IoC container
and initialize the server. We call this library the Server Core.
All RemotePlus servers and proxy servers must utilize a server core to function properly.

!!! note
    A default server core has been provided to you by default

When the server starts, the server will scan the current folder for the `DefaultServerCore.dll` file. If successful, RemotePlus will load it and begin the initialization phase, but
if the server core does not exist, the server will exit giving the following error:
```
Welcome to RemotePlusServer, version: 1.0.0.0


Starting server core to setup and initialize services.
FATAL ERROR: A server core is not present. Cannot start server.
```
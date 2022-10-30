# Connector

A connector is an easy way to include a pre-implemented data upload client in the information of an application.

The idea behind this entity is to allow applications that share commonly used schemas to have information regarding
the library that the developer can use to integrate with the platform directly instead of having to manually implement
the client themselves.

An application can have any number of connectors and a connector is compose of 2 things:

* `language`: The language of the connector library.
* `package_url`: The URL of the connector library to download.

[Official Templates](../official-templates/index.md) make use of connectors to allow you to quickly create applications that
share a common use.

---
description: "Learn more about: Weakly-typed JSON Serialization Sample"
title: "Weakly-typed JSON Serialization Sample"
ms.date: "03/30/2017"
ms.assetid: 0b30e501-4ef5-474d-9fad-a9d559cf9c52
---
# Weakly-typed JSON Serialization Sample

When serializing a user-defined type to a given wire format, or deserializing a wire format back into a user-defined type, the given user-defined type must be available on both the service and the client. Usually to accomplish this, the <xref:System.Runtime.Serialization.DataContractAttribute> attribute is applied to these user-defined types and the <xref:System.Runtime.Serialization.DataMemberAttribute> attribute is applied to their members. This mechanism also applies when working with JavaScript Object Notation (JSON) objects, as described in the topic [How to: Serialize and Deserialize JSON Data](../feature-details/how-to-serialize-and-deserialize-json-data.md).

In some scenarios, a Windows Communication Foundation (WCF) service or client must access JSON objects generated by a service or client that is outside of the control of the developer. As more Web services publicly expose JSON APIs, it can become impractical for the WCF developer to construct local user-defined types into which to deserialize arbitrary JSON objects.

The [WeaklyTypedJson sample](https://github.com/dotnet/samples/tree/main/framework/wcf) provides a mechanism that enables WCF developers to work with deserialized, arbitrary JSON objects, without creating user-defined types. This is known as *weakly-typed serialization* of JSON objects, because the type into which a JSON object deserializes is not known at compile time.

For example, a public Web service API returns the following JSON object, which describes some information about a user of the service.

```json
{"personal": {"name": "Paul", "age": 23, "height": 1.7, "isSingle": true, "luckyNumbers": [5,17,21]}, "favoriteBands": ["Band ABC", "Band XYZ"]}
```

To deserialize this object, a WCF client must implement the following user-defined types.

```csharp
[DataContract]
public class MemberProfile
 {
     [DataMember]
     public PersonalInfo personal;

     [DataMember]
     public string[] favoriteBands;
 }

 [DataContract]
public class PersonalInfo
 {
     [DataMember]
     public string name;

     [DataMember]
     public int age;

     [DataMember]
     public double height;

     [DataMember]
     public bool isSingle;

     [DataMember]
     public int[] luckyNumbers;
 }
```

This can be cumbersome, especially if the client has to handle more than one type of JSON object.

The `JsonObject` type provided by this sample introduces a weakly typed representation of the deserialized JSON object. `JsonObject` relies on the natural mapping between JSON objects and .NET Framework dictionaries, and the mapping between JSON arrays and .NET Framework arrays. The following code shows the `JsonObject` type.

```csharp
// Instantiation of JsonObject json omitted

string name = json["root"]["personal"]["name"];
int age = json["root"]["personal"]["age"];
double height = json["root"]["personal"]["height"];
bool isSingle = json["root"]["personal"]["isSingle"];
int[] luckyNumbers = {
                                     json["root"]["personal"]["luckyNumbers"][0],
                                     json["root"]["personal"]["luckyNumbers"][1],
                                     json["root"]["personal"]["luckyNumbers"][2]
                                 };
string[] favoriteBands = {
                                        json["root"]["favoriteBands"][0],
                                        json["root"]["favoriteBands"][1]
                                    };
```

Note that you can "browse" JSON objects and arrays without the need to declare their type at compile time. For an explanation of the requirement for the top-level `["root"]` object, see the topic [Mapping Between JSON and XML](../feature-details/mapping-between-json-and-xml.md).

> [!NOTE]
> The `JsonObject` class is provided as an example only. It has not been thoroughly tested, and should not be used in production environments. An obvious implication of weakly-typed JSON serialization is the lack of type-safety when working with `JsonObject`.

To use the `JsonObject` type, the client operation contract must use <xref:System.ServiceModel.Channels.Message> as its return type.

```csharp
[ServiceContract]
    interface IClientSideProfileService
    {
        // There is no need to write a DataContract for the complex type returned by the service.
        // The client will use a JsonObject to browse the JSON in the received message.

        [OperationContract]
        [WebGet(ResponseFormat = WebMessageFormat.Json)]
        Message GetMemberProfile();
    }
```

The `JsonObject` is then instantiated as shown in the following code.

```csharp
// Code to instantiate IClientSideProfileService channel omitted…

// Make a request to the service and obtain the Json response
XmlDictionaryReader reader = channel.GetMemberProfile().GetReaderAtBodyContents();

// Go through the Json as though it is a dictionary. There is no need to map it to a .NET CLR type.
JsonObject json = new JsonObject(reader);
```

The `JsonObject` constructor takes a <xref:System.Xml.XmlDictionaryReader>, which is obtained through the <xref:System.ServiceModel.Channels.Message.GetReaderAtBodyContents%2A> method. The reader contains an XML representation of the JSON message received by the client. For more information, see the topic [Mapping Between JSON and XML](../feature-details/mapping-between-json-and-xml.md).

The program produces the following output:

```console
Service listening at http://localhost:8000/.
To view the JSON output from the sample, navigate to http://localhost:8000/GetMemberProfile
This is Paul's page. I am 23 years old and I am 1.7 meters tall.
I am single.
My lucky numbers are 5, 17, and 21.
My favorite bands are Band ABC and Band XYZ.
```

### To set up, build, and run the sample

1. Ensure that you have performed the [One-Time Setup Procedure for the Windows Communication Foundation Samples](one-time-setup-procedure-for-the-wcf-samples.md).

2. Build the solution WeaklyTypedJson.sln as described in [Building the Windows Communication Foundation Samples](building-the-samples.md).

3. Run the solution.

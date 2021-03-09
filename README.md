# Apigee - Message Templates and functions

This API proxy demonstrates how to use the new capability in the Message Template within Apigee, specifically the functions available within Message Templates.

[Message Templates](https://docs.apigee.com/api-platform/reference/message-template-intro) in Apigee are used in numerous places - for the Payload within an AssignMessage policy, as well as in other elements.

- Message Templates have been enhanced to support functions
- the AssignMessage policy has now been enhanced to accept a template when assigning a value to a variable

## Disclaimer

This example is not an official Google product, nor is it part of an official Google product.

## In More Detail

The [Message Template](https://docs.apigee.com/api-platform/reference/message-template-intro) in Apigee Edge has recently been enhanced to support a number of static callable functions.
Things like `substring()` or `sha256base64()`. The full list is in the documentation page.

Shortly afterwards, Apigee extended the AssignMessage/AssignVariable to allow a Template element, which allows you to assign to a variable the result of a template. For example:

```
<AssignMessage name='AM-Variable-From-Template'>
  <IgnoreUnresolvedVariables>false</IgnoreUnresolvedVariables>
  <AssignVariable>
    <Name>assigned</Name>
    <Template>{variable1}-{variable2}</Template>
  </AssignVariable>
</AssignMessage>
```

What the above does is:
1. resolves the values of context variables variable1 and variable2
2. applies the template and concatenates those, with an intervening dash
3. assigns that to variable 'assigned'.


That shows how to insert the value of multiple context variables into a result.  But Message Templates now also accept functions.
For example, you can evaluate a jsonpath expression:

```
<AssignMessage name='AM-Variable-from-JSONPath'>
  <IgnoreUnresolvedVariables>false</IgnoreUnresolvedVariables>
  <AssignVariable>
    <Name>assigned</Name>
    <Template>{jsonPath(json_path_expression,json_value)}</Template>
  </AssignVariable>
</AssignMessage>
```

What the above does is:
1. resolves the values of context variables json_path_expression and json_value
2. Evaluates the json path expression to the value.
3. assigns the result of that to the variable 'assigned'.

There are a number of other functions. Consult the documentation for the full list.

# What's Available Here?

The attached proxy shows examples using these functions:

* [toUpperCase](./apiproxy/policies/AM-01-toUpperCase.xml)
* [toLowerCase](./apiproxy/policies/AM-02-toLowerCase.xml)
* [substring](./apiproxy/policies/AM-06-substring.xml)
* [createUuid](./apiproxy/policies/AM-11-createUuid.xml)
* [xeger](./apiproxy/policies/AM-12-xeger.xml)
* [replaceFirst](./apiproxy/policies/AM-13-replaceFirst.xml)
* [replaceAll](./apiproxy/policies/AM-14-replaceAll.xml)
* [encodeBase64](./apiproxy/policies/AM-15-encodeBase64.xml)
* [decodeBase64](./apiproxy/policies/AM-16-decodeBase64.xml)
* [md5Hex](./apiproxy/policies/AM-21-md5Hex.xml)
* [sha256Hex](./apiproxy/policies/AM-22-sha256Hex.xml)
* [md5Base64](./apiproxy/policies/AM-23-md5Base64.xml)
* [sha256Base64](./apiproxy/policies/AM-24-sha256Base64.xml)
* [timeFormatUTCMs](./apiproxy/policies/AM-31-timeformat.xml)
* [timeFormatUTC](./apiproxy/policies/AM-33-timeformat.xml)
* [randomLong](./apiproxy/policies/AM-41-randomLong.xml)
* [escapeXML](./apiproxy/policies/AM-51-escapeXML.xml)
* [escapeXML11](./apiproxy/policies/AM-52-escapeXML11.xml)
* [escapeJSON](./apiproxy/policies/AM-53-escapeJSON.xml)
* [encodeHTML](./apiproxy/policies/AM-54-encodeHTML.xml)
* [jsonPath](./apiproxy/policies/AM-81-jsonpath.xml)

To use the proxy you need to deploy it into your environment.  You can do that by manually zipping up the apiproxy directory and deploying it with the UI, or use a command-line tool like
[importAndDeploy.js](https://github.com/DinoChiesa/apigee-edge-js-examples/blob/main/importAndDeploy.js)
or Powershell's [Import-EdgeApi](https://github.com/DinoChiesa/Edge-Powershell-Admin/blob/develop/PSApigeeEdge/Public/Import-EdgeApi.ps1)
or similar, to deploy the proxy.

To invoke the examples, use this:

```
ORG=my-apigee-org
ENV=test
curl -i https://$ORG-$ENV.apigee.net/assignvariable-1/test?t=1
```

You can vary the value of the t queryparam from 1 to 99 to exercise the various tests.
(Not all of those numbers correspond to tests.)

For example,

```
$ curl -i https://$ORG-$ENV.apigee.net/messagetemplate-functions/test?t=93
HTTP/1.1 200 OK
Date: Tue, 09 Mar 2021 18:22:12 GMT
Content-Type: application/json
Content-Length: 108
Connection: keep-alive
APIProxy: messagetemplate-functions v8
system.uuid: cfc56ec0-8eaf-452b-ab1b-1b15fc29f837

{
    "status" : "ok",
    "case" : "template with reference to variable",
    "assigned": "2021.March.09"
}


$ curl -i https://$ORG-$ENV.apigee.net/messagetemplate-functions/test?t=92
HTTP/1.1 200 OK
Date: Tue, 09 Mar 2021 18:22:29 GMT
Content-Type: application/json
Content-Length: 118
Connection: keep-alive
APIProxy: messagetemplate-functions v8
system.uuid: c9c403c2-32ad-4476-ba82-4a39405ed63e

{
    "status" : "ok",
    "case" : "cascading timeFormatUTCMs() and toLowerCase()",
    "assigned": "2021.march.09"
}

```

## Hints on using functions within Message Templates

Message templates work in all sorts of situations, not just within the
Template element in AssignVariable. In all of the cases where you use
Message Templates, you can use these functions. The example API proxy here just
serves to demonstrate the capabilities.

When constructing your own function expressions, here are some rules to follow:

1. There should be no spaces between the curly braces and the function name.

   OK: `{createUuid()}`

   NOT OK:  `{ createUuid( ) }`

2. If there are arguments, use variables, or use
   `<IgnoreUnresolvedVariables>true</IgnoreUnresolvedVariables>`.

   For the following table, assume `plus10` has been assigned the value `10`.

   | `IgnoreUnresolvedVariables` | formula                 | ok?  |
   | --------------------------- | ----------------------- | ---- |
   | true                        | `{randomLong(10)}`      |  Y   |
   | true                        | `{randomLong(plus10)}`  |  Y   |
   | false                       | `{randomLong(10)}`      |  N   |
   | false                       | `{randomLong(plus10)}`  |  Y   |

   For example, this will work:
   ```
   <AssignMessage name='AM-FormattedTime'>
     <IgnoreUnresolvedVariables>true</IgnoreUnresolvedVariables> <!-- note -->
     <AssignVariable>
       <Name>assigned</Name>
       <Template>{randomLong(10,1000)}</Template>
     </AssignVariable>
   </AssignMessage>
   ```

   This will not work:
   ```
   <AssignMessage name='AM-FormattedTime'>
     <IgnoreUnresolvedVariables>false</IgnoreUnresolvedVariables> <!-- note -->
     <AssignVariable>
       <Name>assigned</Name>
       <Template>{randomLong(10,1000)}</Template>
     </AssignVariable>
   </AssignMessage>
   ```

   If you do wish to have `IgnoreUnresolvedVariables` as `false`, then, you can
   use a preceding AssignVariable with a Value element in the AssignMessage
   policy, to set a variable to hold a constant. Like this:

   ```
   <AssignMessage name='AM-FormattedTime'>
     <IgnoreUnresolvedVariables>false</IgnoreUnresolvedVariables>
     <AssignVariable>
       <Name>plus10</Name>
       <Value>10</Value>
     </AssignVariable>
     <AssignVariable>
       <Name>assigned</Name>
       <Template>{randomLong(plus10)}</Template>
     </AssignVariable>
   </AssignMessage>
   ```

3. There must be no spaces between the enclosing parens and the arguments.

   OK: `{randomLong(10)}`

   NOT OK: `{randomLong( 10 )}`

2. Use a comma and no spaces between multiple arguments.

   OK: `{substring(alpha,0,4)}`

   NOT OK: `{substring( alpha, 0, 4 )}`

2. Avoid invalid characters when specifying a time format string for `timeFormat` functions.

   OK:
   ```
   <AssignMessage name='AM-FormattedTime'>
     <IgnoreUnresolvedVariables>false</IgnoreUnresolvedVariables>
     <AssignVariable>
       <Name>myformat</Name>
       <Value>yyyyMM</Value>
     </AssignVariable>
     <AssignVariable>
       <Name>assigned</Name>
       <Template>{timeFormatUTCMs(myformat,system.timestamp)}</Template>
     </AssignVariable>
   </AssignMessage>
   ```

   NOT OK:
   ```
   <AssignMessage name='AM-FormattedTime'>
     <IgnoreUnresolvedVariables>false</IgnoreUnresolvedVariables>
     <AssignVariable>
       <Name>myformat</Name>
       <Value>YYYYMM</Value>  <!-- YYYY is not meaningful -->
     </AssignVariable>
     <AssignVariable>
       <Name>assigned</Name>
       <Template>{timeFormatUTCMs(myformat,system.timestamp)}</Template>
     </AssignVariable>
   </AssignMessage>
   ```


4. You cannot nest the function calls. Use separate, successive Templates. They are evaluated in order.

   OK:
   ```
   <AssignMessage name='AM-Cascade'>
     <IgnoreUnresolvedVariables>false</IgnoreUnresolvedVariables>
     <AssignVariable>
       <Name>myformat</Name>
       <Value>yyyy.MMMM.dd</Value>
     </AssignVariable>
     <AssignVariable>
       <Name>assigned</Name>
       <Template>{timeFormatUTCMs(myformat,system.timestamp)}</Template>
     </AssignVariable>
     <AssignVariable>
       <Name>assigned</Name>
       <Template>{toLowerCase(assigned)}</Template>
     </AssignVariable>
   </AssignMessage>
   ```

   NOT OK:
   ```
   <AssignMessage name='AM-Cascade-Not-OK'>
     <IgnoreUnresolvedVariables>false</IgnoreUnresolvedVariables>
     <AssignVariable>
       <Name>myformat</Name>
       <Value>yyyy.MMMM.dd</Value>
     </AssignVariable>
     <AssignVariable>
       <Name>assigned</Name>
       <Template>{toLowerCase(timeFormatUTCMs(myformat,system.timestamp))}</Template>
     </AssignVariable>
   </AssignMessage>
   ```

5. If you specify an invalid function, like `{invalidFunction(xyz)}`, your proxy will not deploy successfully. You will see the status `entities.UnknownStringFunction`, if you query the deployments, like this:

   ```
    "server" : [ {
        "error" : "entities.UnknownStringFunction",
        "pod" : {
          "name" : "flk682-1",
          "region" : "us-west1"
        },
        "status" : "error",
        "type" : [ "message-processor" ],
        "uUID" : "af125932-4d07-4be7-ae1c-57e7d7a5934f"
      }
   ```

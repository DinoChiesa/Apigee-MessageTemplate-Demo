# Apigee Edge - Message Templates and functions

This API proxy demonstrates how to use the new capability in the Message Template within Apigee Edge, specifically the functions available within Message templates.

[Message Templates](https://docs.apigee.com/api-platform/reference/message-template-intro) in Apigee Edge are used in numerous places - for the Payload within an AssignMessage policy, as well as in other elements.

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

* [jsonPath](./apiproxy/policies/AM-20.xml)
* [toUpperCase](./apiproxy/policies/AM-1.xml)
* [replaceAll](./apiproxy/policies/AM-13.xml)
* [encodeBase64](./apiproxy/policies/AM-15.xml)
* [sha256Base64](./apiproxy/policies/AM-16.xml)
* [sha256Hex](./apiproxy/policies/AM-17.xml)
* [escapeXML11](./apiproxy/policies/AM-18.xml)
* [escapeJSON](./apiproxy/policies/AM-19.xml)
* [substring](./apiproxy/policies/AM-6.xml)
* [createUuid](./apiproxy/policies/AM-11.xml)
* [xeger](./apiproxy/policies/AM-12.xml)
* [randomLong](./apiproxy/policies/AM-10.xml)
* [timeFormatUTC](./apiproxy/policies/AM-4.xml),[timeFormatUTCMs](./apiproxy/policies/AM-3.xml)

To use the proxy you need to deploy it into your environment.  You can do that by manually zipping up the apiproxy directory and deploying it with the UI, or use a command-line tool like
[importAndDeploy.js](https://github.com/DinoChiesa/apigee-edge-js/blob/master/examples/importAndDeploy.js)
or Powershell's [Import-EdgeApi](https://github.com/DinoChiesa/Edge-Powershell-Admin/blob/develop/PSApigeeEdge/Public/Import-EdgeApi.ps1)
or similar, to deploy the proxy.

To invoke the examples, use this:

```
ORG=my-apigee-org
ENV=test
curl -i https://$ORG-$ENV.apigee.net/assignvariable-1/test?t=1
```

You can vary the value of the t queryparam from 1 to 20 to exercise the various tests.


## Hints on using functions within Message Templates

Message templates work in all sorts of situations, not just within the
Template element in AssignVariable. In all of the cases where you use
Message Templates, you can use these functions. The example API proxy here just
serves to demonstrate the capabilities.

When constructing your own function expressions, here are some rules to follow:

1. There should be no spaces between the curly braces and the function name.

   OK: `{createUuid()}`

   NOT OK:  `{ createUuid( ) }`

2. If there are arguments, and some are numeric, use variables.

   OK: `{randomLong(plus_10)}`

   NOT OK: `{randomLong(10)}`

   To assist with this, just use a preceding AssignVariable with a Value element in the AssignMessage policy.
   ```
   <AssignMessage name='AM-FormattedTime'>
     <IgnoreUnresolvedVariables>false</IgnoreUnresolvedVariables>
     <AssignVariable>
       <Name>plus_10</Name>
       <Value>10</Value>
     </AssignVariable>
     <AssignVariable>
       <Name>assigned</Name>
       <Template>{randomLong(plus_10)}</Template>
     </AssignVariable>
   </AssignMessage>
   ```

3. There must be no spaces between the enclosing parens and the arguments.

   OK: `{randomLong(plus_10)}`

   NOT OK: `{randomLong( plus_10 )}`

2. Use a comma and no spaces between multiple arguments.

   OK: `{substring(alpha,zero,four)}`

   NOT OK: `{substring( alpha, zero, four )}`

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
       <Value>YYYYMM</Value>  << invalid
     </AssignVariable>
     <AssignVariable>
       <Name>assigned</Name>
       <Template>{timeFormatUTCMs(myformat,system.timestamp)}</Template>
     </AssignVariable>
   </AssignMessage>
   ```


4. You cannot nest the function calls. Use separate Templates.

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


---
date: 2018-01-22 21:49:38
title: Overview
description: Sparta overview
weight: 10
alwaysopen: true
---


This is a brief overview of Sparta's core concepts.  Additional information regarding specific features is available from the menu.

# Terms and Concepts

At a high level, Sparta transforms a **go** binary's registered lambda functions into a set of independently addressable AWS Lambda functions .  Additionally, Sparta provides microservice authors an opportunity to satisfy other requirements such as defining the IAM Roles under which their function will execute in AWS, additional infrastructure requirements, and telemetry and alerting information (via CloudWatch).

The table below summarizes some of the primary Sparta terminology.

<table style="width:90%">
  <!-- Row 1 -->
  <tr>
    <td>
      <h2>Service Name</h2>
      Sparta applications are deployed as a single unit, using the <b>ServiceName</b> as a stable logical identifier.  The <b>ServiceName</b> is used as your application's <a href="http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/pseudo-parameter-reference.html">CloudFormation StackName</a>
        {{< highlight go >}}
    stackName := "MyUniqueServiceName"
    sparta.Main(stackName,
      "Simple Sparta application",
      myLambdaFunctions,
      nil,
      nil)
        {{< /highlight >}}
    </td>
  </tr>
  <!-- Row 2 -->
  <tr>
      <td>
      <h2>Sparta Lambda Function</h2>
A Sparta-compatible lambda is a standard <a href="https://golang.org/pkg/net/http/">http.HandlerFunc</a> instance. Sparta translates the results of the <code>http.ResponseWriter</code> (both status and body) to determine the AWS Lambda response.</h5>
{{< highlight go >}}
func mySpartaHandler(w http.ResponseWriter,
                      r *http.Request) {
  // Lambda code
}
{{< /highlight >}}
    </td>
  </tr>
<!-- Row 3 -->
  <tr>
    <td>
      <h2>Privileges</h2>
      To support accessing other AWS resources in your <b>Go</b> function, Sparta users may define <a href="http://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html">IAM Roles</a> with tightly defined <a href="https://godoc.org/github.com/mweagle/Sparta#IAMRolePrivilege"><code>sparta.IAMRolePrivilege</code></a> values. This allows you to define the <i>minimal</i> set of privileges under which your <b>Go</b> function will execute.  The <code>Privilege.Resource</code> field value may also be a <a href="https://godoc.org/github.com/crewjam/go-cloudformation#StringExpr">StringExpression</a> referencing a CloudFormation dynamically provisioned entity.</h5>
{{< highlight go >}}
lambdaFn.RoleDefinition.Privileges = append(lambdaFn.RoleDefinition.Privileges,
  sparta.IAMRolePrivilege{
    Actions:  []string{"s3:GetObject", "s3:HeadObject"},
    Resource: "arn:aws:s3:::MyS3Bucket",
})
{{< /highlight >}}
    </td>
  </tr>
<!-- Row 4 -->
  <tr>
    <td>
      <h2>Permissions</h2>
      To configure AWS Lambda <a href="http://docs.aws.amazon.com/lambda/latest/dg/intro-core-components.html">Event Sources</a>, Sparta provides both <a href="https://godoc.org/github.com/mweagle/Sparta#LambdaPermission"><code>sparta.LambdaPermission</code></a> and service-specific <i>Permission</i> types; eg: <a href="https://godoc.org/github.com/mweagle/Sparta#CloudWatchEventsPermission"><code>sparta.CloudWatchEventsPermission</code></a>. The service-specific <i>Permission</i> types automatically register your lambda function with the remote AWS service, using each service's specific API.</h5>
{{< highlight go >}}
cloudWatchEventsPermission := sparta.CloudWatchEventsPermission{}
cloudWatchEventsPermission.Rules = make(map[string]sparta.CloudWatchEventsRule, 0)
cloudWatchEventsPermission.Rules["Rate5Mins"] = sparta.CloudWatchEventsRule{
  ScheduleExpression: "rate(5 minutes)",
}
lambdaFn.Permissions = append(lambdaFn.Permissions, cloudWatchEventsPermission)
{{< /highlight >}}
    </td>
  </tr>

<!-- Row 5 -->
  <tr>
    <td>
      <h2>Dynamic Resources</h2>
      Sparta applications can specify other <a href="http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html">AWS Resources</a> (eg, <i>SNS Topics</i>) as part of their application. The dynamic resource outputs can be referenced by Sparta lambda functions via <code>gocf.Ref</code> and <code>gocf.GetAtt</code> functions.</h5>
{{< highlight go >}}
snsTopicName := sparta.CloudFormationResourceName("SNSDynamicTopic")
snsTopic := &gocf.SNSTopic{
  DisplayName: gocf.String("Sparta Application SNS topic"),
})
lambdaFn := sparta.HandleAWSLambda(
  sparta.LambdaName(echoDynamicSNSEvent),
  http.HandlerFunc(echoDynamicSNSEvent),
  sparta.IAMRoleDefinition{})
lambdaFn.Permissions = append(lambdaFn.Permissions, sparta.SNSPermission{
	BasePermission: sparta.BasePermission{
		SourceArn: gocf.Ref(snsTopicName),
	},
})
{{< /highlight >}}
    </td>
  </tr>


<!-- Row 6 -->
  <tr>
    <td>
      <h2>Discovery</h2>
      To support Sparta lambda functions discovering dynamically assigned AWS values (eg, <i>S3 Bucket Names</i>), Sparta provides <code>sparta.Discover</code>. </h5>
{{< highlight go >}}
func echoS3DynamicBucketEvent(event *json.RawMessage,
  context *sparta.LambdaContext,
  w http.ResponseWriter,
  logger *logrus.Logger) {

  config, _ := sparta.Discover()
  // Use config to determine the bucket name to which RawMessage should be stored
}
{{< /highlight >}}
    </td>
  </tr>
</table>

Given a set of registered Sparta lambda function, a typical `provision` build to create a new service follows this workflow. Items with dashed borders are opt-in user behaviors.

{{< spartaflow >}}

During provisioning, Sparta uses [AWS Lambda-backed Custom Resources](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-custom-resources-lambda.html) to support operations for which CloudFormation doesn't yet support (eg, [API Gateway](https://aws.amazon.com/api-gateway/) creation).

At runtime, Sparta uses [NodeJS](http://docs.aws.amazon.com/lambda/latest/dg/programming-model.html) shims to proxy the request to your **go** handler.


# Next Steps

Writing a simple [Sparta Application](/docs/intro_example/intro/).
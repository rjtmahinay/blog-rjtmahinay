# AWS Lambda SnapStart using Micronaut

Recently AWS announced in re:Invent 2022 the AWS Lambda SnapStart feature. This feature improves the cold startup performance of Java applications which can be done by supported cloud-native frameworks like Spring Boot, Quarkus and Micronaut.

In this post, we'll see how it performs using Micronaut and the JDK runtime version used is Java 11 (Amazon Corretto).

## A simple architecture

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1670159955042/S9dBH08Ga.jpg align="left")

The architecture above shows that we will invoke a POST request via the API Gateway and it will be processed by our Lambda function using Micronaut. This function will be invoked a thousand times and with the help of AWS CloudWatch Logs and Insights, we can create a conclusion on SnapStart's performance.

## How to enable Lambda SnapStart?

Initially, when we create a function, SnapStart is disabled. This is a function called On-Demand.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1670160580267/BPUvdpKiU.png align="left")

SnapStart will work in two things, you should **Enable SnapStart** and **publish a new version** of it. You can create an alias to be used as a new target of your API Gateway.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1670160863346/e_w_gsXds.png align="left")

An example alias pointing to the new version.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1670160911853/pSsdWditI.png align="left")

## Request Invocation

I've created a simple POST request which has a query parameter, name.

`?name=Tristan`

The response to this request will be a JSON body with string transformation (concatenate the name with Hi).

```json
{
    "message": "Hi Tristan"
}
```

### Micronaut Code

The string transformation will be handled by a simple Micronaut Lambda code as seen below.

```java
@Slf4j
public class FunctionRequestHandler extends MicronautRequestHandler<APIGatewayProxyRequestEvent, APIGatewayProxyResponseEvent> {
    @Inject
    ObjectMapper objectMapper;

    @Override
    public APIGatewayProxyResponseEvent execute(APIGatewayProxyRequestEvent input) {
        Map<String, String> inputParam = input.getQueryStringParameters();
        APIGatewayProxyResponseEvent response = new APIGatewayProxyResponseEvent();
        try {
            String json = objectMapper.writeValueAsString(
                    Collections.singletonMap("message", "Hi " + inputParam.get("name")));
            response.setStatusCode(200);
            response.setBody(json);
        } catch (JsonProcessingException e) {
            response.setStatusCode(500);
        }
        return response;
    }
}
```

The function will accept an APIGatewayProxyRequestEvent which holds the query parameter object as a Map. We will retrieve the value via the name key. The response will be the name and the appended Hi word.

## Load Testing using Apache JMeter

To test the performance of this new feature. I've used JMeter as my load testing tool to invoke a concurrent request of 50 threads and 1000 loops which spans around ~5-10mins. Both the SnapStart and On-demand invocation of Lambda was tested by these constraints.

## Insights

Using CloudWatch logs and insights, I've performed some monitoring metrics for durations as prescribed in [SnapStart Monitoring](https://docs.aws.amazon.com/lambda/latest/dg/snapstart-monitoring.html) and got the different percentiles for each configuration.

| Configuration | 50th percentile | 99.9th percentile |
| --- | --- | --- |
| Standard (On-demand) | 3077.39ms | 3278.32ms |
| SnapStart - Enabled | 482.11ms | 678.03ms |

## Final Thoughts

This blog post shows the results of a simple Micronaut Function Application for Serverless by comparing the Lambda configuration with or without SnapStart enabled. We can see the app's latency improved due to ahead-of-time initialization optimization.

## Further Reading

For more information about Lambda SnapStart you can check out the ff. links

*   [Improving startup performance with Lambda SnapStart](https://docs.aws.amazon.com/lambda/latest/dg/snapstart.html)
    
*   [Starting up faster with AWS Lambda SnapStart](https://aws.amazon.com/blogs/compute/starting-up-faster-with-aws-lambda-snapstart/)
---
title: Testing
category: clients
permalink: /clients/http-client/testing.html
caption: Testing Http Client (MockEngine)
ktor_version_review: 1.2.0
---

Ktor exposes a `MockEngine` for the HttpClient. This engine allows simulating HTTP calls without actually connecting to the endpoint. It allows to set a code block, that can handle the request and generates a response.

{% include artifact.html kind="engine" class="io.ktor.client.engine.mock.MockEngine" artifact="io.ktor:ktor-client-mock:$ktor_version,io.ktor:ktor-client-mock-jvm:$ktor_version,io.ktor:ktor-client-mock-js:$ktor_version,io.ktor:ktor-client-mock-native:$ktor_version" test="true" %}

## Usage

The usage is straightforward: the MockEngine class has a method `addHandler` in `MockEngineConfig`, that receives a block/callback that will handle the request. This callback receives an `HttpRequest` as a parameter, and must return a `HttpResponseData`. There are many helper methods to construct the response.

Full API description and list of helper methods could be found [here](https://api.ktor.io/{{site.ktor_version}}/io.ktor.client.engine.mock/).

A sample illustrating this:

```kotlin
val client = HttpClient(MockEngine) {
    engine {
        addHandler { request ->
            when (request.url.fullUrl) {
                "https://example.org/" -> {
                    val responseHeaders = headersOf("Content-Type" to listOf(ContentType.Text.Plain.toString()))
                    respond("Hello, world", headers = responseHeaders)
                }
                else -> error("Unhandled ${request.url.fullUrl}")
            }
        }
    }
}

private val Url.hostWithPortIfRequired: String get() = if (port == protocol.defaultPort) host else hostWithPort
private val Url.fullUrl: String get() = "${protocol.name}://$hostWithPortIfRequired$fullPath"
```

If your HttpClient uses a serializer to parse the response, such as the JacksonSerializer, this should be mirrored in the HttpClient used for mocking. The MockEngine should respond with a ByteReadChannel containing the Json response desired.

Please refer to the example below:

```kotlin
data class HelloWorldResponse(val message: String)
val jsonMapper = jacksonObjectMapper()
val resp = HelloWorldResponse("Hello, World")
val mockEngine = MockEngine {
    request ->
    when (request.url.fullUrl) {
        "https://example.org/" -> {
            respond(
                ByteReadChannel(jsonMapper.writeValueAsString(resp).toByteArray(Charsets.UTF_8)),
                HttpStatusCode.OK,
                headersOf("Content-Type" to listOf(ContentType.Application.Json.toString()))
            )
        }
    }
}
val mockClient = HttpClient(mockEngine){
    install(JsonFeature){
        serializer = JacksonSerializer()
    }
}
```

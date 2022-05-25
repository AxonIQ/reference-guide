# Logging format

Axon Server logging is based on Spring logging.
For this reason you can customize logging configuration simply by
using [system properties](../admin-configuration/configuration.md).

## JSON format

If you want to have your logs in JSON format, you can customize the logback configuration. The property that needs to be
configured is `logging.config` with a path to your custom `logback.xml`.
An example how to configure this property would be `logging.config=logback.xml`
In case you have not already configured `logback.xml` for Axon Server, a good starting point for JSON logging format
is [this example](https://raw.githubusercontent.com/AxonIQ/axon-server-se/master/resources/json-logback.xml)

The `<layout>` XML tag is the one that configures the format of the output.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">
            <layout class="ch.qos.logback.contrib.json.classic.JsonLayout">
                <timestampFormat>yyyy-MM-dd'T'HH:mm:ss.SSSX</timestampFormat>
                <timestampFormatTimezoneId>Etc/UTC</timestampFormatTimezoneId>
                <appendLineSeparator>true</appendLineSeparator>

                <jsonFormatter class="ch.qos.logback.contrib.jackson.JacksonJsonFormatter">
                    <prettyPrint>false</prettyPrint>
                </jsonFormatter>
            </layout>
        </encoder>
    </appender>

    <logger name="io.axoniq" level="info" additivity="false">
        <appender-ref ref="STDOUT"/>
    </logger>

    <logger name="io.axoniq.axonserver.AxonServer" level="info" additivity="false">
        <appender-ref ref="STDOUT"/>
    </logger>
    <logger name="io.axoniq.axonserver.grpc.Gateway" level="info" additivity="false">
        <appender-ref ref="STDOUT"/>
    </logger>
    <logger name="io.axoniq.axonserver.grpc.internal.MessagingClusterServer" level="info" additivity="false">
        <appender-ref ref="STDOUT"/>
    </logger>
    <logger name="org.springframework.boot.web.embedded.tomcat.TomcatWebServer" level="info" additivity="false">
        <appender-ref ref="STDOUT"/>
    </logger>

    <logger name="org.springframework.http.converter.json.Jackson2ObjectMapperBuilder" level="error" additivity="false">
        <appender-ref ref="STDOUT"/>
    </logger>

    <logger name="org.hibernate.orm.deprecation" level="error" additivity="false">
        <appender-ref ref="STDOUT"/>
    </logger>

    <root level="warn">
        <appender-ref ref="STDOUT"/>
    </root>
</configuration>
```

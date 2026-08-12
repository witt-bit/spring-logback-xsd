# logback XSD

## Overview

The purpose of this project is to provide an XML Schema Definition file for use in Spring Boot `logback-spring.xml` configuration files, with support for `springProperty`, `springProfile`, and other Spring-specific extensions.

Based on [enricopulatzo/logback-XSD](https://github.com/enricopulatzo/logback-XSD) and [nkatsar/logback-XSD](https://github.com/nkatsar/logback-XSD).

## Usage

Spring Boot's `logback-spring.xml` does not use an XML namespace. Reference the XSD via `xsi:noNamespaceSchemaLocation`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:noNamespaceSchemaLocation="https://raw.githubusercontent.com/witt-bit/spring-logback-xsd/master/src/main/xsd/logback.xsd">

    <springProperty name="APP_NAME" source="spring.application.name" defaultValue="spring"/>

    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <root level="INFO">
        <appender-ref ref="STDOUT"/>
    </root>
</configuration>
```

> Note: `xsi:noNamespaceSchemaLocation` is only used by the IDE for validation and autocompletion. It has no effect at runtime.

Alternatively, configure an XML Catalog mapping in your IDE (see below) and omit the `xsi:noNamespaceSchemaLocation` attribute entirely.

### IntelliJ IDEA

**Preferences** → **Languages & Frameworks** → **Schemas and DTDs** → add a mapping:

- **URI**: `file:///path/to/spring-logback-xsd/src/main/xsd/logback.xsd`

### VS Code

Create `.vscode/settings.json`:

```json
{
  "xml.catalogs": ["catalog.xml"]
}
```

Create `catalog.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<catalog xmlns="urn:oasis:names:tc:entity:xmlns:xml:catalog">
    <uri name="file:///path/to/spring-logback-xsd/src/main/xsd/logback.xsd"
         uri="file:///path/to/spring-logback-xsd/src/main/xsd/logback.xsd"/>
</catalog>
```

> Requires the [Red Hat XML](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-xml) extension.

## Features

| Feature | Support |
|---------|---------|
| Element / attribute autocompletion | ✅ |
| `appender-ref` → `appender` validation (red squiggly when ref is wrong) | ✅ via `xsd:key`/`xsd:keyref` |
| `${...}` placeholder support (no false positive type errors) | ✅ typed elements relaxed to `xsd:string` |
| `${...}` variable navigation to `springProperty` source | ❌ runtime mechanism, out of XSD scope |

## Spring Boot Extensions

### `<springProperty>`

Reads a property from the Spring Environment and exposes it as a Logback variable:

```xml
<springProperty name="APP_NAME" source="spring.application.name" defaultValue="spring"/>
```

| Attribute | Required | Description |
|-----------|----------|-------------|
| `name` | yes | Logback variable name, referenced later as `${name}` |
| `source` | yes | Spring Environment property key |
| `defaultValue` | no | Default value when the property is not set |
| `scope` | no | One of `local`, `context`, or `system` |

### `<springProfile>`

Conditional configuration based on active Spring profiles:

```xml
<springProfile name="dev">
    <logger name="com.example" level="DEBUG"/>
</springProfile>

<springProfile name="staging,production">
    <root level="WARN"/>
</springProfile>
```

Accepts all standard logback child elements.

## Other Supported Elements

Beyond standard logback elements, the XSD also validates:

| Element | Description |
|---------|-------------|
| `<turboFilter>` | Global TurboFilter |
| `<queueSize>` | AsyncAppender queue size |
| `<discardingThreshold>` | AsyncAppender discarding threshold |
| `<neverBlock>` | AsyncAppender never-block-on-full |
| `<includeCallerData>` | AsyncAppender include caller data |
| `<maxFlushTime>` | AsyncAppender max flush time |
| `<define>` | Runtime property definition (with `name` and `class` attributes) |

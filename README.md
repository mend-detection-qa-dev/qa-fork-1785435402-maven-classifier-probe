# maven-classifier-probe

## Probe metadata

| Field           | Value                          |
|-----------------|-------------------------------|
| Pattern         | classifier-dep                |
| Package manager | Maven                         |
| Java version    | 17                            |
| Generated       | 2026-04-29                    |
| Target org      | mend-detection-qa             |

## Feature exercised

This probe exercises detection of Maven dependencies declared with a `<classifier>`
element — a non-default artifact variant published alongside the main jar. Mend SCA
must recognise the classifier and match the artifact to the correct entry in its
vulnerability database rather than treating it as the base artifact.

## Dependencies

### Direct (declared in pom.xml)

| Coordinate                          | Classifier      | Scope   |
|-------------------------------------|-----------------|---------|
| org.lwjgl:lwjgl:3.3.3               | natives-linux   | runtime |
| org.apache.httpcomponents:httpclient:4.5.5 | —         | compile |

### Transitive (pulled in by httpclient:4.5.5)

| Coordinate                                   | Scope   | Required by  |
|----------------------------------------------|---------|--------------|
| org.apache.httpcomponents:httpcore:4.4.9      | compile | httpclient   |
| commons-logging:commons-logging:1.2           | compile | httpclient   |
| commons-codec:commons-codec:1.10              | compile | httpclient   |

LWJGL has no compile-scope transitive dependencies; the natives-linux jar is a
self-contained native binary bundle.

## Expected dependency tree

```
com.example:maven-classifier-probe:jar:1.0.0
+- org.lwjgl:lwjgl:jar:natives-linux:3.3.3:runtime
+- org.apache.httpcomponents:httpclient:jar:4.5.5:compile
   +- org.apache.httpcomponents:httpcore:jar:4.4.9:compile
   +- commons-logging:commons-logging:jar:1.2:compile
   \- commons-codec:commons-codec:jar:1.10:compile
```

Total: 5 dependencies (2 direct, 3 transitive).

## What Mend should detect

1. `org.lwjgl:lwjgl:3.3.3` — identified via the base artifact coordinates; the
   classifier (`natives-linux`) indicates a native binary variant. Mend may or
   may not track the classifier separately; the base artifact must appear.
2. `org.apache.httpcomponents:httpclient:4.5.5` — direct, compile scope.
3. `org.apache.httpcomponents:httpcore:4.4.9` — transitive, compile scope.
4. `commons-logging:commons-logging:1.2` — transitive, compile scope.
5. `commons-codec:commons-codec:1.10` — transitive, compile scope.

## Known Mend behaviour note

Mend's Maven resolver-driven scanner resolves the full dependency tree from the
pom.xml using the Maven resolver. Classifier variants are passed through the
resolver; the scanner records them against the base groupId:artifactId:version
coordinates. If Mend strips the classifier before fingerprinting, the artifact
may appear as `lwjgl-3.3.3.jar` rather than `lwjgl-3.3.3-natives-linux.jar`.
This probe is designed to surface that discrepancy.
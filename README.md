# Pbandk-metadata-collision-reproduction
Reproduction project for an erorr that occurs when including the `pbandk-runtime-jvm` artifact into an Android application project.

When adding the following to a new Android application project:

```groovy
dependencies {
    ...
    implementation "pro.streem.pbandk:pbandk-runtime-jvm:0.9.0-alpha.2"
```

The build fails with the following error:

```
More than one file was found with OS independent path 'META-INF/kotlinx-serialization-runtime.kotlin_module'.
```

# Some observations:
## Caused by dependency on two `kotlinx-serialization` artifacts
This seems to be caused by the fact that `pbandk-runtime-jvm` includes two `kotlinx-serialization` artifacts in its [pom.xml](https://bintray.com/streem/pbandk/download_file?file_path=pro%2Fstreem%2Fpbandk%2Fpbandk-runtime-jvm%2F0.9.0-alpha.2%2Fpbandk-runtime-jvm-0.9.0-alpha.2.pom):

```
    <dependency>
      <groupId>org.jetbrains.kotlinx</groupId>
      <artifactId>kotlinx-serialization-runtime</artifactId>
      <version>0.20.0</version>
      <scope>runtime</scope>
    </dependency>
    ...
    <dependency>
      <groupId>org.jetbrains.kotlinx</groupId>
      <artifactId>kotlinx-serialization-runtime-common</artifactId>
      <version>0.20.0</version>
      <scope>runtime</scope>
    </dependency>
```

And both these artifacts define a `kotlinx-serialization-runtime.kotlin_module` file in their `META-INF` folder.

## Only happens for Android apps
This issue only occurs for Android application projects, not for Android library projects. But the consuming application project that uses the library will run into this issue.

## Short term workaround
Apps that already use `pbandk-runtime-jvm` can work around this issue by including:

```
android {
    ...
    packagingOptions {
        exclude "META-INF/kotlinx-serialization-runtime.kotlin_module"
    }
}
```

in their build.gradle file.

## Kotlinx.serialization 1.0.0
The lastest version of [`kotlinx.serialization`](https://github.com/Kotlin/kotlinx.serialization) seems to have dropped the two separate artifacts in favor of a single artifact: 

```
dependencies {
    ...
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-core:1.0.0-RC")
}
```

Which should also solve the issue.

# SBT Sonatype Apache

sbt-sonatype-apache is a project that is designed to set up Scala [sbt](https://www.scala-sbt.org/) projects to
deploy into [Apache's Maven Nexus](https://infra.apache.org/repository-faq.html) via the usage of
the [sbt-sonatype](https://github.com/xerial/sbt-sonatype)
plugin.

The plugin will handle initializing all the relevant sbt settings dealing with artifact generation and
repository/credentials to their appropriate values, typically speaking only a couple of settings need to be manually
set.

Once this is done you can then publish your projects Maven artifacts into Apache's Nexus by using
`publish`/`publishSigned` as is idiomatically done in sbt projects.

## Usage

Since this plugin is an [auto plugin](https://www.scala-sbt.org/1.x/api/sbt/AutoPlugin.html) that immediately triggers
after sbt-sonatype is loaded, the only steps that are strictly necessary is to add it into your `project/plugins.sbt`,
i.e.

```sbt
addSbtPlugin("org.mdedetrich" % "sbt-apache-sonatype" % "<version>")
```

And then set the only mandatory key which is `apacheSonatypeProjectProfile"`, see [below](#notable-sbt-plugin-keys).

### Enabling project to publish to Apache's Maven Nexus Repository

Initially this project has been setup for Apache Projects that use [github](https://github.com/) along with
[github actions](https://github.com/features/actions). If you haven't
done so already, you need to create an [Apache Infra](https://infra.apache.org/) ticket to add the relevant credentials
as github secrets into your repository/s, here is an example of such a
[ticket](https://issues.apache.org/jira/browse/INFRA-24087).

## Notable Sbt Plugin Keys

These are keys which either need to be set, have a somewhat likely chance to be overridden or have an expectation
of files being in certain locations. Note that if you have a sbt build with multiple projects you will need to use
the `ThisBuild` syntax, i.e. `ThisBuild / apacheSonatypeProjectProfile := "myproject"` to set the keys globally.

Also for the various settings that point to a file, if you happen to have a multi project sbt build and you want to
point to a single file (typically a single file in the root project dir) remember to use `LocalRootProject`, i.e.
if you want to set `apacheSonatypeDisclaimerFile` to point to a `DISCLAIMER` file in the root of your build, do
`ThisBuild / apacheSonatypeDisclaimerFile := Some(LocalRootProject.value / "DISCLAIMER")`.

* `apacheSonatypeProjectProfile`: This is meant to be the name of your Apache project (for example if your project is
  named `myproject` then the Sonatype profile name with be `org.apache.myproject`). This is the only setting that has to
  be set otherwise your sbt build will not load correctly.
* `apacheSonatypeLicenseFile`: A mandatory setting defaulting to a `LICENSE` file in your project's base directory to
  be included in artifacts. If for some reason the `LICENSE` is not in your projects base directory you need to override
  this. See https://infra.apache.org/apply-license.html#new.
* `apacheSonatypeNoticeFile`: A mandatory defaulting to a `NOTICE` file in your project's base directory to
  be included in artifacts. If for some reason the `NOTICE` is not in your projects base directory you need to override
  this. See https://infra.apache.org/apply-license.html#new.
* `apacheSonatypeDisclaimerFile`: An optional setting (defaulting to None) that if set is meant to point to a
  `DISCLAIMER` file to be included in artifacts. Note that disclaimers are typically only necessary for [Incubator
  Projects](https://incubator.apache.org/) or other exceptions, see
  https://incubator.apache.org/policy/incubation.html#disclaimers.
* `apacheSonatypeCredentialsProvider`: How to resolve the Apache Maven Nexus credentials. Defaults to
  `CredentialProvider.Environment` which is typically how it's passed in for Github actions however this can be
  overridden if you want to define how to resolve credentials (see the `CredentialsProvider` trait).
* `apacheSonatypeCredentialsLogLevel`: The log level to be used when logging about potential problems in resolving
  credentials, defaults to `Level.Debug`. If you are trying to diagnose issues with resolving credentials then increase
  the log level to either `Level.Warn` or `Level.Error`.

### Keys for Github Actions

These are keys specific to Apache projects that use Github along with Github Actions.

* `apacheSonatypeCredentialsUserEnvVar`: The environment variable where the Sonatype user is stored, defaults to
  `NEXUS_USER` which is typically the same name as the github secret that gets added by Apache Infrastructure team.
* `apacheSonatypeCredentialsPasswordEnvVar`: The environment variable where the Sonatype user is stored, defaults to
  `NEXUS_PW` which is typically the same name as the github secret that gets added by Apache Infrastructure team.

Note that since this plugin is resolving the credentials via the use of environment variables, a release manager
can also export the same environment variables when doing a full release (which is almost always done on a local machine
and not CI)

## Utility functions

This project exposes the following utility function which can be helpful in other cases

* `SonatypeApachePlugin.addFileToMetaInf`: This function is used internally (i.e. with keys such
  as `apacheSonatypeLicenseFile`) to mark files which will be added to the `META-INF` folder in created artifacts. You
  can manually call this function if you want to add other files to `META-INF` folder in generated artifacts.

## FAQ

### Why is this project loaded automatically instead of explicitly via .enablePlugins?

Since this plugin is designed to be used by Apache project/s, its intended that plugin settings are initialized
immediately and globally. It is actually disallowed for an Apache Project to deploy maven artifacts into a repository
outside of Apache's official Nexus repository (see https://infra.apache.org/release-distribution.html#unreleased) so
there shouldn't be any reason to have different publish settings for different sbt projects (aside from disabling
publishing for a specific sbt subproject which is done via `publish / skip := true` as is standard in sbt builds).

## Future goals

This project is yet to be tested for an actual release so its possible for it to extend other sbt ecosystem plugins
such as [sbt-pgp](https://github.com/sbt/sbt-pgp), i.e. `publishSigned` defaults to using
[bundle deployment](https://help.sonatype.com/repomanager3/integrations/bundle-development) and so if this is not
supported by Apache Maven Nexus repo than sbt-apache-sonatype would configure the relevant sbt-pgp settings to make
sure it works.

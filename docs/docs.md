<!-- Generated with Stardoc: http://skydoc.bazel.build -->

## Rules

- [docker_project](#docker_project)
- [info_file](#info_file)
- [maintainer](#maintainer)
- [privilege_config](#privilege_config)
- [protocol_file](#protocol_file)
- [resource_config](#resource_config)
- [service_config](#service_config)
- [usr_local_linker](#usr_local_linker)

## Functions

- [confirm_binary_matches_platform](#confirm_binary_matches_platform)
- [docker_compose](#docker_compose)
- [images](#images)
- [spk_component](#spk_component)

<a id="docker_project"></a>

## docker_project

<pre>
load("@rules_synology//:defs.bzl", "docker_project")

docker_project(<a href="#docker_project-name">name</a>, <a href="#docker_project-preload_image">preload_image</a>, <a href="#docker_project-projects">projects</a>)
</pre>

## Configure a collection of Docker Projects

The Docker Project is a [Worker](glossary.md#worker) that carries one or more docker-compose
definitions into a Synology NAS, and can be used to turn-up the project defined in the compose
files.

Each project's name and compose file are represented as a `docker_compose()` target; these targets
are collected in an list `projects` in a `docker_project()` target.

For example:

(`BUILD` file)

```
docker_compose(
    name = "nginx"
    compose = ":dc.yaml",
)

docker_compose(
    name = "cryptobot"
    compose = ":some_other_compose.yaml",
)

docker_project(
    name = "green_stack",
    projects = [ ":nginx", "cryptobot" ]
)
```

References:

- [Synology: Docker Project](https://help.synology.com/developer-guide/resource_acquisition/docker-project.html)

**ATTRIBUTES**

| Name                                                   | Description                                                | Type                                                                | Mandatory | Default |
| :----------------------------------------------------- | :--------------------------------------------------------- | :------------------------------------------------------------------ | :-------- | :------ |
| <a id="docker_project-name"></a>name                   | A unique name for this target.                             | <a href="https://bazel.build/concepts/labels#target-names">Name</a> | required  |         |
| <a id="docker_project-preload_image"></a>preload_image | OCI/Docker image to preload at install of the SPK          | String                                                              | optional  | `""`    |
| <a id="docker_project-projects"></a>projects           | List of docker_compose() targets to bundle to this project | <a href="https://bazel.build/concepts/labels">List of labels</a>    | optional  | `[]`    |

<a id="info_file"></a>

## info_file

<pre>
load("@rules_synology//:defs.bzl", "info_file")

info_file(<a href="#info_file-name">name</a>, <a href="#info_file-out">out</a>, <a href="#info_file-arch_strings">arch_strings</a>, <a href="#info_file-ctl_stop">ctl_stop</a>, <a href="#info_file-description">description</a>, <a href="#info_file-maintainer">maintainer</a>, <a href="#info_file-os_min_ver">os_min_ver</a>, <a href="#info_file-package_name">package_name</a>,
          <a href="#info_file-package_version">package_version</a>)
</pre>

## Create an INFO file

The INFO file is the simple metadata k/v dict of the SPK: a number of `key=value` pairs that define
attributes of the package.

This file is needed to inform the Synology NAS of the name and unique ID of the package, quote
maintainers and distributors, and indicate architecture and OS constraints to install.  I
personally expect that the simple format of this file allows the UI to show install dialogs with
minimal complexity.

Likely the reader will notice that the `maintainer()` block collects more information than it needs
to for the `INFO` file: the additional URL is optional, but can help track down a maintainer if a
user or collaborator needs to find them.  This is not necessarily intended to be the upstream
URL(s) of a project (ie github pages, docker URLs,etc) but is intended to be a valid URL that can
help precisely identify where the preferred path to find that maintainer.

This implementation mimics the defaults from Synology's documentation so that the bare minimum
attributes can be provided to unblock immediate progress.

### Examples

(`BUILD` file)

```
load("@rules_synology//:defs.bzl", "info_file", "maintainer")

maintainer(
    name = "chickenandpork",
    maintainer_name = "Allan Clark",
    maintainer_url = "http://linkedin.com/in/goldfish",
    visibility = ["//visibility:public"],
)

info_file(
    name = "floppydog_info",
    package_name = "FloppyDog",
    description = "Provides the FloppyDog set of CLI tools for Great Justice",
    maintainer = ":chickenandpork",
    os_min_ver = "7.0-1",  # correct-format=[^\d+(\.\d+){1,2}(-\d+){1,2}$]
    package_version = "1.0.0-1",
)
```

The `INFO` file generated by this looks similar to:

```
package="FloppyDog"
version="1.0.0-1"
os_min_ver="7.0-1"
description="Provides the FloppyDog set of CLI tools for Great Justice"
maintainer="Allan Clark"
arch="noarch"
ctl_stop="yes"
startable="yes"
thirdparty="yes"
```

`info_file()` uses [maintainer()](docs.md#maintainer) to carry information about the package's
maintainer: referring to a maintainer by target ID allows greatest re-use of a DRY data element,
and should allow the author to associate a maintainer globally maintained in a dependency resource:

(`MODULE.bazel`)

```
bazel_dep(name = "firehydrant_stuff", version = "1.2.3")
```

(`BUILD.bazel`)

```
info_file(
    name = "firehydrant_info",
    package_name = "FireHydrant",
    description = "A great Incident-Management resource made by SREs for SREs",
    maintainer = "@firehydrant_stuff//maint:robertross",
    os_min_ver = "7.0-1",  # correct-format=[^\d+(\.\d+){1,2}(-\d+){1,2}$]
    package_version = "1.0.0-1",
)
```

Ref:

- [Synology: INFO](https://help.synology.com/developer-guide/synology_package/INFO.html)

**ATTRIBUTES**

| Name                                                  | Description                                                                                                                                                                                                                   | Type                                                                | Mandatory | Default      |
| :---------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------ | :-------- | :----------- |
| <a id="info_file-name"></a>name                       | A unique name for this target.                                                                                                                                                                                                | <a href="https://bazel.build/concepts/labels#target-names">Name</a> | required  |              |
| <a id="info_file-out"></a>out                         | Name of the Info file, if INFO is not preferred (changing this is not recommended).                                                                                                                                           | <a href="https://bazel.build/concepts/labels">Label</a>             | optional  | `None`       |
| <a id="info_file-arch_strings"></a>arch_strings       | array of architectures (strings): \[ "alpine", ...\] (default: \["noarch"\]).                                                                                                                                                 | List of strings                                                     | optional  | `["noarch"]` |
| <a id="info_file-ctl_stop"></a>ctl_stop               | Indicates (boolean) whether there is a start-stop-script and the SPK can be started or stopped (previously: startable).                                                                                                       | Boolean                                                             | optional  | `True`       |
| <a id="info_file-description"></a>description         | Brief description of the package: copy-paste from the upstream if permissible.  Although this is permitted be a looooong single-line string, it does display on the UIs to install a package, so brevity is still encouraged. | String                                                              | required  |              |
| <a id="info_file-maintainer"></a>maintainer           | Maintainer of the build logic for the component (primary if multiple, a person)                                                                                                                                               | <a href="https://bazel.build/concepts/labels">Label</a>             | required  |              |
| <a id="info_file-os_min_ver"></a>os_min_ver           | Earliest version of DSM that can install the package; ie "DSM 7.1.1-42962".  There seems to be no handling of the extended hard-to-parse suffixes used by Synology such as "DSM 7.1.1-42962 Update 6".                        | String                                                              | required  |              |
| <a id="info_file-package_name"></a>package_name       | Name of the package, unique within Synology SPKs, hopefully resembles external package name                                                                                                                                   | String                                                              | required  |              |
| <a id="info_file-package_version"></a>package_version | Version of the package; although I recommend semver-ish X.Y.Z-BUILDNUM, Synology describes as being any string of numbers separated by periods, dash, or underscore                                                           | String                                                              | required  |              |

<a id="maintainer"></a>

## maintainer

<pre>
load("@rules_synology//:defs.bzl", "maintainer")

maintainer(<a href="#maintainer-name">name</a>, <a href="#maintainer-maintainer_name">maintainer_name</a>, <a href="#maintainer-maintainer_url">maintainer_url</a>)
</pre>

A simple wrapper for re-use and typing, this produces a Maintainer that can be used as an attribute to targets they maintain.

**ATTRIBUTES**

| Name                                                   | Description                    | Type                                                                | Mandatory | Default |
| :----------------------------------------------------- | :----------------------------- | :------------------------------------------------------------------ | :-------- | :------ |
| <a id="maintainer-name"></a>name                       | A unique name for this target. | <a href="https://bazel.build/concepts/labels#target-names">Name</a> | required  |         |
| <a id="maintainer-maintainer_name"></a>maintainer_name | -                              | String                                                              | required  |         |
| <a id="maintainer-maintainer_url"></a>maintainer_url   | -                              | String                                                              | optional  | `""`    |

<a id="privilege_config"></a>

## privilege_config

<pre>
load("@rules_synology//:defs.bzl", "privilege_config")

privilege_config(<a href="#privilege_config-name">name</a>, <a href="#privilege_config-out">out</a>, <a href="#privilege_config-run_as_package">run_as_package</a>, <a href="#privilege_config-run_as_root">run_as_root</a>)
</pre>

A function (currently stubbed) to define a privilege configuration (conf/privilege) configuring packages installed in Synology.

**ATTRIBUTES**

| Name                                                       | Description                    | Type                                                                | Mandatory | Default |
| :--------------------------------------------------------- | :----------------------------- | :------------------------------------------------------------------ | :-------- | :------ |
| <a id="privilege_config-name"></a>name                     | A unique name for this target. | <a href="https://bazel.build/concepts/labels#target-names">Name</a> | required  |         |
| <a id="privilege_config-out"></a>out                       | -                              | <a href="https://bazel.build/concepts/labels">Label</a>             | optional  | `None`  |
| <a id="privilege_config-run_as_package"></a>run_as_package | -                              | List of strings                                                     | optional  | `[]`    |
| <a id="privilege_config-run_as_root"></a>run_as_root       | -                              | List of strings                                                     | optional  | `[]`    |

<a id="protocol_file"></a>

## protocol_file

<pre>
load("@rules_synology//:defs.bzl", "protocol_file")

protocol_file(<a href="#protocol_file-name">name</a>, <a href="#protocol_file-out">out</a>, <a href="#protocol_file-package_name">package_name</a>, <a href="#protocol_file-service_config">service_config</a>)
</pre>

The Protocol file is used as a wrapper/collector of service_configs: a building-block to generate the actual file as a buildable object for use in a Port Config resource.  Format is an INI file, as discussed in the DSM_Developer_Guide_7_enu.pdf, section "Port" on p 128 in my copy (search for "Configure Format Template").  Effectively, it builds the file that is pointed-to by a port-config.protocol-file entry in a resource JSON file.

**ATTRIBUTES**

| Name                                                    | Description                    | Type                                                                | Mandatory | Default |
| :------------------------------------------------------ | :----------------------------- | :------------------------------------------------------------------ | :-------- | :------ |
| <a id="protocol_file-name"></a>name                     | A unique name for this target. | <a href="https://bazel.build/concepts/labels#target-names">Name</a> | required  |         |
| <a id="protocol_file-out"></a>out                       | -                              | <a href="https://bazel.build/concepts/labels">Label</a>             | optional  | `None`  |
| <a id="protocol_file-package_name"></a>package_name     | -                              | String                                                              | optional  | `""`    |
| <a id="protocol_file-service_config"></a>service_config | -                              | <a href="https://bazel.build/concepts/labels">List of labels</a>    | optional  | `[]`    |

<a id="resource_config"></a>

## resource_config

<pre>
load("@rules_synology//:defs.bzl", "resource_config")

resource_config(<a href="#resource_config-name">name</a>, <a href="#resource_config-resources">resources</a>, <a href="#resource_config-out">out</a>)
</pre>

A function to define a resource configuration (conf/resource) configuring packages installed in Synology.

**ATTRIBUTES**

| Name                                            | Description                    | Type                                                                | Mandatory | Default |
| :---------------------------------------------- | :----------------------------- | :------------------------------------------------------------------ | :-------- | :------ |
| <a id="resource_config-name"></a>name           | A unique name for this target. | <a href="https://bazel.build/concepts/labels#target-names">Name</a> | required  |         |
| <a id="resource_config-resources"></a>resources | -                              | <a href="https://bazel.build/concepts/labels">List of labels</a>    | required  |         |
| <a id="resource_config-out"></a>out             | -                              | <a href="https://bazel.build/concepts/labels">Label</a>             | optional  | `None`  |

<a id="service_config"></a>

## service_config

<pre>
load("@rules_synology//:defs.bzl", "service_config")

service_config(<a href="#service_config-name">name</a>, <a href="#service_config-description">description</a>, <a href="#service_config-dst_ports">dst_ports</a>, <a href="#service_config-port_forward">port_forward</a>, <a href="#service_config-src_ports">src_ports</a>, <a href="#service_config-title">title</a>)
</pre>

A function to define a service configuration (port-forward) for a package installed on Synology

**ATTRIBUTES**

| Name                                                 | Description                    | Type                                                                | Mandatory | Default |
| :--------------------------------------------------- | :----------------------------- | :------------------------------------------------------------------ | :-------- | :------ |
| <a id="service_config-name"></a>name                 | A unique name for this target. | <a href="https://bazel.build/concepts/labels#target-names">Name</a> | required  |         |
| <a id="service_config-description"></a>description   | -                              | String                                                              | required  |         |
| <a id="service_config-dst_ports"></a>dst_ports       | -                              | String                                                              | required  |         |
| <a id="service_config-port_forward"></a>port_forward | -                              | Boolean                                                             | optional  | `True`  |
| <a id="service_config-src_ports"></a>src_ports       | -                              | String                                                              | optional  | `""`    |
| <a id="service_config-title"></a>title               | -                              | String                                                              | required  |         |

<a id="usr_local_linker"></a>

## usr_local_linker

<pre>
load("@rules_synology//:defs.bzl", "usr_local_linker")

usr_local_linker(<a href="#usr_local_linker-name">name</a>, <a href="#usr_local_linker-bin">bin</a>, <a href="#usr_local_linker-etc">etc</a>, <a href="#usr_local_linker-lib">lib</a>)
</pre>

## Configure a /usr/local Linker

The /usr/local linker is a [Worker](glossary.md#worker) that softlinks payload binaries, libraries,
and config files to bin, lib, and etc subdirectories in /usr/local on package start and removes on
package stop.  If the link or file pre-exists, /usr/local worker will unlink() those files,
effectively overwriting.  Any failure to pre-delete or create a link results in the process
failing, triggering any rollback.

### Example

This will cause a softlink to be created in `/usr/local/bin` that points to
`/var/packages/<package>/target/bin/netfilter-mods`:

```
load("@rules_synology//:defs.bzl", "usr_local_linker")

usr_local_linker(
    name = "softlinks",
    bin = ["netfilter-mods"],
)

```

There is currently no method of linking to a target with a different name: the underlying config
that Synology doesn't offer that capability.

References:

- [Synology: /usr/local linker](https://help.synology.com/developer-guide/resource_acquisition/usrlocal_linker.html)

**ATTRIBUTES**

| Name                                   | Description                                      | Type                                                                | Mandatory | Default |
| :------------------------------------- | :----------------------------------------------- | :------------------------------------------------------------------ | :-------- | :------ |
| <a id="usr_local_linker-name"></a>name | A unique name for this target.                   | <a href="https://bazel.build/concepts/labels#target-names">Name</a> | required  |         |
| <a id="usr_local_linker-bin"></a>bin   | binary paths to softlink into /usr/local/bin     | List of strings                                                     | optional  | `[]`    |
| <a id="usr_local_linker-etc"></a>etc   | configfile paths to softlink into /usr/local/etc | List of strings                                                     | optional  | `[]`    |
| <a id="usr_local_linker-lib"></a>lib   | library paths to softlink into /usr/local/lib    | List of strings                                                     | optional  | `[]`    |

<a id="confirm_binary_matches_platform"></a>

## confirm_binary_matches_platform

<pre>
load("@rules_synology//:defs.bzl", "confirm_binary_matches_platform")

confirm_binary_matches_platform(<a href="#confirm_binary_matches_platform-binary">binary</a>, <a href="#confirm_binary_matches_platform-size">size</a>)
</pre>

**PARAMETERS**

| Name                                                      | Description               | Default Value |
| :-------------------------------------------------------- | :------------------------ | :------------ |
| <a id="confirm_binary_matches_platform-binary"></a>binary | <p align="center"> - </p> | none          |
| <a id="confirm_binary_matches_platform-size"></a>size     | <p align="center"> - </p> | `"small"`     |

<a id="docker_compose"></a>

## docker_compose

<pre>
load("@rules_synology//:defs.bzl", "docker_compose")

docker_compose(<a href="#docker_compose-name">name</a>, <a href="#docker_compose-compose">compose</a>, <a href="#docker_compose-project_name">project_name</a>, <a href="#docker_compose-path">path</a>, <a href="#docker_compose-debug">debug</a>)
</pre>

## Configure a single Docker Project (a docker-compose)

The Docker Project is a [Worker](glossary.md#worker) that carries one or more docker-compose
definitions into a Synology NAS, and can be used to turn-up the project defined in the compose
files.

The Synology Docker Project Worker expects a name and a compose file for each project.
docker_compose() is used to provide a name/compose pair, which are then collected in a
docker_project() target:

(`BUILD` file)

```
docker_compose(
    name = "nginx"
    compose = ":dc.yaml",
)

docker_compose(
    name = "cryptobot"
    compose = ":some_other_compose.yaml",
)

docker_project(
    name = "green_stack",
    projects = [ ":nginx", "cryptobot" ]
)
```

References:

- [Synology: Docker Project](https://help.synology.com/developer-guide/resource_acquisition/docker-project.html)

**PARAMETERS**

| Name                                                 | Description               | Default Value |
| :--------------------------------------------------- | :------------------------ | :------------ |
| <a id="docker_compose-name"></a>name                 | <p align="center"> - </p> | none          |
| <a id="docker_compose-compose"></a>compose           | <p align="center"> - </p> | none          |
| <a id="docker_compose-project_name"></a>project_name | <p align="center"> - </p> | `None`        |
| <a id="docker_compose-path"></a>path                 | <p align="center"> - </p> | `None`        |
| <a id="docker_compose-debug"></a>debug               | <p align="center"> - </p> | `False`       |

<a id="images"></a>

## images

<pre>
load("@rules_synology//:defs.bzl", "images")

images(<a href="#images-name">name</a>, <a href="#images-src">src</a>)
</pre>

**PARAMETERS**

| Name                         | Description               | Default Value         |
| :--------------------------- | :------------------------ | :-------------------- |
| <a id="images-name"></a>name | <p align="center"> - </p> | `"images"`            |
| <a id="images-src"></a>src   | <p align="center"> - </p> | `":PACKAGE_ICON.PNG"` |

<a id="spk_component"></a>

## spk_component

<pre>
load("@rules_synology//:defs.bzl", "spk_component")

spk_component(<a href="#spk_component-name">name</a>, <a href="#spk_component-spk">spk</a>, <a href="#spk_component-filename">filename</a>)
</pre>

Pull a member component from the SPK archive for further testing

spk_component pulls a component from the SPK file by passing the filename to an unpack; the result should be usable to confirm that the SPK packed up what was expected.

**PARAMETERS**

| Name                                        | Description               | Default Value |
| :------------------------------------------ | :------------------------ | :------------ |
| <a id="spk_component-name"></a>name         | <p align="center"> - </p> | none          |
| <a id="spk_component-spk"></a>spk           | <p align="center"> - </p> | none          |
| <a id="spk_component-filename"></a>filename | <p align="center"> - </p> | none          |
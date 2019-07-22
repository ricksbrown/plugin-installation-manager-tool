Plugin Installation Manager Tool
================================

[![Join the chat at https://gitter.im/jenkinsci/plugin-installation-manager-cli-tool](https://badges.gitter.im/jenkinsci/plugin-installation-manager-cli-tool.svg)](https://gitter.im/jenkinsci/plugin-installation-manager-cli-tool?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

The plugin manager downloads plugins and their dependencies into a folder so that they can easily be imported into an instance of Jenkins. The goal of this tool is to replace the [Docker install-plugins.sh script](https://github.com/jenkinsci/docker/blob/master/install-plugins.sh) and the many other implementations of plugin management that have been recreated across Jenkins.

### Usage

#### Getting Started
```
mvn clean install 
java -jar plugin-management-cli/target/plugin-management-cli-1.0-SNAPSHOT-jar-with-dependencies.jar --war /file/path/jenkins.war --plugin-file /file/path/plugins.txt --plugins delivery-pipeline-plugin:1.3.2 deployit-plugin
```

#### CLI Options
* `--plugin-file` or `-f`: (optional) Path to plugins.txt, which contains a list of plugins to install.
* `--plugin-download-directory` or `-d`: (optional) Path to the directory in which to install plugins, which can also be set via the PLUGIN_DIR environment variable (which replaces the previously used REF environment variable). If a CLI option is entered, it will override what is set in the environment variable.  If not set via CLI option or environment, will default to /usr/share/jenkins/ref/plugins or C:\ProgramData\Jenkins\Reference\Plugins, depending on the user's OS.
* `--plugins` or `-p`: (optional) List of plugins to install (see plugin format below), separated by a space.
* `--war` or `-w`: (optional) Path to Jenkins war file. If no war file is entered, will default to /usr/share/jenkins/jenkins.war or C:\ProgramData\Jenkins\jenkins.war, depending on the user's OS. Plugins that are already included in the Jenkins war will only be downloaded if their required version is newer than the one included.
* `--list` or `-l`: (optional) Lists plugin names and versions of: installed plugins (plugins that already exist in the plugin directory), bundled plugins (non-detached plugins that exist in the war file), plugins that will be downloaded (highest required versions of the requested plugins and dependencies that are not already installed), and the effective plugin set (the highest versions of all plugins that are already installed or will be installed)
* `--view-security-warnings`: (optional) Set to true to show if any of the user specified plugins have security warnings
* `--view-all-security-warnings`: (optional) Set to true to show all plugins that have security warnings.
* `--available-updates`: (optional) Set to true to show if any requested plugins have newer versions available. If a Jenkins version-specific update center is available, the latest plugin version will be determined based on that update center's data.
* `--jenkins-update-center`: (optional) Sets the main update center, which can also be set via the JENKINS_UC environment variable. If a CLI option is entered, it will override what is set in the environment variable. If not set via CLI option or environment variable, will default to https://updates.jenkins.io.
* `--jenkins-experimental-update-center`: (optional) Sets the experimental update center, which can also be set via the JENKINS_UC_EXPERIMENTAL environment variable. If a CLI option is entered, it will override what is set in the environment variable. If not set via CLI option or environment variable, will default to https://updates.jenkins.io/experimental.
* `--jenkins-incrementals-repo-mirror`: (optional) Sets the incrementals repository mirror, which can also be set via the JENKINS_INCREMENTALS_REPO_MIRROR environment variable. If a CLI option is entered, it will override what is set in the environment variable. If not set via CLI option or environment variable, will default to https://repo.jenkins-ci.org/incrementals.


#### Plugin Input Format
The expected format for plugins is `artifact Id:version:url`

Use plugin artifact ID, without -plugin extension. If a plugin cannot be downloaded, -plugin will be appended to the name and download will be retried. This is for cases in which plugins don't follow the rules about artifact ID (i.e. docker plugin).

The version and  download url are optional. By default, the latest version of the plugin will be downloaded. If both a version and an url are supplied, the version will not be used to determine the plugin download location and the library will attempt to download the plugin from the given url.

The following custom version specifiers can also be used: 

* `latest` - downloads the latest version from a version specific update center if one exists for the version in the Jenkins war file. If no version specific update center exists, will use the main update center [https://updates.jenkins.io](https://updates.jenkins.io)
* `experimental` - downloads the latest version from the [experimental update center](https://jenkins.io/doc/developer/publishing/releasing-experimental-updates/), which offers Alpha and Beta versions of plugins. Default value: [https://updates.jenkins.io/experimental](https://updates.jenkins.io/experimental)
* `incrementals;org.jenkins-ci.plugins.workflow;2.19-rc289.d09828a05a74` - downloads the plugin from the [incrementals repo](https://jenkins.io/blog/2018/05/15/incremental-deployment/). For this option you need to specify groupId of the plugin. Note that this value may change between plugin versions without notice. More information on incrementals and their use for Docker images can be found [here](https://github.com/jenkinsci/incrementals-tools#updating-versions-for-jenkins-docker-images).  

#### Examples
If an url is included, then a placeholder should be included for the version. Examples of plugin inputs:

* `github-branch-source` - will download the latest version
* `github-branch-source:latest` - will download the latest version
* `github-branch-source:2.5.3` - will download version 2.5.3
* `github-branch-source:experimental` - will download the latest version from the experimental update center
* `github-branch-source:2.5.2:https://updates.jenkins.io/2.121/latest/github-branch-source.hpi` - will download version of plugin at url regardless of requested version
* `github-branch-source:https://updates.jenkins.io/2.121/latest/github-branch-source.hpi` - will treat the url like the version, which is not likely the behavior you want
* `github-branch-source::https://updates.jenkins.io/2.121/latest/github-branch-source.hpi` - will download plugin from url 

If a plugin to be downloaded from the incrementals repository is requested using the -plugins option from the CLI, the plugin name should be enclosed in quotes, since the semi-colon can be interpretted as the end of the command.

```
java -jar plugin-management-cli/target/plugin-management-cli-1.0-SNAPSHOT-jar-with-dependencies.jar -p "workflow-support:incrementals;org.jenkins-ci.plugins.workflow;2.19-rc289.d09828a05a74"
```

For plugins listed in a .txt file, each plugin must be listed on a new line. Other [import formats](https://issues.jenkins-ci.org/browse/JENKINS-58147) coming soon. 

# Maven assembly 官方文档翻译

https://maven.apache.org/plugins/maven-assembly-plugin/examples/sharing-descriptors.html

## part  1 文件格式

程序集定义了通常以存档格式（如zip、tar或tar.gz）分发的文件集合，这些文件是由项目产生的。例如，一个项目可以生成一个ZIP程序集，该程序集在根目录中包含一个项目的JAR工件、在lib/directory中的运行时依赖项，以及一个用于启动独立应用程序的shell脚本。

```xml
<assembly xmlns="http://maven.apache.org/ASSEMBLY/2.1.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/ASSEMBLY/2.1.0 http://maven.apache.org/xsd/assembly-2.1.0.xsd">
  <id/>
  <formats/>
  <includeBaseDirectory/>
  <baseDirectory/>
  <includeSiteDirectory/>
  <containerDescriptorHandlers>
    <containerDescriptorHandler>
      <handlerName/>
      <configuration/>
    </containerDescriptorHandler>
  </containerDescriptorHandlers>
  <moduleSets>
    <moduleSet>
      <useAllReactorProjects/>
      <includeSubModules/>
      <includes/>
      <excludes/>
      <sources>
        <useDefaultExcludes/>
        <outputDirectory/>
        <includes/>
        <excludes/>
        <fileMode/>
        <directoryMode/>
        <fileSets>
          <fileSet>
            <useDefaultExcludes/>
            <outputDirectory/>
            <includes/>
            <excludes/>
            <fileMode/>
            <directoryMode/>
            <directory/>
            <lineEnding/>
            <filtered/>
            <nonFilteredFileExtensions/>
          </fileSet>
        </fileSets>
        <includeModuleDirectory/>
        <excludeSubModuleDirectories/>
        <outputDirectoryMapping/>
      </sources>
      <binaries>
        <outputDirectory/>
        <includes/>
        <excludes/>
        <fileMode/>
        <directoryMode/>
        <attachmentClassifier/>
        <includeDependencies/>
        <dependencySets>
          <dependencySet>
            <outputDirectory/>
            <includes/>
            <excludes/>
            <fileMode/>
            <directoryMode/>
            <useStrictFiltering/>
            <outputFileNameMapping/>
            <unpack/>
            <unpackOptions>
              <includes/>
              <excludes/>
              <filtered/>
              <nonFilteredFileExtensions/>
              <lineEnding/>
              <useDefaultExcludes/>
              <encoding/>
            </unpackOptions>
            <scope/>
            <useProjectArtifact/>
            <useProjectAttachments/>
            <useTransitiveDependencies/>
            <useTransitiveFiltering/>
          </dependencySet>
        </dependencySets>
        <unpack/>
        <unpackOptions>
          <includes/>
          <excludes/>
          <filtered/>
          <nonFilteredFileExtensions/>
          <lineEnding/>
          <useDefaultExcludes/>
          <encoding/>
        </unpackOptions>
        <outputFileNameMapping/>
      </binaries>
    </moduleSet>
  </moduleSets>
  <fileSets>
    <fileSet>
      <useDefaultExcludes/>
      <outputDirectory/>
      <includes/>
      <excludes/>
      <fileMode/>
      <directoryMode/>
      <directory/>
      <lineEnding/>
      <filtered/>
      <nonFilteredFileExtensions/>
    </fileSet>
  </fileSets>
  <files>
    <file>
      <source/>
      <sources/>
      <outputDirectory/>
      <destName/>
      <fileMode/>
      <lineEnding/>
      <filtered/>
    </file>
  </files>
  <dependencySets>
    <dependencySet>
      <outputDirectory/>
      <includes/>
      <excludes/>
      <fileMode/>
      <directoryMode/>
      <useStrictFiltering/>
      <outputFileNameMapping/>
      <unpack/>
      <unpackOptions>
        <includes/>
        <excludes/>
        <filtered/>
        <nonFilteredFileExtensions/>
        <lineEnding/>
        <useDefaultExcludes/>
        <encoding/>
      </unpackOptions>
      <scope/>
      <useProjectArtifact/>
      <useProjectAttachments/>
      <useTransitiveDependencies/>
      <useTransitiveFiltering/>
    </dependencySet>
  </dependencySets>
  <repositories>
    <repository>
      <outputDirectory/>
      <includes/>
      <excludes/>
      <fileMode/>
      <directoryMode/>
      <includeMetadata/>
      <groupVersionAlignments>
        <groupVersionAlignment>
          <id/>
          <version/>
          <excludes/>
        </groupVersionAlignment>
      </groupVersionAlignments>
      <scope/>
    </repository>
  </repositories>
  <componentDescriptors/>
</assembly>
```

### 标签说明

### assembly

|                          Element                          |   Type    | Description                                                  |
| :-------------------------------------------------------: | :-------: | :----------------------------------------------------------- |
|                           `id`                            | `String`  | 设置此assembly的id。这是此项目中文件特定assembly的符号名称。<br/>此外，除了通过将组装包的值附加到生成的归档文件来清楚地命名组装包外，在部署时，id还用作工件的分类器。 |
|                     `formats/format*`                     |  `List`   | 打包格式：jar等                                              |
|                  `includeBaseDirectory`                   | `boolean` | 在最终存档中包括一个基本目录。例如，如果您正在创建一个名为“your-app”的程序集，将includeBaseDirectory设置为true将创建一个包含此基本目录的存档。如果此选项设置为false，则创建的存档文件将其内容解压缩到当前目录**默认值为**：`true`。 |
|                      `baseDirectory`                      | `String`  | 设置生成的assembly archive的基目录。如果未设置此值且includeBaseDirectory==true，则将使用${project.build.finalName}。（自2.2-beta-1起） |
|                  `includeSiteDirectory`                   | `boolean` | 在 final archive中包括site directory。项目的site directory位置由Assembly Plugin的siteDirectory参数确定**默认值为**：`false`。 |
| `containerDescriptorHandlers/containerDescriptorHandler*` |  `List`   | **(Many)** 一组组件，将各种容器描述符从 normal archive stream中过滤出来，这样就可以对它们进行聚合然后添加。 |
|                  `moduleSets/moduleSet*`                  |  `List`   | **(Many)** 指定要包含在assembly中的模块文件。模块集通过提供一个或多个<moduleSet> 子元素来指定。 |
|                    `fileSets/fileSet*`                    |  `List`   | **(Many)** 指定要包含在assembly中的文件组。通过提供一个或多个子元素 <fileSet>来指定文件集。 |
|                       `files/file*`                       |  `List`   | **(Many)** 指定要包含在assembly中的单个文件。通过提供一个或多个子元素<file>来指定文件。 |
|              `dependencySets/dependencySet*`              |  `List`   | **(Many)** 指定要包含在assembly中的依赖项。dependencySet是通过提供一个或多个子元素<dependencySet>来指定的。 |
|                `repositories/repository*`                 |  `List`   | **(Many)** 指定要包含在assembly中的存储库（repository files）文件。通过提供一个或多个<repository>子元素来指定存储库。 |
|        `componentDescriptors/componentDescriptor*`        |  `List`   | 指定要包含在部件中的共享零部件xml文件位置。指定的位置必须相对于描述符的基本位置。如果描述符是通过类路径中的元素找到的，那么它指定的任何组件也将在类路径上找到。如果它是通过pathname通过<descriptor/>元素找到的，那么这里的值将被解释为相对于项目basedir的路径。当找到多个组件描述符时，它们的内容将被合并。通过提供一个或多个子 元素<componentDescriptor>来指定componentDescriptor。 |



### containerDescriptorHandler

为指向assembly archive的文件配置筛选器，以启用各种类型描述符片段（如组件）的聚合，xml，web.xml等。

| Element         | Type     | Description                                  |
| :-------------- | :------- | :------------------------------------------- |
| `handlerName`   | `String` | 处理程序的plexus角色提示，用于从容器中查找。 |
| `configuration` | `DOM`    | 处理程序的配置选项。                         |



### moduleSet

A moduleSet represent one or more project <module> present inside a project's pom.xml. This allows you to include sources or binaries belonging to a project's <modules>.

**NOTE:** When using <moduleSets> from the command-line, it is required to pass first the package phase by doing: "mvn package assembly:assembly". This bug/issue is scheduled to be addressed by Maven 2.1.

| Element                 | Type             | Description                                                  |
| :---------------------- | :--------------- | :----------------------------------------------------------- |
| `useAllReactorProjects` | `boolean`        | 如果设置为true，插件将包括当前reactor中的所有项目，以便在此模块集中处理。这些将遵守包括/排除规则。 (Since 2.2) **Default value is**: `false`. |
| `includeSubModules`     | `boolean`        | 如果设置为false，插件将在此模块集中排除子模块。否则，它将处理所有子模块，每个主题包括/排除规则。(Since 2.2-beta-1) **Default value is**: `true`. |
| `includes/include*`     | `List`           | **(Many)**当存在<include>子元素时，它们定义一组要包含的项目坐标。如果不存在，则<includes>表示所有有效值。工件坐标可以以简单的groupId:artifactId形式给出，也可以以groupId:artifactId:type[：classifier]：version形式完全限定。此外，还可以使用通配符，如*：maven-* |
| `excludes/exclude*`     | `List`           | **(Many)**当存在<exclude>子元素时，它们定义一组要排除的项目工件坐标。If none is present，则<excludes>表示不排除。工件坐标可以以简单的groupId:artifactId形式给出，也可以以groupId:artifactId:type[：classifier]：version形式完全限定。此外，还可以使用通配符，如*:maven-* |
| `sources`               | `ModuleSources`  | 如果存在，插件将在生成的assembly中包含此集合中包含的模块的源文件。 |
| `binaries`              | `ModuleBinaries` | 当出现这种情况时，插件将在生成的assembly中包含此集合中包含的模块的二进制文件。 |



### sources

Contains configuration options for including the source files of a project module in an assembly.

| Element                       | Type      | Description                                                  |
| :---------------------------- | :-------- | :----------------------------------------------------------- |
| `useDefaultExcludes`          | `boolean` | 在计算受此集合影响的文件时，是否应使用标准排除模式，例如那些匹配CVS和Subversion元数据文件的模式。对于向后兼容性，默认值为true。(Since 2.2-beta-1) **Default value is**: `true`. |
| `outputDirectory`             | `String`  | 相对于assembly根目录的根设置输出目录。例如，“log”将把指定的文件放在log目录中。 |
| `includes/include*`           | `List`    | **(Many)**当存在<include>子元素时，它们定义一组要包含的文件和目录。如果不存在，则<includes>表示所有有效值。 |
| `excludes/exclude*`           | `List`    | **(Many)** When <exclude> subelements are present, they define a set of files and directory to exclude. If none is present, then <excludes> represents no exclusions. |
| `fileMode`                    | `String`  | Similar to a UNIX permission, sets the file mode of the files included. THIS IS AN OCTAL VALUE. Format: (User)(Group)(Other) where each component is a sum of Read = 4, Write = 2, and Execute = 1. For example, the value 0644 translates to User read-write, Group and Other read-only. The default value is 0644 [(more on unix-style permissions)](http://www.onlamp.com/pub/a/bsd/2000/09/06/FreeBSD_Basics.html) |
| `directoryMode`               | `String`  | Similar to a UNIX permission, sets the directory mode of the directories included. THIS IS AN OCTAL VALUE. Format: (User)(Group)(Other) where each component is a sum of Read = 4, Write = 2, and Execute = 1. For example, the value 0755 translates to User read-write, Group and Other read-only. The default value is 0755. [(more on unix-style permissions)](http://www.onlamp.com/pub/a/bsd/2000/09/06/FreeBSD_Basics.html) |
| `fileSets/fileSet*`           | `List`    | **(Many)** Specifies which groups of files from each included module to include in the assembly. A fileSet is specified by providing one or more of <fileSet> subelements. (Since 2.2-beta-1) |
| `includeModuleDirectory`      | `boolean` | Specifies whether the module's finalName should be prepended to the outputDirectory values of any fileSets applied to it. (Since 2.2-beta-1) **Default value is**: `true`. |
| `excludeSubModuleDirectories` | `boolean` | Specifies whether sub-module directories below the current module should be excluded from fileSets applied to that module. This might be useful if you only mean to copy the sources for the exact module list matched by this ModuleSet, ignoring (or processing separately) the modules which exist in directories below the current one. (Since 2.2-beta-1) **Default value is**: `true`. |
| `outputDirectoryMapping`      | `String`  | Sets the mapping pattern for all module base-directories included in this assembly. NOTE: This field is only used if includeModuleDirectory == true. Default is the module's ${artifactId} in 2.2-beta-1, and ${module.artifactId} in subsequent versions. (Since 2.2-beta-1) **Default value is**: `${module.artifactId}`. |



### fileSet

A fileSet allows the inclusion of groups of files into the assembly.

| Element                                               | Type      | Description                                                  |
| :---------------------------------------------------- | :-------- | :----------------------------------------------------------- |
| `useDefaultExcludes`                                  | `boolean` | Whether standard exclusion patterns, such as those matching CVS and Subversion metadata files, should be used when calculating the files affected by this set. For backward compatibility, the default value is true. (Since 2.2-beta-1) **Default value is**: `true`. |
| `outputDirectory`                                     | `String`  | Sets the output directory relative to the root of the root directory of the assembly. For example, "log" will put the specified files in the log directory. |
| `includes/include*`                                   | `List`    | **(Many)** When <include> subelements are present, they define a set of files and directory to include. If none is present, then <includes> represents all valid values. |
| `excludes/exclude*`                                   | `List`    | **(Many)** When <exclude> subelements are present, they define a set of files and directory to exclude. If none is present, then <excludes> represents no exclusions. |
| `fileMode`                                            | `String`  | Similar to a UNIX permission, sets the file mode of the files included. THIS IS AN OCTAL VALUE. Format: (User)(Group)(Other) where each component is a sum of Read = 4, Write = 2, and Execute = 1. For example, the value 0644 translates to User read-write, Group and Other read-only. The default value is 0644. [(more on unix-style permissions)](http://www.onlamp.com/pub/a/bsd/2000/09/06/FreeBSD_Basics.html) |
| `directoryMode`                                       | `String`  | Similar to a UNIX permission, sets the directory mode of the directories included. THIS IS AN OCTAL VALUE. Format: (User)(Group)(Other) where each component is a sum of Read = 4, Write = 2, and Execute = 1. For example, the value 0755 translates to User read-write, Group and Other read-only. The default value is 0755. [(more on unix-style permissions)](http://www.onlamp.com/pub/a/bsd/2000/09/06/FreeBSD_Basics.html) |
| `directory`                                           | `String`  | Sets the absolute or relative location from the module's directory. For example, "src/main/bin" would select this subdirectory of the project in which this dependency is defined. |
| `lineEnding`                                          | `String`  | Sets the line-endings of the files in this fileSet. Valid values:**"keep"** - Preserve all line endings**"unix"** - Use Unix-style line endings (i.e. "\n")**"lf"** - Use a single line-feed line endings (i.e. "\n")**"dos"** - Use DOS-/Windows-style line endings (i.e. "\r\n")**"windows"** - Use DOS-/Windows-style line endings (i.e. "\r\n")**"crlf"** - Use carriage-return, line-feed line endings (i.e. "\r\n") |
| `filtered`                                            | `boolean` | Whether to filter symbols in the files as they are copied, using properties from the build configuration. (Since 2.2-beta-1) **Default value is**: `false`. |
| `nonFilteredFileExtensions/nonFilteredFileExtension*` | `List`    | **(Many)** Additional file extensions to not apply filtering (Since 3.2.0) |



### binaries

Contains configuration options for including the binary files of a project module in an assembly.

| Element                         | Type            | Description                                                  |
| :------------------------------ | :-------------- | :----------------------------------------------------------- |
| `outputDirectory`               | `String`        | Sets the output directory relative to the root of the root directory of the assembly. For example, "log" will put the specified files in the log directory, directly beneath the root of the archive. |
| `includes/include*`             | `List`          | **(Many)** When <include> subelements are present, they define a set of artifact coordinates to include. If none is present, then <includes> represents all valid values. Artifact coordinates may be given in simple groupId:artifactId form, or they may be fully qualified in the form groupId:artifactId:type[:classifier]:version. Additionally, wildcards can be used, as in *:maven-* |
| `excludes/exclude*`             | `List`          | **(Many)** When <exclude> subelements are present, they define a set of dependency artifact coordinates to exclude. If none is present, then <excludes> represents no exclusions. Artifact coordinates may be given in simple groupId:artifactId form, or they may be fully qualified in the form groupId:artifactId:type[:classifier]:version. Additionally, wildcards can be used, as in *:maven-* |
| `fileMode`                      | `String`        | Similar to a UNIX permission, sets the file mode of the files included. THIS IS AN OCTAL VALUE. Format: (User)(Group)(Other) where each component is a sum of Read = 4, Write = 2, and Execute = 1. For example, the value 0644 translates to User read-write, Group and Other read-only. The default value is 0644 [(more on unix-style permissions)](http://www.onlamp.com/pub/a/bsd/2000/09/06/FreeBSD_Basics.html) |
| `directoryMode`                 | `String`        | Similar to a UNIX permission, sets the directory mode of the directories included. THIS IS AN OCTAL VALUE. Format: (User)(Group)(Other) where each component is a sum of Read = 4, Write = 2, and Execute = 1. For example, the value 0755 translates to User read-write, Group and Other read-only. The default value is 0755. [(more on unix-style permissions)](http://www.onlamp.com/pub/a/bsd/2000/09/06/FreeBSD_Basics.html) |
| `attachmentClassifier`          | `String`        | When specified, the attachmentClassifier will cause the assembler to look at artifacts attached to the module instead of the main project artifact. If it can find an attached artifact matching the specified classifier, it will use it; otherwise, it will throw an exception. (Since 2.2-beta-1) |
| `includeDependencies`           | `boolean`       | If set to true, the plugin will include the direct and transitive dependencies of of the project modules included here. Otherwise, it will only include the module packages only. **Default value is**: `true`. |
| `dependencySets/dependencySet*` | `List`          | **(Many)** Specifies which dependencies of the module to include in the assembly. A dependencySet is specified by providing one or more of <dependencySet> subelements. (Since 2.2-beta-1) |
| `unpack`                        | `boolean`       | If set to true, this property will unpack all module packages into the specified output directory. When set to false module packages will be included as archives (jars). **Default value is**: `true`. |
| `unpackOptions`                 | `UnpackOptions` | Allows the specification of includes and excludes, along with filtering options, for items unpacked from a module artifact. (Since 2.2-beta-1) |
| `outputFileNameMapping`         | `String`        | Sets the mapping pattern for all NON-UNPACKED dependencies included in this assembly. (Since 2.2-beta-2; 2.2-beta-1 uses ${artifactId}-${version}${dashClassifier?}.${extension} as default value) NOTE: If the dependencySet specifies unpack == true, outputFileNameMapping WILL NOT BE USED; in these cases, use outputDirectory. See the plugin FAQ for more details about entries usable in the outputFileNameMapping parameter. **Default value is**: `${module.artifactId}-${module.version}${dashClassifier?}.${module.extension}`. |



### dependencySet

A dependencySet allows inclusion and exclusion of project dependencies in the assembly.

| Element                     | Type            | Description                                                  |
| :-------------------------- | :-------------- | :----------------------------------------------------------- |
| `outputDirectory`           | `String`        | Sets the output directory relative to the root of the root directory of the assembly. For example, "log" will put the specified files in the log directory, directly beneath the root of the archive. |
| `includes/include*`         | `List`          | **(Many)** When <include> subelements are present, they define a set of artifact coordinates to include. If none is present, then <includes> represents all valid values. Artifact coordinates may be given in simple groupId:artifactId form, or they may be fully qualified in the form groupId:artifactId:type[:classifier]:version. Additionally, wildcards can be used, as in *:maven-* |
| `excludes/exclude*`         | `List`          | **(Many)** When <exclude> subelements are present, they define a set of dependency artifact coordinates to exclude. If none is present, then <excludes> represents no exclusions. Artifact coordinates may be given in simple groupId:artifactId form, or they may be fully qualified in the form groupId:artifactId:type[:classifier]:version. Additionally, wildcards can be used, as in *:maven-* |
| `fileMode`                  | `String`        | Similar to a UNIX permission, sets the file mode of the files included. THIS IS AN OCTAL VALUE. Format: (User)(Group)(Other) where each component is a sum of Read = 4, Write = 2, and Execute = 1. For example, the value 0644 translates to User read-write, Group and Other read-only. The default value is 0644 [(more on unix-style permissions)](http://www.onlamp.com/pub/a/bsd/2000/09/06/FreeBSD_Basics.html) |
| `directoryMode`             | `String`        | Similar to a UNIX permission, sets the directory mode of the directories included. THIS IS AN OCTAL VALUE. Format: (User)(Group)(Other) where each component is a sum of Read = 4, Write = 2, and Execute = 1. For example, the value 0755 translates to User read-write, Group and Other read-only. The default value is 0755. [(more on unix-style permissions)](http://www.onlamp.com/pub/a/bsd/2000/09/06/FreeBSD_Basics.html) |
| `useStrictFiltering`        | `boolean`       | When specified as true, any include/exclude patterns which aren't used to filter an actual artifact during assembly creation will cause the build to fail with an error. This is meant to highlight obsolete inclusions or exclusions, or else signal that the assembly descriptor is incorrectly configured. (Since 2.2) **Default value is**: `false`. |
| `outputFileNameMapping`     | `String`        | Sets the mapping pattern for all dependencies included in this assembly. (Since 2.2-beta-2; 2.2-beta-1 uses ${artifactId}-${version}${dashClassifier?}.${extension} as default value). See the plugin FAQ for more details about entries usable in the outputFileNameMapping parameter. **Default value is**: `${artifact.artifactId}-${artifact.version}${dashClassifier?}.${artifact.extension}`. |
| `unpack`                    | `boolean`       | If set to true, this property will unpack all dependencies into the specified output directory. When set to false dependencies will be includes as archives (jars). Can only unpack jar, zip, tar.gz, and tar.bz archives. **Default value is**: `false`. |
| `unpackOptions`             | `UnpackOptions` | Allows the specification of includes and excludes, along with filtering options, for items unpacked from a dependency artifact. (Since 2.2-beta-1) |
| `scope`                     | `String`        | Sets the dependency scope for this dependencySet. **Default value is**: `runtime`. |
| `useProjectArtifact`        | `boolean`       | Determines whether the artifact produced during the current project's build should be included in this dependency set. (Since 2.2-beta-1) **Default value is**: `true`. |
| `useProjectAttachments`     | `boolean`       | Determines whether the attached artifacts produced during the current project's build should be included in this dependency set. (Since 2.2-beta-1) **Default value is**: `false`. |
| `useTransitiveDependencies` | `boolean`       | Determines whether transitive dependencies will be included in the processing of the current dependency set. If true, includes/excludes/useTransitiveFiltering will apply to transitive dependency artifacts in addition to the main project dependency artifacts. If false, useTransitiveFiltering is meaningless, and includes/excludes only affect the immediate dependencies of the project. By default, this value is true. (Since 2.2-beta-1) **Default value is**: `true`. |
| `useTransitiveFiltering`    | `boolean`       | Determines whether the include/exclude patterns in this dependency set will be applied to the transitive path of a given artifact. If true, and the current artifact is a transitive dependency brought in by another artifact which matches an inclusion or exclusion pattern, then the current artifact has the same inclusion/exclusion logic applied to it as well. By default, this value is false, in order to preserve backward compatibility with version 2.1. This means that includes/excludes only apply directly to the current artifact, and not to the transitive set of artifacts which brought it in. (Since 2.2-beta-1) **Default value is**: `false`. |



### unpackOptions

Specifies options for including/excluding/filtering items extracted from an archive. (Since 2.2-beta-1)

| Element                                               | Type      | Description                                                  |
| :---------------------------------------------------- | :-------- | :----------------------------------------------------------- |
| `includes/include*`                                   | `List`    | **(Many)** Set of file and/or directory patterns for matching items to be included from an archive as it is unpacked. Each item is specified as <include>some/path</include> (Since 2.2-beta-1) |
| `excludes/exclude*`                                   | `List`    | **(Many)** Set of file and/or directory patterns for matching items to be excluded from an archive as it is unpacked. Each item is specified as <exclude>some/path</exclude> (Since 2.2-beta-1) |
| `filtered`                                            | `boolean` | Whether to filter symbols in the files as they are unpacked from the archive, using properties from the build configuration. (Since 2.2-beta-1) **Default value is**: `false`. |
| `nonFilteredFileExtensions/nonFilteredFileExtension*` | `List`    | **(Many)** Additional file extensions to not apply filtering (Since 3.2.0) |
| `lineEnding`                                          | `String`  | Sets the line-endings of the files. (Since 2.2) Valid values:**"keep"** - Preserve all line endings**"unix"** - Use Unix-style line endings**"lf"** - Use a single line-feed line endings**"dos"** - Use DOS-style line endings**"crlf"** - Use Carraige-return, line-feed line endings |
| `useDefaultExcludes`                                  | `boolean` | Whether standard exclusion patterns, such as those matching CVS and Subversion metadata files, should be used when calculating the files affected by this set. For backward compatibility, the default value is true. (Since 2.2) **Default value is**: `true`. |
| `encoding`                                            | `String`  | Allows to specify the encoding to use when unpacking archives, for unarchivers that support specifying encoding. If unspecified, archiver default will be used. Archiver defaults generally represent sane (modern) values. |



### file

A file allows individual file inclusion with the option to change the destination filename not supported by fileSets. Note: either source or sources is required

| Element           | Type      | Description                                                  |
| :---------------- | :-------- | :----------------------------------------------------------- |
| `source`          | `String`  | Sets the absolute or relative path from the module's directory of the file to be included in the assembly. |
| `sources/source*` | `List`    | **(Many)** Set of absolute or relative paths from the module's directory of the files be combined and included in the assembly. |
| `outputDirectory` | `String`  | Sets the output directory relative to the root of the root directory of the assembly. For example, "log" will put the specified files in the log directory. |
| `destName`        | `String`  | Sets the destination filename in the outputDirectory. Default is the same name as the source's file. |
| `fileMode`        | `String`  | Similar to a UNIX permission, sets the file mode of the files included. THIS IS AN OCTAL VALUE. Format: (User)(Group)(Other) where each component is a sum of Read = 4, Write = 2, and Execute = 1. For example, the value 0644 translates to User read-write, Group and Other read-only. The default value is 0644 [(more on unix-style permissions)](http://www.onlamp.com/pub/a/bsd/2000/09/06/FreeBSD_Basics.html) |
| `lineEnding`      | `String`  | Sets the line-endings of the files in this file. Valid values are:**"keep"** - Preserve all line endings**"unix"** - Use Unix-style line endings (i.e. "\n")**"lf"** - Use a single line-feed line endings (i.e. "\n")**"dos"** - Use DOS-/Windows-style line endings (i.e. "\r\n")**"windows"** - Use DOS-/Windows-style line endings (i.e. "\r\n")**"crlf"** - Use carriage-return, line-feed line endings (i.e. "\r\n") |
| `filtered`        | `boolean` | Sets whether to determine if the file is filtered. **Default value is**: `false`. |



### repository

Defines a Maven repository to be included in the assembly. The artifacts available to be included in a repository are your project's dependency artifacts. The repository created contains the needed metadata entries and also contains both sha1 and md5 checksums. This is useful for creating archives which will be deployed to internal repositories.

**NOTE:** Currently, only artifacts from the central repository are allowed.

| Element                                         | Type      | Description                                                  |
| :---------------------------------------------- | :-------- | :----------------------------------------------------------- |
| `outputDirectory`                               | `String`  | Sets the output directory relative to the root of the root directory of the assembly. For example, "log" will put the specified files in the log directory, directly beneath the root of the archive. |
| `includes/include*`                             | `List`    | **(Many)** When <include> subelements are present, they define a set of artifact coordinates to include. If none is present, then <includes> represents all valid values. Artifact coordinates may be given in simple groupId:artifactId form, or they may be fully qualified in the form groupId:artifactId:type[:classifier]:version. Additionally, wildcards can be used, as in *:maven-* |
| `excludes/exclude*`                             | `List`    | **(Many)** When <exclude> subelements are present, they define a set of dependency artifact coordinates to exclude. If none is present, then <excludes> represents no exclusions. Artifact coordinates may be given in simple groupId:artifactId form, or they may be fully qualified in the form groupId:artifactId:type[:classifier]:version. Additionally, wildcards can be used, as in *:maven-* |
| `fileMode`                                      | `String`  | Similar to a UNIX permission, sets the file mode of the files included. THIS IS AN OCTAL VALUE. Format: (User)(Group)(Other) where each component is a sum of Read = 4, Write = 2, and Execute = 1. For example, the value 0644 translates to User read-write, Group and Other read-only. The default value is 0644 [(more on unix-style permissions)](http://www.onlamp.com/pub/a/bsd/2000/09/06/FreeBSD_Basics.html) |
| `directoryMode`                                 | `String`  | Similar to a UNIX permission, sets the directory mode of the directories included. THIS IS AN OCTAL VALUE. Format: (User)(Group)(Other) where each component is a sum of Read = 4, Write = 2, and Execute = 1. For example, the value 0755 translates to User read-write, Group and Other read-only. The default value is 0755. [(more on unix-style permissions)](http://www.onlamp.com/pub/a/bsd/2000/09/06/FreeBSD_Basics.html) |
| `includeMetadata`                               | `boolean` | If set to true, this property will trigger the creation of repository metadata which will allow the repository to be used as a functional remote repository. **Default value is**: `false`. |
| `groupVersionAlignments/groupVersionAlignment*` | `List`    | **(Many)** Specifies that you want to align a group of artifacts to a specified version. A groupVersionAlignment is specified by providing one or more of <groupVersionAlignment> subelements. |
| `scope`                                         | `String`  | Specifies the scope for artifacts included in this repository. (Since 2.2-beta-1) **Default value is**: `runtime`. |



### groupVersionAlignment

Allows a group of artifacts to be aligned to a specified version.

| Element             | Type     | Description                                                  |
| :------------------ | :------- | :----------------------------------------------------------- |
| `id`                | `String` | The groupId of the artifacts for which you want to align the versions. |
| `version`           | `String` | The version you want to align this group to.                 |
| `excludes/exclude*` | `List`   | **(Many)** When <exclude> subelements are present, they define the artifactIds of the artifacts to exclude. If none is present, then <excludes> represents no exclusions. An exclude is specified by providing one or more of <exclude> subelements. |



## 预定义的描述符文件（Pre-defined Descriptor Files）

There are four predefined descriptor formats available for reuse, packaged within the Assembly Plugin. Their descriptorIds are:

bin 、jar-with-dependencies、src、project.

### bin

使用bin作为程序集插件配置的描述符，以便创建项目的二进制分发存档。此内置描述符以三种存档格式生成具有分类器bin的程序集：tar.gz,tar.gz2和zip。
组装的归档文件包含通过运行mvn包生成的二进制JAR，以及项目根目录中可用的任何自述文件、许可证和通知文件。
以下是bin描述符格式：

```xml
<assembly xmlns="http://maven.apache.org/ASSEMBLY/2.1.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/ASSEMBLY/2.1.0 http://maven.apache.org/xsd/assembly-2.1.0.xsd">
  <id>bin</id>
  <formats>
    <format>tar.gz</format>
    <format>tar.bz2</format>
    <format>zip</format>
  </formats>
  <fileSets>
    <fileSet>
      <directory>${project.basedir}</directory>
      <outputDirectory></outputDirectory>
      <includes>
        <include>README*</include>
        <include>LICENSE*</include>
        <include>NOTICE*</include>
      </includes>
    </fileSet>
    <fileSet>
      <directory>${project.build.directory}</directory>
      <outputDirectory></outputDirectory>
      <includes>
        <include>*.jar</include>
      </includes>
    </fileSet>
    <fileSet>
      <directory>${project.build.directory}/site</directory>
      <outputDirectory>docs</outputDirectory>
    </fileSet>
  </fileSets>
</assembly>
```

实例操作

```xml
<!--语法-->
<plugin>  
    <artifactId>maven-assembly-plugin</artifactId>  
    <configuration>      
        <descriptorRefs>  
            <descriptorRef>bin</descriptorRef>  
        </descriptorRefs>  
    </configuration>  
</plugin>
<!--实例-->
<plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-assembly-plugin</artifactId>
        <version>3.3.0</version>
        <executions>
          <execution>
            <id>make-assembly</id>
            <phase>package</phase>
            <goals>
              <goal>single</goal>
            </goals>
            <configuration>
              <finalName>demo</finalName>
              <descriptorRefs>
                <descriptorRef>bin</descriptorRef>
              </descriptorRefs>
              <outputDirectory>output</outputDirectory>
              <archive>
                <manifest>
                  <addClasspath>true</addClasspath>
                  <mainClass>com.sugon.ConnectToMysql</mainClass>                             <!-- 你的主类名 -->
                </manifest>
              </archive>
            </configuration>
          </execution>
        </executions>
      </plugin>
```

![image-20220627143327562](E:\image\image-20220627143327562.png)

![image-20220627143703770](E:\image\image-20220627143703770.png)



![image-20220627143727849](E:\image\image-20220627143727849.png)



### jar-with-dependencies

使用`jar-with-dependencies`作为程序集插件配置的描述符，以便创建一个jar，其中包含项目的二进制输出及其未打包的依赖项。这个内置描述符使用jar归档格式生成一个带有分类器jar的程序集，该分类器jar具有依赖关系。
请注意，带有依赖项的jar只提供对`uber-jar`的基本支持。要获得更多控制，请使用Maven -Shade插件。

```xml
<assembly xmlns="http://maven.apache.org/ASSEMBLY/2.1.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/ASSEMBLY/2.1.0 http://maven.apache.org/xsd/assembly-2.1.0.xsd">
  <!-- TODO: a jarjar format would be better -->
  <id>jar-with-dependencies</id>
  <formats>
    <format>jar</format>
  </formats>
  <includeBaseDirectory>false</includeBaseDirectory>
  <dependencySets>
    <dependencySet>
      <outputDirectory>/</outputDirectory>
      <useProjectArtifact>true</useProjectArtifact>
      <unpack>true</unpack>
      <scope>runtime</scope>
    </dependencySet>
  </dependencySets>
</assembly>
```

```xml
<!--语法-->
<plugin>  
    <artifactId>maven-assembly-plugin</artifactId>  
    <configuration>      
        <descriptorRefs>  
            <descriptorRef>jar-with-dependencies</descriptorRef>  
        </descriptorRefs>  
    </configuration>  
</plugin>
<!--实例-->
```



![image-20220627144350560](E:\image\image-20220627144350560.png)



![image-20220627144512916](E:\image\image-20220627144512916.png)

会将程序的依赖一起打入jar包

### src

Use src as the descriptorRef in your assembly-plugin configuration to create source archives for your project. 归档文件将包含`project's /src`目录结构的内容，供用户参考。src描述符Id使用分类器src以三种格式生成程序集存档：tar.gz,tar.gz2和zip
以下是src描述符格式：

```xml
<assembly xmlns="http://maven.apache.org/ASSEMBLY/2.1.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/ASSEMBLY/2.1.0 http://maven.apache.org/xsd/assembly-2.1.0.xsd">
  <id>src</id>
  <formats>
    <format>tar.gz</format>
    <format>tar.bz2</format>
    <format>zip</format>
  </formats>
  <fileSets>
    <fileSet>
      <directory>${project.basedir}</directory>
      <includes>
        <include>README*</include>
        <include>LICENSE*</include>
        <include>NOTICE*</include>
        <include>pom.xml</include>
      </includes>
      <useDefaultExcludes>true</useDefaultExcludes>
    </fileSet>
    <fileSet>
      <directory>${project.basedir}/src</directory>
      <useDefaultExcludes>true</useDefaultExcludes>
    </fileSet>
  </fileSets>
</assembly>
```



```xml
<!--语法-->
<plugin>  
    <artifactId>maven-assembly-plugin</artifactId>  
    <configuration>      
        <descriptorRefs>  
            <descriptorRef>src</descriptorRef>  
        </descriptorRefs>  
    </configuration>  
</plugin>
<!--实例-->
<plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-assembly-plugin</artifactId>
        <version>3.3.0</version>
        <executions>
          <execution>
            <id>make-assembly</id>
            <phase>package</phase>
            <goals>
              <goal>single</goal>
            </goals>
            <configuration>
              <finalName>demo</finalName>
              <descriptorRefs>
                <descriptorRef>src</descriptorRef>
              </descriptorRefs>
              <outputDirectory>output</outputDirectory>
              <archive>
                <manifest>
                  <addClasspath>true</addClasspath>
                  <mainClass>com.sugon.ConnectToMysql</mainClass>                             <!-- 你的主类名 -->
                </manifest>
              </archive>
            </configuration>
          </execution>
        </executions>
      </plugin>
```

![image-20220627144130326](E:\image\image-20220627144130326.png)



![image-20220627144057619](E:\image\image-20220627144057619.png)



### project

Using the project <descriptorRef> in your Assembly Plugin configuration will produce an assembly containing your entire project, minus any build output that lands in the /target directory.

```xml
<assembly xmlns="http://maven.apache.org/ASSEMBLY/2.1.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/ASSEMBLY/2.1.0 http://maven.apache.org/xsd/assembly-2.1.0.xsd">
  <id>project</id>
  <formats>
    <format>tar.gz</format>
    <format>tar.bz2</format>
    <format>zip</format>
  </formats>
  <fileSets>
    <fileSet>
      <directory>${project.basedir}</directory>
      <outputDirectory></outputDirectory>
      <useDefaultExcludes>true</useDefaultExcludes>
      <excludes>
        <exclude>**/*.log</exclude>
        <exclude>**/${project.build.directory}/**</exclude>
      </excludes>
    </fileSet>
  </fileSets>
</assembly>
```

```xml
<!--语法-->
<plugin>  
    <artifactId>maven-assembly-plugin</artifactId>  
    <configuration>      
        <descriptorRefs>  
            <descriptorRef>project</descriptorRef>  
        </descriptorRefs>  
    </configuration>  
</plugin>
<!--实例-->
```

![image-20220627144720267](E:\image\image-20220627144720267.png)

![image-20220627144831891](E:\image\image-20220627144831891.png)

会将整个工程打包

### 扩展

```
Spring MVC中的路径匹配要比标准的web.xml要灵活的多。默认的策略实现了 org.springframework.util.AntPathMatcher，就像名字提示的那样，路径模式是使用了Apache Ant的样式路径，Apache Ant样式的路径有三种通配符匹配方法（在下面的表格中列出)这些可以组合出很多种灵活的路径模式

 

Wildcard    Description         

?    匹配任何单字符         

*    匹配0或者任意数量的字符         

**    匹配0或者更多的目录 

 

Path    Description         

/app/*.x    匹配(Matches)所有在app路径下的.x文件         

/app/p?ttern    匹配(Matches) /app/pattern 和 /app/pXttern,但是不包括/app/pttern         

/**/example    匹配(Matches) /app/example, /app/foo/example, 和 /example         

/app/**/dir/file.    匹配(Matches) /app/dir/file.jsp, /app/foo/dir/file.html,/app/foo/bar/dir/file.pdf, 和 /app/dir/file.java         

/**/*.jsp    匹配(Matches)任何的.jsp 文件
```



# Assembly 插件教程

https://blog.csdn.net/z69183787/article/details/102519491?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-0-102519491-blog-122193500.pc_relevant_antiscanv4&spm=1001.2101.3001.4242.1&utm_relevant_index=3

## maven中打包依赖的路径配置

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>2.6</version>
    <configuration>
        <excludes>
            <exclude>*.properties</exclude>
            <exclude>*.xml</exclude>
            <exclude>*.sh</exclude>
        </excludes>
        <archive>
            <manifest>
                <addClasspath>true</addClasspath>
                <classpathPrefix>lib</classpathPrefix>
                <mainClass>com.hhht.riskcontrol.thirdparty.tongdun.LoginServer</mainClass>
            </manifest>
            <manifestEntries>
                <Class-Path>conf/</Class-Path>
            </manifestEntries>
        </archive>
    </configuration>
</plugin>

```

<classpathPrefix>系统会将这个路径下所有的jar包加入到classpath路径中，
<Class-Path>系统会将这个路径加入到classpath中，主要是用于加载配置文件。

```
 <fileSet>
         <directory>src/main/lib</directory>
         <outputDirectory>lib</outputDirectory>
         <includes>
            <include>*.jar</include>
         </includes>
         <fileMode>0644</fileMode>
 </fileSet>
```

语法：将`<directory>src/main/lib</directory>`下的文件打包到`<outputDirectory>lib</outputDirectory>`,包含所有的`.jar`的jar'包，<fileMode></fileMode>标签指定文件属性，使用八进制表达，分别为(User)(Group)(Other)所属属性，默认为 0644。

## 使用内置的Assembly Descriptor

要使用maven-assembly-plugin，需要指定至少一个要使用的assembly descriptor 文件。默认情况下，maven-assembly-plugin内置了几个可以用的assembly descriptor：

- bin ： 类似于默认打包，会将bin目录下的文件打到包中；
- jar-with-dependencies ： 会将所有依赖都解压打包到生成物中；
- src ：只将源码目录下的文件打包；
- project ： 将整个project资源打包。









## 知识点1，assembly普通用法

https://blog.csdn.net/z69183787/article/details/102519491?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-0-102519491-blog-122193500.pc_relevant_antiscanv4&spm=1001.2101.3001.4242.1&utm_relevant_index=3

```xml
<plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-assembly-plugin</artifactId>
        <version>3.3.0</version>
        <executions>
          <execution>
            <id>make-assembly</id>
            <phase>package</phase>
            <goals>
              <goal>single</goal>
            </goals>
            <configuration>
              <finalName>demo</finalName>
              <descriptors>
                <!--描述文件路径-->
                <descriptor>src/assembly/assembly.xml</descriptor>
              </descriptors>
              <outputDirectory>output</outputDirectory>
            </configuration>
          </execution>
        </executions>
      </plugin>
</plugins>
```

![image-20220627132134347](E:\image\image-20220627132134347.png)

```xml
<assembly>
    <id>demo</id>

    <formats>
        <format>jar</format>
    </formats>

    <includeBaseDirectory>false</includeBaseDirectory>

    <fileSets>
        <fileSet>
            <directory>${project.build.directory}/classes</directory>
            <outputDirectory>classes</outputDirectory>
        </fileSet>
    </fileSets>

</assembly>
```

![image-20220627132422368](E:\image\image-20220627132422368.png)

classes里面包含的路径是`demo-demo.jar\classes\com\sugon`下以及目录就是class文件。

 

## 知识点2 assembly dependencySets

用来定制工程依赖 jar 包的打包方式，核心元素如下表所示。

| 元素            | 类型         | 作用                                 |
| :-------------- | :----------- | :----------------------------------- |
| outputDirectory | String       | 指定包依赖目录，该目录是相对于根目录 |
| includes        | List<String> | 包含依赖                             |
| excludes        | List<String> | 排除依赖                             |

```xml
<assembly>
    <id>demo</id>

    <formats>
        <format>jar</format>
    </formats>

    <includeBaseDirectory>false</includeBaseDirectory>
    
    <dependencySets>
        <dependencySet>
            <outputDirectory>/lib</outputDirectory>
            <excludes>
                <exclude>${project.groupId}:${project.artifactId}</exclude>
            </excludes>
        </dependencySet>
        <dependencySet>
            <outputDirectory>/</outputDirectory>
            <includes>
                <include>${project.groupId}:${project.artifactId}</include>
            </includes>
        </dependencySet>
    </dependencySets>
</assembly>
```

## 学习使用

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>pro-all</artifactId>
        <groupId>cn.whbing.pro</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
 
    <artifactId>mybatis-genarator-modify</artifactId>
 
    <!--mybatis-generator-plus源码找那个需要以下依赖-->
    <dependencies>
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>
        <dependency>
            <groupId>org.apache.ant</groupId>
            <artifactId>ant</artifactId>
            <version>1.9.1</version>
        </dependency>
    </dependencies>
 
    <build>
 
        <finalName>mybatis-genarator-modify</finalName>
 
        <!--该resource节点的加入是为了将dtd文件也打包，否则出错-->
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                    <include>**/*.tld</include>
                    <include>**/*.dtd</include>
                </includes>
                <filtering>false</filtering>
            </resource>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                    <include>**/*.tld</include>
                    <include>**/*.dtd</include>
                </includes>
                <filtering>false</filtering>
            </resource>
        </resources>
 
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>2.6</version>
                <!--assembly的配置-->
                <configuration>
                    <finalName>mybatis-genarator-modify</finalName>
 
                    <!--这里先使用自带的打包，后续再修改成复杂的-->
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                    <!--入口文件-->
                    <archive>
                        <manifest>
                            <addClasspath>true</addClasspath>
                            <mainClass>org.mybatis.generator.api.ShellRunner</mainClass>                             <!-- 你的主类名 -->
                        </manifest>
                    </archive>
                </configuration>
 
                <!--在package生命周期执行-->
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

## 问题

使用Java -jar 的方式运行程序，如果在assembly.xml里面定义了输出文件，该如何加载主类呢？

如配置

```xml
          <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-jar-plugin</artifactId>
          <configuration>
            <archive>
              <manifest>
                <addClasspath>true</addClasspath>
                <classpathPrefix>lib/</classpathPrefix>
                <mainClass>com.sugon.ConnectToMysql</mainClass>
              </manifest>
            </archive>
          </configuration>
          </plugin>
```

```xml
<assembly>
    <id>demo</id>
    <formats>
        <format>jar</format>
    </formats>

    <includeBaseDirectory>false</includeBaseDirectory>

    <fileSets>
        <fileSet>
            <directory>${project.build.directory}/classes</directory>
            <outputDirectory>classes</outputDirectory>
        </fileSet>
    </fileSets>
</assembly>
```

# 附录：xml模板

```xml
<assembly xmlns="http://maven.apache.org/ASSEMBLY/2.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/ASSEMBLY/2.0.0 http://maven.apache.org/xsd/assembly-2.0.0.xsd">
    <!--
        设置此程序集的标识。这是来自此项目的特定文件组合的符号名称。此外，除了用于通过将生成的归档的值附加到组合包以明确命名组合包之外，该ID在部署时用作工件的分类器。
    -->
    <!--string-->
    <id/>
    <!--
        (许多） 指定程序集的格式。通过目标参数而不是在这里指定格式通常会更好。例如，允许不同的配置文件生成不同类型的档案。
        可以提供多种格式，装配体插件将生成每种所需格式的档案。部署项目时，所有指定的文件格式也将被部署。
        通过在<format>子元素中提供以下值之一来指定格式：
        “zip” - 创建一个ZIP文件格式
        “tar” - 创建一个TAR格式
        “tar.gz”或“tgz” - 创建一个gzip'd TAR格式
        “tar.bz2”或“tbz2” - 创建一个bzip'd TAR格式
        “tar.snappy” - 创建一个灵活的TAR格式
        “tar.xz”或“txz” - 创建一个xz'd TAR格式
        “jar” - 创建一个JAR格式
        “dir” - 创建分解的目录格式
        “战争” - 创建一个WAR格式
    -->
    <!--List<String>-->
    <formats/>
    <!--
        在最终归档中包含一个基本目录。例如，如果您正在创建一个名为“your-app”的程序集，则将includeBaseDirectory设置为true将创建一个包含此基本目录的归档文件。
        如果此选项设置为false，则创建的存档将其内容解压缩到当前目录。
        默认值是：true。
    -->
    <!--boolean-->
    <includeBaseDirectory/>
    <!--
        设置生成的程序集归档的基本目录。如果没有设置，并且includeBaseDirectory == true，则将使用$ {project.build.finalName}。（从2.2-beta-1开始）
    -->
    <!--string-->
    <baseDirectory/>
    <!--
        在最终档案中包含一个网站目录。项目的站点目录位置由Assembly Plugin的siteDirectory参数确定。
        默认值是：false。
    -->
    <!--boolean-->
    <includeSiteDirectory/>

    <!--
        （许多） 从常规归档流中过滤各种容器描述符的组件集合，因此可以将它们聚合然后添加。
    -->
    <!--List<ContainerDescriptorHandlerConfig>-->
    <containerDescriptorHandlers>
        <!--
            配置文件头部的过滤器，以启用各种类型的描述符片段（如components.xml，web.xml等）的聚合。
        -->
        <containerDescriptorHandler>
            <!--
                处理程序的plexus角色提示，用于从容器中查找。
            -->
            <!--string-->
            <handlerName/>
            <!--
                处理程序的配置选项。
            -->
            <!--DOM-->
            <configuration/>
        </containerDescriptorHandler>
    </containerDescriptorHandlers>
    <!--
        （许多） 指定在程序集中包含哪些模块文件。moduleSet是通过提供一个或多个<moduleSet>子元素来指定的。
    -->
    <!--List<ModuleSet>-->
    <moduleSets>
        <!--
            moduleSet表示一个或多个在项目的pom.xml中存在的<module>项目。这使您可以包含属于项目<modules>的源代码或二进制文件。
            注意：从命令行使用<moduleSets>时，需要先通过“mvn package assembly：assembly”来传递包阶段。这个bug计划由Maven 2.1解决。
        -->
        <moduleSet>
            <!--
                如果设置为true，则该插件将包含当前反应堆中的所有项目，以便在此ModuleSet中进行处理。这些将被 纳入/排除(includes/excludes) 规则。（从2.2开始）
                默认值是：false。
            -->
            <!--boolean-->
            <useAllReactorProjects/>
            <!--
                如果设置为false，则该插件将从该ModuleSet中排除子模块的处理。否则，它将处理所有子模块，每个子模块都要遵守包含/排除规则。（从2.2-beta-1开始）
                默认值是：true。
            -->
            <!--boolean-->
            <includeSubModules/>
            <!--
                （许多） 当存在<include>子元素时，它们定义一组包含的项目坐标。如果不存在，则<includes>表示所有有效值。
                工件坐标可以以简单的groupId：artifactId形式给出，或者可以以groupId：artifactId：type [：classifier]：version的形式完全限定。
                另外，可以使用通配符，如*：maven- *
            -->
            <!--List<String>-->
            <includes/>
            <!--
                （许多） 当存在<exclude>子元素时，它们定义一组要排除的项目工件坐标。如果不存在，则<excludes>不表示排除。
                工件坐标可以以简单的groupId：artifactId形式给出，或者可以以groupId：artifactId：type [：classifier]：version的形式完全限定。
                另外，可以使用通配符，如*：maven- *
            -->
            <!--List<String>-->
            <excludes/>
            <!--
                当存在这个时，插件将在生成的程序集中包含这个集合中包含的模块的源文件。
                包含用于在程序集中包含项目模块的源文件的配置选项。
            -->
            <!--ModuleSources-->
            <sources>
                <!--
                    在计算受该集合影响的文件时，是否应该使用标准排除模式，例如那些匹配CVS和Subversion元数据文件的排除模式。为了向后兼容，默认值是true。（从2.2-beta-1开始）
                    默认值是：true。
                -->
                <!--boolean-->
                <useDefaultExcludes/>
                <!--
                    设置输出目录相对于程序集根目录的根目录。例如，“日志”将把指定的文件放在日志目录中。
                -->
                <!--string-->
                <outputDirectory/>
                <!--
                    （许多） 当<include>子元素存在时，它们定义一组要包含的文件和目录。如果不存在，则<includes>表示所有有效值。
                -->
                <!--List<String>-->
                <includes/>
                <!--
                    （许多） 当存在<exclude>子元素时，它们定义一组要排除的文件和目录。如果不存在，则<excludes>不表示排除。
                -->
                <!--List<String>-->
                <excludes/>
                <!--
                    与UNIX权限类似，设置所包含文件的文件模式。这是一个 OCTAL VALUE。格式：（用户）（组）（其他）其中每个组件是Read = 4，Write = 2和Execute = 1的总和。
                    例如，值0644转换为用户读写，组和其他只读。默认值是0644
                -->
                <!--string-->
                <fileMode/>
                <!--
                    与UNIX权限类似，设置包含的目录的目录模式。这是一个 OCTAL VALUE。格式：（用户）（组）（其他）[Format: (User)(Group)(Other) ] 其中每个组件是Read = 4，Write = 2和Execute = 1的总和。
                    例如，值0755转换为用户读写，Group和其他只读。默认值是0755.
                -->
                <!--string-->
                <directoryMode/>
                <!--
                    （许多） 指定包含在程序集中的每个包含模块的哪些文件组。fileSet通过提供一个或多个<fileSet>子元素来指定。（从2.2-beta-1开始）
                -->
                <!--List<FileSet>-->
                <fileSets>
                    <!--
                        fileSet允许将文件组包含到程序集中。
                    -->
                    <fileSet>
                        <!--
                            在计算受该集合影响的文件时，是否应该使用标准排除模式，例如那些匹配CVS和Subversion元数据文件的排除模式。为了向后兼容，默认值是true。（从2.2-beta-1开始）
                            默认值是：true。
                        -->
                        <!--boolean-->
                        <useDefaultExcludes/>
                        <!--
                            设置输出目录相对于程序集根目录的根目录。例如，“日志”将把指定的文件放在日志目录中。
                        -->
                        <!--string-->
                        <outputDirectory/>
                        <!--
                            （许多） 当<include>子元素存在时，它们定义一组要包含的文件和目录。如果不存在，则<includes>表示所有有效值。
                        -->
                        <!--List<String>-->
                        <includes/>
                        <!--
                            （许多） 当存在<exclude>子元素时，它们定义一组要排除的文件和目录。如果不存在，则<excludes>不表示排除。
                        -->
                        <!--List<String>-->
                        <excludes/>
                        <!--
                            与UNIX权限类似，设置所包含文件的文件模式。这是一个 OCTAL VALUE。格式：（用户）（组）（其他）其中每个组件是Read = 4，Write = 2和Execute = 1的总和。
                            例如，值0644转换为用户读写，组和其他只读。默认值是0644.
                        -->
                        <!--string-->
                        <fileMode/>
                        <!--
                            与UNIX权限类似，设置包含的目录的目录模式。这是一个 OCTAL VALUE。格式：（用户）（组）（其他）其中每个组件是Read = 4，Write = 2和Execute = 1的总和。
                            例如，值0755转换为用户读写，Group和其他只读。默认值是0755.
                        -->
                        <!--string-->
                        <directoryMode/>
                        <!--
                            设置模块目录的绝对或相对位置。例如，“src / main / bin”会选择定义这个依赖关系的项目的这个子目录。
                        -->
                        <!--string-->
                        <directory/>
                        <!--
                            设置此文件集中文件的行结束符。有效值：
                            “keep” - 保留所有的行结束
                            “unix” - 使用Unix风格的行尾（即“\ n”）
                            “lf” - 使用一个换行符结束符（即“\ n”）
                            “dos” - 使用DOS / Windows风格的行尾（即“\ r \ n”）
                            “windows” - 使用DOS / Windows风格的行尾（即“\ r \ n”）
                            “crlf” - 使用回车，换行符结尾（即“\ r \ n”）
                        -->
                        <!--string-->
                        <lineEnding/>
                        <!--
                            是否在复制文件时过滤符号，使用构建配置中的属性。（从2.2-beta-1开始）
                            默认值是：false。
                        -->
                        <!--boolean-->
                        <filtered/>
                    </fileSet>
                </fileSets>
                <!--
                    指定模块的finalName是否应该添加到应用于它的任何fileSets的outputDirectory值。（从2.2-beta-1开始）
                    默认值是：true。
                -->
                <!--boolean-->
                <includeModuleDirectory/>
                <!--
                    指定是否应从应用于该模块的文件集中排除当前模块下方的子模块目录。如果仅仅意味着复制与此ModuleSet匹配的确切模块列表的源，忽略（或单独处理）当前目录下目录中存在的模块，这可能会很有用。（从2.2-beta-1开始）
                    默认值是：true。
                -->
                <!--boolean-->
                <excludeSubModuleDirectories/>
                <!--
                    设置此程序集中包含的所有模块基本目录的映射模式。注意：只有在includeModuleDirectory == true的情况下才会使用此字段。
                    缺省值是在 2.2-beta-1中是$ {artifactId}，以及后续版本中是$ {module.artifactId}。（从2.2-beta-1开始）
                    默认值是：$ {module.artifactId}。
                -->
                <!--string-->
                <outputDirectoryMapping/>
            </sources>
            <!--
                    如果存在，插件将在生成的程序集中包含来自该组的所包含模块的二进制文件。
                    包含用于将项目模块的二进制文件包含在程序集中的配置选项。
            -->
            <!--ModuleBinaries-->
            <binaries>
                <!--
                    设置输出目录相对于程序集根目录的根目录。例如，“log”会将指定的文件放在归档根目录下的日志目录中。
                -->
                <!--string-->
                <outputDirectory/>
                <!--
                    （许多） 当存在<include>子元素时，它们定义一组要包含的工件坐标。如果不存在，则<includes>表示所有有效值。
                    工件坐标可以以简单的groupId：artifactId形式给出，或者可以以groupId：artifactId：type [：classifier]：version的形式完全限定。
                    另外，可以使用通配符，如*：maven- *
                -->
                <!--List<String>-->
                <includes/>
                <!--
                    （许多） 当存在<exclude>子元素时，它们定义一组依赖项工件坐标以排除。如果不存在，则<excludes>不表示排除。
                    工件坐标可以以简单的groupId：artifactId形式给出，或者可以以groupId：artifactId：type [：classifier]：version的形式完全限定。
                    另外，可以使用通配符，如*：maven- *
                -->
                <!--List<String>-->
                <excludes/>
                <!--
                    与UNIX权限类似，设置所包含文件的文件模式。这是一个 OCTAL VALUE。格式：（用户）（组）（其他）其中每个组件是Read = 4，Write = 2和Execute = 1的总和。
                    例如，值0644转换为用户读写，组和其他只读。默认值是0644
                -->
                <!--string-->
                <fileMode/>
                <!--
                    与UNIX权限类似，设置包含的目录的目录模式。这是一个 OCTAL VALUE。格式：（用户）（组）（其他）[Format: (User)(Group)(Other) ] 其中每个组件是Read = 4，Write = 2和Execute = 1的总和。
                    例如，值0755转换为用户读写，Group和其他只读。默认值是0755.
                -->
                <!--string-->
                <directoryMode/>
                <!--
                    指定时，attachmentClassifier将使汇编器查看附加到模块的工件，而不是主工程工件。如果能够找到与指定分类符匹配的附件，则会使用它; 否则，会抛出异常。（从2.2-beta-1开始）
                -->
                <!--string-->
                <attachmentClassifier/>
                <!--
                    如果设置为true，插件将包含这里包含的项目模块的直接和传递依赖关系。否则，它将只包含模块包。
                    默认值是：true。
                -->
                <!--boolean-->
                <includeDependencies/>
                <!--List<DependencySet>-->
                <dependencySets>
                    <!--
                        依赖关系集允许在程序集中包含和排除项目依赖关系。
                    -->
                    <dependencySet>
                        <!--
                                设置输出目录相对于程序集根目录的根目录。例如，“log”会将指定的文件放在归档根目录下的日志目录中。
                            -->
                        <!--string-->
                        <outputDirectory/>
                        <!--
                            （许多） 当存在<include>子元素时，它们定义一组要包含的工件坐标。如果不存在，则<includes>表示所有有效值。
                            工件坐标可以以简单的groupId：artifactId形式给出，或者可以以groupId：artifactId：type [：classifier]：version的形式完全限定。
                            另外，可以使用通配符，如*：maven- *
                        -->
                        <!--List<String>-->
                        <includes/>
                        <!--
                            （许多） 当存在<exclude>子元素时，它们定义一组依赖项工件坐标以排除。如果不存在，则<excludes>不表示排除。
                            工件坐标可以以简单的groupId：artifactId形式给出，或者可以以groupId：artifactId：type [：classifier]：version的形式完全限定。
                            另外，可以使用通配符，如*：maven- *
                        -->
                        <!--List<String>-->
                        <excludes/>
                        <!--
                            与UNIX权限类似，设置所包含文件的文件模式。这是一个 OCTAL VALUE。格式：（用户）（组）（其他）其中每个组件是Read = 4，Write = 2和Execute = 1的总和。
                            例如，值0644转换为用户读写，组和其他只读。默认值是0644
                        -->
                        <!--string-->
                        <fileMode/>
                        <!--
                            与UNIX权限类似，设置包含的目录的目录模式。这是一个 OCTAL VALUE。格式：（用户）（组）（其他）[Format: (User)(Group)(Other) ] 其中每个组件是Read = 4，Write = 2和Execute = 1的总和。
                            例如，值0755转换为用户读写，Group和其他只读。默认值是0755.
                        -->
                        <!--string-->
                        <directoryMode/>
                        <!--
                            如果指定为true，那么在程序集创建过程中任何用于过滤实际构件的包含/排除模式都将导致构建失败，并显示错误。这是为了强调过时的包含或排除，或者表示程序集描述符配置不正确。（从2.2开始）
                            默认值是：false。
                        -->
                        <!--boolean-->
                        <useStrictFiltering/>
                        <!--
                            为此程序集中包含的所有依赖项设置映射模式。（从2.2-beta-2开始； 2.2-beta-1使用$ {artifactId} - $ {version} $ {dashClassifier？}。$ {extension}作为默认值）。
                            默认值是：$ {artifact.artifactId} - $ {artifact.version} $ {dashClassifier？}。$ {artifact.extension}。
                        -->
                        <!--string-->
                        <outputFileNameMapping/>
                        <!--
                            如果设置为true，则此属性将所有依赖项解包到指定的输出目录中。设置为false时，依赖关系将被包含为档案（jar）。只能解压jar，zip，tar.gz和tar.bz压缩文件。
                            默认值是：false。
                        -->
                        <!--boolean-->
                        <unpack/>
                        <!--
                            允许指定包含和排除以及过滤选项，以指定从相关性工件解压缩的项目。（从2.2-beta-1开始）
                        -->
                        <unpackOptions>
                            <!--
                                （许多） 文件和/或目录模式的集合，用于匹配将在解压缩时从归档文件中包含的项目。每个项目被指定为<include> some / path </ include>（从2.2-beta-1开始）
                            -->
                            <!--List<String>-->
                            <includes/>
                            <!--
                                （许多） 用于匹配项目的文件和/或目录模式的集合，在解压缩时将其从归档文件中排除。每个项目被指定为<exclude> some / path </ exclude>（从2.2-beta-1开始）
                            -->
                            <!--List<String>-->
                            <excludes/>
                            <!--
                                是否使用构建配置中的属性过滤从档案中解压缩的文件中的符号。（从2.2-beta-1开始）
                                默认值是：false。
                            -->
                            <!--boolean-->
                            <filtered/>
                            <!--
                                设置文件的行尾。（从2.2开始）有效值：
                                “keep” - 保留所有的行结束
                                “unix” - 使用Unix风格的行结尾
                                “lf” - 使用单个换行符结束符
                                “dos” - 使用DOS风格的行尾
                                “ crlf ” - 使用Carraige返回，换行符结束
                            -->
                            <!--string-->
                            <lineEnding/>
                            <!--
                                在计算受该集合影响的文件时，是否应该使用标准排除模式，例如那些匹配CVS和Subversion元数据文件的排除模式。为了向后兼容，默认值是true。（从2.2开始）
                                默认值是：true。
                            -->
                            <!--boolean-->
                            <useDefaultExcludes/>
                            <!--
                                允许指定解压档案时使用的编码，支持指定编码的unarchiver。如果未指定，将使用归档程序默认值。Archiver默认值通常代表理智（modern）的values。
                            -->
                            <!--string-->
                            <encoding/>
                        </unpackOptions>
                        <!--
                            为此dependencySet设置依赖项范围。
                            默认值是：runtime。
                        -->
                        <!--string-->
                        <scope/>
                        <!--
                            确定当前项目构建过程中产生的工件是否应该包含在这个依赖集中。（从2.2-beta-1开始）
                            默认值是：true。
                        -->
                        <!--boolean-->
                        <useProjectArtifact/>
                        <!--
                            确定当前项目构建过程中产生的附件是否应该包含在这个依赖集中。（从2.2-beta-1开始）
                            默认值是：false。
                        -->
                        <!--boolean-->
                        <useProjectAttachments/>
                        <!--
                            确定是否将传递依赖项包含在当前依赖项集的处理中。如果为true，那么include / excludes / useTransitiveFiltering将应用于传递依赖项构件以及主项目依赖项构件。
                            如果为false，则useTransitiveFiltering无意义，并且包含/排除仅影响项目的直接依赖关系。
                            默认情况下，这个值是真的。（从2.2-beta-1开始）
                            默认值是：true。
                        -->
                        <!--boolean-->
                        <useTransitiveDependencies/>
                        <!--
                            确定此依赖项集中的包含/排除模式是否将应用于给定工件的传递路径。
                            如果为真，并且当前工件是由包含或排除模式匹配的另一个工件引入的传递依赖性，则当前工件具有与其相同的包含/排除逻辑。
                            默认情况下，此值为false，以保持与2.1版的向后兼容性。这意味着包含/排除仅仅直接应用于当前的工件，而不应用于传入的工件。（从2.2-beta-1）
                            默认值为：false。
                        -->
                        <!--boolean-->
                        <useTransitiveFiltering/>
                    </dependencySet>
                </dependencySets>
                <!--
                    如果设置为true，则此属性将所有模块包解包到指定的输出目录中。当设置为false时，模块包将作为归档（jar）包含在内。
                    默认值是：true。
                -->
                <!--boolean-->
                <unpack/>
                <!--
                    允许指定包含和排除以及过滤选项，以指定从相关性工件解压缩的项目。（从2.2-beta-1开始）
                -->
                <unpackOptions>
                    <!--
                        （许多） 文件和/或目录模式的集合，用于匹配将在解压缩时从归档文件中包含的项目。每个项目被指定为<include> some / path </ include>（从2.2-beta-1开始）
                    -->
                    <!--List<String>-->
                    <includes/>
                    <!--
                        （许多） 用于匹配项目的文件和/或目录模式的集合，在解压缩时将其从归档文件中排除。每个项目被指定为<exclude> some / path </ exclude>（从2.2-beta-1开始）
                    -->
                    <!--List<String>-->
                    <excludes/>
                    <!--
                        是否使用构建配置中的属性过滤从档案中解压缩的文件中的符号。（从2.2-beta-1开始）
                        默认值是：false。
                    -->
                    <!--boolean-->
                    <filtered/>
                    <!--
                        设置文件的行尾。（从2.2开始）有效值：
                        “keep” - 保留所有的行结束
                        “unix” - 使用Unix风格的行结尾
                        “lf” - 使用单个换行符结束符
                        “dos” - 使用DOS风格的行尾
                        “ crlf ” - 使用Carraige返回，换行符结束
                    -->
                    <!--string-->
                    <lineEnding/>
                    <!--
                        在计算受该集合影响的文件时，是否应该使用标准排除模式，例如那些匹配CVS和Subversion元数据文件的排除模式。为了向后兼容，默认值是true。（从2.2开始）
                        默认值是：true。
                    -->
                    <!--boolean-->
                    <useDefaultExcludes/>
                    <!--
                        允许指定解压档案时使用的编码，支持指定编码的unarchiver。如果未指定，将使用归档程序默认值。Archiver默认值通常代表理智（modern）的values。
                    -->
                    <!--string-->
                    <encoding/>
                </unpackOptions>
                <!--
                    设置此程序集中包含的所有非UNPACKED依赖关系的映射模式。（由于2.2-beta-2; 2.2-beta-1使用$ {artifactId} - $ {version} $ {dashClassifier？}。$ {extension}作为默认值）注意：如果dependencySet指定unpack == true，则outputFileNameMapping将不要使用; 在这些情况下，使用outputDirectory。有关可用于outputFileNameMapping参数的条目的更多详细信息，请参阅插件FAQ。
                    默认值是：$ {module.artifactId} - $ {module.version} $ {dashClassifier？}。$ {module.extension}。
                -->
                <!--string-->
                <outputFileNameMapping/>
            </binaries>
        </moduleSet>
    </moduleSets>
    <!--
        （许多） 指定在程序集中包含哪些文件组。fileSet通过提供一个或多个<fileSet>子元素来指定。
    -->
    <!--List<FileSet>-->
    <fileSets>
        <!--
            fileSet允许将文件组包含到程序集中。
        -->
        <fileSet>
            <!--
                在计算受该集合影响的文件时，是否应该使用标准排除模式，例如那些匹配CVS和Subversion元数据文件的排除模式。为了向后兼容，默认值是true。（从2.2-beta-1开始）
                默认值是：true。
            -->
            <!--boolean-->
            <useDefaultExcludes/>
            <!--
                设置输出目录相对于程序集根目录的根目录。例如，“日志”将把指定的文件放在日志目录中。
            -->
            <!--string-->
            <outputDirectory/>
            <!--
                （许多） 当<include>子元素存在时，它们定义一组要包含的文件和目录。如果不存在，则<includes>表示所有有效值。
            -->
            <!--List<String>-->
            <includes/>
            <!--
                （许多） 当存在<exclude>子元素时，它们定义一组要排除的文件和目录。如果不存在，则<excludes>不表示排除。
            -->
            <!--List<String>-->
            <excludes/>
            <!--
                与UNIX权限类似，设置所包含文件的文件模式。这是一个 OCTAL VALUE。格式：（用户）（组）（其他）其中每个组件是Read = 4，Write = 2和Execute = 1的总和。
                例如，值0644转换为用户读写，组和其他只读。默认值是0644.
            -->
            <!--string-->
            <fileMode/>
            <!--
                与UNIX权限类似，设置包含的目录的目录模式。这是一个 OCTAL VALUE。格式：（用户）（组）（其他）其中每个组件是Read = 4，Write = 2和Execute = 1的总和。
                例如，值0755转换为用户读写，Group和其他只读。默认值是0755.
            -->
            <!--string-->
            <directoryMode/>
            <!--
                设置模块目录的绝对或相对位置。例如，“src / main / bin”会选择定义这个依赖关系的项目的这个子目录。
            -->
            <!--string-->
            <directory/>
            <!--
                设置此文件集中文件的行结束符。有效值：
                “keep” - 保留所有的行结束
                “unix” - 使用Unix风格的行尾（即“\ n”）
                “lf” - 使用一个换行符结束符（即“\ n”）
                “dos” - 使用DOS / Windows风格的行尾（即“\ r \ n”）
                “windows” - 使用DOS / Windows风格的行尾（即“\ r \ n”）
                “crlf” - 使用回车，换行符结尾（即“\ r \ n”）
            -->
            <!--string-->
            <lineEnding/>
            <!--
                是否在复制文件时过滤符号，使用构建配置中的属性。（从2.2-beta-1开始）
                默认值是：false。
            -->
            <!--boolean-->
            <filtered/>
        </fileSet>
    </fileSets>
    <!--
        （许多） 指定在程序集中包含哪些单个文件。通过提供一个或多个<file>子元素来指定文件。
    -->
    <!--List<FileItem>-->
    <files>
        <!--
            一个文件允许单个文件包含选项来更改不受fileSets支持的目标文件名。
        -->
        <file>
            <!--
                设置要包含在程序集中的文件的模块目录的绝对路径或相对路径。
            -->
            <!--string-->
            <source/>
            <!--
                设置输出目录相对于程序集根目录的根目录。例如，“日志”将把指定的文件放在日志目录中。
            -->
            <!--string-->
            <outputDirectory/>
            <!--
                在outputDirectory中设置目标文件名。默认是与源文件相同的名称。
            -->
            <!--string-->
            <destName/>
            <!--
                与UNIX权限类似，设置所包含文件的文件模式。这是一个八卦价值。格式：（用户）（组）（其他）其中每个组件是Read = 4，Write = 2和Execute = 1的总和。
                例如，值0644转换为用户读写，组和其他只读。默认值是0644
            -->
            <!--string-->
            <fileMode/>
            <!--
                设置此文件中文件的行结束符。有效值是：
                “keep” - 保留所有的行结束
                “unix” - 使用Unix风格的行尾（即“\ n”）
                “lf” - 使用一个换行符结束符（即“\ n”）
                “dos” - 使用DOS / Windows风格的行尾（即“\ r \ n”）
                “windows” - 使用DOS / Windows风格的行尾（即“\ r \ n”）
                “crlf” - 使用回车，换行符结尾（即“\ r \ n”）
            -->
            <!--string-->
            <lineEnding/>
            <!--
                设置是否确定文件是否被过滤。
                默认值是：false。
            -->
            <!--boolean-->
            <filtered/>
        </file>
    </files>
    <!--List<DependencySet>-->
    <dependencySets>
        <!--
            依赖关系集允许在程序集中包含和排除项目依赖关系。
        -->
        <dependencySet>
            <!--
                    设置输出目录相对于程序集根目录的根目录。例如，“log”会将指定的文件放在归档根目录下的日志目录中。
                -->
            <!--string-->
            <outputDirectory/>
            <!--
                （许多） 当存在<include>子元素时，它们定义一组要包含的工件坐标。如果不存在，则<includes>表示所有有效值。
                工件坐标可以以简单的groupId：artifactId形式给出，或者可以以groupId：artifactId：type [：classifier]：version的形式完全限定。
                另外，可以使用通配符，如*：maven- *
            -->
            <!--List<String>-->
            <includes/>
            <!--
                （许多） 当存在<exclude>子元素时，它们定义一组依赖项工件坐标以排除。如果不存在，则<excludes>不表示排除。
                工件坐标可以以简单的groupId：artifactId形式给出，或者可以以groupId：artifactId：type [：classifier]：version的形式完全限定。
                另外，可以使用通配符，如*：maven- *
            -->
            <!--List<String>-->
            <excludes/>
            <!--
                与UNIX权限类似，设置所包含文件的文件模式。这是一个 OCTAL VALUE。格式：（用户）（组）（其他）其中每个组件是Read = 4，Write = 2和Execute = 1的总和。
                例如，值0644转换为用户读写，组和其他只读。默认值是0644
            -->
            <!--string-->
            <fileMode/>
            <!--
                与UNIX权限类似，设置包含的目录的目录模式。这是一个 OCTAL VALUE。格式：（用户）（组）（其他）[Format: (User)(Group)(Other) ] 其中每个组件是Read = 4，Write = 2和Execute = 1的总和。
                例如，值0755转换为用户读写，Group和其他只读。默认值是0755.
            -->
            <!--string-->
            <directoryMode/>
            <!--
                如果指定为true，那么在程序集创建过程中任何用于过滤实际构件的包含/排除模式都将导致构建失败，并显示错误。这是为了强调过时的包含或排除，或者表示程序集描述符配置不正确。（从2.2开始）
                默认值是：false。
            -->
            <!--boolean-->
            <useStrictFiltering/>
            <!--
                为此程序集中包含的所有依赖项设置映射模式。（从2.2-beta-2开始； 2.2-beta-1使用$ {artifactId} - $ {version} $ {dashClassifier？}。$ {extension}作为默认值）。
                默认值是：$ {artifact.artifactId} - $ {artifact.version} $ {dashClassifier？}。$ {artifact.extension}。
            -->
            <!--string-->
            <outputFileNameMapping/>
            <!--
                如果设置为true，则此属性将所有依赖项解包到指定的输出目录中。设置为false时，依赖关系将被包含为档案（jar）。只能解压jar，zip，tar.gz和tar.bz压缩文件。
                默认值是：false。
            -->
            <!--boolean-->
            <unpack/>
            <!--
                允许指定包含和排除以及过滤选项，以指定从相关性工件解压缩的项目。（从2.2-beta-1开始）
            -->
            <unpackOptions>
                <!--
                    （许多） 文件和/或目录模式的集合，用于匹配将在解压缩时从归档文件中包含的项目。每个项目被指定为<include> some / path </ include>（从2.2-beta-1开始）
                -->
                <!--List<String>-->
                <includes/>
                <!--
                    （许多） 用于匹配项目的文件和/或目录模式的集合，在解压缩时将其从归档文件中排除。每个项目被指定为<exclude> some / path </ exclude>（从2.2-beta-1开始）
                -->
                <!--List<String>-->
                <excludes/>
                <!--
                    是否使用构建配置中的属性过滤从档案中解压缩的文件中的符号。（从2.2-beta-1开始）
                    默认值是：false。
                -->
                <!--boolean-->
                <filtered/>
                <!--
                    设置文件的行尾。（从2.2开始）有效值：
                    “keep” - 保留所有的行结束
                    “unix” - 使用Unix风格的行结尾
                    “lf” - 使用单个换行符结束符
                    “dos” - 使用DOS风格的行尾
                    “crlf ” - 使用Carraige返回，换行符结束
                -->
                <!--string-->
                <lineEnding/>
                <!--
                    在计算受该集合影响的文件时，是否应该使用标准排除模式，例如那些匹配CVS和Subversion元数据文件的排除模式。为了向后兼容，默认值是true。（从2.2开始）
                    默认值是：true。
                -->
                <!--boolean-->
                <useDefaultExcludes/>
                <!--
                    允许指定解压档案时使用的编码，支持指定编码的unarchiver。如果未指定，将使用归档程序默认值。Archiver默认值通常代表理智（modern）的values。
                -->
                <!--string-->
                <encoding/>
            </unpackOptions>
            <!--
                为此dependencySet设置依赖项范围。
                默认值是：runtime。
            -->
            <!--string-->
            <scope/>
            <!--
                确定当前项目构建过程中产生的工件是否应该包含在这个依赖集中。（从2.2-beta-1开始）
                默认值是：true。
            -->
            <!--boolean-->
            <useProjectArtifact/>
            <!--
                确定当前项目构建过程中产生的附件是否应该包含在这个依赖集中。（从2.2-beta-1开始）
                默认值是：false。
            -->
            <!--boolean-->
            <useProjectAttachments/>
            <!--
                确定是否将传递依赖项包含在当前依赖项集的处理中。如果为true，那么include / excludes / useTransitiveFiltering将应用于传递依赖项构件以及主项目依赖项构件。
                如果为false，则useTransitiveFiltering无意义，并且包含/排除仅影响项目的直接依赖关系。
                默认情况下，这个值是真的。（从2.2-beta-1开始）
                默认值是：true。
            -->
            <!--boolean-->
            <useTransitiveDependencies/>
            <!--
                确定此依赖项集中的包含/排除模式是否将应用于给定工件的传递路径。
                如果为真，并且当前工件是由包含或排除模式匹配的另一个工件引入的传递依赖性，则当前工件具有与其相同的包含/排除逻辑。
                默认情况下，此值为false，以保持与2.1版的向后兼容性。这意味着包含/排除仅仅直接应用于当前的工件，而不应用于传入的工件。（从2.2-beta-1）
                默认值为：false。
            -->
            <!--boolean-->
            <useTransitiveFiltering/>
        </dependencySet>
    </dependencySets>
    <!--
        定义要包含在程序集中的Maven仓库。可用于存储库中的工件是项目的依赖工件。创建的存储库包含所需的元数据条目，并且还包含sha1和md5校验和。这对创建将被部署到内部存储库的档案很有用。
        注意：目前，只有来自中央存储库的工件才被允许。
    -->
    <!--List<Repository>-->
    <repositories>
        <repository>
            <!--
                设置输出目录相对于程序集根目录的根目录。例如，“log”会将指定的文件放在归档根目录下的日志目录中。
            -->
            <!--string-->
            <outputDirectory/>
            <!--
                （许多） 当存在<include>子元素时，它们定义一组包含的项目坐标。如果不存在，则<includes>表示所有有效值。
                工件坐标可以以简单的groupId：artifactId形式给出，或者可以以groupId：artifactId：type [：classifier]：version的形式完全限定。
                另外，可以使用通配符，如*：maven- *
            -->
            <!--List<String>-->
            <includes/>
            <!--
                （许多） 当存在<exclude>子元素时，它们定义一组要排除的项目工件坐标。如果不存在，则<excludes>不表示排除。
                工件坐标可以以简单的groupId：artifactId形式给出，或者可以以groupId：artifactId：type [：classifier]：version的形式完全限定。
                另外，可以使用通配符，如*：maven- *
            -->
            <!--List<String>-->
            <excludes/>
            <!--
                    与UNIX权限类似，设置所包含文件的文件模式。这是一个 OCTAL VALUE。格式：（用户）（组）（其他）其中每个组件是Read = 4，Write = 2和Execute = 1的总和。
                    例如，值0644转换为用户读写，组和其他只读。默认值是0644
                -->
            <!--string-->
            <fileMode/>
            <!--
                与UNIX权限类似，设置包含的目录的目录模式。这是一个 OCTAL VALUE。格式：（用户）（组）（其他）[Format: (User)(Group)(Other) ] 其中每个组件是Read = 4，Write = 2和Execute = 1的总和。
                例如，值0755转换为用户读写，Group和其他只读。默认值是0755.
            -->
            <!--string-->
            <directoryMode/>
            <!--
                如果设置为true，则此属性将触发创建存储库元数据，这将允许存储库用作功能性远程存储库。
                默认值是：false。
            -->
            <!--boolean-->
            <includeMetadata/>
            <!--
                （许多） 指定要将一组工件与指定的版本对齐。groupVersionAlignment通过提供一个或多个<groupVersionAlignment>子元素来指定。
                允许一组工件与指定的版本对齐。
            -->
            <!--List<GroupVersionAlignment>-->
            <groupVersionAlignments>
                <groupVersionAlignment>
                    <!--
                        要为其对齐版本的工件的groupId。
                    -->
                    <!--string-->
                    <id/>
                    <!--
                        您想要将该组对齐的版本。
                    -->
                    <!--string-->
                    <version/>
                    <!--
                        （许多） 当存在<exclude>子元素时，它们定义要排除的构件的artifactIds。如果不存在，则<excludes>不表示排除。排除是通过提供一个或多个<exclude>子元素来指定的。
                    -->
                    <!--List<String>-->
                    <excludes/>
                </groupVersionAlignment>
            </groupVersionAlignments>
            <!--
                指定此存储库中包含的工件的范围。（从2.2-beta-1开始）
                默认值是：runtime。
            -->
            <!--string-->
            <scope/>
        </repository>
    </repositories>
    <!--
        （许多） 指定要包含在程序集中的共享组件xml文件位置。指定的位置必须相对于描述符的基本位置。
        如果描述符是通过类路径中的<descriptorRef />元素找到的，那么它指定的任何组件也将在类路径中找到。
        如果通过路径名通过<descriptor />元素找到，则此处的值将被解释为相对于项目basedir的路径。
        当找到多个componentDescriptors时，它们的内容被合并。检查 描述符组件 了解更多信息。
        componentDescriptor通过提供一个或多个<componentDescriptor>子元素来指定。
    -->
    <!--List<String>-->
    <componentDescriptors/>
</assembly>


```


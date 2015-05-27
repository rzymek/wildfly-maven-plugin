## Problem
Executing `module add` command with `mvn wildfly:run` causes an error:

    The server failed to start:
    Command execution failed for command 'module add --name=testing --resources=empty.jar'.
    JBOSS_HOME environment variable is not set.

Error call stack:

    org.apache.maven.lifecycle.LifecycleExecutionException: Failed to execute goal org.wildfly.plugins:wildfly-maven-plugin:1.1.0.Alpha3-SNAPSHOT:run (default-cli) on project testing: The server failed to start
        at org.apache.maven.lifecycle.internal.MojoExecutor.execute(MojoExecutor.java:216)
        at org.apache.maven.lifecycle.internal.MojoExecutor.execute(MojoExecutor.java:153)
        at org.apache.maven.lifecycle.internal.MojoExecutor.execute(MojoExecutor.java:145)
        at org.apache.maven.lifecycle.internal.LifecycleModuleBuilder.buildProject(LifecycleModuleBuilder.java:116)
        at org.apache.maven.lifecycle.internal.LifecycleModuleBuilder.buildProject(LifecycleModuleBuilder.java:80)
        at org.apache.maven.lifecycle.internal.builder.singlethreaded.SingleThreadedBuilder.build(SingleThreadedBuilder.java:51)
        at org.apache.maven.lifecycle.internal.LifecycleStarter.execute(LifecycleStarter.java:128)
        at org.apache.maven.DefaultMaven.doExecute(DefaultMaven.java:307)
        at org.apache.maven.DefaultMaven.doExecute(DefaultMaven.java:193)
        at org.apache.maven.DefaultMaven.execute(DefaultMaven.java:106)
        at org.apache.maven.cli.MavenCli.execute(MavenCli.java:862)
        at org.apache.maven.cli.MavenCli.doMain(MavenCli.java:286)
        at org.apache.maven.cli.MavenCli.main(MavenCli.java:197)
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:483)
        at org.codehaus.plexus.classworlds.launcher.Launcher.launchEnhanced(Launcher.java:289)
        at org.codehaus.plexus.classworlds.launcher.Launcher.launch(Launcher.java:229)
        at org.codehaus.plexus.classworlds.launcher.Launcher.mainWithExitCode(Launcher.java:415)
        at org.codehaus.plexus.classworlds.launcher.Launcher.main(Launcher.java:356)
    Caused by: org.apache.maven.plugin.MojoExecutionException: The server failed to start
        at org.wildfly.plugin.server.RunMojo.doExecute(RunMojo.java:233)
        at org.wildfly.plugin.deployment.AbstractDeployment.execute(AbstractDeployment.java:111)
        at org.apache.maven.plugin.DefaultBuildPluginManager.executeMojo(DefaultBuildPluginManager.java:134)
        at org.apache.maven.lifecycle.internal.MojoExecutor.execute(MojoExecutor.java:208)
        ... 20 more
    Caused by: java.lang.IllegalArgumentException: Command execution failed for command 'module add --name=testing --resources=empty.jar'. JBOSS_HOME environment variable is not set.
        at org.wildfly.plugin.cli.Commands.executeCommands(Commands.java:176)
        at org.wildfly.plugin.cli.Commands.execute(Commands.java:134)
        at org.wildfly.plugin.deployment.AbstractDeployment.executeDeployment(AbstractDeployment.java:118)
        at org.wildfly.plugin.server.RunMojo.doExecute(RunMojo.java:218)
        ... 23 more
    Caused by: org.jboss.as.cli.CommandLineException: JBOSS_HOME environment variable is not set.
        at org.jboss.as.cli.handlers.module.ASModuleHandler.getModulesDir(ASModuleHandler.java:420)
        at org.jboss.as.cli.handlers.module.ASModuleHandler.addModule(ASModuleHandler.java:281)
        at org.jboss.as.cli.handlers.module.ASModuleHandler.doHandle(ASModuleHandler.java:250)
        at org.jboss.as.cli.handlers.CommandHandlerWithHelp.handle(CommandHandlerWithHelp.java:88)
        at org.jboss.as.cli.impl.CommandContextImpl.handle(CommandContextImpl.java:674)
        at org.wildfly.plugin.cli.Commands.executeCommands(Commands.java:172)
        ... 26 more

## Reproduce

Minimal project to reproduce: https://github.com/rzymek/wildfly-maven-plugin/tree/add-module-reproduce

    git clone https://github.com/rzymek/wildfly-maven-plugin.git -b add-module-reproduce add-module-reproduce
    cd add-module-reproduce
    mvn wildfly:run

It generally boils down to calling `mvn wildfly:run` with:

    <plugin>
        <groupId>org.wildfly.plugins</groupId>
        <artifactId>wildfly-maven-plugin</artifactId>
        <version>1.1.0.Alpha3-SNAPSHOT</version>
        <configuration>
            <before-deployment>
                <commands>
                    <command>
                        module add --name=testing --resources=empty.jar
                    </command>
                </commands>
            </before-deployment>
        </configuration>
    </plugin>

## Fix

As `wildfly:run` unpacks the server by itself, it knows what value should `JBOSS_HOME` have (and not the caller). That's why it can pass the value to `ASModuleHandler` via `jboss.home.dir` Java system property.
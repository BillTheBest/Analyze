Configuring the "audit logging to file" example project
=======================================================

In i2 Analyze, you can use audit logging to record information about user activity in the Information Store. To do so, you must write and configure an implementation of the `IAuditLogger` interface. In Developer Essentials, IBM provides the `awc-audit-file-example` example, which writes audit logging information to a log file.

Before you begin
----------------

The audit logging to file example project requires the development version of i2 Analyze, prepared according to the Developer Essentials deployment instructions. However, the example has no additional requirements.

About this task
---------------

When you build the audit logging to file example, you create a JAR file that contains an implementation of the `IAuditLogger` interface. To enable audit functionality, you must redeploy i2 Analyze to include the new JAR file, and configure the platform to use the class that the JAR file contains.

Procedure
---------

The examples in Developer Essentials require links to the i2 Analyze libraries, and to the API documentation. These links are configured in a separate project directory that you must import into Eclipse before you can build any of the example projects.

1.  If you did not already add the `master` directory and the related artifacts to your Eclipse workspace while you worked with one of the other examples, add it now.
    1.  In Eclipse, click **Window** &gt; **Preferences** to open the Preferences window.
    2.  In the Preferences window, select **General** &gt; **Workspace** &gt; **Linked Resources**.
    3.  Add an entry to the **Defined path variables** list:

        -   Name: `TOOLKIT_ROOT`
        -   Location: `C:\IBM\i2analyze\SDK\sdk-projects\master\build\toolkit`

        The shared libraries and several of the example projects rely on the presence of `TOOLKIT_ROOT` in your development environment.

    4.  In Eclipse, click **File** &gt; **Import** to open the Import window.
    5.  In the Import window, click **General** &gt; **Existing Projects into Workspace**, and then click **Next**.
    6.  Click **Browse** at the top of the window, and then select the `C:\IBM\i2analyze\SDK\sdk-projects\master` directory.
    7.  Click **Finish** to complete the import process.

Developing and testing implementations of `IAuditLogger` requires a development version of the `awc` server. Developer Essentials includes a project that you can import into to Eclipse and configure to be that server.

1.  If you did not already add the `awc` directory to your Eclipse workspace while you worked with one of the other audit logging examples, add it now.
    1.  In your Eclipse workspace, stop any running servers.
    2.  In Windows Explorer, navigate to the `C:\IBM\i2analyze\deploy-dev\wlp\usr\servers\awc\apps` directory. Delete the `awc.war` subdirectory, which contains the `awc` application.
    3.  Back in Eclipse, refresh your view of the Enterprise Explorer tab.
    4.  Repeat the instructions that you followed when you added the `master` directory to your Eclipse workspace. This time, add the `C:\IBM\i2analyze\SDK\sdk-projects\awc` directory to Eclipse.
    5.  In the **Servers** tab at the bottom of the Eclipse application window, right-click the **awc** server and select **Add and Remove**.
    6.  In the Add and Remove window, move **awc** from the **Available** list to the **Configured** list, and then click **Finish** to close the window.
    7.  Still in the **Servers** tab, right-click the **awc** server and select **Publish** to ensure that the server and its representation in Eclipse are synchronized.
    8.  Verify that you can still start both servers without seeing any new errors in the **Console** tab in Eclipse.
        Note: If you restart the **awc** server while a client is connected to it, you might see an error in the **Console** tab that reports an unavailable server. If the server then initializes normally, you can safely disregard this report.
        If you see error messages about the Solr indexing service, run the following command at your command prompt to ensure that the service is running:

        ``` pre
        build -t startSolr -s awc
        ```

The audit logging examples in Developer Essentials all require the contents of the `awc-audit-example-common` project.

1.  If you have not already done so, repeat the instructions that you followed when you added the `master` directory to your Eclipse workspace. This time, add the `C:\IBM\i2analyze\SDK\sdk-projects\awc-audit-example-common` directory to Eclipse.

Additionally, the audit logging to file example requires the `awc-audit-file-example` project, which contains the source code that is specific to this example.

1.  Repeat the instructions again to import `C:\IBM\i2analyze\SDK\sdk-projects\awc-audit-file-example` into Eclipse.
2.  In the `awc` project directory, navigate to `fragment\WEB-INF\classes\DiscoServerSettingsMandatory.properties`. Uncomment the class for the `IAuditLogger` implementation in this example project, and comment out the other settings.

    ``` pre
    #AuditLogger=com.example.audit.CSVAuditLogger
    #AuditLogger=com.example.audit.DatabaseAuditLogger
    AuditLogger=com.example.audit.FileAuditLogger
    #AuditLogger=com.i2group.disco.audit.NoOpAuditLogger
    ```

3.  Add the JAR files that the common and specific projects generate to the deployment assembly for the `awc` project.
    1.  In Enterprise Explorer, right-click **awc**, and select **Properties** to display the Properties window.
    2.  Click **Deployment Assembly**, and then **Add**. In the New Assembly Directive window, select **Project**, and then click **Next**.
    3.  Select both **awc-audit-example-common** and **awc-audit-file-example**, and then click **Finish**.

4.  In the **Servers** tab, right-click the **awc** server and select **Publish** to ensure that the server and its representation in Eclipse are synchronized.
5.  Still in the **Servers** tab, right-click the **awc** server, and then click **Start** (or **Restart**).

    This action starts the server with audit logging to file enabled. The logging behavior is governed by the settings file at `awc-audit-file-example\fragment\WEB-INF\classes\FileAudit.properties`, which controls where the log files are saved, and how large they can be. Typical messages look like this one:

    ``` pre
    2016-10-03T11:21:27.232Z. User: Analyst, IP: 0:0:0:0:0:0:0:1,
    UserAgent: Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko)
    Chrome/53.0.2785.116 Safari/537.36, Groups: [Analyst, Security Controller, Controlled],
    Operation: QuickSearch, DataStores: [InfoStore], Expression: green, Filters: []
    ```

    The supplied version of `FileAudit.properties` specifies the `wlp\usr\servers\awc\logs\awc` directory as the target for audit log files. The files are named `auditn.log`, where n is an integer that increments with each new file.

After you develop and test an audit logging solution for i2 Analyze, the next step is to publish it to a live deployment. These instructions assume that you have access to two instances of i2 Analyze:

-   The development version of the platform, deployed according to the instructions in IBM i2 Analyze Developer Essentials. This instance contains the custom application that you want to publish.
-   The production version of the platform, deployed according to the instructions in the IBM i2 Analyze deployment documentation. This instance has all of its default settings.

To prepare your development version of i2 Analyze for production, you must edit the `topology.xml` file, and then use the deployment toolkit to redeploy the platform.

1.  If the development version of the `awc` server is running, stop the server from Eclipse.
2.  In Windows Explorer, navigate to the `C:\IBM\i2analyze\deploy-dev\wlp\usr\servers\awc\apps` directory. Delete the `awc.war` subdirectory, which contains the `awc` application.
3.  Open the topology file that represents your deployment of i2 Analyze in an XML editor. By default, this file is at `C:\IBM\i2analyze\SDK\sdk-projects\master\build\toolkit\configuration\environment\topology.xml`.
4.  Edit the `<war>` element that defines the contents of the `awc` web application to include the common and specific fragments for the audit logging to file example.

    ``` pre
    <war context-root="awc" i2-data-source-id="infostore" name="awc" target="awc">
       <data-sources>
          <data-source database-id="infostore" create-database="true"/>
       </data-sources>
       <file-store-ids>
          <file-store-id value="awc-job-store"/>
          <file-store-id value="awc-recordgroup-store"/>
       </file-store-ids>
       <fragments>
          <fragment name="awc"/>
          <fragment name="common"/>
          <fragment name="default-user-profile-provider"/>
          <fragment name="awc-audit-example-common"/>
          <fragment name="awc-audit-file-example"/>
       </fragments>
       <solr-collection-ids>
          <solr-collection-id collection-id="main_index"
                              data-dir="C:/IBM/i2analyze/data-dev/solr"
                              cluster-id="is_cluster"/>
       </solr-collection-ids>
    </war>
    ```

5.  Open a command prompt as Administrator, and navigate to the `C:\IBM\i2analyze\SDK\sdk-projects\master` directory.
6.  Run the following command to redeploy the `awc` application with all the changes that you made:

    ``` pre
    build -t deploy -s awc
    ```

Before you publish the audit logging functionality to a live deployment of i2 Analyze, test that your changes to the topology file are behaving correctly.

1.  Back in the Servers tab in your Eclipse environment, ensure that the `awc` application is marked as **Synchronized**. Right click and **Publish** if it is not.
2.  Start the **awc** server, connect to the application through a client, and verify that the audit logging behavior is unchanged from the version that you deployed with Eclipse.
3.  Stop the **awc** server again.

At this stage, you are ready to copy the changes that you made to your development deployment to a production deployment of i2 Analyze.

1.  Copy `C:\IBM\i2analyze\SDK\sdk-projects\master\build\toolkit\configuration` from the development version into the production version of the deployment toolkit.
2.  Edit the following files to ensure that the settings match the production environment:
    -   `configuration\environment\awc\environment.properties`
    -   `configuration\environment\i2analyze\environment.properties`
    -   `configuration\environment\http-server.properties`
    -   `configuration\environment\credentials.properties`
    -   `configuration\environment\topology.xml`

3.  On the production version of the platform, open a command prompt as Administrator, and navigate to the `toolkit\scripts` directory.
4.  To redeploy the platform, run the following command:

    ``` pre
    setup -t deploy -s awc
    ```

5.  Use the Services application in Windows (`services.msc`) to restart IBM HTTP Server.

Results
-------

The result of following all of these steps is that your development and production deployments of i2 Analyze contain the same implementation of audit logging to file. When users interact with the Information Store, the platform writes data about those interactions to the log files that you specified.

What to do next
---------------

It is likely that after you use the deployment toolkit to deploy the example, you want to return to developing and deploying it in Eclipse. That process involves just a handful of steps:

1.  In your Eclipse workspace, stop any running servers.
2.  In Windows Explorer, navigate to the `C:\IBM\i2analyze\deploy-dev\wlp\usr\servers\awc\apps` directory. Delete the `awc.war` subdirectory, which contains the `awc` application.
3.  Back in Eclipse, refresh your view of the Enterprise Explorer tab.
4.  In the **Servers** tab, right-click the **awc** server, and click **Clean**.

This operation restores Eclipse to its state when you first added the `awc` project to the **awc** server. You can change the `awc-audit-file-example` project and see the effects on the development version of the platform exactly as before.

**Parent topic:** <a href="developer_essentials_welcome.md" class="link" title="IBM i2 Analyze Developer Essentials contains tools, libraries, and examples that enable development and deployment of custom extensions to i2 Analyze.">IBM i2 Analyze Developer Essentials</a>

------------------------------------------------------------------------

© Copyright IBM Corporation 2014, 2016.



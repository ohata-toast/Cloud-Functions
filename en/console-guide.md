## Compute > Cloud Functions > Console User Guide
This document describes how to create and manage functions in Cloud Functions console.

## Manage functions
You can create, edit, delete, and copy functions.

### Create functions
Configure the function, write your code, and build it. Then, click **Create** to deploy the function using the latest built package.

![console-guide-07](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/2025-11-25/console-guide-en-01.png)
![console-guide-08](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/2025-11-25/console-guide-en-02.png)

#### Function settings
<table class="it">
    <tr>
        <th>Category</th>
        <th>No.</th>
        <th>Item</th>
        <th>Description</th>
    </tr>
    <tr>
        <td rowspan="2">Basic Information</td>
        <td>1.</td>
        <td>Name</td>
        <td>Name of the function<br> Duplicate function names are not allowed.<br> Used as the function endpoint URL.</td>
    </tr>
    <tr>
        <td>2.</td>
        <td>Description</td>
        <td>Description of the function, up to 250 characters</td>
    </tr>
    <tr>
        <td rowspan="7">Function settings</td>
        <td>3.</td>
        <td>Endpoint URL</td>
        <td>Function call HTTP endpoint URL<br>It changes automatically as you enter the function name.</td>
    </tr>
    <tr>
        <td>4.</td>
        <td>Type</td>
        <td>Pool Manager<br> New Deployment</td>
    </tr>
    <tr>
        <td>5.</td>
        <td>Resource</td>
        <td>Select function behavior environment resources<br>If it’s Pool Manager type, only the specified resources can be selected<br>New Deployment type allows you to enter custom resources</td>
    </tr>
    <tr>
        <td>6.</td>
        <td>Execution timeout</td>
        <td>Specify how long the function will run (Time out). If the function runs and exceeds the specified time, the function is forced to abort and responds with an error</td>
    </tr>
    <tr>
        <td>7.</td>
        <td>(Pool Manager) Concurrent execution settings</td>
        <td>Specify the number of concurrent executions of the function. When concurrent function calls occur, instances are executed concurrently as many times as the number of calls, and the maximum number of instances is limited.</td>
    </tr>
    <tr>
        <td>8.</td>
        <td>(New Deployment) Minimum number of instances</td>
        <td>Specify the minimum number of instances to retain.</td>
    </tr>
    <tr>
        <td>9.</td>
        <td>(New Deployment) Maximum number of instances</td>
        <td>Specify the maximum number of instances that can scale up automatically.</td>
    </tr>
    <tr>
        <td>Log settings</td>
        <td>10.</td>
        <td>Integrate with log service</td>
        <td>Choose to integrate using Log &amp; Crash Search service<br>The logs of the function are available directly from Log &amp; Crash Search service.<br>Logs are sent in 10-line units.</td>
    </tr>
</table>

> **[Note]** <br>
> **(Pool Manager) Concurrent execution settings**


![console-guide-05](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/2025-07-29/console-guide-05.png)

<br>

> **[Note]** <br>
> **(New Deployment) Number of instances** - Depending on resource usage, the specified number of instances might not be created.

<br>


#### Code writing

![console-guide-09](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/2025-11-25/console-guide-en-03.png)
![console-guide-10](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/2025-11-25/console-guide-en-04.png)
![console-guide-11](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/2025-11-25/console-guide-en-05.png)

<table class="it">
    <tr>
        <th>Category</th>
        <th>No.</th>
        <th>Item</th>
        <th>Description</th>
    </tr>
    <tr>
        <td rowspan="2">Source code</td>
        <td>1.</td>
        <td>Runtime Environment</td>
        <td>Select the function's runtime environment</td>
    </tr>
    <tr>
        <td>2.</td>
        <td>Entry Point</td>
        <td>Specify the entry point for the function, such as function name <br>It is automatically completed to match the template in the runtime environment.<br>It must match the entry point in the source code that you wrote when you modified randomly.<br>If you entered it incorrectly, <strong>you cannot check it with build</strong>, <strong>but you can check it with with the log during testing</strong>.</td>
    </tr>
    <tr>
        <td rowspan="3">Code</td>
        <td>3.</td>
        <td>Code editor</td>
        <td>The template files in the runtime environment are loaded, and you modify them to write the function.<br>You cannot add directories on the code editor. To edit the directory structure, you must download the template file, edit it locally, and then upload the ZIP file.</td>
    </tr>
    <tr>
        <td>4.</td>
        <td>User’s local environment</td>
        <td>Write function code in the local environment and upload as a ZIP file</td>
    </tr>
    <tr>
        <td>5.</td>
        <td>Build</td>
        <td>Build a package from the code you have written or uploaded.<br>The built packages are used for testing. When creating or updating a function, the latest built package is automatically integrated into the function version</td>
    </tr>
    <tr>
        <td rowspan="2">Test</td>
        <td>6.</td>
        <td>Test events</td>
        <td>Create a JSON Body to send to the function</td>
    </tr>
    <tr>
        <td>7.</td>
        <td>Test</td>
        <td>Test the function by sending the JSON Body you created to the event. (This must be built first.) <br>You can verify function behavior with logs.<br>Call with the GET method.</td>
    </tr>
    <tr>
        <td></td>
        <td>8.</td>
        <td>Creation</td>
        <td>Use the Generate button to generate a function based on the function settings and the function you created.</td>
    </tr>
</table>

<br>

> **[Note]** <br> Packages generated via Build button are linked to the function version during creation or updates. You must perform at least one build before creating a function

### Modify functions
To modify the function settings and code of an existing function, click **Modify** button to modify the function.
#### Non-modifiable item
- Name, runtime environment
    - You can edit everything except the item.
#### Source code
- If you're using the code editor, the existing code is loaded.
- If you created a function by uploading a ZIP file from the local environment, when you switch to the code editor, it loads the default template code without showing the ZIP file.

### Delete a function
![console-guide-14](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/2025-11-25/console-guide-en-06.png)
Select an existing function to delete it. You can delete multiple functions at once.

### Copy a function
![console-guide-13](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/2025-11-25/console-guide-en-07.png)
Copy a function that is identical to an existing function. The name cannot be duplicated, so you can rewrite the name before copying.
- Triggers are not copied. (HTTP triggers are provided by default).
- Only the currently applied version will be copied.

## About functions
### List of functions
![console-guide-01](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/2025-11-25/console-guide-en-08.png)
- You can see a list of functions that users have created.
- Build status shows the build status of the current version.
### Basic information of functions
![console-guide-02](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/2025-11-25/console-guide-en-09.png)
- You can see basic information about the function.
- Click Log & Crash Search in the Log Management section to view your logs in the Log & Crash Search service.

### Function versioning
![console-guide-XX]()
- You can manage function versions.
- You can view version history and rollback to any previous version.

#### Versioning overview
Build your code by clicking **Build** in the function creation or modification screen to generate a package.
- When you create or update a function, the latest built package is integrated into the function version.
- Each version is managed independently, allowing you to apply a tested version directly to your function.
- At least one build is required to enable function creation.

#### Version information
<table class="it">
    <tr>
        <th>No.</th>
        <th>Item</th>
        <th>Description</th>
    </tr>
    <tr>
        <td>1.</td>
        <td>Version List</td>
        <td>You can check the full version history of the function.<br>This list displays information such as the version name, runtime, source code, and build status.</td>
    </tr>
    <tr>
        <td>2.</td>
        <td>Currently applied version</td>
        <td>It diplays the version currently applied to the function.</td>
    </tr>
    <tr>
        <td>3.</td>
        <td>Download source code</td>
        <td>You can download the source code of the selected version as a ZIP file.</td>
    </tr>
    <tr>
        <td>4.</td>
        <td>Check logs</td>
        <td>You can check the selected version's build logs.<br>This is the same as the feature to check build logs in the Basic Information tab.</td>
    </tr>
</table>

#### Deploy versions
- Select a version from the list and click **Deploy Version** to update the function to the version.
- The currently applied version cannot be selected for deployment.
- Only one version can be deployed at a time; multiple selection is not supported.
- Versions with a failed build status can still be deployed.

> **[Note]**
> <br>A confirmation popup will appear upon deployment. Once the deployment is successful, the version list will refresh automatically.

#### Delete versions
- To delete a version, select it from the list and click **Delete Version**.
- The currently applied version cannot be deleted.
- You can select and delete multiple versions at once.
- Deleting a version also removes its associated source code.

> **[Note]**
> <br>A confirmation popup will appear when deleting a version. Please note that deleted versions cannot be recovered.

#### Constraints
- Canceling function creation or modification will prevent the built package from being integrated into a function version.
- Deleting a function will permanently remove all integrated versions.
- When copying a function, only the currently active version is copied. The full version history will not be copied.
- While multiple runtimes are available during function creation, you can only build using the initially selected runtime when modifying the function.

### Manage function triggers
You can manage triggers that can perform functions.
- HTTP triggers are built-in when creating a function.
    - You can set whether to use through the Enable/Disable features.

![console-guid-12](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/2025-11-25/console-guide-en-10.png)

- You can execute the function you created via a given HTTP trigger.
    - Example: `https://{userdomain}/{function name}`
    - Method : GET, POST

#### Create/edit triggers
- Timer
    - Value: Enter the cycle as a Cron string.
    - API Gateway
    - You can add HTTP Endpoint by using the API Gateway service.

#### Delete triggers
- You can select multiple triggers to delete. You can't delete the HTTP trigger, which is the default trigger.

### Monitor functions
![console-guide-06](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/2025-11-25/console-guide-en-11.png)
- You can check the usage of a function.
- Provide metrics for the number of function calls, number of call rejections, number of errors, success rate, and function execution time.
- Provide metrics within a set time.
- If logs are integrated, you can navigate to the Log & Crash Search service using the Log & Crash Search button.

| Item | Description |
| --- | --- |
| Number of function calls | Total number of function calls (per second) |
| Number of call rejections | Number of function calls that failed as instances were not created (per second) |
| Number of errors | Number of responses with a non-200 response code (per second) |
| Success rate | Ratio of successful calls to the total number of function calls |
| Function execution time | Response time for function calls |

> **[Note]**
> <br>The number of call rejections is only relevant if the type is Pool Manager.
> <br>The number of function calls, call rejections, and errors are displayed as an average number of times per second within the range of the metric step. For example, if the step is 15 seconds and 1 event occurred within the time period, it is displayed as 0.0667 (1÷15).
> <br>The average of the function execution time is the overall cumulative average, and the maximum is the maximum execution time during the period.
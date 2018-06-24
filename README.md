# Ready CI
A no-fuss CI/CD service and collection of build scripts

## Why use Ready CI?

### :+1: It comes with scripts
Ready CI comes with build scripts so that you spend less time setting up your automated CI/CD infrastructure, and get to making automated builds faster. Ready CI scripts currently support:
* iOS apps
* Maven projects


### :+1: Command-line or web-service
You can run Ready CI on the command-line within another CI environment like Jenkins, or run Ready CI as a service on it's own and accept web-hook calls from GIT services.

### :+1: Supports GIT web-hooks
Ready CI supports GIT commit web-hooks when you run it as a service so that your automated builds start as soon as you push to your git repository. Ready CI supports web-hooks from:
* GitHub
* BitBucket

### :+1: Parses iOS provisioning profiles
Configuring iOS builds is tricky. Ready CI uses the .mobileprovision file generated by Apple Developer Portal to automatically configure your build and remove some of the guess-work behind making your iOS app build successful.

### :+1: Handles whitespace
Paths, filenames and target names look great when you use whitespace. However, whitespace can be a nightmare to manage in CI scripts so Ready CI handles whitespace like a pro* so that your builds keep working while your project files are clean and readable.

\* The `pipeline` command-line parameter doesn't handle whitespace well. A work in process :-)

## How to use Ready CI
### Building Ready CI
Use Maven to create a jar: target/readyci-0.1.jar
```bash
$ mvn install
```

### Configure pipelines
Make your own copy of the `readyConfigExample.yml` file, specifying all of your pipelines and associated build tasks.

### Running a command-line build
Run a once off command-line build by specifying a `yml` configuration file and the `pipeline=` parameter. It's only fitting that Ready CI be able to build it's self! Try this out by using the example configuration `readyConfigExample.yml` to run a Ready CI build named `ready-ci`. 
```bash
$ java -jar target/readyci-0.1.jar readyConfigExample.yml pipeline=ready-ci

Loaded configuration readyConfigExample.yml with 2 pipelines
...
Ready CI is in command-line mode
Building pipline ready-ci
...
FINISHED BUILD 74e404d8-6bae-41fa-8aa1-4d786c797c58 
```

A successful build will deploy `readyci.jar` to your `/tmp/` directory. You can check that it's there like this:
```bash
$ ls -la /tmp/readyci.jar 
-rw-r--r--  1 bradley  wheel  16612035 Jun 13 12:30 /tmp/readyci.jar
```

####Examples
Point ReadyCI at a repository which has a readyci.yml configuration file in the root of the repository. ReadyCI will fetch the repository, load the configuration in `readyci.yml`, and execute the `ready-ci` pipeline. 
```bash
$ java -jar target/readyci-0.1.jar pipeline=ready-ci gitPath=git@github.com:dotb/ready_ci.git
```

Load a local configuration file and build the `ready-ci` pipeline. The local configuration file needs to specify the gitPath to be used to fetch the repository. The repository can also contain a `readyci.yml` configuration file which will be loaded before the build commences.
```
$ java -jar target/readyci-0.1.jar pipeline=ready-ci readyConfigExample.yml 
```

### Running a build service
Ready CI can run as a web-service and listen out for web-hook calls. Configure your GIT repository to post to http://<your address>:8080/webhook and then run Ready CI with a `yml` configuration time and the `server` parameter.
```bash
java -jar target/readyci-0.1.jar readyConfigExample.yml server

Loaded configuration readyConfigExample.yml with 2 pipelines
...
Ready CI is in server mode
Started ReadyCI in 3.655 seconds (JVM running for 4.612)
```

You can test your web-hook using cURL
```bash
curl -X POST \
  http://localhost:8080/webhook \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -d '{
   "push":{
      "changes":[
         {
            "new":{
               "type":"branch",
               "name":"master",
               "target":{
                  "author":{
                     "raw":"Bradley Clayton <me@cheese.toast.com>"
                  }
               }
            }
         }
      ]
   },
   "repository":{
      "name":"ready_ci"
   }
}'
```

This will kick off a build of Ready CI and you'll see output like this:
```bash
Webhook proceeding with build for pipline ready-ci
RUNNING BUILD c7a56eec-f303-4ad9-8de9-ffe5da68bef5 
STARTING TASK build_path_clean | Finish up by cleaning the build folder
...
STARTING TASK checkout_git |  
Cloning into '/tmp/readyci//c7a56eec-f303-4ad9-8de9-ffe5da68bef5'...
COMPLETED TASK checkout_git
STARTING TASK maven_install | Run maven install
Executing command: mvn install 
...
FINISHED BUILD c7a56eec-f303-4ad9-8de9-ffe5da68bef5 
```
Ready CI currently supports web-hook calls from GitHub and Bitbucket.

## Configuration explained
Ready CI is configured by supplying a simple YML file on the command line, and an optional configuration file in the root of your repository named readyci.yml. For example, the configuration below builds Ready CI using Maven and the code on GitHub.
```yml
  pipelines:
  - name: ready-ci # every pipeline needs a name 
    gitPath: git@github.com:dotb/ready_ci.git
    gitBranch: master
    parameters:
      deploySrcPath: target/readyci-0.1.jar
      deployDstPath: /tmp/readyci.jar

    tasks:
    - description: Run maven install
      type: maven_install

    - description: Copy the built binary to a deployment destination
      type: deploy_copy

    - description: Finish up by cleaning the build folder
      type: build_path_clean
```
Lets take a look at some of these parameters 

| Parameter | Description |
| :-------- | :---------- |
| instanceName      | A name for your instance. This name is used to populate git commit messages so that you can identify automated commits. Ready CI also uses the instanceName to avoid cyclic builds triggered by the web-hook receiving one of it's own commit notifications. 
| pipelines         | An array of as many pipeline configurations as you want |
| - name            | Each pipeline is named, and you use this name to start a command-line build |
|   gitPath         | The path to your code repository |
|   gitBranch       | Use the gitBranch parameter to specify which branch should trigger builds when web-hook requests are received | 
|   parameters      | Parameters are used to customise the build tasks |
|     deploySrcPath | In this example the deploy_copy task needs to know the source and destination paths for the `readyci.jar` file, so that it can copy it to the right place |
|   tasks:          | The array of tasks is used to configure each build step |
|   - description   | You can set any description you like. It'll be displayed in the build logs |
|     type          | The type of task is important, it tells Ready CI which task should be run |

## Task types
Ready CI includes a collection of task types that currently supports Maven and iOS builds.

| Task                             | Description |
| :---                             | :--- |
| *Maven*                          | |
| maven_install                    | Run maven install |
| *iOS*                            | |
| ios_carthage_update              | Install dependencies using Carthage |
| ios_pod_install                  | Install dependencies using CocoaPods |
| ios_install_provisioning_profile | Install a .mobileprovisioning file onto the build host |
| ios_provisioning_profile_read    | Read build information from a .mobileprovisioning file |
| ios_increment_build_number       | Increments the buld number in Info.plist |
| ios_export                       | Compile your app and export an archive |
| ios_export_options_create        | Creates a populated .plist with export options |
| ios_archive                      | Generate an archived .ipa|
| ios_upload_hockeyapp             | Upload app builds to HockeyApp |
| ios_upload_itunes_connect        | Upload your build .ipa to iTunes connect |
| *GIT*                            | |
| checkout_git                     | Clone a git repository. This step is automatically run and you don't need to reference this task |
| *Build*                          | |
| build_path_create                | Creates a temporary build folder. This step is automatically run and you don't need to reference this task |
| build_path_clean                 | Cleans the build folder. This step is automatically run and you don't need to reference this task ||
| *Deploy*                         | |
| deploy_copy                      | A simple copy based deployment task |

## Release notes
| Release | Features |
| :---  | :---|
| 0.2   |   0.2 Kicks things off with a whole host of features, like allowing you to build iOS app projects and maven projects. Upload iOS binaries to Hockeyapp and iTunes connect. Increment the iOS build number. Automatically commit modified files back to GIT.  |
| 0.3   |   Added the ability to read configuration from both the ReadyCI host and the repository. Simply add a readyci.yml file to the root of your repository and it'll be included in the build. |
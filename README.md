# Ready CI
A no-fuss CI/CD service and collection of build scripts

## Why use Ready CI?

### It comes with scripts
Ready CI comes with build scripts so that you spend less time setting up your automated CI/CD infrastructure, and get to making automated builds faster. Ready CI scripts currently support:
* iOS apps

### Command-line or web-service
You can run Ready CI on the command-line within another CI environment like Jenkins, or run Ready CI as a service on it's own.

### Supports GIT web-hooks
Ready CI supports GIT commit web-hooks when you run it as a service so that your automated builds start as soon as you push to your git repository.

### Parses iOS provisioning profiles
Configuring iOS builds is tricky. Ready CI uses the .mobileprovision file generated by Apple Developer Portal to automatically configure your build and remove some of the guess-work.

### Handles whitespace
Paths, filenames and target names look great when you use whitespace. However, whitespace can be a nightmare to manage in CI scripts so Ready CI handles whitespace like a pro so that your builds keep working while your project files are clean and readable.   
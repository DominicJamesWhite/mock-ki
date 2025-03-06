CLI cheat sheet
This page contains a list of commonly used humctl commands and flags.

Installation
Visit CLI for instructions how to install the humctl CLI.

Shell completions
Completions are available for bash, fish, PowerShell, and zsh.

# To see available shells

humctl completion --help

# To generate the autocompletion script for your shell, e.g. bash

humctl completion bash

# To get setup instructions for your shell, e.g. bash

humctl completion bash --help
bash

# Set up autocomplete in bash into the current shell, bash-completion package should be installed first.

source <(humctl completion bash)

# Add autocomplete permanently to your bash shell

echo "source <(humctl completion bash)" >> ~/.bashrc
fish

# Add autocomplete to your fish shell

echo 'humctl completion fish | source' > ~/.config/fish/completions/humctl.fish
zsh

# To load completions in your current shell session:

source <(humctl completion zsh)

# To load completions for every new session, execute once:

#### Linux:

humctl completion zsh > "${fpath[1]}/\_humctl"

#### macOS:

humctl completion zsh > $(brew --prefix)/share/zsh/site-functions/\_humctl
Configuring humctl

# If at any point you don't know what flags are accepted or what subcommands are present

# please use the --help -or -h command

humctl create app my-app --help
humctl create -h

# Append the verbosity level (0-9) to any command

humctl get orgs -v9

humctl login # Authenticate humctl using web-based browser flow
humctl api get /current-user | jq .email # Verify that you are logged in with the correct email ID
humctl config view # View the config that has been set
humctl get orgs # Get a list of all orgs you have access to
humctl config set org my-org # Set your default Organization to my-org
humctl config set app my-app # Set your default Application to my-app
humctl config set env staging # Set your default Environment to staging
humctl config set env --delete # Unset your default Environment
humctl config org # View which org has been set as default org

# Set defaults using environment variables prefixed with HUMANITEC\_<key name in caps>

# Order of preference will be:

# flags on command > environment variables > config file

set -Ux HUMANITEC_ORG my-org # Set the default org using environment variables in fish shell
export HUMANITEC_APP my-app # Set the default Application in bash
export HUMANITEC_ENV staging # Set the default Application in bash
export HUMANITEC_TOKEN token-value # Set a Humanitec token. Overrides a user token obtained via 'humctl login'
Viewing and finding objects

# Flags like --org, --app, --env, --token are universal

# Get commands with table output

humctl get orgs # List all orgs the user / token has access to
humctl get apps # List all apps the user / token has access to in the set org
humctl get envs # List all envs the user / token has access to in the set org / app

# Get the JSON output of any command

humctl get orgs -o json

# Get the YAML output of any command

humctl get orgs -o yaml

# Get the beautified YAML output of a Resource Definition

humctl get def default-humanitec-dns -o yaml | yq .

# To get details of a single org, use the org name as a parameter

humctl get orgs my-org
Creating and updating objects
For all commands, specify the Organization via --org or set it in the config via humctl config set org.

humctl create app my-app # Create a new Application with ID my-app
humctl create env my-env --app my-app --type staging # Create a new Environment of type "staging" inside the Application "my-app"
humctl create env my-env --app my-app --type staging --from development # Create a new Environment of type "staging" based on the existing "development" Environment

humctl apply -f a.yaml # Create or update objects based on a Humanitec manifest file
humctl apply -f a.yaml -f b.yaml # Create or update objects based on several Humanitec manifest files
humctl apply -f ./mydir # Create or update all objects based on all Humanitec manifest files in directory "mydir"
cat a.yaml | humctl apply -f - # Create or update objects using stdin

# Create a Shared Value on the Application level

humctl create value <YOUR_KEY> <your-value> \
 -d "The value of your-value stored in YOUR_KEY" \
 --app my-app

# Create a Shared Value override on the Environment level

humctl create value <YOUR_KEY> <your-value> \
 -d "The environment value of your-value stored in YOUR_KEY" \
 --app my-app \
 --env development

# Create a Shared Values on the Application level as a secret reference

humctl create value <YOUR_SECRET_KEY> "name-of-secret-in-secret-store" \
 --app my-app \
 --description "my secret" \
 --is-secret-ref \
 --secret-store dev-vault
Deleting objects
For all commands, specify the Organization via --org or set it in the config via humctl config set org.

humctl delete app my-app # Delete the Application with ID "my-app"
humctl delete env staging --app my-app # Delete the Environment "staging" inside the Application "my-app"
humctl delete def my-res-def # Delete the Resource Definition "my-res-def"

humctl delete -f a.yaml # Delete an object based on a Humanitec manifest file
humctl delete -f a.yaml -f b.yaml # Create or update several objects based on Humanitec manifest files
humctl delete -f ./mydir # Create or update all objects based on all Humanitec manifest files in directory "mydir"
Working with the Orchestrator API
You can use the humctl CLI to issue HTTP requests against the Platform Orchestrator API . This is useful if the target resource is not (yet) natively covered by the CLI.

humctl api get /orgs/my-org/secretstores # Get all secret stores registered for the Organization "my-org"
humctl api get /orgs/my-org/agents # Get all Agents registered for the Organization "my-org"
humctl api get /orgs/my-org/users # Get all users for the Organization "my-org"

# Update the Organization role of the user with ID "some-user-id" to "manager"

humctl api patch /orgs/my-org/users/some-user-id \
-d '
{
"role": "manager"
}
Working with Deployments
Add --org, --app, and --env flags to commands if not set in your humctl config .

Performing rollbacks

# Roll back a deployment to the previous deployment in the current environment (as set by humctl config)

# This will also restore the values to that previous version

humctl deploy deploy +1 .

# Roll back a deployment to the previous deployment in the current environment (as set by humctl config)

# but keep the current version of values

humctl deploy deploy +1 . --values-version .
Working with Deployment Sets and Deltas

# Get the diff between the Deployment Sets of the previous and the current Deployment

humctl diff sets deploy/+1 deploy/. --app my-app --env development

# Compare the "development" environment to the "production" environment

humctl diff sets env/development env/production --app my-app

# Create a Delta to add an environment variable to the "main" container in the "sample-app" workload

humctl create delta --name "add variable" --app my-app --env development

# Update the Delta using its ID from the output of the previous command

humctl update delta 3a30b4cc6c89f5ec3a2dad902b4d28e6525dfe12 add /modules/sample-app/spec/containers/main/variables/NEW_VAR 'Hello World!' --app my-app

# Deploy the Delta to the current environment:

humctl deploy delta 3a30b4cc6c89f5ec3a2dad902b4d28e6525dfe12 development --app my-app --message "Adding NEW_VAR to sample-app"

# Remove a Workload from an Environment. If you have the Score file, check the "Working with Score" section as well

DELTA_ID=$(humctl create delta --name "Remove workload" --app my-app --env development -o json | jq -r .metadata.id)
humctl update delta ${DELTA_ID} remove /modules/my-workload --app my-app
humctl deploy delta ${DELTA_ID} development --app my-app --message "Remove workload"
Working with Applications and Environments
Pausing and resuming Environments

# Pause the Environment "development" of Application "my-app"

humctl api put "/orgs/my-app/apps/my-app/envs/development/runtime/paused" -d 'true'

# Resume the Environment

humctl api put "/orgs/my-app/apps/my-app/envs/development/runtime/paused" -d 'false'
Working with Active Resources
Add --org, --app, and --env flags to commands if not set in your humctl config .

# Get all Active Resources for an Environment

humctl get active-resources

# or

humctl resources active-resource-usage

# Get the URL for an Environment which has a "dns" resource (assuming a TLS/SSL setup)

humctl get active-resources \
 -o json \
 | jq -r '. | map(. | select(.metadata.type == "dns")) | map((.metadata.res_id | split(".") | .[1]) + ": [" + .status.resource.host + "](https://" + .status.resource.host + ")") | join("\n")'

# Get the Kubernetes namespace name for an Environment

humctl get active-resources \
 -o yaml \
 | yq -r '.[] | select (.metadata.type == "k8s-namespace") | .status.resource.namespace'

# Force-delete an Active Resource. Specify ../resources/<type>/<res_id>?force=true

# (!) Cannot be undone, use with caution

humctl api delete \
 /orgs/my-org/apps/my-app/envs/development/resources/config/modules.my-workload.externals.my-resource?force=true
Analyzing infrastructure
humctl get resource-account # Get available Cloud Accounts
humctl resources check-account my-account # Check access for a Cloud Account named "my-account"
humctl resources graph # Output the Resource Graph for an Environment

# Test the ability for the Platform Orchestrator to connect to the cluster for a given Environment

humctl resources check-connectivity --app my-app --env development --env-type development

# Get container logs

humctl api get \
 "/orgs/my-org/apps/my-app/envs/development/logs?workload_id=demo-workload&container_id=demo-container"
Working with Score
Add --org, --app, and --env flags to commands if not set in your humctl config .

humctl score init # Create a sample Score file
humctl score deploy # Deploy the Score file "score.yaml"
humctl score deploy -f my-score.yaml # Deploy a Score file of any name
humctl score deploy --image myimage:latest # Specify the container image to use
humctl score deploy --dry-run # Create a draft Delta only. Use "humctl get deployment-delta" to inspect it
humctl score deploy --wait # Display deployment output and wait for it to be completed
humctl score deploy --remove -f scoreToRemove.yaml # Remove workloads instead of adding them
humctl score deploy --deploy-config deployConfig.yaml # Use a deploy config for multiple workloads
humctl score deploy --extensions humanitec.score.yaml # Use a Score extension file
humctl score deploy --overrides overrides.yaml # Use an overrides file
humctl score validate # Validate the Score file "score.yaml"
humctl score validate --extensions humanitec.score.yaml # Validate the Score file "score.yaml" and the extension file "humanitec.score.yaml"

# Override any Score file properties

humctl score deploy --property "containers.demo.image=myimage:latest"
Working with Resources
Add --org, --app, and --env flags to commands if not set in your humctl config .

# List available Resource Types for current Organization

humctl score available-resource-types

# Output a specific Resource Type (option 1)

humctl score available-resource-types --filter-type dns -o yaml

# Output a specific Resource Type (option 2)

humctl get resource-type -o yaml | yq e '.[] | select(.metadata.type == "dns")'

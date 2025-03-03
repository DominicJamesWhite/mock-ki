{{ metadata }}

actions:
action-one
source: humanitec/paths/create-project
input: {{ project_name }}
action-two
source: humanitec/paths/create-application
inputs: {{ project_name }} {{ application_name}}
action-three
source: humanitec/paths/create-environment
inputs: {{ project_name }} {{ application_name}} {{environment_name}}
action-four
source: github-actions
inputs: gh clone {{ repo }} {{ target: {{project_name}}-{{application-name}} }}
action-five
source: humanitec/paths/add-metadata
input: {{gh repo}}

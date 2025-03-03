apiVersion: core.api.humanitec.io/v2
kind: Path
metadata:
id: template
name: "Template path"
description: "This is a template path. All it does is call the 'get resource' path. Or something!?"
success: "This went well. This is a success message".
on-fail: "Here's the ways to troubleshoot this going wrong; A: Something bad happened. B: Another bad thing. C: So-and-so.

entity:
actions:
find-some-resource:
source: humanitec/cli
inputs: "humctl get resource {{ user-input }}"

## Ops First Spec


### Goal

An easy process to ensure operational integrity.

### Strategy

Create a template file of the spec to make it easy to start new components.

When doing one or more refactors, each element must be worked over again.

### Spec Elements

#### Refactor

List the refactors to be completed in this round, in the style of commit messages.

Determine which semver elements this package of refactors bumps.

Create Github issues for each item on the list.

#### Troubleshoot

This section describes flow through the component, with the principle input and outcomes.

It also shows examples of commands used to troubleshoot.
These should raw, so clipboard-copy is ready to paste and run.

#### Deploy

State the deployment process and time/effort expectation.

#### Scale

State the scaling process and time/effort expectation.

#### Recover

State the recovery process and time/effort expectation.

State any data loss.

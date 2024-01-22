# Flux CDEvents Provider

<!--
The title must be short and descriptive.
-->

**Status:** provisional

<!--
Status represents the current state of the RFC.
Must be one of `provisional`, `implementable`, `implemented`, `deferred`, `rejected`, `withdrawn`, or `replaced`.
-->

**Creation date:** 2024-01-19

**Last update:** 2024-01-19

## Summary

CDEvents Provider for flux. Sends a CDEvent to the specified address based on the flux event that triggered the provider, using a user-defined mapping of events.
<!--
CDEvents Provider for flux. Sends a CDEvent to the specified address based on the flux event that triggered the provider.
-->

## Motivation

CDEvents enables interoperability between supported tools in a workflow, and flux is a very popular continuous delivery tool, as such we have received many questions about implementing CDEvents into the tool.
<!--
CDEvents enables interoperability between supported tools in a workflow, and flux is a very popular continuous delivery tool, as such we have received many questions about implementing CDEvents into the tool.
-->

### Goals

Integrate CDEvents into Flux with a CDEvents Provider.

<!--
Integrate CDEvents into Flux with a CDEvents Provider.
-->

### Non-Goals

A CDEvent receiver is already in progress.

<!--
A CDEvent receiver is already in progress.
-->

## Proposal

Add CDEvents to the list of available providers in Flux Notification controller. The user should be able to create mappings that tell it which CDEvents to send based on the flux event. It should then send that CDEvent to the specified URL.
<!--
Add CDEvents to the list of available providers in Flux Notification controller. The user should be able to create mappings that tell it which CDEvents to send based on the flux event. It should then send that CDEvent to the specified URL.
-->

### User Stories

<!--
Optional if existing discussions and/or issues are linked in the motivation section.
-->

### Alternatives

There are currently no viable alternatives for the CDEvent Provider.
<!--
Certain use cases for CDEvents could be done alternatively using available providers such as the generic webhook.
-->

## Design Details

Add a provider for CDEvents that will send a CDEvent upon receiving a flux event from an alert. The user will be able to write a yaml file for the provider and deploy it to their cluster. This provider will take a user defined mapping of flux events to CDEvents, with a default mapping for a handful of events if not user-provided. Once an alert is triggered that matches the flux events, it will send the mapped CDEvent to an address. This CDEvent will be created with the [CDEvents Go SDK](https://github.com/cdevents/sdk-go).

Example Provider YAML:

```yaml
apiVersion: notification.toolkit.fluxcd.io/v1beta2
kind: Provider
metadata:
  name: cdevents-provider
  namespace: flux-system
spec:
  type: cdeventss
  secretRef:
    name: generic-token
  address: http://10.100.0.17:5000
  mapping: 
    ReconciliationFinished: dev.cdevents.pipelinerun.finished
    GitOperationFailed: dev.cdevents.incident.reported

```

<!--
This section should contain enough information that the specifics of your
change are understandable. This may include API specs and code snippets.

The design details should address at least the following questions:
- How can this feature be enabled / disabled?
- Does enabling the feature change any default behavior?
- Can the feature be disabled once it has been enabled?
- How can an operator determine if the feature is in use?
- Are there any drawbacks when enabling this feature?
-->

## Implementation History

<!--
Major milestones in the lifecycle of the RFC such as:
- The first Flux release where an initial version of the RFC was available.
- The version of Flux where the RFC graduated to general availability.
- The version of Flux where the RFC was retired or superseded.
-->

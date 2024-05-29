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

**Last update:** 2024-05-17

## Summary

This RFC proposes to add a `Provider` type to the Flux notification-controller API for the sending of CDEvents.

When `Provider` objects configured to send CDEvents are alerted by a Flux event, they will utilise a user defined mapping of Flux and CDEvents events to send an appropriate CDEvent payload to a defined URL.
<!--
CDEvents Provider for flux. Sends a CDEvent to the specified address based on the flux event that triggered the provider.
-->

## Motivation

CDEvents enables interoperability between supported tools in a workflow, and flux is a very popular continuous delivery tool, as such we have received many questions about implementing CDEvents into the tool. The receiver part of this integration is already implemented in flux 2.3.0
<!--
CDEvents enables interoperability between supported tools in a workflow, and flux is a very popular continuous delivery tool, as such we have received many questions about implementing CDEvents into the tool.
-->

### Goals

Integrate [CDEvents](https://github.com/cdevents) into Flux with a CDEvents Provider that supports user mapping of flux to CDEvent events.

<!--
Integrate CDEvents into Flux with a CDEvents Provider.
-->

### Non-Goals

A CDEvent receiver is already implemented into Flux 2.3.0.

<!--
A CDEvent receiver is already in progress.
-->

## Proposal

Add CDEvents to the list of available `Providers` in Flux Notification controller. The user should be able to create mappings within the configuration of the `Provider` that dictates the CDEvent payload to send depending on the Flux event that triggered the alert. These CDEvent payloads should have meaningful data from the source event.
<!--
Add CDEvents to the list of available providers in Flux Notification controller. The user should be able to create mappings that tell it which CDEvents to send based on the flux event. It should then send that CDEvent to the specified URL.
-->

### User Stories

Users of multiple CI/CD tools such as Tekton and Flux could use CDEvents as a way to enable interoperability.

For example, a user may want a Tekton `pipeline` to run once a HelmRelease flux resource has succeeded in a Helm install. On successful helm install, Flux will emit an event with reason `InstallSucceeded` which the user will have mapped in their `Provider` configuration to correspond to an `Environment.Modified` CDEvent. The CDEvent `Provider` will then send a payload with that CDEvent, which will also contain data from the Flux event, to a CloudEvents broker that Tekton is subscribed to, and trigger a Pipeline Run within Tekton. 

![User Stories Tekton](user-stories-provider.drawio.png)


<!--
Optional if existing discussions and/or issues are linked in the motivation section.
-->

### Alternatives

There are currently no viable alternatives for the CDEvent Provider.
<!--
Certain use cases for CDEvents could be done alternatively using available providers such as the generic webhook.
-->

## Design Details

Adding a Flux `Provider` for CDEvents that will send a CDEvent payload upon receiving a flux event from an alert. 

The user will be able to define a Flux `Provider` custom resource and deploy it to their cluster. This provider configuration will allow the user to define a mapping of which Flux events correspond to which CDEvent to send. Once an alert is triggered for this provider, it will send the corresponding CDEvent, based on the Flux event that caused the alert. This CDEvent will be created using the [CDEvents Go SDK](https://github.com/cdevents/sdk-go). As well as the user-defined mapping, the provider will include some default mappings for Flux-Helm events in the case that the user does not provide a custom mapping.

The CDEvents broker is not a part of this design and is left to the users to set up however they wish.

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

| Helm Event                | CDEvent                         |
| ------------------------- | ------------------------------- |
| InstallSucceeded          | Environment.Modified            |
| InstallFailed             | Incident.Detected               |
|UpgradeSucceeded           | TaskRun.Finished                |
|UpgradeFailed              | Incident.Detected               |
|TestSucceeded              | TestCaseRun.finished            | 
|TestFailed                 | Incident.Detected               | 
|RollbackSucceeded          | Environment.Modified            |
|RollbackFailed             | Incident.Detected               |
|UninstallSucceeded         | Environment.Modified            |
|UninstallFailed            | Incident.Detected               |
|ReconciliationSucceeded    | Environment.Modified            | 
|ReconciliationFailed       | Incident.Detected               | 

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

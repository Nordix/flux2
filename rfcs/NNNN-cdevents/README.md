# Flux CDEvents Receiver

<!--
The title must be short and descriptive.
-->

**Status:** provisional

<!--
Status represents the current state of the RFC.
Must be one of `provisional`, `implementable`, `implemented`, `deferred`, `rejected`, `withdrawn`, or `replaced`.
-->

**Creation date:** 2023-12-08

**Last update:** 2023-12-08

## Summary

CDEvents Receiver for flux. Takes a cdevent sent to the receiver webhook URL, verifies it and checks the cdevent type against spec.Events, then triggers reconciliation.
<!--
One paragraph explanation of the proposed feature or enhancement.
-->

## Motivation

CDEvents enables interoperability between supported tools in a workflow, and flux is a very popular continuous delivery tool, as such we have received many questions about implementing CDEvents into the tool. 
<!--
This section is for explicitly listing the motivation, goals, and non-goals of
this RFC. Describe why the change is important and the benefits to users.
-->

### Goals

Integrate CDEvents into Flux with a CDEvents Receiver.
<!--
List the specific goals of this RFC. What is it trying to achieve? How will we
know that this has succeeded?
-->

### Non-Goals

a CDEvent provider will be handled as a separate RFC in the future.

<!--
What is out of scope for this RFC? Listing non-goals helps to focus discussion
and make progress.
-->

## Proposal

Add CDEvents to the list of available receivers in Flux Notification controller. Similar to other receivers such as the github, the user will be able to specify the event types that causes Flux to act on inside the spec. The receiver will also verify using the [CDEvents Go SDK](https://github.com/cdevents/sdk-go) that the payload sent to the webhook URL is a valid CDEvent.

<!--
This is where we get down to the specifics of what the proposal actually is.
This should have enough detail that reviewers can understand exactly what
you're proposing, but should not include things like API designs or
implementation.

If the RFC goal is to document best practices,
then this section can be replaced with the actual documentation.
-->

### User Stories

<!--
Optional if existing discussions and/or issues are linked in the motivation section.
-->

### Alternatives

Certain use cases for CDEvents could be done alternatively using available receivers such as the generic webhook.

<!--
List plausible alternatives to the proposal and explain why the proposal is superior.

This is a good place to incorporate suggestions made during discussion of the RFC.
-->

## Design Details

Adding a receiver for CDEvents that works much like the other event-based receivers already implemented. The user will be able to write a yaml file for the receiver and deploy it to their cluster. The receiver takes the payload sent to the webhook URL, checks the headers for the event type, and filters out events based on the user-defined list of events in spec.Events. If left empty this filtering will not take place. It then creates a CDEvent, using the [CDEvents Go SDK](https://github.com/cdevents/sdk-go) to create a CDEvent from the payload body, and verifies it also using the Go SDK. Valid events will then trigger reconciliation of the resources.

```yaml
apiVersion: notification.toolkit.fluxcd.io/v1
kind: Receiver
metadata:
  name: cdevents-receiver
  namespace: flux-system
spec:
  type: cdevents
  events:
    - "cd.change.merged.v1"
  secretRef:
    name: receiver-token
  resources:
    - kind: Kustomization
      apiVersion: kustomize.toolkit.fluxcd.io/v1
      name: podinfo
      namespace: flux-system
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

---
title: A tale of a Software Supply Chain Attestation
slug: a-tale-of-supply-chain-attestation
authors: miguel
image: /img/v2/chainloop-dev-overview.png
---

In our previous [blog post](/blog/introducing-chainloop), I alluded to team dynamics being one of the main problems tackling Software Supply Chain Security. Today we are going to dig into that with a role-playing exercise.

Your name is Dyson, you work on the Security and Operations (SecOps) team at Cyberdyne solutions. You are in charge of making sure that the organization's Software Supply Chain Security is improved end-to-end, from source code to packaging and distribution.

<!--truncate-->

You feel pretty good about the current state of things. Your development teams are already signing their commits, scanning their first and third-party components, generating Software Bill of Materials (SBOMs), signing container images and even blocking releases based on vulnerability scanning or signature verification. Life is good!

The next day you receive an email from your CISO saying that all the progress in application security is great but an additional effort needs to be done to make sure we can trust all that metadata.

He seems to be **concerned** about all that **information** being **scattered around, not being trustworthy or easily available for auditing**. He sends a link to [slsa.dev](https://slsa.dev/) and says we need to be at least level 3 at that, whatever that means.

Got it! It seems that what we need to do is to **implement an attestation and provenance layer in our SSC**, we "just" need to

- figure out the format of this new piece of metadata (attestation/provenance) that can glue artifacts together.
- put together a place where the provenance information and their associated artifacts (SBOMs, static analysis, vulnerability scanner output) can be stored.
- ensure authenticity and integrity via some sort of signing process.

Choosing solutions for each of those areas was surprisingly straightforward since there are great building blocks for attestation crafting (e.g [in-toto](https://in-toto.io/)), signing/verification ([Sigstore](https://www.sigstore.dev/)), and storing ([Storage/OCI](https://github.com/opencontainers/image-spec/blob/main/spec.md)), but figuring out an end-to-end wasn't easy.

In any case, your team spends the next couple of months deploying an OCI artifact storage, defining the expected attestation specification, building some tooling to glue it all together, and writing a lot of documentation to share with the development teams for their integration.

Now to the easy part (or what you think), to make developer teams adopt it.

## Day one integration

Let's introduce team Skynet, they are building (in their words) a revolutionary AI.

They leverage a pretty standard CI/CD setup using GitHub Actions. Build/Test a couple of GoLang/Java services, package them into signed container images, craft and store SBOMs in S3 and then deploy them to Kubernetes.

You open a chat with Sarah, one of their lead developers and it goes like this (pleasantries skipped):

D - Hi Sarah, I have some security requirements that I'll need you to put in place in your CI/CD as soon as possible.

S - Hi Dyson, but we already did a ton of work on that!

D - Yes I know, but this is different. What you did was implement application security. What we need now is for us to understand more about the CI/CD setup and collect all the additional information that you are already creating, the SBOM remember?

S - Oh, I see. how much work is that? We are quite constrained already, we have to launch soon.

D - I'll send you a document with all the details, but long story short we need you to craft an attestation using in-toto that will contain a SLSA provenance statement, sign it with cosign and store it in an OCI registry that our team has put in place. Ahh, and you'll need to start pushing your existing artifacts to that OCI registry too.

D - Could you carve out some time in your future sprints to start looking into that?

S - Wait but we have already a lot on our plate, and to be frank I understood half of the words you just said.

D - I understand, but all this work is required to meet new compliance requirements so I am afraid that delaying it's not an option.

S - I'll see what I can do

After weeks of back and forth, the Skynet team integration is done and the SecOps teams now have access to the workflow build information via signed attestations and their associated artifacts (sBOM, logs, ...) in a centralized OCI registry. You will tell our boss that the **Skynet team reached SLSA level 3 provenance compliance**! Life is good, again.

## Day two challenges

You quickly realized that applying this process to all teams doesn't scale. It's time-consuming, requires manual work and frustrates both sides.

But the **worst part is yet to come. Day two operations**.

### Enforcing new requirements

After some trial and error querying the information you realize all CI pipelines that build container images will also need to send the Dockerfile and their base image. In addition, signing/verification capabilities need to be elevated by leveraging our brand-new transparent log and ephemeral key management [1].

Implementing such iterative changes requires a similar manual, **overloaded communication and interaction pattern** as seen before, over and over again.

### Enforcing best practices

You realize that it's hard to make sure all the teams are following the documented best practices. For example, some teams might not be storing their container images content digest in their attestation or storing the artifacts in the provided artifact storage.

Now you have to inspect each attestation to make sure they are compliant with your requirements.

### Prevent configuration drift or regressions

You also notice it's very hard to keep track of the state of the attestation integration each team is in. Some might have fallen behind the latest requirements. In fact, It's very hard to detect if a regression has been added or an integration has been removed altogether.

### Lack of visibility and actionability

Although you are receiving attestation and artifacts from different teams, you still lack information about your Organization's Software Supply Chain as a whole, their owners, their readiness, implementation state and so on.

Even worse, if you detect an issue during auditing or because of a new security issue, there aren’t any enforcement mechanisms for even blocking a CI workflow if needed.

## How can Chainloop help?

Chainloop is built on the promise of providing a similar **state-of-the-art attestation/provenance compliance but through a mechanism that has lower friction, removes manual steps and makes day-two operations first-class citizens**.

If instead, you, Dyson, choose Chainloop

- You will save months implementing a custom end-to-end solution for attestation, artifact storage, specification or documentation.
- You can [onboard](/getting-started/workflow-definition#workflow-and-contract-creation) and [keep track](/getting-started/operator-view) of the teams and workloads that are being integrated with the system.
- You can [declaratively define contracts](/getting-started/workflow-definition#add-materials-to-the-contract) with the attestation requirements and associate them with such workloads.
- You can rest assured that on the developer side, best practices are followed and the attestation is following the declared contract.
- You can detect Supply Chain anomalies since your control plane records workflow runs either successfully or not.

Sarah from the Skynet team

- Will just need the Chainloop attestation tool plus some credentials provided by Dyson to do the integration following some [simple steps](/getting-started/attestation-crafting). No additional security expertise is required.
- When new requirements are set by Dyson, Sarah will get notified and can follow the same procedure as before. No manual communication channel is required.

In the next blog post, I'll get into details on how this scenario looks like using Chainloop but for now you can follow our [getting started guide](/category/getting-started) for more details.

And remember, send feedback [our way](https://us21.list-manage.com/contact-form?u=801f42b3abafc40b1a17c5f25&form_id=3f3bbfe15e6fcd4a60be9b966652cfd5)!

Thanks!

[1] This refers to the use of Sigstore's [Rekor](https://docs.sigstore.dev/rekor/overview/) and [Fulcio](https://docs.sigstore.dev/fulcio/overview) not being covered in this post. The point of mentioning them is to illustrate that requirements and practices evolve.

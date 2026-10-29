---
layout: post
title: "Seqera Actions to automatically launch pipelines"
date: 2026-10-27
author: "Amit Indap"
tags:
  - Seqera
  - Nextflow
  - automation
---

[Sasha Dagayev](https://www.linkedin.com/in/sasha-dagayev/) recently posted [a demo](https://lnkd.in/p/gGgX76Jh) of a new Seqera Platform feature called Pipeline Actions that launches pipelines automatically.



I've become a big fan of Seqera Platform over the past two years, and what I like most is its UI for launching and tracking pipelines. That's high praise from someone who used to monitor pipeline progress in the AWS Batch console 😆

I set up an action to launch my [HiFi WGS trio pipeline](https://github.com/embedbio/nextflow-hifi-wgs). Here's how.

First, go to **Actions** in [Seqera Platform](https://cloud.seqera.io/).

![The Actions menu in Seqera Platform](/assets/img/seqera_actions_menu.png)

Click **Add Action**, then fill in a name and tag. You can choose from several triggers, which are described in detail in the [Seqera documentation](https://docs.seqera.io/platform-cloud/pipeline-actions/overview).

I chose **Bucket event**. Make sure the bucket has the right IAM permissions, as described in the [bucket events documentation](https://docs.seqera.io/platform-cloud/pipeline-actions/overview#bucket-events).

![The new action form in Seqera Platform](/assets/img/newaction.png)

For a bucket event, choose the bucket you want to watch and the event that fires the trigger (object created or object deleted).

Next, choose the marker file that signals your data is ready. This part confused me a little: I set the marker file to the name of the samplesheet my pipeline expects.

![Configuring the bucket event trigger](/assets/img/trigger.png)

Then point the action at its target. In my case, that's my [pipeline repo on GitHub](https://github.com/embedbio/nextflow-hifi-wgs).

![Setting the action's target pipeline](/assets/img/action-target.png)

Finally, test the action. I uploaded my samplesheet to the bucket, and it launched my pipeline.

![The pipeline run launched by the action](/assets/img/pipeline-launch.png)

You might be wondering: why not do the same thing with AWS Lambda? I asked Seqera Co-Scientist:

> Can't a Seqera action on a bucket event also be implemented on another platform, like AWS Lambda? Is the advantage that it's launched from the Platform UI?

Here's its answer:

> Yes. Under the hood, a Seqera bucket event action is the same pattern as S3 → Lambda. When you create the action, Seqera sets up an S3 bucket notification and an SNS topic in your AWS account, and S3 sends events through SNS to Platform. A Lambda can do the same job by calling the Platform API, using the launch hook from earlier or the standard launch endpoint. The run still shows up in Platform either way.
>
> The UI is part of the advantage, but the bigger part is that Seqera handles the plumbing and the safeguards for you.
>
> Rule of thumb:
>
> If one fixed input path fits your workflow, as in your current setup, the Seqera action is simpler and gives you more visibility.
>
> If each event needs its own parameters, such as multiple trio samplesheets in parallel, separate outdirs per batch, or custom filtering, you need Lambda (or an agent target). The Seqera action can't pass event details to the run.
>
> You can also combine them. Let Lambda handle only the "which file arrived" logic and call a Seqera launch hook. The pipeline, compute environment and defaults stay managed in Platform, and Lambda only supplies the per-batch parameters.

| | Seqera bucket event action | S3 → Lambda → Platform API |
|---|---|---|
| **Setup** | A form in the UI. Seqera creates the S3 notification and SNS topic | You write the function, IAM role, secret and S3 trigger |
| **Code to maintain** | None | The Lambda function, its runtime and its token |
| **Trigger history** | Built in: each event, its status and payload | CloudWatch logs, unless you build something better |
| **Safety rails** | Rate limiting, loop suppression, and auto-pause when a launch fails | Whatever you write yourself |
| **Pause/resume and editing** | One click in the UI | Disable the trigger or redeploy |
| **Chaining** | Can feed pipeline-run-event actions (run B after run A finishes) | You build it |
| **Using event data (file path, batch name) as parameters** | No. Parameters are fixed on the action | Yes. Any logic you want |
| **Storage support** | S3 only | S3, or anything that can call an HTTP API |

I'm excited about this feature. I've wanted to launch pipelines automatically for a while, but I'm still an AWS Lambda novice. Actions make it much easier.

---
layout: post
title: "Seqera Actions to automatically launch pipeiines"
date: 2026-10-27
author: "Amit Indap"
tags:
  - Seqera
  - Nextflow
  - automation
---

[Sasha Dagayev ](https://www.linkedin.com/in/sasha-dagayev/) recently posted [this demo](https://lnkd.in/p/gGgX76Jh) on a new Seqera platform feature to launch pipeines automatically. 

The  new feature called Seqera Actions make it easy to automate launch jobs on Seqera Platform. Over the past two years, I've become a big fan of Seqera Platform. The thing I like the most is the UI to track and launch pipelines. This is coming from a person who used to track pipeline progress on AWS Batch console :laughing: 

I set up an action to launch my [HiFi WGS trio pipeline](https://github.com/embedbio/nextflow-hifi-wgs)

Step one:

1. Navigate to **Actions** on [Seqera platform](https://cloud.seqera.io/)

 ![ Actions menu in Seqera Platform](/assets/img/seqera_actions_menu.png)

2. Click  Add Action and the fill in the information for the name and tag. There are various Triggers you can choose from, described in detail [here](https://docs.seqera.io/platform-cloud/pipeline-actions/overview)

For my purposes I chose bucket event. Make sure the bucket has appropriate IAM permissions, as described [here](https://docs.seqera.io/platform-cloud/pipeline-actions/overview#bucket-events)

![newaction.png](/assets/img/newaction.png)

For bucket event, choose the bucket you want to watch. Choose your trigger ( object created, object deleted)

Choose your marker file that signals your data is ready. I'm a little confused on this, becasue I set my marker file the name of the samplesheet that my pipeline expects. 

![trigger.png](/assets/img/trigger.png)

3. Next, you point your action towards your Target. For my case, I pointed it to my [pipeline repo on github](https://github.com/embedbio/nextflow-hifi-wgs)

![action-target.png](/assets/img/action-target.png)

4. Finally, you can test your action. I uploaded my samplesheet to the bucket and it triggered my pipeline

![pipeline-launch.png](/assets/img/pipeline-launch.png)

You might ask yourself, why can't I implement the same thing via AWS Lambda.  I gave this prompt to Seqera Co-Scientist:

>Can't a Seqera action on a bucket event also be implmemeted on a onther platform like AWS lambda? Is the advantage  being its launched via platform UI?

It answered the following:

>Yes. Under the hood, a Seqera bucket event action is the same pattern as S3 → Lambda. When you create the action, Seqera sets up an S3 bucket notification and an SNS topic in your AWS account, and S3 sends events through SNS to Platform. A Lambda can do the same job by calling the Platform API, using the launch hook from earlier or the standard launch endpoint. The run still shows up in Platform either way.

>The UI is part of the advantage, but the bigger part is that Seqera handles the plumbing and the safeguards for you.

>Rule of thumb:

>If one fixed input path fits your workflow, as in your current setup, the Seqera action is simpler and gives you more visibility.

>If each event needs its own parameters, such as multiple trio samplesheets in parallel, separate outdirs per batch, or custom filtering, you need Lambda (or an agent target). The Seqera action can't pass event details to the run.
You can also combine them. Let Lambda handle only the "which file arrived" logic and call a Seqera launch hook. The pipeline, compute environment and defaults stay managed in Platform, and Lambda only supplies the per-batch parameters.

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

I'm really excited for this feature because I've wanted to do something like this for a while now, and I'm still a AWS Lambda neophyte. But now with Actions, it makes it a lot easier to launch pipelines automatically. 

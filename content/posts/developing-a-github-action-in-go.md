---
title: "Developing a Github Action in Go"
date: 2020-03-23T16:43:10+01:00
draft: false
toc: false
images:
tags: 
  - github
  - azure
  - go
---

![GitHub actions logo](https://avatars0.githubusercontent.com/u/44036562?s=200&v=4)

I've taken part in this month's GitHub Hackathon. This time it consisted of creating a [GitHub Action](https://github.com/features/actions) and publishing it on the Marketplace.

Browsing through the marketplace, I did not see any connector to Azure Queues or Azure Event Hub, so I decided to write an action in Go that would send the last job status of the GitHub Workflow (failure, success, etc...), along with some Git metadata, to an Azure Queue, for further processing through any consumer application that would read from that Azure Queue.

I can say that writing an Action is as simple as using them in your Workflows, and the Marketplace is a really good idea with quality Actions, as both the community and the vendors such as Microsoft or Amazon Web Services are publishing pug-and-play open-sourced Actions to integrate with their systems easily.

You can check my Action code repository [here](https://github.com/amongil/gh-action-push-workflow-last-job-status-to-azure-queue) and the published Action itself [here on the Marketplace](https://github.com/marketplace/actions/send-job-status-to-azure-queue).
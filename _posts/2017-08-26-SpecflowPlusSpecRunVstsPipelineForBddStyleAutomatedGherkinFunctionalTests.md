---
layout: post
title: "SpecFlow VSTS Build pipeline with SpecRunner For BDD Style Automated Gherkin Functional Tests"
date: 2017-08-26
author: tarun
tags: ["DevOps", "BuildPipeline", "SpecFlow"]
categories:
- "DevOps"
image: "/images/screenshots/tarun/SpecFlowVstsBuildPipelineBlogCover.png"
description: "Walk through to set up an automated build pipeline for SpecFlow in Visual Studio Team Services (VSTS). How to use SpecRunner as the test adapter with Visual Studio Test task and SpecFlow+ to put SpecFlow+Living documentation with in VSTS. We'll also look at putting together a testing dashboard to surface useful application quality metrics on a dashboard."
permalink: /DevOps/SpecflowPlusSpecRunVstsPipelineForBddStyleAutomatedGherkinFunctionalTests
published: true
keywords: "DevOps, SpecFlow, SpecFlow+, SpecFlowPlus, BDD, Gherkin, SpecFlow+ Living Documentation, Gherkin, Functional Test Pipeline, SpecRunner, VSTS, Automated Build Pipleine for SpecFlow, SpecRunner with Visual Studio Test Task, SpecFlow Analysis Results in VSTS, Email Spec Analysis, Attach SpecFlow Analysis with VSTS Build, Given When Then SpecFlow Gherkin VSTS, Automated Functional Testing Pipeline in VSTS, Azure, SpecFlow Agent Setup, Set up VSTS Agent for SpecFlow, How to setup SpecFlow in VSTS, Example SpecFlow Build Pipeline, SpecRunner SpecFlow setup for VSTS, SpecFlow VSTS NuGet Packages, SpecFlow Cucumber for .NET in VSTS & TFS, SpecFlow Plus TFS, SpecRunner TFS Build Pipeline"
---
In this blogpost I'll show you how to create an automated build pipeline for SpecFlow with VSTS using SpecRunner to execute your automated BDD functional tests written using Gherkin syntax. We won't stop just here, I'll show you how to add the SpecFlow+ extension to create living functional test documentation accessible right from VSTS... We'll end by looking at how to pin some of this on a dashboard in VSTS to surface some key testing metrics for visibility with your development and operation teams... 
<!--more-->

![Image]({{site.url}}/images/screenshots/tarun/SpecFlowTarget.jpg)

# Build Pipeline 
Simply select one of the existing build template that includes the Visual Studio Build and Test task, but to really put your SpecFlow (plus) tests to best use watch out for the for the following points... 

+ __Using SpecFlow+ SpecRunner test adapter__
SpecFlow tests don't necessarily need the SpecRunner for execution, they can be run using MSTestv2 or any other compatible framework. However, using SpecRunner provides great benefits, for example you can get some very useful analysis out of the tests that wouldn't necessarily be available if you used other test execution frameworks. Luckily using SpecRunner for test execution doesn't require any installation on the agent! 

    The Visual Studio Unit Test Task can be used to invoke other test adapters, as you can see in the screen shot below, it is possible to pass the path to the custom test adapter. 

    ![Image]({{site.url}}/images/screenshots/tarun/UseVisualStudioTestTaskToRunSpecRunner.jpg)

    If you already have a reference to the custom test adapter in your code then you don't need to explicitly pass this in this task. Just make sure you add the following NuGet packages in your project... As you can see in the screen shot below, I am using SpecFlow+ Runner, you can find out more about the [benefits of SpecFlow+ here](http://specflow.org/plus/)

    ![Image]({{site.url}}/images/screenshots/tarun/SpecFlowPlusRunner.jpg)

    If the packages are added correctly, in the build summary for this step you'll be able to see that the tests are executed using the SpecFlow+ Runner test adapter... 

    ![Image]({{site.url}}/images/screenshots/tarun/BuildLogVsTestSpecFlowPlusRunner.jpg)

+ __SpecFlow Analysis__ 
When you run SpecFlow tests in Visual Studio you may see the analysis report generated by SpecFlow... 

    ![Image]({{site.url}}/images/screenshots/tarun/SpecFlowAnalysisReport.jpg)

    In your visual studio Test Task if you check the option to `Upload Test Attachments` then these analysis reports along with the SpecFlow execution logs would get attached to your test runs, that you can view from the Test Runner explorer... 

    ![Image]({{site.url}}/images/screenshots/tarun/SpecFlowAnalysisAttachment.jpg)

    Instead I would prefer to see the analysis result attached to the build as an artifact, even better have the analysis results emailed out to the developers, testers and ops folks. You can do that by using a few good tasks in your pipeline to pick up the test results, zip them up and upload them to the as build artifacts. More on the email task in the next section... 

    Using the copy task, you can selectively pick the *.html or *.log files from the build test results folder to the agent test results directory. 

    ![Image]({{site.url}}/images/screenshots/tarun/SpecFlowCopyFileTestResults.jpg)

    Next zip the agent test results directory... 

    ![Image]({{site.url}}/images/screenshots/tarun/SpecFlowTestAnalysisArchive.jpg)

    Next publish the test analysis as a build artifact... 

    ![Image]({{site.url}}/images/screenshots/tarun/PublishSpecAnalysisAsBuildArtifact.jpg)

    Now when you run the build you can see the analysis results attached as an artifact...

    ![Image]({{site.url}}/images/screenshots/tarun/SpecFlowAnalysisAttachAsArtifact.jpg)

+ __SpecFlowPlus Living Documentation__
When you are writing all these functional tests using a `Given When Then` Gherkin BDD style syntax it makes all the more sense to make these tests easily discoverable to your stakeholders. SpecFlowPlus now offers a Visual Studio Team Services [SpecFlow+LivingDoc](https://marketplace.visualstudio.com/items?itemName=techtalk.techtalk-specflow-plus) extension that allows you to point your build to the csproj file that contains the specflow tests, it then uses [picklesdoc](http://docs.picklesdoc.com/en/latest/) under the hood to generate the documentation and upload it to a page in VSTS under the test hub. The document stays up to date with the changes in the tests as the build likely gets run on every change, in addition to that the tests are searchable which makes the discovery of specific tests very easy. 

    > What better way to preserve your cukes than to pickle them !

    Simply install the extension from the marketplace and include it in your build pipeline. Point it to the spec project you want to generate the documentation for, you can obviously have multiple instances of this extension if you want to generate documentation from multiple sources.  

    ![Image]({{site.url}}/images/screenshots/tarun/SpecFlowPlusLivingDocumentationExample.jpg)

    Personally I think the SpecFlow+Living Documentation is pretty rough and needs some more work, but none the less it provides great value even in it's current state.

    ![Image]({{site.url}}/images/screenshots/tarun/SpecFlowPlusLivingDocumentationVsts.jpg)

+ __Email SpecFlow Analysis__
I love my slack and team rooms, but every once in while you find email still has relevance. You can extend the build pipeline to generate email notifications with a custom message and attachments that you find useful sharing with others. You can download the free [Send Email](https://marketplace.visualstudio.com/items?itemName=rvo.SendEmailTask) task created by a fellow MVP. You can use this task in your pipeline, it supports multiple email addresses, custom SMTP, attachments and HTML in the body of the message. 

    ![Image]({{site.url}}/images/screenshots/tarun/SpecFlowEmailTask.jpg)

    Putting all of these to work, I am able to generate some pretty useful and actionable emails for the devs, testers, ops and stakeholders that care about some of this stuff... As you can see in the screen shot below, there are some useful links to the SpecFlow+ documentation, testing dashboard, code changes that shows a div between the last two versions and details about why and by whom was the build trigger in addition to the agent the tests were executed.  

    ![Image]({{site.url}}/images/screenshots/tarun/VstsSpecFlowTestResultsEmailSummary.jpg)

+ __Testing Dashboard__ 
Information radiators help bring the visibility of key stats to the people it makes most sense to. Luckily with VSTS, there are a great range of widgets that support pinning live updates onto dashboards. As you can see in the example below, I am tracking a few things...

    ![Image]({{site.url}}/images/screenshots/tarun/SampleTestingDashboardVsts.jpg)

    - Markdown 
    - Last build status on a traffic light signal  
    - Build Status Trend 
    - Code Churn in the repository 
    - Activity Feed against the repository and the specs build 
    - Spec execution - Outcome & Duration 
    - Spec execution - Passed & Failed
    - Code coverage by blocks and lines covered 
    - Quick access to SpecFlow+ Living Documentation
    - Release Definition Status & Trend 
    - Deployment status by branch 

Hope you found this useful... Would love your feedback... Any questions on the specflow build pipeline or specflow plus spec runner, feel free to ask. If there are others things that you track in VSTS for your functional tests or know any other cool extensions to improve the use of SpecFlow within VSTS then feel free to leave a comment... 

Tarun 
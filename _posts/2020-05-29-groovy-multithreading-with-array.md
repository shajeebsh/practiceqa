---
title: "Groovy Multithreading with 1 Array"
date: 2020-05-29 21:29:38 
author: shajeebhameed
layout: page
slug: groovy-multithreading-with-array
status: publish
---

import hudson.model.*

import jenkins.model.*

import groovy.json.JsonSlurper

import groovy.transform.Field

import jenkins.model.Jenkins

import java.util.concurrent.*

import hudson.AbortException

import groovy.json.*

@Field def TEST_SET_WAITING = "Waiting"

@Field def TEST_SET_IN_PROGRESS = "InProgress"

@Field def TEST_SET_COMPLETE = "Complete"

@Field def ALL = true

@Field def AVAILABLE = true

println "Json File Data";

println "----------------------------\n"

String jsonString= GetFileData("data.json")

def jsonData = new JsonSlurper().parseText(jsonString)

def buildPara= jsonData.build

def environment= jsonData.environment

// Log the parameters.

println "Build: ${buildPara}"

println "Environment: ${environment}"

println "JsonData: ${jsonData}"

println "VM Data";

println "----------------------------\n"

String vmData= GetFileData("VM.csv")

def TestMachines=[];

vmData.split("\n").each { t ->

def vmVals = t.split(',')

def vmName=vmVals[0] + " " + vmVals[1]

def ipPara=vmVals[2];

def modJsonData1= ModifyJsonMachines(jsonData, vmName)

def modJsonData=modJsonData1.replace("\n","");

TestMachines.add(new TestMachine(vmName, ipPara, buildPara, modJsonData))

}

println TestMachines

LogTestMachines(TestMachines, ALL, AVAILABLE)

LogTestMachines(TestMachines, !ALL, AVAILABLE)

// Run jobs when machines are available.

while (GetTestMachinesByStatus(TestMachines, TEST_SET_COMPLETE).size() < TestMachines.size()) {

def statusChanged = false

for(tmInProgress in GetTestMachinesByStatus(TestMachines, TEST_SET_IN_PROGRESS)) {

def jenkins = Jenkins.instance

def jobs = jenkins.items.findAll { job -> job.name.equals('myNewJob_Thread') }

jobs.each

{ job ->

def build = job.getBuildByNumber(tmInProgress.JobID)

if (build != null) {

if (!build.isBuilding())

{

tmInProgress.Status = TEST_SET_COMPLETE

statusChanged = true

println "Completed ${tmInProgress.Name}"

}

}

}

}

for(tmWaiting in GetTestMachinesByStatus(TestMachines, TEST_SET_WAITING)) {

def jobInfo = Jenkins.instance.getItemByFullName('myNewJob_Thread')

def jobID = jobInfo.getLastBuild().getNumber() + 1

def params =

[

new StringParameterValue('IP', tmWaiting.Ip),

new StringParameterValue('JSONRequest', tmWaiting.JsonRequest),

new StringParameterValue('Build', tmWaiting.Build),

]

job = Hudson.instance.getJob('myNewJob_Thread')

def execution = job.scheduleBuild2(0, new Cause.UpstreamCause(build), new ParametersAction(params))

tmWaiting.Status = TEST_SET_IN_PROGRESS

tmWaiting.JobID = jobID

statusChanged = true

break

}

if (statusChanged)

{

println ""

println ""

LogTestMachines(TestMachines, ALL, AVAILABLE)

LogTestMachines(TestMachines, !ALL, !AVAILABLE)

println ""

}

else

{

println "."

}

int waitTime = 8000

println "Waiting $waitTime milliseconds..."

sleep(waitTime)

}

println "DONE!"

def GetTestMachinesByStatus(machines, status)

{

def filteredList = machines.findAll{ it.Status.equals(status) }

return filteredList;

}

def LogTestMachines(machines, all, available)

{

if (all)

{

println "Test machines (All = ${GetTestMachineCount(machines, ALL, available)}):"

}

else

{

def availability = (available) ? "Available" : "Unavailable"

println "Test machines (${availability} = ${GetTestMachineCount(machines, !ALL, available)}):"

}

machines.each()

{

if (all || it.Available.equals(available))

{

println "\t${it.toString()}"

}

}

}

def GetTestMachineCount(machines, all, status)

{

def count = 0

machines.each()

{

if (all || it.Status.equals(status))

{

count++

}

}

return count

}

def GetFileData(fileName)

{

String path=build.workspace.toString() + "\\\" + fileName

def fp;

String fileData;

if(build.workspace.isRemote())

{

channel = build.workspace.channel;

fp = new hudson.FilePath(channel, path)

println "channel=$channel";

} else {

fp = new hudson.FilePath(new File(path))

}

fileData =fp.read().getText();

return fileData

}

def ModifyJsonMachines(jsonData, vmName)

{

jsonData.machines= vmName

def result = JsonOutput.toJson(jsonData)

def modJsonData =JsonOutput.prettyPrint(result)

return modJsonData

}

class TestMachine

{

String Name

String JsonRequest

String Ip

String Build

String Status

int JobID

Boolean Available

TestMachine(name, ip, build, jsonRequest)

{

this.Name = name

this.JsonRequest=jsonRequest

this.Ip= ip

this.Build= build

this.Available = true

this.Status = "Waiting"

this.JobID = -1

}

String toString()

{

if(this.JobID != -1)

return "[ Name=${Name}, IP=${Ip}, Build=${Build}, Status=${Status}, JobID=${this.JobID}, URL: ${Hudson.instance.getRootUrl()}job/myNewJob_Thread/${JobID}/console]"

else

return "[ Name=${Name}, IP=${Ip}, Build=${Build}, Status=${Status}, JobID=${this.JobID} ]"

}

}

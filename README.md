# Better DevOps with LogDNA and IBM Cloud Kubernetes service

# Introduction
	

In the [What's Going On In My Cluster](https://haralduebele.blog/2019/04/08/whats-going-on-in-my-cluster/), Harald Uebele describes how to set up
the Native Cloud Starter app with IBM LogDNA service. In this tutorial, we'll expand on that and show some use cases where LogDNA helps
developers and operators with some common scenarios in the context of an IBM LogDNA cluster. Most of this is adapted from personal
experience in building and troubleshooting the [IBM Developer](https://developer.ibm.com/blogs/introducing-the-ibm-developer-mobile-app/). 

# Scenario 1: Create views


By default, everything is displayed, which can be overwhelming in a kubernetes cluster with many system and internal components. We want to filter
the view to only see the applications we're interested in.

1. Filtering by app. LogDNA can separate its logs into buckets, but the type of bucket depends on the kind of cluster. In Kubernetes, this view represents containers. So, in our case we can select containers representing our application to limit the data show:

From the drop-down list, select the apps we want to view: ![screenshot](logdna_screenshots/no-filter.png).

After filtering for just the containers we want, the noisy system info is removed and we just see events.
![screenshot](logdna_screenshots/filter-app.png).

This filtered view can then be saved: ![screenshot](logdna_screenshots/new-view.png). 

Takeway: Filter the view by a combination of "app" selection.


Next, let's look at how we can filter by entering a search query. This will be especially helpful for creating alerts based on conditions in the log.

We click on the saved view on the left side of the screen, then enter a search term.  For example, we can set it up to view Java container shutdowns with the phrase "JVM is exiting".

Then select "Save as new view/alert" and choose a name. We'll call it "JVM Exiting" - ![screenshot](logdna_screenshots/jvm-exiting.png).

Now we have a new view available on list of views, which includes both the initial app filtering as well as the new search based filter. 

# Scenario 2: Alerts from views 

![screenshot](logdna_screenshots/alert-setup.png).

This alert setup will send emails when the lines are triggered. We can batch up entries to keep from being overwhelmed repeatedly if matches 

Let's try to trigger a log based alert. We'll delete one of our pods from the command line.


```
$ [fix-text] [k|istio-south] #  kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
articles-dddf44c85-xqdqt     2/2     Running   0          5d23h
authors-7bc9995896-f8kv2     2/2     Running   0          8d
client-59b69f46c4-7k6j7      1/1     Running   0          15d
logdna-agent-4rwxl           1/1     Running   0          15d
logdna-agent-jspz8           1/1     Running   0          15d
logdna-agent-xtkfj           1/1     Running   0          15d
web-api-v1-cc9b5c5d5-9ws9z   2/2     Running   0          8d
web-app-f8f47cdfd-qr8rr      2/2     Running   0          8d


[22|Thu May 23, 02:58 PM| yan@kb16-vmware|~/src/adv/tracing/ibm/opentracing-istio-troubleshooting]
$ [fix-text] [k|istio-south] #  kubectl delete pod articles-dddf44c85-xqdqt
pod "articles-dddf44c85-xqdqt" deleted
```

![screenshot](logdna_screenshots/alert-slack.png).

We can also set-up email alerts - multiple alerts can bethe same alert 
![screenshot](logdna_screenshots/alert-email.png).

Notice how the view updates immediately with a message about the container's JVM stopping.

# Scenario 3: Map frequency

Next, we'll take a look at the Board feature of LogDNA. This is a feature I love because it's a great way to visualize
a large quantity of data quickly, find trends and zero-in on error conditions. We'll start by simply getting a graph
of authors requested by the API. We'll create a graph that uses the authors app and the filters it by author names. 
We'll create two graphs, one for "Niklas" and for "Harald", and then drive traffic with `artillery` and we can examine
the histogram in the view and the frequencies of various requests.  We launched requests about 5 minutes apart
request content with different parameters. This is a synthetic test but shows the kind of visualizations 
that are trivial to build in LogDNA by simply filtering on a single term.


```
Sun May 26, 11:35 AM #  artillery run simtraffic_harald.yml 
```

```
574|Sun May 26, 11:41 AM| # artillery run simtraffic_niklas.yml 
```
You can see the frequency spikes based on when artillery tests sent data to our test URL.

![screenshot](logdna_screenshots/author-freq.png)

# Scenario 4: Map errors - drill down into logs

Let's create some error conditions and use LogDNA to figure out what happened. As before, we'll break the app by deleting
a pod. We'll create graphs for each app in our microservice and filter on "error".  You may want to create graphs for more
specific conditions (for example, in a different app we have I have filters specific for network errors.) But what I like about
the broad kind of error filter is the ability to drill down by selecting a time range when an interesting event happens and
immediatly jumping into the raw log view, correlated by timestamps. This is a huge improvement over traditional `grep` based
workflows for event search in log files. 

![screenshot](logdna_screenshots/error2.png)


Scenario 7: Time based filtering - natural langauge time ranges



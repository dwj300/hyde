---
layout: post
title: Computing PageRank of a Large Graph on Hadoop
---

<div class="message">
  tl;dr Lets use a lot of computers to do some math.
</div>

For the <a href="https://wsdmcupchallenge.azurewebsites.net/">WSDM Cup Challenge</a> we wanted to compute the <a href="https://www.wikiwand.com/en/PageRank">pagerank</a> of a citation graph. In this case, the graph provided by Microsoft Research was a directed graph with 73,543,432 vertices and 757,462,733 edges. In order to do this efficiently, we are going to spin up a HD Insight Cluster on Microsoft Azure. I chose 5 D3 nodes, but this configuration can get pretty pricy. Once the cluster is deployed, ssh into the head node using the account you specified during cluster creation. Then lets get going:

I recommend doing all of this within a screen (or tmux) session.
{% highlight bash %}
$ screen
{% endhighlight %}

First download the graph with wget:
{% highlight bash %}
$ wget https://academicgraph.blob.core.windows.net/graph-2015-08-20/PaperReferences.zip
{% endhighlight %}

Then unzip it. But first we need to install p7zip (trust me, the stock unzip won't work), then we can actually unzip it. 

{% highlight bash %}
$ sudo apt-get install p7zip-full
$ 7za x PaperReferences.zip
{% endhighlight %}

Now we hit the first roadblock. The graph package we will be using requires an edge file with node id's as integers, but the file we are given has strings. Never fear, let's just whip up some python to fix it.

{% highlight python %}
f = open('PaperReferences.txt', 'r')
out = open('output.edges', 'w+')
ids = {}
counter = 0
for l in f:
    a, b = l.split('\t')
    if a in ids:
        a = ids[a]
    else:
        ids[a] = counter
        a = counter
        counter += 1
    if b in ids:
        b = ids[b]
    else:
        ids[b] = counter
        b = counter
        counter += 1
    out.write("{0}\t{1}\n".format(a, b))
out.close()
f.close()
{% endhighlight %}

Now that we have our edge file in the correct format, lets put it in HDFS:
{% highlight bash %}
$ hdfs dfs -mkdir [username]
$ hdfs dfs -mkdir pagerank
$ hdfs dfs -put output.edges pagerank/
{% endhighlight %}

One last thing - we need to know how many vertices are in the edge file:
{% highlight python %}
f = open('output.edges')
m = 0
for l in f:
    a, b = l.split('\t')
    a, b = int(a), int(b)
    if a > m:
        m = a
    if b > m:
        m = b
print m
{% endhighlight %}

Finally, now that the edge file is in HDFS, we can run pegasus. The first argument is the number of nodes in the graph (make sure to add one to the output of the compute-max script), and the second argument is the number of reducers. The general recomendation is to use 2*n reducers where n is the number of worker nodes.
{% highlight bash %}
$ wget http://www.cs.cmu.edu/~pegasus/PEGASUSH-2.0.tar.gz
$ tar -xvzf PEGASUSH-2.0.tar.gz
$ cd PEGASUS/
$ ./run_pr.sh 73543432 10 pagerank nosym
{% endhighlight %}  

If all goes well, the hadoop job will be submitted and you will be able to monitor the progress of the map and reduce phase of the job with output like:
{% highlight bash %}
15/11/08 22:55:57 INFO mapreduce.Job:  map 87% reduce 14%
15/11/08 22:55:59 INFO mapreduce.Job:  map 88% reduce 15%
15/11/08 22:56:05 INFO mapreduce.Job:  map 88% reduce 16%
15/11/08 22:56:08 INFO mapreduce.Job:  map 89% reduce 16%
15/11/08 22:56:18 INFO mapreduce.Job:  map 90% reduce 16%
15/11/08 22:56:25 INFO mapreduce.Job:  map 90% reduce 17%
{% endhighlight %}

When it finishes, you should see output like: 
{% highlight bash %}
[PEGASUS] PageRank computed.
[PEGASUS] The final PageRanks are in the HDFS pr_vector.
[PEGASUS] The minium and maximum PageRanks are in the HDFS pr_minmax.
[PEGASUS] The histogram of PageRanks in 1000 bins between min_PageRank and max_PageRank are in the HDFS pr_distr.
{% endhighlight %}

Finally, you can get the files off of HDFS:
{% highlight bash %}
$ hdfs dfs -copyToLocal pr_vector/part-0000
{% endhighlight %}

The format of the output files are:
{% highlight bash %}
(nodeid TAB "v"PageRank_of_the_node)
{% endhighlight %}

Hopefully this post, while abit long, demonstrated the power of the MapReduce programming paradigm as well as how to work with large graphs. Once you're finished with the HDInsight cluster, make sure to turn it off or else you may rack up a nasty bill.

## Links
* <a href="http://www.cs.cmu.edu/~pegasus/">PEGASUS</a>
* <a href="https://azure.microsoft.com/en-us/services/hdinsight/">Azure HD Insight</a>
* <a href="https://hadoop.apache.org/">Hadoop</a>

-----

Want to see something else added? <a href="https://github.com/dwj300/dwj300.github.io/issues/new">Open an issue.</a>

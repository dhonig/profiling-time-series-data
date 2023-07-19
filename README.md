# Profiling Streaming Time Series Data

- Skill level
    
    **Beginner, no prior knowledge requirements**
    
- Time to complete
    
    **Approx. 25 min**
    

Introduction: *In this guide, we will show you how you can combine bytewax with ydata-profiling to profile and improve the quality of your streaming flows!*

## ****Prerequisites****

**Python modules** bytewax==0.16.2 ydata-profiling==4.3.1

## Your Takeaway

*You'll be able to handle and structure data streams into snapshots using Bytewax, and then analyze it with ydata-profiling creating a comprehensive report of data characteristics.*

## Table of content

- [Resources](https://github.com/bytewax/-Profiling-Time-Series-Data/tree/main#resources)
- [Data Profiling](https://github.com/bytewax/-Profiling-Time-Series-Data/tree/main#data-profiling)
- [Step 1. Environmental Sensor Telemetry Dataset](https://github.com/bytewax/-Profiling-Time-Series-Data/tree/main#step-1-environmental-sensor-telemetry-dataset)
- [Step 2. Inputs and parsing](https://github.com/bytewax/-Profiling-Time-Series-Data/tree/main#step-2-inputs-and-parsing)
- [Step 3. Windowing](https://github.com/bytewax/-Profiling-Time-Series-Data/tree/main#step-3-windowing)
- [Step 4. Profile report](https://github.com/bytewax/-Profiling-Time-Series-Data/tree/main#step-4-profile-report)
- [Summary](https://github.com/bytewax/-Profiling-Time-Series-Data/tree/main#summary)

## Resources

[Github link](https://github.com/bytewax/-Profiling-Time-Series-Data)

[Jupyter Notebook link](https://colab.research.google.com/gist/awmatheson/d30d520f693d1ddc4319ab3bc87eccf2/ydata-profiling-streaming.ipynb)

[Data sample link](https://www.kaggle.com/datasets/garystafford/environmental-sensor-data-132k)

## Data Profiling

Unlike traditional models, where data quality is usually assessed during the creation of the data warehouse or dashboard solution, streaming data requires continuous monitoring.

It is essential to maintain data quality throughout the entire process, from collection to feeding downstream applications.

In what concerns data profiling, [ydata-profiling](https://github.com/ydataai/ydata-profiling) has consistently been a [crowd favorite](https://medium.com/ydata-ai/auditing-data-quality-with-pandas-profiling-b1bf1919f856), either for [tabular](https://ydata-profiling.ydata.ai/docs/master/pages/getting_started/examples.html) or [time-series](https://medium.com/towards-data-science/how-to-do-an-eda-for-time-series-cbb92b3b1913) data. And no wonder why — it’s one line of code for an extensive set of analysis and insights.

Let's see it in action!

## Step 1. Environmental Sensor Telemetry Dataset 

Let's download the [Environmental Sensor Telemetry Dataset](https://www.kaggle.com/datasets/garystafford/environmental-sensor-data-132k) (License — CC0: Public Domain), which contains several measurements of temperature, humidity, carbon monoxide liquid petroleum gas, smoke, light, and motion from different IoT devices

https://github.com/bytewax/-Profiling-Time-Series-Data/blob/9b5985e778157b55b2bef412a5cda0cd790d0dc2/utils/get-dataset.sh#L1

In a production environment, these measurements would be continuously generated by each device, and the input would look like what we expect in a streaming platform such as [Kafka](https://bytewax.io/guides/enriching-streaming-data). 

## Step 2. Inputs and parsing

To simulate the context we would find with streaming data, we will read the data from the CSV file one line at a time and create a dataflow.

First, let’s make some necessary imports:

https://github.com/bytewax/-Profiling-Time-Series-Data/blob/119059e709a639e9fd3ac10f1c83e3cc4d91954c/dataflow.py#L1-L12

Then, we define our dataflow object and add our CSV input.

https://github.com/bytewax/-Profiling-Time-Series-Data/blob/9b5985e778157b55b2bef412a5cda0cd790d0dc2/dataflow.py#L14-L15

Afterwards, we will use a stateless `map` method where we pass in a function to convert the string to a `datetime` object and restructure the data to the format (device_id, data).

https://github.com/bytewax/-Profiling-Time-Series-Data/blob/9b5985e778157b55b2bef412a5cda0cd790d0dc2/dataflow.py#L17-L26

The `map` method will make the change to each data point in a stateless way. The reason we have modified the shape of our data is so that we can easily group the data in the next steps to profile data for each device separately rather than for all of the devices simultaneously.


## Step 3. Windowing
Now we will take advantage of the stateful capabilities of bytewax to gather data for each device over a duration of time that we have defined. ydata-profiling expects a snapshot of the data over time, which makes the `window` operator the perfect method to use to do this.

https://github.com/bytewax/-Profiling-Time-Series-Data/blob/9b5985e778157b55b2bef412a5cda0cd790d0dc2/dataflow.py#L28-L52

In ydata-profiling, we are able to produce summarizing statistics for a dataframe which is specified for a particular context. For instance, in our example, we can produce snapshots of data referring to each IoT device or to particular time frames.


## Step 4. Profile report

After the snapshots are defined, leveraging ydata-profiling is as simple as calling the PorfileReport for each of the dataframes we would like to analyze:

https://github.com/bytewax/-Profiling-Time-Series-Data/blob/9b5985e778157b55b2bef412a5cda0cd790d0dc2/dataflow.py#L56-L73

## Step 5. Kicking off
Once the profile is complete, the dataflow expects some output, so we can use the built-in `StdOutput` to print the device that was profiled and the time it was profiled at that was passed out of the profile function in the map step:

https://github.com/bytewax/-Profiling-Time-Series-Data/blob/9b5985e778157b55b2bef412a5cda0cd790d0dc2/dataflow.py#L75

And are ready to run our program! You can clone this repository to your machine and run the following commands:

https://github.com/bytewax/-Profiling-Time-Series-Data/blob/main/run.sh#L3

## Summary

We can now use the profiling reports to validate the data quality, check for changes in schemas or data formats, and compare the data characteristics between different devices or time windows.

Being able to process and profile incoming data appropriately opens up a plethora of use cases across different domains, from the correction of errors in data schemas and formats to the highlighting and mitigation of additional issues that derive from real-world activities, such as *anomaly detection* (e.g., fraud or intrusion/threats detection), *equipment malfunction*, and other events that deviate from the expectations (e.g., *data drifts* or misalignment with business rules).

_This article was written with the support of the Ydata team_

## We want to hear from you!

If you have any trouble with the process or have ideas about how to improve this document, come talk to us in the #troubleshooting Slack channel!

## Where to next?

- [Switch this tutorial to use Kafka inputs](https://bytewax.io/guides/enriching-streaming-data)

See our full gallery of tutorials →

[Share your tutorial progress!](https://twitter.com/intent/tweet?text=I%27m%20mastering%20data%20streaming%20with%20%40bytewax!%20&url=https://bytewax.io/tutorials/&hashtags=Bytewax,Tutorials)

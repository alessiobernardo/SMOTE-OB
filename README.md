# MOA (Massive Online Analysis)
[![Build Status](https://travis-ci.org/Waikato/moa.svg?branch=master)](https://travis-ci.org/Waikato/moa)
[![Maven Central](https://img.shields.io/maven-central/v/nz.ac.waikato.cms.moa/moa-pom.svg)](https://mvnrepository.com/artifact/nz.ac.waikato.cms)
[![DockerHub](https://img.shields.io/badge/docker-available-blue.svg?logo=docker)](https://hub.docker.com/r/waikato/moa)
[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)

![MOA][logo]

[logo]: http://moa.cms.waikato.ac.nz/wp-content/uploads/2014/11/LogoMOA.jpg "Logo MOA"

MOA is the most popular open source framework for data stream mining, with a very active growing community ([blog](http://moa.cms.waikato.ac.nz/blog/)). It includes a collection of machine learning algorithms (classification, regression, clustering, outlier detection, concept drift detection and recommender systems) and tools for evaluation. Related to the WEKA project, MOA is also written in Java, while scaling to more demanding problems.

http://moa.cms.waikato.ac.nz/

## Using MOA

* [Getting Started](http://moa.cms.waikato.ac.nz/getting-started/)
* [Documentation](http://moa.cms.waikato.ac.nz/documentation/)
* [About MOA](http://moa.cms.waikato.ac.nz/details/)

MOA performs BIG DATA stream mining in real time, and large scale machine learning. MOA can be extended with new mining algorithms, and new stream generators or evaluation measures. The goal is to provide a benchmark suite for the stream mining community. 

## SMOTE-OB
The SMOTE-OB algorithm is in the `moa/src/main/java/moa/classifiers/meta/imbalanced` folder, while the streams used are in the `Data Streams/` folder.

We run all the experiments on a virtual machine inside a Docker container to correctly extract the memory consumed by each model through InfluxDB. After launching the container, we created a `.sh` file to run all the experiments, where each line corresponding to a single model tested on a particular data stream. For example, the following line shows the `first` run of the `SMOTE-OB` model with `ARF` as base learner using the `appearing-clustersincremental` stream having imbalance ratio `1:9`, and an `incremental` drift starting at the `45000˚` instance. The output of the results is then redirected to a folder. Each model, for each data stream and base learner, was tested `10` times with a different seed value.

`docker run --rm --name="appearing-clusters_incremental_1_9_ARF_1" -v $(pwd)/results:/src/results test_moa bash -c "java -Xmx15g -Xss50M -cp moa.jar -javaagent:sizeofag-1.0.4.jar moa.DoTask \"EvaluatePrequential -l (meta.imbalanced.SMOTEOB -s 10 -d -e 1) -s (ArffFileStream -f (Data Streams/Artificial/appearing-clusters/1/appearing-clustersincremental.arff)) -e (WindowFixedClassificationPerformanceEvaluator -w 45000 -j 10000 -r -f -g) -i -1 -f 5000\" 1> results/appearing-clusters/incremental_1_9/ARF/1_results.csv 2> results/appearing-clusters/incremental_1_9/ARF/1_running.csv"`

Once completed, we downloaded them together with the respective memory consumed. To download the latter, we run the following query in InfluxDB for each model and stream tested. The following query downloads the average memory consumed by the model previously tested on `10` runs.

```
from(bucket: "bucket_name")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r._measurement == "docker_container_mem")
  |> filter(fn: (r) => r._field == "usage")
  |> filter(fn: (r) => r.container_name == "appearing-clusters_incremental_1_9_ARF_1" or r.container_name == "appearing-clusters_incremental_1_9_ARF_10" or r.container_name == "appearing-clusters_incremental_1_9_ARF_2" or r.container_name == "appearing-clusters_incremental_1_9_ARF_3" or r.container_name == "appearing-clusters_incremental_1_9_ARF_4" or r.container_name == "appearing-clusters_incremental_1_9_ARF_5" or r.container_name == "appearing-clusters_incremental_1_9_ARF_6" or r.container_name == "appearing-clusters_incremental_1_9_ARF_7" or r.container_name == "appearing-clusters_incremental_1_9_ARF_8" or r.container_name == "appearing-clusters_incremental_1_9_ARF_9")
  |> aggregateWindow(every: v.windowPeriod, fn: last)   
  |> last()
  |> group(columns: ["_usage"])
  |> mean()
  |> yield(name: "appearing-clusters_incremental_1_9_ARF")
  
```  

Finally, we averaged the results over the `10` repetitions of all the models tested, and we used those results to apply the Nemeyi test.

The original paper is in print at the IEEE BigData 2021 conference. You can find the pre-print version [here](https://www.researchgate.net/publication/356287208_SMOTE-OB_Combining_SMOTE_and_Online_Bagging_for_Continuous_Rebalancing_of_Evolving_Data_Streams#:~:text=10.13140/RG.2.2.35335.32165):

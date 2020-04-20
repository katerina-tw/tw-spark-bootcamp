
## Startup
We will be using Python 3 in jupyter to access spark API.
To install spark+jupyter notebook bundle run the command below. It will pull
jupyter/pyspark-notebook image from docker and run the container to enable
us to connect to the notebook. The whole process is taking about 20min. 
Time to get a coffee while listening to the presentation
```
 docker run -it --rm -p 8888:8888 -p 4040:4040 -v ~:/home/jovyan/workspace -v $(pwd)/scripts:/scripts jupyter/pyspark-notebook
```

## Set up spark config
By default spark UI is available via 4040 port on the spark running machine.
However, this only works while a spark code is running.
In order to monitor the historical job executions in spark we need to enable this functionality first:
```
docker ps
docker exec -u 0 -it CONTAINER_ID bash
cd $SPARK_HOME
cp conf/spark-defaults.conf.template conf/spark-defaults.conf
nano conf/spark-defaults.conf         
```
uncomment the line 
```
spark.eventLog.enabled true
```
 and save. Then run:
 ```
./sbin/start-history-server.sh 
```
to start spark history and monitor historical jobs by 4040 port.

## Login 
Open a browser and paste the link to open jupyter:
```
http://127.0.0.1:8888

```
if asked for a token - please, copy the one from the output of the previous command.
Similar to:
```
http://127.0.0.1:8888/?token=f57f1d648692576233455580ef7a99d7ef2dd128e1
```

## Developing in Pyspark
See guide [here](scripts/README.md)

## Shutdown
When finished press Ctrl+C in the terminal to shutdown the container

## Docker remove image
If you never need this again then run in the terminal
```
docker image rm jupyter/all-spark-notebook 
```

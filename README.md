
## Startup
We will be using Python 3 in jupyter to access spark API.
To install spark+jupyter notebook bundle run the command below. It will pull
jupyter/pyspark-notebook image from docker and run the container to enable
us to connect to the notebook. The whole process is taking about 20min. 
Time to get a coffee while listening to the presentation
```
 docker run -it --rm -p 8888:8888 -p 4040:4040 --name spark_machine -v ~:/home/jovyan/workspace -v $(pwd)/scripts:/scripts jupyter/pyspark-notebook
```

## Login 
In an output of the previous command find and click on the link similar to the following:
```
http://127.0.0.1:8888/?token=f57f1d648692576233455580ef7a99d7ef2dd128e1
```
This should open a new browser window with jupyter.

## Set up spark config
By default spark UI is available via 4040 port on the spark running machine.
However, this only works while a spark code is running.
In order to monitor the historical job executions in spark we need to enable this functionality first.
Open the terminal and run the commands:
```
mkdir /tmp/spark-events
chown jovyan:users /tmp/spark-events
docker ps
# get the CONTAINER ID of the running spark_maching container and insert in the next command
docker exec -u 0 -it CONTAINER_ID bash 
cd $SPARK_HOME
cp conf/spark-defaults.conf.template conf/spark-defaults.conf
nano conf/spark-defaults.conf         
```
uncomment the line 
```
spark.eventLog.enabled true
```
save and close. Now run:
 ```
./sbin/start-history-server.sh 
```
to start spark history and monitor historical jobs by 4040 port on the local machine.

Configuration for accessing the historical spark jobs is done.
By default the events logs go under /tmp/spark-events. Once we run some of the jobs, the log will start filling in 
and we will be able to access historical information. We will observe them in UI in the next steps.
Now just open in the browser in a new tab:
http://127.0.0.1:4040

## Developing in Pyspark
See guide [here](scripts/README.md)

## Shutdown the container
When finished press Ctrl+C in the terminal to shutdown the container and confirm the operation by pressing 'y'

## Docker remove the image
If you never need this again then run in the terminal
```
docker image rm jupyter/pyspark-notebook 
```

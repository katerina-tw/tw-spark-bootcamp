
## Startup
We will be using Python 3 in jupyter to access spark API.
To install spark+jupyter notebook bundle run the command below. It will pull
jupyter/pyspark-notebook image from docker and run the container to enable
us to connect to the notebook. The whole process is taking about 20min. 
Time to get a coffee while listening to the presentation
```
 docker run -it --rm -p 8888:8888 -p 4040:4040 -v ~:/home/jovyan/workspace -v $(pwd)/scripts:/home/jovyan/workspace/scripts jupyter/pyspark-notebook
```

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

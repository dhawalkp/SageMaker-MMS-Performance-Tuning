# Amazon SageMaker Multi-Model Server Framework Performance Tuning Tips

*Disclaimer: This is based on my understanding of the MMS code, public documentation available on MMS and my experience working on MMS so far. Complete disclaimer that this is only my understanding and may not be fully accurate. The views and opinions expressed does not reflect my employer's i.e. Amazon Web Services Opinions*

## High level Performance Tuning Tips

* You can set MMS Config file properties location as an environment variable  and set all the MMS configuration key values in that file. You can do so by setting the environment variable from your entrypoint script of the SageMaker docker Inference serving image. You need to set environment variable `MMS_CONFIG_FILE` to point to the config.properties location. You can then set the values of `OMP_NUM_THREADS`, including all the other available performance tuning parameters mentioned here.
* Please note below considerations while tuning these values –
Generally I would recommend exercising extreme care before changing the default values of `number_of_netty_threads` and `netty_client_threads` values of Netty. They generally control the incoming HTTP connection serving worker threads and the default values which is directly proportional to number of logical CPUs is most of the times optimal.
* `job_queue_size` parameter is useful to tune when you have a scenario wherein the type of the inference request payload is huge and due to the size of payload being larger may lead to higher heap memory consumption of the JVM in which this queue is being maintained. Ideally you might want to keep the heap memory requirements of JVM lower and allow Python workers to allot more memory for actual model serving. JVM is only for receiving the HTTP Requests, queuing them and dispatching them to the Python-based workers for inferences. If you increase the job_queue_size, you might end up increasing the heap memory consumption of the JVM and thus ultimately snatching away the memory from the host that could have been used by Python workers. Thus, extreme care must be exercised in tuning this parameter as well.
* `default_workers_per_model` parameter is for the backend model serving and might be very valuable to tune since this the critical component of the overall model serving based on which the python processes will spawn threads per Model. If this component is slower (or not tuned properly), the frontend tuning might not be effective.
* Performance tuning is an art and generally requires multiple iterations of tuning based on performance testing results and is dependent on multiple factors including type and size of the model being served, response time of inference request, Throughput expected, type of ML instance selected for serving, Payload size of the inference request etc.
 
Below is the diagram I have put together based on my understanding on how MMS works and what performance knobs do we have for which components –


![Multi Model Server Framework Runtime Architecture](https://github.com/dhawalkp/SageMaker-MMS-Performance-Tuning/blob/main/MMS.png "Multi-Model Server Runtime Architecture")

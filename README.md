# TPC-DS io-intensive workload with Presto on OpenShift

This repos contains the tooling to easily run workloads based on the TPC-DS benchmark with a selection of the most io-intensive queries.

It is designed to run as Jobs in an OpenShift environment with OpenShift Container Storage to provide shared volumes to store reports, but zero installation of any other tools.

Reports are generated by JMeter and available through a simple Web server.

## Requirements

### Deploy a Presto Cluster using S3 to store data

Procedure available [here](https://github.com/guimou/presto-ocs)

#### Data bucket

A data bucket must have been created during the Presto installation. This is the **bucketname** that must be referenced in the **Jobs** files.

### Logs and reports

All the logs and reports will be written to a shared volume for easy retrieval. Create this volume with:

```bash
oc apply -f 00_pvc_presto-data-share.yaml
```

### Web Server

To easily browse logs and reports you can deploy a simple Web Server that will give you access to the shared folder where they are stored.

```bash
oc apply -f 01_dc-web-server.yaml
```

### Presto CLI (Optional)

If you want an ever running Presto CLI env, you can use the provided Deployment Config:

```bash
oc apply -f 02_dc_presto-cli.yaml
```

### Container images (Optional as they are available on quay.io)

#### TPC-DS dataset generator and Presto CLI

From the **docker/tpcds-gen** folder, edit the `build-tpcds-gen.sh` and `Dockerfile` to change the repo name and/or the Presto version you want to buid, then run:

```bash
./build-tpcds-gen.sh
```

#### TPC-DS benchmark

From the **docker/tpcds-run** folder, edit the `build-tpcds-run.sh` and `Dockerfile` to change the repo name and/or optionally the Presto version you want to use.

You can also modify the `jmeter-tpcds-io.jmx` to change/modify the queries you want to run.

Then run:

```bash
./build-tpcds-run.sh
```

## Benchmark

### Create TPC-DS Data sets

Create the different Data sets at different scale directly with the Jobs in the **jobs** folder:

```bash
oc apply -f 01_tpcds_create_table_sf1.yaml
```

Then proceed in the same way for sf10, sf100, sf1000.

### Run the Benchmark

Run the benchmark against the different scale factor data sets with the Jobs in the **jobs** folder:

```bash
oc apply -f 11_tpcds_benchmark_sf1.yaml
```

### Get logs and reports

Jobs logs and reports are available on the Web serber in the `/data` folder.

Ex: http://webserver-presto.apps.\<openshift-address\>/data/

`generate_data_sfxx` and `run_benchmark_sfxx` contain the logs for the respective jobs, timestamped.

`reports` contain the JMeter reports for the different runs, grouped by scale factor and timestamped.

### Credits

Original code by Yifeng Jiang available [here](https://github.com/uprush/kube-presto/tree/master/benchmark).

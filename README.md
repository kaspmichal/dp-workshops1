## GID DataOps CLI: Setting up environment and loading data to dwh 

Welcome to the DataOps CLI Labs workshop repository #1. By the end of this tutorial, you will know how to:
- configure and deploy GID DataOps CLI tool as a user-managed notebook in GCP Vertex AI environment
- move around various DataOps tools in JupyterLab environment
- load static seed data to the warehouse with the use of dbt

Target environment will be Google Cloud Platform's: BigQuery, Vertex AI Managed Notebook, VSCode as IDE. This tutorial was written with GID DataOps 1.0.9 as a current release.


# Exercise
## Setting up environment
1. Go to: https://console.cloud.google.com/vertex-ai?project=dataops-demo-prod&supportedpurview=project or to the Vertex AI Dashboard in your project and choose "Workbench" and then "User-managed notebooks"  
#TODO [Screen]

2. Click on New Notebook located in the topbar and then "Customize..."
   ![Screenshot 2022-04-25 at 22 33 26](https://user-images.githubusercontent.com/77925576/165170160-a08af36a-d022-4c5d-b5cd-a181576a6f76.png)

3. Type in notebook name (preferrably your first and last name).
4. In environment section, choose Debian 10 and "Custom container"
5. Provide a link to the GID DataOps CLI image: [gcr.io/getindata-images-public/jupyterlab-dataops:bigquery-1.0.9](gcr.io/getindata-images-public/jupyterlab-dataops:bigquery-1.0.9)
   ![Screenshot 2022-04-25 at 22 42 09](https://user-images.githubusercontent.com/77925576/165171403-93633875-3f5c-429c-a40a-014a863cd10d.png)
   #TODO [Screen]
6. In machine configuration section, choose n1-standard-1 machine 1vCPU/3.75GB RAM (~0.044 USD / hour) #TODO
7. Leave everything else on default. 
8. Click on "Create Jupyter notebook".
9. Wait until it spuns up correctly and click on "Open Jupyterlab"

You can find full documentation of our GID DataOps CLI tool on https://github.com/getindata/data-pipelines-cli and also https://data-pipelines-cli.readthedocs.io/en/latest/index.html

## Inside the notebook with GID DataOps CLI
You are now inside managed Vertex AI Workbench instance, which will serve as our data pipelines development workflow. This image lets you open:
- VSCode instance
- CloudBeaver, open source SQL IDE
- dbt docs
- python3 interactive terminal

In this tutorial, we will only cover on how to operate within VSCode Instance.

1. Open a VSCode instance. At the top, click on explore and open a home directory so you can easily create new files and track changes to directories inside VSCode.

>-> Tip: In the toolbar click on 'Explore' and then 'Open Folder'. Click OK. You should be located in JUPYTER directory.

![Screenshot 2022-04-25 at 22 59 10](https://user-images.githubusercontent.com/77925576/165173963-c2aaa4c9-d68b-4709-8ddf-1e1c63f79fe6.png)

3. Open a new terminal instance.

![Screenshot 2022-04-25 at 23 01 27](https://user-images.githubusercontent.com/77925576/165174292-ed5b1cc0-0516-40ec-89f9-aa6de7de833f.png)

5. Make sure you are in `/home/jupyter` directory and then execute command `dp init https://github.com/getindata/data-pipelines-cli-init-example`. This will initialize dp-cli tool in the environment. Provide any username when prompted.

>-> Tip: when copy+pasting for the 1st time, you might be asked for permissions to access your clipboard by Chrome. Accept.

6. Run `dp create .` This command will create a full data-pipelines-cli environment with dbt project as a core part of it. IMPORTANT: provide __dataops-test-project__ as a GCP project name. #TODO

>-> Tip: when prompted, you can simply press ENTER to use default values. Don't use it for GCP Project ID!

>-> Tip: use underscores _

>-> Tip: Example of provided values
> 
>![](../../../var/folders/w_/g7pk42215d3_12jkm76_0y900000gn/T/TemporaryItems/NSIRD_screencaptureui_y7pJ42/Screenshot 2022-09-05 at 14.43.57.png)


7. Your environment is now ready to execute some dbt code against BigQuery dwh!

## Loading data to dwh with dbt seed

For this and succeeding exercises we will be using **thelook_ecommerce** [dataset](https://console.cloud.google.com/marketplace/product/bigquery-public-data/thelook-ecommerce?project=gid-dataops-labs). Exercises are meant to slowly build upon data available in the dataset
and arrange transformations into staging (bronze), intermediate (silver) and presentation (gold) layers in dwh.

thelook_ecommerce dataset is organized as 7 tables: ![]
<img width="317" alt="Screenshot 2022-09-05 at 15 59 41" src="https://user-images.githubusercontent.com/77925576/188466518-2e342f7a-6c5b-4c00-b956-bf1cee9093bb.png">
Here, we will start by seeding data into dwh.


We will extend the dataset with our customary data and put it into the new table called:
1. To up some seeds to load your static data to warehouse. You need to provide .yml file with a definition, and a .csv with actual data to be loaded in. Put them under `seeds` directory. You can make additional directories inside `seeds` for clarity.

>-> Tip: you can find documentation on seeds on https://docs.getdbt.com/docs/building-a-dbt-project/seeds


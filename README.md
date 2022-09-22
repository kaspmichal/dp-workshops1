## GID DataOps CLI: Setting up environment and loading data to dwh 

Welcome to the DataOps CLI Labs workshop repository #1. By the end of this tutorial, you will know how to:
- configure and deploy GID DataOps CLI tool as a user-managed notebook in GCP Vertex AI environment
- move around various DataOps tools in JupyterLab environment
- load static seed data to the warehouse with the use of `dp seed`
- create a simple transformation and execute it using `dp run`

Target environment will be Google Cloud Platform's: BigQuery, Vertex AI Managed Notebook, VSCode as IDE. This tutorial was written with GID DataOps 1.0.9 as a current release.


# Exercise
## Setting up environment
1. Go to: https://console.cloud.google.com/welcome?project=datamass-mdp-workshop&supportedpurview=project or to the Vertex AI Dashboard in your project and choose "Workbench" and then "User-managed notebooks"
<p align="center">
<img src="https://user-images.githubusercontent.com/54064594/191755592-58e86b63-3cc2-4392-8c50-3be722ae1d2c.png" height="500" align="center">
   
<img src="https://user-images.githubusercontent.com/54064594/191755932-d96c6cad-7b8e-454e-abcc-50b4af7765f3.png" width="600" align="center">
</p>



#TODO [Screen]

2. Click on New Notebook located in the topbar and then "Customize..."
<p align="center">
   <img src="https://user-images.githubusercontent.com/77925576/165170160-a08af36a-d022-4c5d-b5cd-a181576a6f76.png" align="center">
</p>
3. Type in notebook name (preferrably your first and last name).
4. In environment section, choose Debian 10 and "Custom container"
5. Provide a link to the GID DataOps CLI image: [gcr.io/getindata-images-public/jupyterlab-dataops:bigquery-1.0.9](gcr.io/getindata-images-public/jupyterlab-dataops:bigquery-1.0.9)
<p align="center">
<img width="539" alt="image" src="https://user-images.githubusercontent.com/54064594/191758015-10e4d023-5fe7-4f9c-8fa2-f6818ae20484.png" align="center">
</p>
<!-- <img src="https://user-images.githubusercontent.com/77925576/188915356-19d91e45-4115-40fc-bbd8-a2857993dddc.png" height="500" width="600"> -->

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

5. Make sure you are in `/home/jupyter` directory and then execute command:
`dp init https://github.com/getindata/data-pipelines-cli-init-example`
This will initialize dp-cli tool in the environment. Provide any username when prompted.

>-> Tip: when copy+pasting for the 1st time, you might be asked for permissions to access your clipboard by Chrome. Accept.

6. Run `dp create .`  This command will create a full data-pipelines-cli environment with dbt project as a core part of it.
From options choose 'pipeline-project'
<p align="center">
<img height="100" alt="image" src="https://user-images.githubusercontent.com/54064594/191789935-59c8b12b-a2b0-4f64-ab67-fcd5866fa38c.png" align="center">
</p>

For "gcp_dev_project_id" and "gcp_prod_project_id"  provide the same project id: `datamass-mdp-workshop`. For the rest questions you can answer as follow:
<p align="center">
<img width="460" alt="image" src="https://user-images.githubusercontent.com/54064594/191799731-6399b7df-b254-44fb-9ca8-0c2009834b9b.png">
</p>
IMPORTANT: provide __dataops-test-project__ as a GCP project name. #TODO

>-> Tip: when prompted, you can simply press ENTER to use default values. Don't use it for GCP Project ID!

>-> Tip: use underscores _

>-> Tip: Example of provided values
> 
>![](https://user-images.githubusercontent.com/77925576/165175393-660a9fec-9a07-4179-93bd-abd337f9d285.png)


7. Your environment is now ready to execute some dbt code against BigQuery dwh!

## Loading data to dwh with dbt seed

For this and succeeding exercises we will be using **thelook_ecommerce** [dataset](https://console.cloud.google.com/marketplace/product/bigquery-public-data/thelook-ecommerce?project=gid-dataops-labs). Exercises are meant to slowly build upon data available in the dataset
and organize transformations into staging (bronze), intermediate (silver) and presentation (gold) layers in dwh.

thelook_ecommerce dataset is arranged in 7 tables:  

<img src="https://user-images.githubusercontent.com/77925576/188466518-2e342f7a-6c5b-4c00-b956-bf1cee9093bb.png" height="410" width="500">

It is a dataset which resembles a typical ecommerce shop data warehouse, with events, orders, inventory_items and users facts tables and 2 dim tables: distribution_centers and order_items.
Those tables could've been extracted from different companies' backend applications' databases and collected to a single schema.  

Here, we will want to extend this dataset by "seeding" (loading) additional data into dwh.
This additional data is a static mapping table **mapping_tracking** and was sent to you by someone from the software department
in the form of csv file. This table contains information about software that had been used throughout company's history to track users' behavior across different user sessions and will help prepare a transformation leading to the
final requested dashboard. The mapping is between **browser_name** and **browser** in `events` table ('Chrome', 'IE', 'Safari'..)

To load this data into the warehouse, you will use dbt command called `dbt seed` by executing `dp seed` in the main project directory.
1. You need to set up a .yml file which will serve as a definition of this new table. You need to provide .yml file with the name of the table i.e. mapping_tracking.yml. Put it under `seeds` directory of your dbt project. You can make additional directories inside `seeds` for clarity.
>-> Tip: you can find documentation on seeds with examples at https://docs.getdbt.com/docs/building-a-dbt-project/seeds
2. Create a csv file of the exact same name as the .yml file i.e mapping_tracking.csv and copy+paste the data there from a file inside this repository.
3. Execute `dp seed`
4. You should now have a mapping_tracking table inside your personal working schema.

![Screenshot1](https://user-images.githubusercontent.com/77925576/188682826-085d84ca-c83c-43aa-9e87-c4ec3ce0e3bb.png)

## Basic SQL transformation using dp run

Using our freshly supplied data we can now prepare a basic dbt transformation which will reside
inside the `models` directory of the project. A model is a `SELECT` statement inside .sql file which paired with a definition
from corresponding .yml file together allows to materialize an object (table, view..) inside the dwh.

1. The request for a dashboard was to investigate how many events entries are in the `events` table for every pair 'browser-tracking software' app.
2. The task for you is to finish the query below, insert it into SQL file
3. Create and fill in a .yml file which corresponds to the SQL with column names and description
````
with mapping_tracking_converted_unix_epochs as (
   select 
      id, 
      app_name, 
      browser_name, 
      date_from, 
      case when date_to = 'current' then null else date_to end as date_to 
   from `<workshop_project>.<mapping_schema>.mapping_tracking`
)
select
   distinct(concat(app_name, browser)), 
   timestamp_seconds(date_from) as date_from, 
   case 
      when date_to != 'null' then timestamp_seconds(cast (date_to as int64)) 
      else current_timestamp 
   end as date_to
FROM mapping_tracking_converted_unix_epochs a 
INNER JOIN `<workshop_project>.thelook_ecommerce.events` b on 
   a.browser_name=b.browser;
````
### (Optional) Add generic test and description 
[To do]
### (Optional) Inspect lineage graph
[To do]
## 

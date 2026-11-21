# Overview

The n8n workflow we developed for the project is given below.

![](/pictures/readme_pictures/workflow_before.png)

The following steps give an overview of the workflow.

1. The workflow automatically executes according to a predetermined schedule. It begins with the first search term in the list, and keeps iterating one by one until the end of the list.
2. Job scrapers form the Apify platform are used to automatically scrapes jobs from various job sites, with the relevant information then extracted from these jobs.
3. The jobs are merged into a single list. 
4. The list is filtered for relevancy and duplicates.
5. Each job in the list is given a score based on how well the candidate is suited for the job.
6. The jobs are uploaded to Google Sheets.

# APIs Needed

In order for the workflow to succeed, API keys need to be configured so that n8n can communicate with the required applications. The following lists where the keys need to be supplied from and their reasons. 

- Apify: required to import job scrapers into our workflow
- Adzuna: required to scrape Adzuna jobs
- Google Sheets: required to upload our jobs into Google Sheets
- Anthropic: required to use Claude AI to score the jobs

# Detailed Implementation

In this section we give more detailed information regarding each nodes used in the workflow. Nodes in n8n are the building blocks of a workflow and each one performs a single task. 

## Schedule Trigger

![](/pictures/methodology_pictures/node_1.png)

![](/pictures/methodology_pictures/node_1_details.png)

This is the first node in our workflow and its job is to provide a schedule according to which our workflow will execute. In our implementation we set the workflow to execute every day at 12:00 AM. 

## Input Keywords

![](/pictures/methodology_pictures/node_2.png)

![](/pictures/methodology_pictures/node_2_details.png)

This node contains the search terms that the job scraper uses. Here we use "warehouse" and "data scientist". The user may change this list according to their particular needs. This allows users to search for different types of roles in a single workflow. 

## Loop over Keywords

![](/pictures/methodology_pictures/node_3.png)

![](/pictures/methodology_pictures/node_3_details.png)

This node loops over each search term.  

## Scrape Jobs

![](/pictures/methodology_pictures/node_4.png)

![](/pictures/methodology_pictures/node_4_details.png)

We group these four nodes together since they perform the same role, which is to scrape jobs. As seen from images, the jobs sites that these nodes scrape are Seek, CareerOne, Jora, and Indeed. These nodes are where we need to supply the Apify API key from an Apify account. Once this is supplied, we can then select a particular job scraper (called an "actor") from the account. We can also configure the settings for the job scraper in the "Input JSON" field. 

## HTTP Request

![](/pictures/methodology_pictures/node_5.png)

![](/pictures/methodology_pictures/node_5_details.png)

We use a separate node for scraping Adzuna since it provides a public API, which we take advantage of here.

## Split Out

![](/pictures/methodology_pictures/node_6.png)

![](/pictures/methodology_pictures/node_6_details.png)

The scraper nodes earlier only give us a dataset ID associated with the scraped jobs. To get the actual job listings we need the "Split Out" nodes. These nodes extract the jobs and puts them into a list. 

## Split Out Adzuna

![](/pictures/methodology_pictures/node_7.png)

![](/pictures/methodology_pictures/node_7_details.png)

This node does the same job as the previous nodes, but is technically a different type of node due to the Adzuna scraper node being different to the other scraper nodes. 

## Edit Fields

![](/pictures/methodology_pictures/node_8.png)

![](/pictures/methodology_pictures/node_8_details.png)

The jobs we scraped earlier contains unnecessary information (not used in Google Sheets). The purpose of these "Edit Fields" nodes is to extract only the required information. These are the job title, company, description, and link.

## Merge Jobs

![](/pictures/methodology_pictures/node_9.png)

![](/pictures/methodology_pictures/node_9_details.png)

This node merges all the job data from the various job sites into a single list.

## Filter Relevant Jobs

![](/pictures/methodology_pictures/node_10.png)

![](/pictures/methodology_pictures/node_10_details.png)

This node filters the jobs that are relevant. In the implementation, we deem the job as being relevant if the search term is contained in either the title or the description.

## Remove In-Batch Duplicates

![](/pictures/methodology_pictures/node_11.png)

![](/pictures/methodology_pictures/node_11_details.png)

This node removes duplicates from the list of jobs that were scraped. In the implementation, we deem two jobs as being duplicates if they share the job title and company name.

## Read Jobs in Google Sheet & Remove Duplicates

![](/pictures/methodology_pictures/node_12.png)

![](/pictures/methodology_pictures/node_12_details.png)

![](/pictures/methodology_pictures/node_13.png)

![](/pictures/methodology_pictures/node_13_details.png)

We group these two nodes together since they work together to perform an important task: to remove the scraped jobs that already exist in the Google Sheets spreadsheet. Recall that the previous node removes in-batch duplicates. This means that duplicates within the list of scrpaed jobs are removed, and the output is a list of unique jobs. It does not, however, account for the event where a scraped job may already exist in the spreadsheet. The purpose of these two nodes is to deal with this problem. The first node returns the jobs in the spreadsheet that matches any of the scraped jobs, based on the job title and company (filter 1 and 2 in the implemention photo). The second node takes these two lists as input. That is, input 1 is the list of scraped jobs, and input 2 is the list of jobs in the spreadsheet that matches any of the scraped jobs. To remove the second type of duplicity, and thereby achieve our goal, we simply output the jobs in input 1 that does not match those from input 2. This is done using a Merge node and the details of this can be found in the implementation photo. 

## Score Jobs

![](/pictures/methodology_pictures/node_14.png)

![](/pictures/methodology_pictures/node_14_1_details.png)

![](/pictures/methodology_pictures/node_14_2_details.png)

This node give a score out of 10 to each job based on the candidate's ability to do the job. It uses Claude LLM and requires a Claude API key. To do this, We first give this goal to Claude in the "System Message" field. This is to contorl its role and behaviour. Next we provide the job description and candidate resume in the "Prompt" field. The Claude model will then expend tokens and use the provided information to score the jobs. 

## Parse Scores

![](/pictures/methodology_pictures/node_15.png)

![](/pictures/methodology_pictures/node_15_details.png)

This node reformats the score output form the previous node so that it can be used properly. It just solves a technical issue, nothing more. 

## Edit Fields - Final

![](/pictures/methodology_pictures/node_16.png)

![](/pictures/methodology_pictures/node_16_details.png)

Recall that the earlier "Edit Fields" nodes extracts the job title, company, description, and link. This node simply adds the job score to that list.

## Upload to Google Sheet

![](/pictures/methodology_pictures/node_17.png)

![](/pictures/methodology_pictures/node_17_1_details.png)

![](/pictures/methodology_pictures/node_17_2_details.png)

Finally, this node uplaods the scraped jobs to our Google Sheet spreadsheet. The job title, company, description, link, and score, which were extracted earlier, are uploaded to their respective columns in the spreadsheet. The job deadline and notes are intentionally left blank since they are optional and are meant for manual user entry. The job status is set to pending by default. 






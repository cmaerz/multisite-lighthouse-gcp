# Google Cloud Function for Lighthouse Url Crawling

Run Lighthouse audits on URLs, and write the results daily into a BigQuery table.

Thanks to https://github.com/sahava/multisite-lighthouse-gcp

# Steps (needs rewrite)

1. Clone repo.
2. Run `npm install` in directory.
3. Install [Google Cloud SDK](https://cloud.google.com/sdk/).
4. Authenticate with `gcloud auth login`.
5. Create a new GCP project.
6. Enable Cloud Functions API and BigQuery API.
7. Create a new dataset in BigQuery.
8. Run `gcloud config set project <projectId>` in command line.
9. Edit `config.json`, edit `projectId` to your GCP project ID, edit `datasetId` to the BigQuery dataset ID, specify a API-Key for Security `apiKey`.
10. Run `gcloud functions deploy launchLighthouse --trigger-http --memory 2048 --timeout 540 --runtime=nodejs8`.
11. Run a http request against your function
13. Verify with Cloud Functions logs and a BigQuery query that the performance data ended up in BQ. Might take some time, especially the first run when the BQ table needs to be created.

# How it works

When you deploy the Cloud Function to GCP, it waits for a http Request.

When a message corresponding with a URL defined in `config.json` is registered, the function fires up a lighthouse instance and performs the basic audit on the URL.

This audit is then parsed into a BigQuery schema, and written into a BigQuery table named `report` under the dataset you created.

The BigQuery schema currently only includes items that have a "weight", i.e. those that impact the scores also provided in the audit. 

You can also send the message `all` to the Pub/Sub topic, in which case the Cloud Function self-executes a new function for every URL in the list, starting the lighthouse processes in parallel.

# Problems

The main problem is with the Performance audit. The lighthouse instances aren't meant for heavy lifting with default settings, so they don't necessarily reflect actual performance costs of the site. Some configuration for network conditions needs to be done in the future.

# Cost

This is extremely low-cost. You should basically be able to work with the free tier for a long while, assuming you don't fire the functions dozens of times per day. 

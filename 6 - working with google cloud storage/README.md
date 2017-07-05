### Google Cloud Storage

This is where you can store stuff. Literally stuff. Anything.
Structured data of a CSV, or your Ed Sheeran photo collection, to a website.
Yep, you can make a website out of a google cloud storage bucket. This is just for holding stuff.
 
It's like a USB stick, or like google drive, but on steroids, with more APIs than you can shake a stick at!

Despite that, it's surprisingly difficult to find [this documentation](https://googlecloudplatform.github.io/google-cloud-node/#/docs/google-cloud/v0.55.0/google-cloud), even when you know what you're looking for.

Here's the [Cloud Storage](https://googlecloudplatform.github.io/google-cloud-node/#/docs/google-cloud/v0.55.0/storage) page, definitely give it a whirl, but I'll detail a few things here.

**You need to set up authorisation if your code is running anywhere other than on Google Cloud Platform (GCP).**

### What does this mean for me?
You'll have a nicer time if you make as much use of GCP as you can.

But here's how to DIY.

You'll need a service account set up. Go to the project you intend to use, by logging into [GCP console](http://console.cloud.google.com/),
look under the IAM and Admin section, under service accounts. Make yourself a new one or get the credentials from an existing one.

Download the credentials file. I've renamed mine to `credentials.json` and put it in the folder `volume/credentials`, for a total path of `volume/credentials/credentials.json`. Obviously you can name and put this wherever is your favourite. Always pick your favourite.

It should look something like this. Obviously I've shortened and edited this one.

`credentials.json`
```json
{
  "type": "service_account",
  "project_id": "my-cloud-project",
  "private_key_id": "45...37e61...3f0",
  "private_key": "-----BEGIN PRIVATE KEY-----\nMJIEv ...loads of giberish... 0yAM=\n-----END PRIVATE KEY-----\n",
  "client_email": "your-service-account@my-cloud-project.iam.gserviceaccount.com",
  "client_id": "355..(longNumber)..24",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://accounts.google.com/o/oauth2/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/your-service-account%40my-cloud-project.iam.gserviceaccount.com"
}
```

### Is that it?

Pretty much. Run `npm install @google-cloud/storage` (set up [package.json](../Setting%20Things%20Up/package.json%20and%20installing.md) first). Add `GCLOUD_PROJECT` to your environment variables.

Add the relevant setup in your pipeline or script (pipeline still sounds cooler)

```typescript
const storage = require("@google-cloud/storage");

const gcs = storage({
    projectId: process.env.GCLOUD_PROJECT,
    keyFilename: './volume/credentials/credentials.json'
});
```
And you're good to go.
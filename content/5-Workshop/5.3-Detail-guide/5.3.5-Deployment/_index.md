---
title : "Deploy Game to Web"
date : 2026-08-12
weight : 5
chapter : false
pre : " <b> 5.3.5. </b> "
---

#### Step-by-Step WebGL Game Deployment with Amazon S3 and AWS CloudFront

After completing the Unity Client and Backend API development, the final step is to build the game into **WebGL** format and deploy it onto AWS cloud infrastructure. This allows players to access the game directly from any web browser via a secure HTTPS connection with optimized loading performance.

---

### Phase 1: Build Game to WebGL in Unity

1. Open the project in **Unity Editor**.
2. On the main menu, navigate to **File** → **Build Settings...** (or press `Ctrl + Shift + B`).
3. Select **WebGL** under the **Platform** list:
   - If WebGL is not the active platform, click **Switch Platform** and wait for Unity to re-import assets.
4. Click **Build** (or **Build and Run** to test immediately after building).
5. Create a new folder on your computer (e.g., `WebGL_Build`) and select it as the build destination.
6. **Result:** Once the build completes, the `WebGL_Build` directory will contain:
   - `index.html` (The primary web page launching the game).
   - `Build/` folder (Containing compiled Unity WebGL code and data assets).
   - `TemplateData/` folder (Containing loading progress UI, icons, and layout scripts).

---

### Phase 2: Host Game on Amazon S3 (Static Website Hosting)

Amazon S3 acts as the centralized storage hosting all static files produced by the Unity WebGL build.

1. **Access Amazon S3 Service:** Log in to the [AWS Management Console](https://console.aws.amazon.com/), search for **S3** in the top bar, and select the **S3** service.
   ![Select S3 Service in AWS Console](images/S3/1.jpg)

2. **Create S3 Bucket:** In the Amazon S3 Buckets dashboard, click **Create bucket**.
   ![Create new S3 Bucket](images/S3/2.jpg)

3. **Configure Bucket Name:** Under **Bucket name**, enter a unique name (lowercase, no spaces or special characters, e.g., `ai-test-game`).
   ![Specify Bucket Name](images/S3/3.jpg)

4. **Configure Public Access Settings:**
   - Under **Block Public Access settings for this bucket**, uncheck **Block *all* public access**.
   - Check the acknowledgment box: *"I acknowledge that the current settings might result in this bucket and the objects within becoming public."*
   - Scroll down to the bottom and click **Create bucket**.
   ![Uncheck Block All Public Access and Acknowledge Warning](images/S3/4.jpg)

5. **Enable Static Website Hosting:**
   - Select your newly created bucket from the Buckets list and switch to the **Properties** tab.
     ![Select Properties tab](images/S3/5.jpg)
   - Scroll down to **Static website hosting** and click **Edit**.
     ![Edit Static Website Hosting Settings](images/S3/6.jpg)
   - Select **Enable**.
   - Under **Hosting type**, select **Host a static website**.
   - In the **Index document** field, type `index.html`.
   - Click **Save changes**.
     ![Enable Static Website Hosting and set index.html](images/S3/7.jpg)

6. **Configure Bucket Policy (Public Read Permissions):**
   - Switch to the **Permissions** tab.
     ![Select Permissions tab](images/S3/8.jpg)
   - Locate the **Bucket policy** section and click **Edit**.
     ![Edit Bucket Policy](images/S3/9.jpg)
   - Copy and paste the following JSON policy into the editor (replace `YOUR_BUCKET_NAME` with your actual bucket name from step 3):
     ```json
     {
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Sid": "PublicReadGetObject",
                 "Effect": "Allow",
                 "Principal": "*",
                 "Action": "s3:GetObject",
                 "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME/*"
             }
         ]
     }
     ```
   - Click **Save changes**.
     ![Paste Bucket Policy JSON and Save](images/S3/10.jpg)

7. **Upload WebGL Files to S3:**
   - Verify **Block public access** displays `Off`. Switch to the **Objects** tab.
     ![Verify Public Access Off and switch to Objects tab](images/S3/11.jpg)
   - Click **Upload** and upload all generated build items from Phase 1 (`index.html`, `Build/` folder, `TemplateData/` folder) to the bucket.
     ![Upload WebGL build files to S3](images/S3/12.jpg)

---

### Phase 3: Deliver Game via AWS CloudFront (HTTPS CDN)

Using AWS CloudFront CDN secures the web traffic over HTTPS and accelerates game load speeds globally.

1. **Access CloudFront Service:** On the AWS Console home, search for and select **CloudFront**.
   ![Select CloudFront in AWS Console](images/CloudFront/1.jpg)

2. **Create CloudFront Distribution:** In the Distributions dashboard, click **Create distribution**.
   ![Create new CloudFront Distribution](images/CloudFront/2.jpg)

3. **Choose Service Plan:** In the *Choose a plan* step, select the **Free ($0/month)** tier for development and testing purposes.
   ![Choose Free 0$/month tier](images/CloudFront/3.jpg)

4. **Name Distribution:** In the *Get started* step, enter a descriptive name under **Distribution name** (e.g., `AI-DUNGEON-RPG-GAME`).
   ![Specify Distribution Name](images/CloudFront/4.jpg)

5. **Configure Origin Type:** In the *Specify origin* step, select **Amazon S3** under **Origin type**.
   ![Select Amazon S3 as Origin Type](images/CloudFront/5.jpg)

6. **Connect to S3 Bucket:**
   - Click **Browse S3** next to the *S3 origin* input field.
     ![Click Browse S3 button](images/CloudFront/6.jpg)
   - In the modal dialog, select your WebGL S3 bucket created in Phase 2 (e.g., `ai-dungeon-rpg-game-webgl`) and click **Choose**.
     ![Select S3 Bucket from list](images/CloudFront/7.jpg)

7. **Enforce HTTPS & Deploy Distribution:**
   - Scroll down to **Default cache behavior** → **Viewer protocol policy**, and select **Redirect HTTP to HTTPS**.
   - Scroll to the bottom and click **Create distribution**.
   - The global deployment will take approximately **5 to 10 minutes**. Once the **Status** shows **Enabled**, copy the URL in the **Distribution domain name** column (e.g., `https://d1xxxxxxxxxxxx.cloudfront.net`).
   - Paste the domain name into any web browser to launch and play your game!

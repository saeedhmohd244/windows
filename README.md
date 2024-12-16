How to Deploy a Next.js App on Windows Server 2022 IIS: A Step-by-Step Guide
Rajesh Kumar Yadav
Rajesh Kumar Yadav

·
Follow

5 min read
·
Nov 15, 2024

3



Deploying a Next.js app on Windows Server 2022 might seem like a daunting task, but with the right steps, it’s a smooth and manageable process. In this post, I’ll walk you through the steps I followed to successfully deploy a Next.js app on my Windows Server 2022 environment.

Prerequisites
Before we begin, make sure your system meets the following prerequisites:

A Windows Server 2022 instance (physical or virtual).
Node.js and npm installed.
PM2 installed to manage the application processes.
IIS (Internet Information Services) configured on your server for reverse proxying.
If you don’t have Node.js installed yet, download the latest version from the official Node.js website.

My app name is ‘rky-next’ and domain is rky.app, this tutorial will be around this, for you, you just have to use your app name, your domain and folder path in this guide.

Create an ecosystem.config.js file in the root of your project (C:\Projects\rky-next) with the following content:

module.exports = {
    apps: [
      {
        name: "nextjs-app",
        script: "npm",
        args: "start",
        cwd: "C:\\Projects\\rky-next",
      }
    ]
  };
In above code:

script: "npm" specifies that we want to run npm.
args: "start" passes the start command to npm.
cwd is set to the project directory so PM2 runs from the correct path.
Create a file named server.js in the root of your project (C:\Projects\rky-next) with the following code:

const { exec } = require("child_process");

exec("npm start", (error, stdout, stderr) => {
  if (error) {
    console.error(`Error starting Next.js: ${error.message}`);
    return;
  }
  if (stderr) {
    console.error(`stderr: ${stderr}`);
    return;
  }
  console.log(`stdout: ${stdout}`);
});
Step 1: Install PM2
PM2 is a process manager for Node.js applications. It helps manage your Next.js application in the background and keeps it running even after you close the terminal.

Installing PM2
Open PowerShell as Administrator.
Run the following command to install PM2 globally:
npm install -g pm2
Once PM2 is installed, you can verify its installation by running:

pm2 -v

If you don’t have a Next.js app ready, you can create one by running the following commands:

Create a new Next.js app:
npx create-next-app@latest my-next-app
cd my-next-app
Install dependencies:

npm install
Alternatively, if you already have a Next.js app, just ensure it’s placed in the desired directory (e.g., C:\Projects\rky-next).

Step 3: Build the Next.js Application
Navigate to your project directory in PowerShell:
cd C:\Projects\rky-next
Build the application:

npm run build
This step compiles your Next.js app into an optimized version for production.

Step 4: Start the Application with PM2
To start your Next.js app using PM2, run the following command:

pm2 start npm --name "nextjs-app" -- run start
Or

pm2 start server.js --name "nextjs-app"

If the above command fails due to the script not being found, you can directly use PM2 to start the next server with:

pm2 start node_modules/next/dist/bin/next --name "nextjs-app" -- start
This will start the Next.js application on port 3000 by default.

Step 5: Configure IIS as a Reverse Proxy
To make your app accessible via your domain (e.g., rky.app), we need to configure IIS (Internet Information Services) to forward requests to the Next.js app running on port 3000.

1. Install IIS URL Rewrite Module
Install URL Rewrite: Go to URL Rewrite Module for IIS and download/install it.
Install Application Request Routing (ARR): Download and install Application Request Routing (ARR). This module allows IIS to act as a reverse proxy.
2. Enable Application Request Routing (ARR)
Open IIS Manager and select the server node.
In the Features View, double-click Application Request Routing.
Click Server Proxy Settings in the right-hand pane and check Enable Proxy. Click Apply.
3. Set Up URL Rewrite Rules
In IIS Manager, go to your site, rky.app.
Open URL Rewrite and add a Blank Rule:
Name: NextJsProxy
Match URL:
Requested URL: Matches the Pattern
Using: Regular Expressions
Pattern: .*
Action:
Action Type: Rewrite
Rewrite URL: http://localhost:3000/{R:0}
Ensure “Append query string” is checked.
Click Apply to save the changes.

This rule ensures that all requests to rky.app are forwarded to your Next.js application.

4. Restart IIS
Once you’ve set up the reverse proxy, restart IIS to apply all changes. Open PowerShell and run:

iisreset
Step 6: Test the Deployment
Open your browser and visit http://localhost:3000 and http://rky.app.

If you have not added the DNS records, then add @ and www two records with your server IP and wait for 2 min to 24 hours for DNS propagation.
Your Next.js app should now be live, running on Windows Server 2022!
Step 7: Optional — Set Up SSL (HTTPS)
To secure your domain with SSL, you can configure an SSL certificate in IIS and enable HTTPS:

Obtain an SSL certificate for rky.app (e.g., via Let's Encrypt or a commercial provider).
In IIS Manager, go to the site settings for rky.app, click on Bindings, and add an HTTPS binding using the SSL certificate.
Alternatively, you can visit win-acme and download this software, run it as administrator and install the SSL automatically to IIS

Conclusion
Deploying a Next.js app on Windows Server 2022 can be done relatively easily by following these steps. You’ll have a robust environment with PM2 managing your app and IIS serving as a reverse proxy to route traffic to your Next.js app. This setup provides a reliable, production-ready environment for hosting web applications.

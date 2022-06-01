# How to create a react app and deploy it to Azure via Source Control

The steps below outline the procedure to create a react app on your machine, push it to Azure DevOps and create a CI/CD workflow using pipeline/release to deploy the webapp in an Azure App Services.

## 1. Prerequesites

### npm / Node.js

- To verify
```
node -v
npm -v
```

- To update
`npm install -g npm`

- To install
Follow guidelines on [this page](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm).

### git

- To verify
`git version`

- To install
Follow guidelines on [this page](https://github.com/git-guides/install-git#install-git-on-windows).

### Code editor

*Visual Studio Code* is preferred. You can install directly via the Windows store or through [this page](https://code.visualstudio.com/Download).

---

## 2. Create a project

Select a path on your machine to host your project and create your project.

Please note the project name should be all lowercase.

```
cd C:\Users\abiencourt\OneDrive - Microsoft\Documents\Workspace\
npx create-react-app react-cicd
```

---

## 3. Start your app

To run your application, navigate in your newly created folder and start it.

```
cd react-cicd
npm start
```

---

## 4. Push to Azure DevOps

Now that we are happy with our app, let's create an Azure DevOps project and push our app to its repo.

1. Head to [dev.azure.com](https://dev.azure.com/)
2. Select your organization or create one (I recommend setting one with your alias (e.g. abiencourt))
3. Create a **private** project in this organization
4. Browse to the *repo* section of the project (e.g. https://dev.azure.com/abiencourt/_git/react-cicd) where you will find *git commands* to **Push an existing repository from command line**
```
git remote add origin https://abiencourt@dev.azure.com/abiencourt/react-cicd/_git/react-cicd
git push -u origin --all
```

---

## 5. Create a pipeline

1. Navigate in your project to the **Pipelines** section (e.g. https://dev.azure.com/abiencourt/react-cicd/_build)
2. Click on `Create Pipeline`
3. Select `Azure Repos Git`
4. Select your project
5. Select starter pipeline (we already have a pipeline ready)
6. Copy/Paste [the content of this yaml file](./build.yml) into the code editor
7. Edit the name of the file (by default `azure-pipelines.yml`) to `build.yml`
7. Click `Save and run`

Wait a few moments and if everything went well your build should be successful.

---

## 6. Create an App Service

### Automated

1. Copy the content of the [azuredeploy-appservice.json](./azuredeploy-appservice.json)
2. Click on the button below
3. Select **build your own template**
4. Paste the *azuredeploy-appservice.json* template
5. Click **Save**
6. Edit the details to match your naming scheme
7. Click **Create**


[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://ms.portal.azure.com/#create/Microsoft.Template)


### Manually

1. Go to your Azure Portal
2. Give your App Service name
3. Select **Publish as Code**
4. Select **Runtime stack Node 16 LTS**
5. Select **Operating System Windows**
6. Under Monitoring you can disable **Application Insights**
7. Rest is up to you or leave as default
8. Click `Review + create` and `create`
9. When deployed click on `Go to resource`
10. Navigate to **Deployment Slots**
11. Select **Add Slot**
12. Enter the name `dev` and click `add`

---

## 7. Create a release to push to Azure App Services

### 1. Configure the dev stage

1. Head to Releases, under Pipelines (e.g. https://dev.azure.com/abiencourt/react-cicd/_release)
2. Click `New Pipeline`
3. Select `Azure App Service deployment`
4. Rename `Stage 1` to `Dev`
5. Select `1 job, 1 task`
6. In the drop down under Azure subscription Select your AIA and **Authorize**
7. Under **App service name** select the App service created on step 6
8. Select the task **Deploy Azure App Service**
9. Tick the box **Deploy to Slot or App Service Environment** and select the slot `dev`
10. Under **Package or folder** replace the location by `$(System.DefaultWorkingDirectory)/_build/drop`

### 2. Configure the pipeline

1. After configuring the Dev stage, click on Pipeline on the top left (on the left of **Tasks**)
2. Click on **Add an artifact**
3. In the **Source** select your pipeline created on step 5
4. Click **Add**
5. Click on the lightning icon and enable **Continuous deployment trigger**

### 3. Configure the prod stage

1. Highlight the **Dev** stage and slect clone
2. Click on **Copy of Dev** and rename it to `Prod`
3. Click on 1 job, 1 task > Deploy Azure App Service
4. Untick **Deploy to Slot or App Service Environment**

### 4. Release is ready

1. Rename the release if you wish (e.g. `Release to Azure`)
2. Click on **save**
3. Leave default location
4. As we previously ran our pipeline (build) successfully, we can click on the top right **Create release** and click `Create`
5. Monitor your release and observe your slots being deployed
**git checkout <commit\_id>** : lets you checkout to a particular version of the code by going to the commit

**git log** : displays the commit ids and the commit messages and where the HEAD is currently being pointed to

  
**Note:** Once checked-out to a previous commit and after you run git log, it'll only show the commits that had occurred before **checked-out commit**, it gets **branched out** from the **main branch**. And the **HEAD** will point to the checked-out commit.   
Also note, that the main branch will not be displayed after running git checkout command because git has branched it to a different version   
For you to be able to go back to the branch with all the commits that you've made previously, just checkout to the branch. For example: **git checkout <branch> (main)**

---

Suppose, you've made a change in the code that you no longer need. To get this sorted, you can remove the added code and commit again but there's a better way to do it.

**git revert <commit\_id>** : Git revert let's you revert the changes that were made in a commit and creates another commit on top of that. 

If you had 2 commits previously and you want to revert the changes made in the second commit, git then reverts the changes but creates a 3rd commit

---

**git reset --hard <id>** : Undos changes by deleting all commits since <id>. Basically getting rid of all the commits after that commit and by doing that you can never get back the code and you'll be rewriting history

---

**git branch** <**branch\_name**\>: Lets you create a new branch. The new branch will have the files from the commit the branch was created.   
In short, when ran git branch, git tries to create a new branch with all the existing codes as it is into a different branch. This will not conflict the main branch and will be running as in like a parallel world. 

At some point in future, you can even merge the branches together, the main branch and the newly created branch and git will try to merge them without disturbing either of the codes from the two branches.

In case, a conflict arises between the two merges, then it needs to be sorted out before the merge is complete.

---

To push the code to the github repository, we first need to be authenticated.

to push code to github repo, you first need to add the local repo as the remote repo by passing the name to the local repo: **git remote add <any\_local\_repo> (origin) <repo\_link>**

where origin is the local repo name & repo link is the repo link

after adding the remote repo, **git push** still won't work because we are not authenticated yet

---

to push, we need to pass: **git push remote origin** but we'd still get permission denied

to authenticate, pass: **git remote set-url origin <repo\_link>** but need to add your username before github.com as **repo link: https://<github\_username>@github.com/repo\_link.git**

After that step, you can try to run: **git push origin main** command and this will prompt for a password in the terminal.

For password, go to your **github account settings > developer settings > personal access token > select repo access scope > generate a token** and pass it in the password

---

when run simply **git push** this will fail saying that there's no branch called as upstream branch

to get past this, you need to pass: **git push --set-upstream origin main**

**Note:** This command is needed to configure a strict connection between your remote repository and it's main branch and your local main branch

**origin** being the **local repo name** & **main** being the **branch** **name**

After that you can simply use **git push** command to push to your remote repo

---

when using github repo and you create a new branch in the local and make some changes, you need to push it to the remote repository by passing: **git push origin <new\_branch\_name>**

from the github console, you can see 2 branches now and also a pop-up that say, **compare and pull request**

You can now merge the changes to the main branch by passing **git merge <new\_branch\_name>** and then pushing it to remote repo by passing: **git push**

---

When you can create a file from Github repo directly and commit it to main branch but that newly added file **won't magically appear in your local repo**.

This will **create a conflict**, if in **future**, you **commit** to the **remote repo** **without** **adding the file locally**, you'll actually be **deleting the file** since the **local doesn't match with remote repo**, just like terraform.

To fix this, we need some way to **download the remote repo files to our local repo**

**git pull :** this pulls the code from github and applies to your local repo

This is also useful if a different team member has updated the code and you want to pull those into your local repo so that you can start working from that commit or snapshot of code

---

when cloning a git repo: **git clone <clone\_repo\_url> <folder\_name>**

When passed the **folder\_name**, all the code will be copied under the folder name and if we do not pass it, it'll get copied under the same name as the repo name

---

after cloning the remote repository, if you pass: **git remote** you can see the list of all connected repositories

if you pass in: **git remote get-url origin** (origin being the name of the repository) you can see the url of the remote repository

---

GitHub actions consists of 3 things: **Workflows, Jobs, Steps**

1. **Workflows** \- Workflows are attached to a **GitHub repository** & can have as many **workflows** as needed. Workflows contains **one or more jobs** and are **triggered** upon **Events**
2. **Jobs** \- Can have as many jobs as needed in a workflow.  
   1. Every job defines a **runner**, which is an execution env (e.g. linux, windows, macOS).  
   2. Could be **predefined** by **GitHub** or can have custom **runners.** Jobs contain one or more **Steps**.  
   3. You can run these jobs in **parallel** or in a **sequential order**  
   4. You can also **configure** to **run jobs** on a **condition**

---

1. **Steps** \- These are the actual stepping ladder where the actual work happens, this is where you define what needs to be done.  
   1. Can contain as many steps as needed in a **Job** (e.g. executing a shell script or an action)  
   2. **Actions** in the context of GitHub actions are predefined scripts that perform a certain task  
   3. You can use the **predefined actions** in your **Steps** or can use **custom** or **third party actions**  
   4. These **Steps** are then **executed in order** and they could be **conditional**

---

**_NOTE:_** In **public repositories**, you can use GitHub Actions for **free**. For **private repositories**, **only a certain amount of monthly usage is available for free** \- extra usage on top must be paid.

---

GitHub action workflows and all of the files are stored in the same repository in the **/.github/workflows/** folder

A workflow consists of some reserved parameters, below is a sample workflow for understanding: 

```html
name: first-workflow    #reserved
on: workflow_dispatch   #reserved
jobs:                   #reserved, needs to jobs and not job
  first-job:            #custom-job name
    runs-on: ubuntu-latest      #reserved
    steps:                      #reserved
      - name: Hello World step   
        run: echo "Hello, World!"
      - name: Bye Bye step
        run: echo "Bye, bye!"
```

---

The below example was to show how to run commands in a sequential order, if you need to run multiple shell commands (or multi-line commands, e.g., for readability), you can easily do so by adding the pipe symbol (`|`) as a value after the `run:` key.

Like this:

```html
      - name: Multi-line step
        run: |
          echo "Hello, World!"
          echo "Bye, Bye!"
```

This will run both commands in one step.

---

What is an **Action** in the context of GitHub Actions? 

* An Action is a (custom) application that performs a (**typically complex**) **frequently repeated task**
* You can build your **own Actions** and you can also use **official** or **community Actions**

---

You can pass, **working\_directory** parameter if your application code lives in path other than root directory to build and run tests

---

By default, all jobs in a workflow run parallelly. If you want jobs to run in a sequential order, on a condition if one job is successful, other job needs to be run, we need to add a parameter in the second job, i.e., **needs: <job\_name> . needs parameter can also be a list**

```html
    testing:
        runs-on: ubuntu-latest
        steps:
            - name: Checkout Repository
              uses: actions/checkout@v3
    deployment:
        needs: testing # this parameter tells to run deployment, only if test succeeds
        runs-on: ubuntu-latest
```

---

Jobs can be triggered on multiple events, such as **_push, workflow\_dispatch._** 

Note that **on: parameter is a list** and it can accept multiple triggers

For each event, you can configure them to run on a much granular level, for example, **push:** 

```html
on:
  push:
    branches:
      - 'releases/**'
```

The above workflow is **triggered** on **push event** on the **releases branch.**

---

Workflows also accepts **contexts** & **expressions** in the jobs. These contexts carry the metadata regarding that workflow and about the github. 

The **contexts** are presented in the workflow as an **expression**. 

An expression is passed in this way: **${{ .. }}.** An example for the expression is:   
**echo "${{ toJSON(github) }}"**

---

example context & expressions workflow:

```html
name: Output Workflow
on: workflow_dispatch
jobs: 
    output:
        runs-on: ubuntu-slim
        steps:
            - name: Output Message.
              run: echo "${{ toJSON(github.repository_owner) }}" # specifically printing repository owner for a single value
```

---

A workflow can be triggered on events and we can further filter out certain activities rather than as a whole **(push or pull\_request).**

There are two types of events, **Activity Types** and **Filters.** Both offer more detailed control over when a workflow will be triggered. Some are event based and some are based on the **filters.**

For example: **pull\_request** event has **opened, closed or edited** events

And **push** event is a **filter** based on event where we can **filter** on the defined **branch** the **push occurred.**

---

Example event type trigger:

```html
name: Event Types Workflow
on: 
  pull_request:
    types: [opened, created, reopened]
  workflow_dispatch:
jobs:
```

---

**Fork Pull Request Workflows:** Previously, we had configured the workflow to be triggered **on pull requests**, but when **forked** and created a **pull request** by an anonymous person, by default, **Pull Requests** based on **Forks do NOT trigger** a **workflow.** Because:

1. Everyone can fork your repository and open a pull requests
2. Malicious workflow runs, anyone could spam with **PRs** and that will in return cause your workflow to be triggered which can create unnecessary runs and excess costs

**Note:** First-time contributors must be **approved manually**. If you already added a **collaborator**, then the workflow will be **triggered automatically**.

---

Cancelling & Skipping Workflow Runs

**Cancelling**: You can **cancel** or **stop** a workflow in the middle. This by default happens if:

1. **Workflow** gets **cancelled** if one of the **Jobs** fail
2. **Job fails** if at least one of the **Step fails**
3. You can also **cancel** **workflows manually**

**Skipping Workflows**: To **skip** or to not trigger a workflow on any event, you need to **append a special command** in the commit message. e.g., **git commit -m "commit message \[skip ci\]"**

Other similar commands are: `[skip ci]`

* `[ci skip]`
* `[no ci]`
* `[skip actions]`
* `[actions skip]`

---

**What is Job Artifact:** A job artifact are the outputs generated by Jobs. Like in codepipeline, we provide artifact to be passed down to another job. 

**For example**: **Artifact** produced by Testing job **\--> Build Job** to build **\--> Deploy Job** to be deployed.

**Another example**: A **log file** that gets generated as an **output** is also an **artifact**

---

If your repository contains **multiple applications** (for example: `project-01/`, `project-02/`, `project-03/`), make sure each job or step runs commands inside the **correct subdirectory** using the `working-directory` option (or by setting it once under `defaults.run.working-directory`).

```html
  build:
    needs: testing
    runs-on: ubuntu-latest
    defaults:
      run:
        shell: bash
        working-directory: ./starting-project-03
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v3
      - name: Upload Artifact
        uses: actions/upload-artifact@v4
        with:
          name: project-03-artifact
          path: |
            starting-project-03/dist
            starting-project-03package.json
```

---

**Download Artifact**: We can also automatically download the artifact or pass it to a different job in the workflow to be further build / deploy. 

**Note:** if the project is in a specified directory, then you need to pass the path in **ls** command to output its contents

```html
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Download Artifact
        uses: actions/download-artifact@v4
        with:
          name: project-03-artifact
      - name: List Files of Project 03 artifact
        run: ls -la starting-project-03/
      - name: Deploy Website
        run: echo "Deploying Artifact..."
```

---

**Outputs**: We don't just deal with **Artifacts** which outputs files or folders. Typically used for sharing log files, app binaries. etc. **Artifacts** and **Job Outputs** are 2 different things.

**Job Outputs**: **Job outputs** could also be **simple values**, like a **key, value** pair, that could be used in a **different job**. Typically used for reusing a value in different jobs.  
**Example:** Used for _passing metadata or small values_ (like version numbers, filenames, commit IDs, URLs, etc.) between jobs

**Note**: It's not always the file(artifact) that is necessary, but also the name of the file (**job output simple value**)

---

For example: If you want to output a filename to a different job in the workflow, you need to first create an outputs parameter under the job. 

Then add the step to Publish the filename with an id to be referenced in the outputs parameter with the command below: **find project-03/dist/assets/\*.js -type f -execdir echo 'file={}' >> $GITHUB\_OUTPUT ';'**

a sample workflow would look like this: 

```html
  build:
    needs: testing
    runs-on: ubuntu-latest
    outputs:
      output_file: ${{ steps.output_file_step.outputs.file }} #important
    steps:
      - name: Job Output
        id: output_file_step
        run: find dist/assets/*.js -type f -execdir echo 'file={}' >> $GITHUB_OUTPUT ';'
  deploy:
    needs: build #important
    runs-on: ubuntu-latest
    steps:
      - name: Output Jobs
        run: echo "The output file is ${{ needs.build.outputs.output_file }}"
```

---

**Note**: The jobs should be run in a **sequential** **order** so that the **job output** is then **made available** for the **subsequent jobs**. 

---

**Caching Dependencies**: For the steps that are repetitive in each job and take longer time, we can use caching to reduce the time taken to run the workflow. 

---

To add **Caching** in your workflow, you need to **add it before the job.step.** you want to cache. 

**For example**, if you want to **cache dependencies (npm ci)**, then you need to **add caching step before** this step.

**Cached Files** are not just stored in that particular job, after a job is **successfully ran**, it will **cache** those files and **keep it with GH-Actions creating a hashFile** that is **unique** for each file. It's then re-usable in different job or workflow

**hashFiles**: hashFiles id is generated based on the content and path in a machine. the hashFiles hash changes when the content inside changes. 

---

The same **cachedfile**, in our case, **package-lock.json** if not being changed then it will be **reused** in the upcoming **builds**.

---

**Artifacts**: Jobs often product assets that should be shared or analyzed. 

* Examples: Deployable website files, logs, binaries, etc.
* These assets are referred to as "Artifacts" (or "Job Artifacts")
* GitHub Actions provides Actions for uploading & downloading artifcats

---

**Artifacts**: Jobs often **produces** assets that should be **shared or analyzed**. 

* **Examples**: Deployable **website files, logs, binaries**, etc.
* These assets are referred to as "**Artifacts**" (or "**Job Artifacts**")
* GitHub Actions provides **Actions** for **uploading** & **downloading** artifacts

---

**Outputs**: Besides Artifacts, Steps can product & share simple values, like the file\_name, or the commit\_id

* These outputs are shared via **::set-output**  
**(**Should use **echo '{output-name}="{output-value}' >> $GITHUB\_OUTPUT**
* Jobs can pickup & share step outputs via the **steps context**
* Other Jobs can use Job Outputs via the **needs** context

---

**Caching**: **Caching** can help **speed** up **repeated**, slow Steps 

* **Typical use-case**: Caching **dependencies** but any **files & folders** can be **cached**
* The cache Action **automatically stores & updates cache values** (based on the cache key)
* **Cache key** changes if the **contents** of the **cached file changes**, it then **recreates** a new cache-key
* **Never use Cache action for Artifacts**

---

**Environment Variables**: Just like any tool uses env variables, you could also configure variables to be used in the workflow at **workflow level(global), job level, steps level.**

---

**env** variables can be used in two ways, in with **":"** and **"="**. Example below

```html
jobs:
  test:
    env: 
      MONGODB_USERNAME: kyouma
      MONGODB_PASSWORD: pRQtfIvRkhmHfLa3
      PORT: 8080
        run: npx wait-on http://127.0.0.1:${{ env.PORT }} #like this
      - name: Output information
        run: echo 'MONGODB_USERNAME=${{ env.MONGODB_USERNAME }}' #and this
```

  
---

**Secrets**: Some env variables should never be exposed, like the **DB\_PASSWORD** variable

* Use **Secrets together environment variables**

---

The **Env variables & the Secrets** can be stored directly in the GitHub repo settings > **Environments > Create environment** and by adding **environment variables & secret** respectively. 

They can be fetched by using **${{ vars.ENV\_NAME }}** and **${{ secrets.SECRET\_NAME }}**

```html
jobs:
  deploy:
    environment: testing #env where info is stored
    runs-on: ubuntu-latest
    steps:
      - name: Output information
        run: |        
          echo "The database name is ${{ vars.MONGODB_DB_NAME }}."
          echo "The username is ${{ secrets.MONGODB_USERNAME }}."
          echo "The password is ${{ secrets.MONGODB_PASSWORD }}."
```

---

**Environments in GitHub Action**: Environments in GitHub provides a isolated environment to store environment variables & secrets. 

* They also allow you to setup extra protection rules (ex: env or secrets should only be accessed if pushed to a particular branch)

---

**Controlling Workflow Jobs & Steps Execution**

* We can run jobs and steps on a conditional execution via **if** field
* We can also use **expressions** as conditional execution

By default, **if** a **step fails** then the **subsequent steps** are **not executed** and **exits the job**.

* We can configure steps to Ignore Errors via **continue-on-error** field

---

A key note about referencing S**teps** or **Jobs** in different job or step**.**

* For jobs, the **id** is the name under the **job**:
* For the **Steps**, you can add a field for **id** that will be used as reference in other jobs or steps.

```html
jobs:
  build_job: # Job ID
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        id: caching # Step ID
        uses: actions/checkout@v4
```

---

To fix that we need to use a special condition function **failure() && steps.run-tests.outcome == 'failure'**

```html
test:
  runs-on: ubuntu-latest
  steps:
    - name: Test code
      id: test-case
      run: npm run test # Test fails here
    - name: Upload test report # This will still be executed because of function()
      if: failure() && steps.test-case.outcome == 'failure'
      uses: actions/upload-artifact@v4
      with:
        name: test-report
        path: test.json
```

---

**Status check functions:**

Use the status check functions as expressions in `if` conditionals. A default status check of `success()` is applied unless you include one of these functions

Below are the status check functions available:

* `success()` Returns **true** when **all previous steps** have **succeeded**
* `always()` Causes the step to **always** **execute**, and returns **true**, even when **cancelled**
* `cancelled()` Returns **true** if the **workflow** was **cancelled**
* `failure()` Returns **true** when **any previous step** or a job **fails**. If you have a **chain of dependent jobs**, failure() returns **true** if any **ancestor job fails**

---

Use cases for each functions

* **success()** : build only if tests passed
* **always()** : To send logs even when a job is cancelled

You can use **failure()** function with additional conditions as shown below.

```html
if: ${{ failure() && steps.test.conclusion == 'failure' }}
```

---

We can combine if and failure() conditions on job-level. Suppose, if any of the job fails, then this job **OnFailure** should be executed.

```html
output:
  runs-on: ubuntu-latest
  needs: [lint, deploy]
  if: failure()
  steps:
    - name: Output message
      run: |
        echo "Something went wrong. One of the jobs has failed."
```

---

**continue-on-error**:

* The `continue-on-error` keyword allows you to control how your workflow behaves when a step or job encounters an error.
* By default, a step or job failure will immediately stop the current job and mark the workflow as failed. However, `continue-on-error` can be used to prevent this from happening
* You can apply `continue-on-error` at **job\_level** & **steps\_level**

---

`continue-on-error` at **job\_level:**

* Setting `continue-on-error: true` at the job level will allow the entire workflow to continue running even if **any step** **within** that **job fails**.
* The job itself will be marked as "**neutral**" or "**succeeded**" in the workflow summary, depending on the overall outcome, but the **individual failing steps will still be indicated**

`continue-on-error` at **step\_level**: 

* Setting `continue-on-error: true` on a specific step will allow the job to continue running even if that **particular step fails**.
* The step will be marked as **failed**, but **subsequent steps** within the same job will still execute.

---

**Matrix Strategy:** 

* With matrix you can run **variations** of a **job** in a **workflow**.
* A matrix strategy lets you use **variables** in a **single job** definition to **automatically** create **multiple job runs** that are based on the **combinations of the variables**.
* For example, you can use a matrix strategy to **test your code** in **multiple versions** of a **language** or on **multiple OS**

```html
jobs:
  matrix-job:
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest]
        node-version: [18, 20, 24]
    runs-on: ${{ matrix.os }}
    steps:
      - name: Setup NodeJS
        uses: actions/setup-node@v5
        with:
          node-version: ${{ matrix.node-version }}
```

---

**Matrix** also lets us allow **include & exclude flags** to either **include or exclude** a **set** of **combinations** to be used. For example: 

```html
jobs:
  matrix-job:
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest]
        node-version: [18, 20]
        include:  
          - os: ubuntu-latest  # This combo will be added to the existing matrix
            node-version: 24
        exlude: 
          - os: macos-latest   # This combo will be removed from the existing matrix
            node-version: 20
```

We could directly **include/exclude** them in the existing lists but it will **iterate** over **all values** and that's not what we want to do

---

**Reusable Workflow:** 

* Reusable workflows are very similar to any other workflow file.
* As with other workflow files, you locate reusable workflows in the `.github/workflows` directory of a repository.
* For a workflow to be reusable, the values for `on` must include `workflow_call`

```html
name: Workflow Call  # workflow.yml
on: workflow_call
jobs:
  reusable-workflow:
    runs-on: ubuntu-latest
    steps:
      - name: Download Artifact
        uses: actions/download-artifact@v4
        with: 
          name: dist-files
      - name: Output Hello
        run: echo "Hello"
```

```html
  deploy:         # another workflow.yml
    uses: ./.github/workflows/workflow.yml
    needs: build
```

---

You can also provide inputs to a reusable workflow making it more dynamic. Lets say you create a re-usable workflow that downloads the artifact and deploy to any cloud. This artifact name changes and we can't hardcode it in it the workflow, hence we provide an input.

```html
name: Workflow Call
on: 
  workflow_call:
    inputs: 
      artifact-name:
        required: false
        description: 'Name of the artifact to download'
        default: dist-files # defaults to this value
        type: string
jobs:
  reusable-workflow:
    runs-on: ubuntu-latest
    steps:
      - name: Get code
        uses: actions/download-artifact@v4
        with: 
          name: ${{ inputs.artifact-name }}
```

```html
  deploy:
    uses: ./.github/workflows/workflow.yml
    needs: build
    with:
      artifact-name: dist-files # overrides the default value
```

---

secrets can also be provided to a reusable workflow going by the same requirement that it needs to be used by many other workflows.

```html
name: Workflow Call  # the environment needs to be provided in the workflow_call flow
on:                  # for secrets / env vars to be picked up
  workflow_call:  
    secrets:
      some-secret:
        required: true
        description: 'A secret for deployment'
```

```html
  deploy:
    uses: ./.github/workflows/workflow.yml
    needs: build
    with:
      artifact-name: dist-files
    secrets: 
      some-secret: ${{ secrets.SUPER_SECRET }} 
```

---

We can also use Outputs in the workflow\_call to output a value. This output is then used in a different workflow.

```html
name: Workflow Call
on: 
  workflow_call:
    outputs:
      result:
        description: 'Deployment status'
        value: ${{ jobs.reusable-workflow.outputs.status}}
jobs:
  reusable-workflow:
    outputs:
      status: ${{ steps.set-output.outputs.deployment-status }}
    steps:
      - name: Set Output
        id: set-output
        run: echo "deployment-status=success" >> $GITHUB_OUTPUT
```

```html
  deploy:
    uses: ./.github/workflows/workflow.yml
    needs: build
    with:
      artifact-name: dist-files
 
  print-output:
    needs: deploy
    runs-on: ubuntu-latest
    steps:
      - name: Printing Deployment Status
        run: echo ${{ needs.deploy.outputs.result }}
```

---

**Containers in GitHub Actions**

You can run your jobs on a custom container(environment & installed software).   
The Docker container packages that contain code + its execution environment.

* Advantage: Reproducible execution environment & results.
* These Docker containers still runs on GitHub Actions Runners (ubuntu-latest, macOS-latest).
* Your containerized Job is hosted by the runner
* Steps execute inside the container
* You can also create Services: Utility containers used by your Steps (Example: Testing Database)

---

container example:

```html
jobs:
  test:
    environment: testing
    runs-on: ubuntu-latest
    container: 
      image: node:25
      env: 
        SOME_ENV: ${{ vars.SOME_ENV }}
    env:
      MONGODB_CONNECTION_PROTOCOL: mongodb+srv
```

---

**Service Containers:** Suppose you have a testing job that tests a database but you do not want to manipulate the production database. So, for testing jobs, you can run a testing database to which you can run your tests against.

* However, running a database just for testing purpose will incur charges and you have to maintain that somewhere.
* Instead you could host a testing database inside a container hosted by the runner(ubuntu-latest)
* These testing databases are ran and terminated only when the workflows are running.
* Job steps can communicate with service containers (and the services exposed by them)

---

To run an image inside a job, you need to specify **services** parameter under **jobs.** You need to define the configuration as provided in the docker compose file. For example:

```html
name: Deployment (Container)
on:
  push:
jobs:
  test:
    environment: testing
    runs-on: ubuntu-latest
    env:
      MONGODB_CONNECTION_PROTOCOL: mongodb
      MONGODB_CLUSTER_ADDRESS: 127.0.0.1:27017
      MONGODB_USERNAME: root_user
      MONGODB_PASSWORD: root_password
      PORT: 8080
    services:
      mongodb:
        image: mongo:6.0
        ports:
          - 27017:27017
        env: 
          MONGO_INITDB_ROOT_USERNAME: root_user
          MONGO_INITDB_ROOT_PASSWORD: root_password
```

  
---

When running the job inside a container, you can reference the cluster address with it's service name, i.e., mongodb

```html
name: Deployment (Container)
on:
  push:
jobs:
  test:
    environment: testing
    runs-on: ubuntu-latest
    container:  
      image: node:25  # within mongodb container
    env:
      MONGODB_CONNECTION_PROTOCOL: mongodb
      MONGODB_CLUSTER_ADDRESS: mongodb
      MONGODB_USERNAME: root_user
      MONGODB_PASSWORD: root_password
      PORT: 8080
    services:
      mongodb:
        image: mongo:6.0
        ports:
          - 27017:27017
        env: 
          MONGO_INITDB_ROOT_USERNAME: root_user
          MONGO_INITDB_ROOT_PASSWORD: root_password
```

---

**Custom Actions**

* Instead of writing multiple possible very complex step definitions, you can build and use a single custom action
* You can also group multiple steps into a single custom action. For example: Caching, Installing dependencies, Running Tests - we can group these multiple steps into a single custom action
* **No Existing (Community Action)** : Another reason could be that there's isn't an Action to use or has been created to use. Or existing public actions might not solve the specific problem you have in your workflow.
* Custom Actins can contain any logic you need to solve your specific Workflow Problems and can publish it in the market place.

---

There are **three** different type of **Custom Actions**

1. **JavaScript Actions**  
   * These are written using **JavaScript (NodeJS)** \+ any packages of your chose  
   * Whenever this action is used, it **executes** a JavaScript file  
   * Pretty easy to setup if you know JavaScript
2. Docker Actions  
   * It's basically a containerized action created using Dockerfile with your required configuration  
   * Perform any tasks of your with any language  
   * Has a lot of flexibility but requires Docker knowledge
3. Composite Actions  
   * You don't really write any code with any programming language, instead use multiple community available actions and club them together to be used as a single custom action  
   * It can combine multiple workflow steps in one single action. It can also combine run (commands) and uses (Actions)  
   * It allows for reusing shared Steps (without extra skills)

---

* Custom Actions are stored inside the **.github/actions/** directory, though the name could be anything but it makes sense to name it as **actions**.
* When stored in the **same repo** as your **workflow**, that action is only available for any **workflows running in that repo**
* To make an action be **available** to use for any other workflows, you need to create a **standalone repo** that holds your defined actions
* Each custom action has it's own separate directory in the **.github/actions/your-action-name/** directory.
* Each custom action directory must have **action.yaml** file present which **defines the action** or you can say **configures the action**

---

Example of Composite custom-action

```html
name: Cache & Install Dependencies
description: Caches and installs project dependencies (npm)
runs:
  using: 'composite'
  steps: 
    - name: Cache dependencies
      id: cache
      uses: actions/cache@v3
      with:
        path: node_modules
        key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
    - name: Install dependencies
      if: steps.cache.outputs.cache-hit != 'true'
      run: npm ci
      shell: bash
```

---

To use a composite action in a workflow, if the custom action is the same repo as your workflow, you can reference it with it's path. 

For example: 

```html
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - name: Get code
        uses: actions/checkout@v3
      - name: Cache dependencies
        uses: ./.github/actions/cache-deps 
```

---

Here's an example of Input action in Custom composite action

```html
name: Loading Cache Dependencies Custom Action
description: To offload Caching Dependency Steps and simplify
inputs:
  caching:
    description: 'Whether to cache or not'
    required: false
    default: 'true'
runs:
  using: 'composite'
  steps:
    - name: Cache NPM Dependencies
      if: inputs.caching == 'true'
```

You can then use this to either cache or not in specific jobs/steps by passing the input in the workflow, like this: 

```html
jobs:
  linting:
    runs-on: ubuntu-latest
    steps:
      - name: Installing Dependencies
        id: install-deps
        uses: ./.github/actions/npm-cache
        with:
          caching: 'false'
```

---

Workflow output action in conjunction with input action

```html
name: Loading Cache Dependencies Custom Action
description: To offload Caching Dependency Steps and simplify
inputs:
  caching:
    description: 'Whether to cache or not'
    required: false
    default: 'true'
outputs:
  cache-status:
    description: "Check whether caching happened?"
    value: ${{ steps.install.outputs.npm-cache-status }}
runs:
  using: 'composite'
  steps:
    - name: Cache NPM Dependencies
      if: inputs.caching == 'true'
      uses: actions/cache@v5
      id: npm-cache
    - name: Install Dependencies
      if: steps.npm-cache.outputs.cache-hit != 'true' || inputs.caching != 'true'
      id: install
      run: |
        npm ci
        echo "npm-cache-status=${{ inputs.caching }}" >> $GITHUB_OUTPUT
      shell: bash
```

---

Here, we're printing the output, if caching happened or not, look closely at how referencing works

```html
jobs:
  linting:
    runs-on: ubuntu-latest
    steps:
      - name: Get Code
        uses: actions/checkout@v5
      - name: Installing Dependencies
        id: install-deps
        uses: ./.github/actions/npm-cache
        with:
          caching: 'false'
      - name: Output Cache Status
        run: echo "Did Caching Happen? = ${{ steps.install-deps.outputs.cache-status }}"
```

---
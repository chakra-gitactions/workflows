Jenkins or github actions
==
- concepts are similar
1. pipeline/workflow
2. tirggers
3. events
4. event types
5. event filters
6. options - (timeout and concurrency)
7. environment - (key,value)
8. conditions - (if:, when{})
9. status - (success, failure,abort, cancel)
10. post
11. sequential
12. parallel
13. statges/jobs
14. steps/name
15. runners/agents
16. inputs/parameters
17. shared library/actions


Advantage of github actions
==
- No additional tool
- Easy to integrate
- Must use github as repository
- We can use self hosted runners

- ***Workflow:*** An automation process in yaml that runs multiple jobs and triggered by events in github.
- ***Events:*** Any changes in git repo is considered as an event. Like push, pull, create PR, merge PR, create issues, update tags and so on.
- ***JOBS:*** It is collection of steps executed in CI/CD process. Like checkout, build, test, scan, deploy and so on.
- ***Action:*** Like plugins in jenkins, reusable code snippet, like checkout,setup terraform etc.
- ***Runners:*** Agents in Jenkins where our workflow runs. Runners are associated with organization in github(group of repo).
- ***New organization:** Create it, runners are associated with organizations (We can create multiple repos inside org)

- Path for workflow: .github/workflows/%.yaml
- Hello world program
```
name: Hello                 # Name of the pipeline
on:                         
 workflow_dispatch          #event, trigger manually
jobs:                       # Can run multiple jobs inside pipeline
 hello:                     # name of the job
  runs-on: ubuntu-lastest   # again in jenkins
  steps:                    # Sequentials steps
  - name: print name
    run: echo "hello world"
```

- Github provides their hosted runners (ubuntu, windows and mac), we can configure our own runners. Those are self hosted runners.
- To get inputs from users and trigger events manually, the event should be workflow_dispatch.
- We can specify list of events to trigger.
- ***Refer:** inputs.yaml

- ***Options**:
   - Define timeout or concurrency
   - timeout-minutes is defined at job level
   - concurrency is defined at trigger level
   - Need to create a unique group for concurrency

  Key elements
  ==
  - Workflow
    - Attached to github repository
    - Contains one or more jobs
    - Triggered upon events
  - Jobs
    - We choose runners like agents in jenkins
    - Contains one or more steps
    - Jobs run in parallel by default
    - It can be sequential by defining dependencies(needs)
   - Steps
     - Execute commands or call actions
     - Always run in order and can be conditional

Manual events
==
- workflow_dispatch: Used to trigger manually.
- repository_dispatch: API request triggers workflow.
- Schedule: Workflow scheduled.
- Workflow_call: It can be called by other workflows.

Actions
==
- Reusable snippet or code
- It is called into workflow using "uses" in steps
- To download repo into runner we use ***uses: actions/checkout@v3***
- We use version to keep it stable and wont take new/untested versions.


Capabilities
==

- Writing re-usable workflows in github actions, developing actions.
- Migrating pipelines from Jenkins to github action's workflows.


Lifecycle of CI/CD
==

***Feature pipeline:***
- The workflow is created in feature branch.
- Check out code
- Install dependencies
- Run Unit test
- Build image
- scan image
- Set cloud credentials
- Push to cloud registry
- Use the trigger type push to trigger the CI pipeline at each commit/push
- Use github commit statusess API to update each steps commit status to commit id
- Use resuable workflow and actions and permissions.
- At the end an image with commit id as tag will be pushed to registry.

***Dev pipeline:***
- Trigger dev pipeline when a PR is raised and merged with main branch.
- Merge creates additional commit, so it will run the feature and dev pipeline.
- The dev pipeline runs the feature steps and finally deploy the image/manifests to dev cluster.
- Helm is used to deploy the chart to dev cluster.

***UAT pipeline:***
- A re-usable pipeline is used to perform commit status checks using API.
- If successful, it will deploy the helm chart to UAT.
- As input, we provide environment (QA, UAT) and version which is commit id.
- This workflow is triggered manually, it also does integration testing.
- Check commit status -> trigger env -> trigger end-to-end test -> deploy in UAT success -> E2E-Test sucess.

***PROD pipeline:***
- A re-usable pipeline is used to perform commit status checks using API.
- If successful including UAT deployment, it will deploy the helm chart to UAT.
- As input, we provide version which is commit id.
- This workflow is triggered manually and follows CR process.

***CR Process***
- Checks, time, CR is approved or not.
- Trigger prod deploy. L2 support or dev support
- Jira ticket
   - Change ticket number
   - project code
   - component
   - version
   - approvals -> TL, TM, TESTING, MANAGER, Client
   - upload scan, test results
   - time fixed
   - rollback plan -> using helm automatic rollback
   - Call api to get jira ticket details
   - Check version
   - Check time window < deployment window
   - Status - approved or not
   - Check commit status and trigger prod deploy
   - Developer do the sanity checks

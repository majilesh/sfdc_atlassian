Missing Atlassian SFDC documentation developer.atlassian.com/sfdc

# The Salesforce Development Workflow

Team-based development is hard to get right. At Atlassian we make use of some state-of-the-art development tools and agile techniques to make collaboration easier and ensure the quality of our Salesforce customizations.

# Salesforce Organizations
We make use of several orgs to allow developers to build and test customizations in isolation, then integrate and promote them to production in a safe manner.

* Individual Developer Sandboxes: each developer has their own Developer Sandbox where features are initially developed and tested in isolation
* Shared Developer Sandbox: finished features are then integrated in a shared Developer Sandbox to ensure there are no conflicts or other integration failures
* User Acceptance Testing: changes are then promoted to a Full Data Sandbox known as UAT, where end-users can manually test and verify features before they are promoted to production
* Production: is our live Salesforce organization containing fully tested customizations and all of our production data.

This is the general flow of how customizations (or "features") are developed and promoted between our Salesforce organizations.

# Issue Tracking

Every new feature, improvement or bug fix applied to our Salesforce org starts off as an issue in JIRA, our issue tracker. In order to properly manage and track changes to the org, no customization (no matter how small) is made without first creating an issue. For simplicity, we'll be referring to all code changes regardless of their nature as "features".

#### A very simple feature development workflow has four states:


* Open: the feature is ready to be worked on
* In Progress: the developer has commenced work
* In Review: the developer is having their work peer reviewed and iterating on it
* Done: the work is complete

Issues transition through them in a linear fashion:

In our Salesforce development workflow, we use a slightly more sophisticated workflow to encompass two additional lifecycle stages: User Acceptance Testing and deployment to production.

These two additional stages represent workflow states where our issue is "code complete" but still needs to pass some critical criteria before it is deployed to our end users:


* Ready for Testing: the feature has been deployed to UAT
* Ready for Deployment: the feature has been tested and is ready for deployment to Production

See [Setting Up Issue Tracking](https://developer.atlassian.com/sfdc/setting-up-your-workflow/setting-up-issue-tracking) for a step-by-step guide for setting up the Salesforce workflow in JIRA.

# Version Control & Feature Branching

All of our metadata is version controlled in a Git repository hosted by [Stash](http://www.atlassian.com/software/stash/). 
All of our feature development is done using a hybrid [feature branching](https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow) and [forking](https://www.atlassian.com/git/tutorials/comparing-workflows/forking-workflow) workflow. We create a new feature branch for each and every issue, which means that each branch in our repository represents a single feature, bugfix or improvement. This feature is worked on in complete isolation on the feature branch until it is considered "ready to ship", which involves passing several criteria relating to quality:

* All development work is finished and the feature is "code complete"
* All tests are passing (indicating that the branch is deployable)
* The code has been reviewed and approved by at least two approvers

Once these criteria are met, the feature branch is merged into the master branch of the developer sandbox repository.

Our Salesforce workflow requires three separate repositories, each representing a different Salesforce org.

1. Developer Sandbox: this repository is the first repository to which a new feature is merged. Features are first merged to the Developer Sandbox's master branch to ensure that all integration tests are passing before promoting the changes to UAT.
2. UAT: if the master branch of the Developer Sandbox is passing, it can be merged into the master branch of the UAT repository. Changes to UAT's master branch are automatically deployed to the UAT Salesforce org, where end users can test that the new feature is working correctly.
3. Production: once the end users are satisfied, the UAT master branch can be merged to the Production master branch. Like UAT, changes to Production's master branch are automatically deployed to the Production Salesforce org.

The feature branch, master branches and merge commits across the three repositories can be visualized like this:

To track a feature's progression from development to production, we can bind the feature branch creation and subsequent merges to our issue tracking workflow:

Keeping in line with Salesforce's "clicks not code" philosophy, the branch merges between repositories are done via the Stash user interface using Pull Requests. Importantly, this allows developers to peer code review and iterate on features before they're merged and deployed.

See [Setting Up Version Control](https://developer.atlassian.com/sfdc/setting-up-your-workflow/setting-up-version-control) for a step-by-step guide for setting up the git Salesforce workflow in Stash.

# Continuous Integration & Deployment

We make heavy use of our Continuous Integration server, Bamboo, to continuously test and deploy features. We have several different build plans that run at different points of the feature lifecycle:

* All Tests plans: run the tests against feature branches and master branches to ensure that all unit tests are currently passing and the branch is deployable
* Deployment plans: automatically deploy master branches to their associated Salesforce organizations
..* DevSandbox/master is continuously deployed to the Shared Developer Sandbox
..* UAT/master is continuously deployed to the UAT Full Data Sandbox
..* Prod/master is continuously deployed to the production Salesforce org
* Destructive Changes plan: deploys a set of destructive changes (defined in a destructiveChanges.xml descriptor) across all our Salesforce organizations
* Save Changes plan: pulls down "click" changes made via the Salesforce UI and commits them to a repository, allowing Salesforce Administrators to make changes alongside Developers
* Full Backup plan: pulls down all Salesforce metadata and commits it to an orthogonal branch in a repository, which can be used for auditing and analysis of how your metadata has changed over time

See [Setting Up Continuous Integration and Deployment](https://developer.atlassian.com/sfdc/setting-up-your-workflow/setting-up-continuous-integration-and-deployment) for a step-by-step guide for setting up the Salesforce build plans in [Bamboo](http://www.atlassian.com/software/bamboo/).

# Team Collaboration

To work more effectively as a distributed team and facilitate asynchronous workflow, we use a variety of HipChat integrations to keep the team abreast of current development and deployments. All team members are part of a single Salesforce HipChat room during working hours. Whenever a build passes or fails, the results are posted to the room so that every team member is aware when there are failing tests or a deployment has succeeded. We've also experimented in the past with notifying the room when issues are transitioned or pull requests are created and merged.

See Setting Up Collaboration Tools for a step-by-step guide for setting up useful Salesforce integration in [HipChat](https://www.atlassian.com/software/hipchat).

# Continuous releasing

We develop our Salesforce customizations as a traditional software team might develop a SaaS product: by continuously release features "when they're ready" rather than packaging up collections of features as a discrete release. We've experimented with building "releases", but we find that continuously deploying speeds up our overall issue resolution time and removes the needless overhead of release management. This may differ from your current workflow, but it has worked well for us so far. There's nothing stopping you from following the rest of the Atlassian workflow and then creating stable release branches in your Stash repository, but I'd encourage you to pause and consider what value it is really adding to your workflow.

# Clicks versus code

One of the challenges we faced when designing our workflow was reconciling the two parallel streams of Salesforce development that occur when your Org reaches a certain level of sophistication:

1. Clicks: changes made by Salesforce Administrators via the user interface
2. Code: changes to Apex classes and metadata made by Salesforce Developers and deployed via the tooling API

The two workflows are fundamentally different, and the next section of the cookbook is split into two sections to document the two flows separately. You may wish to stick to the part most relevant to your role, but feel free to read both to gain deeper understanding of how the Atlassian Salesforce workflow hangs together:

## CodeCommit
* A private git-based repository hosted by AWS

### What is CodeCommit?
* Managed source code control service that hosts private git repos.
* Benefits:
	* Highly available, scalable, fault tolerant
	* No size limit
	* Integrates easily with other AWS services (CodePipeline, Lambda, SNS)
	* Migrate files from other git-based repos
	* Works with existing git-based tools
* Developers can easily manage, share, update, and coordinate the code they are independently working on
	* Facilitated by a centralized repo

### Basic git Concepts
* Clone - a developer can copy a repository's contents to their local machine
* Commit - a placeholder or queue where a developer can place changes or features added to code without pushing them to the remote repository
* Push - this will push a new version of the code to the central respository (v1 -> v2)
* Pull - this allows a developer with an earlier working version to pull changes to the central repo to their local machine
* Branch - versions of code can potentially branch off as new features are developed/tested. Generally the "master" branch is treated as the stable release
* Merge - you can merge different branches into the master branch to create a new version of the master
* Conflict - when multiple developers are working on the same portion of code. A developer pulls the current version from the central repo and there are conflicts with the changes they've made locally

### Interacting with the Repository
* You can access the repository either over HTTPS or SSH
	* Tip: SSH tends to be easier with Mac OSX because of keychain issues
	* Both protocols encrypt data in transit, but one major difference is that there is a credentials helper when using HTTPS
* You can create, view, edit, and delete a repo either from the AWS Console or CLI
	* Create: `aws codecommit create-repository --repository-name <RepoName> --repository-description "<Description>"`
	* List: `aws codecommit list-repositories`
	* View info: `aws codecommit get-repository --repository-name <RepoName>`
	* View batch info: `aws codecommit batch-get-repositories <Repo1> <Repo2> etc.`
	* Update: `aws codecommit update-repository-name --old-name <OldName> --new-name <NewName>`
	* Delete: `aws codecommit delete-repository --repository-name <RepoName>`

#### Using git commands to interact with the repo:
* **Clone**: `git clone <RepoURL> <LocalRepoName>`
* **Commit**:
	* `git add <file name>` -- adds file to a commit
	* `git rm --cached <file name>` -- removes a file from a commit
	* `git status` -- view files that have been added to a commit
	* `git commit` -- finalize a commit
* **Push**:
	* `git remote` -- view remote name
	* `git branch` -- view branch name
	* `git diff` --stat -- view which files will be pushed
	* `git push` -- push the commit to the remote repo
* **Pull**:
	* If someone has a previous version, they can pull the newer version using:
		* `git pull` or `git pull <remote name> <branch name>`
* **Resolving basic merge conflicts**:
	
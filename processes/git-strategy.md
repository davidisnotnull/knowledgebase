# Git Strategy
## Goals
This branching strategy document has the following intentions
* To minimise chaos within the branches
* To ensure clarity of the changes that are committed
* To prevent errors and bugs by minimising conflicts and development overlap
* To improve reliability with a list of best practices

Whenever possible, these guidelines will include a specific justification for each strategy choice. Unjustified guidelines must be either justified or removed. Whereas these styles draw mostly from the experience of the author, it also includes ideas from Vincent Dressen.

### Fixing problems in the branching strategy
Unless otherwise stated, this branching strategy is not optional, nor is it up for interpretation.
* If a guideline is not sufficiently clear, recommend a clearer formulation
* If you don't like a guideline, try to get it changed or removed; don't just ignore it. Others might not be aware that you are special and not subject to the same rules as everyone else.

## Branching
### Main branches
At the base of our branching strategy are two evergreen branches - `master`, and `stable`.

Master is our main branch, and reflects the latest delivered development changes for our next release. As a developer, you should be branching from and merging into master.

Stable should always represent the code that currently exists in the wild on our production environment. It should not be interacted with in our day-to-day development.

When the source code within master has been deployed to production, all of these changes must be merged into the stable branch and tagged with a release number.

Both of these branches must be locked so that force pushes cannot be made to them (cue Star Wars puns), and standard pushes can only be made via approved pull requests.

### Feature branches
A feature branch comes under the guise of **Supporting Branches**. These are used to aid in parallel development, to ease the tracking of features, and to quickly fix problems in production.

Feature branches have a limited lifetime, and are used to develop code features in isolation within a sprint.

When working on feature branches, we should keep an eye on what's happening in the master branch. If commits have been made to master after the feature branch was created, we must merge master into the feature branch before merging the feature branch back into master.

So, the rules for feature branch merging are
* Must branch from master
* Must merge back into master
* Naming convention is **feature/identifier-description**

Often, we find ourselves using some sort of task management software, such as Jira or Azure DevOps. In this case, we need to be naming our feature branches with the identifier of the ticket, followed by a brief description of what the feature contains. An example would be **feature/DSB-545-marketing-preferences-service**.

#### Highlander's Rule
You can probably guess what this is going to be...
> There can be only one.

Clearly, we aren't lopping people's heads off to cure ourselves of immortality. In this context, **Highlander's Rule** means that there should only ever be one feature branch for a particular story.

Never should we see two feature branches created by developers for the same feature. Not only is it deeply confusing for the person who has to put together a release candidate, but it's shoddy practice.

### Hotfix branches
A hotfix branch is branched from master, and should only be used to fix issues that are found in production. Due to the priority nature of hotfixes, they fall outside of the usual deployment schedule.

## Commit Messages
A well crafted commit message is the best way to convey the context of a change to other developers, and your future self. Be kind to your future self, and adhere to the following rules for commit messages.

### The rules of commit messages
##### Separate the subject from the body
Though not required, it's a good idea to begin the commit message with a single short (less than 50 character) line summarizing the change, followed by a blank line and then a more thorough description. The text up to the first blank line in a commit message is treated as the commit title, and that title is used throughout Git. For example, Git-format-patch(1) turns a commit into email, and it uses the title on the Subject line and the rest of the commit in the body.

##### Limit the subject line to 50 characters
This is not a hard and fast limit, just a recommendation. By keeping our subject lines to this limit, it makes them readable in git GUI tools and when examining the commits in GitHub. It also forces the author to think a bit about being concise when they are explaining what’s included in their commit.

GitHub’s UI has a hard limit of 72 characters, and will provide an ellipsis at the end of any commit subject messages that exceed this. So, 50 characters is the recommendation. 72 is the absolute limit.

##### Proper casing on the commit subject
I don’t think I need to explain any more than the heading for this section - make sure you capitalise the first word on the subject of the git commit, e.g.
> Resolve merge conflicts from branch hotfix/BUG-1448

##### Do not end the subject line with a period
Trailing punctuation is not necessary on commit messages. Remember, space is precious when we are trying to keep within a 50 character limit.

##### Use the imperative mood in the subject line
Imperative mood simply means “spoken or written as if giving a command or instruction”. For example:
* Tidy your room
* Take out the rubbish
* Do the dishes

Being (mostly) British, we don’t like being outwardly rude. It’s perfect for commit subject lines, however. One reason for this is that git uses the imperative whenever it creates a git commit on your behalf, and computers aren’t renowned for their politeness (in some cases, neither are developers).

For example, whenever we do a merge using the git command line, we get:

> Merging branch ‘feature/FEA-1234-new-navigation’

When you write in the imperative, all you’re doing is following git’s inbuilt conventions. The following are poor examples of subject messages:
* More fixes for broken stuff
* Sassy new API methods
* Changing structure of Y

No. This is what we call writing in the indicative mood. Being indicative is all about reporting facts, which is not what a commit subject should be, as counterintuitive as that sounds.

##### A properly formed git commit subject line should always be able to complete the following sentence
> If applied, this commit will [your subject line here]

The following are good examples of commit subject lines:
* If applied, this commit will **remove deprecated methods**
* If applied, this commit will **resolve merge conflicts**
* If applied, this commit will **merge pull request #101 from feature/branch**

You can see how our first few commit messages won’t work in the above format:
* If applied, this commit will **changing structure of Y**
* If applied, this commit will **more fixes for broken stuff**
* If applied, this commit will **sassy new API methods**

##### Fixed is not acceptable
This one is simple.
> Under no circumstances is "Fixed" an acceptable commit message.

It's lazy, not descriptive, and doesn't help anyone.

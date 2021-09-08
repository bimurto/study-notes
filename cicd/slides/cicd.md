---
marp: true
theme: gaia
_class: lead
paginate: false
backgroundColor: #fff
backgroundImage: url('https://marp.app/assets/hero-background.jpg')
---

# Continuous Integration



---

# What is Continuous Integration?

* Continuous Integration is a software development practice where members of a team integrate their work frequently, usually each person integrates at least daily - leading to multiple integrations per day.
* Each integration is verified by an automated build (including test) to detect integration errors as quickly as possible.

---

# Why do you need it ?
* The longer development continues on a branch without merging back to the mainline, the greater the risk of multiple integration conflicts and failures when the developer branch is eventually merged back.
* In the past, developers on a team might work in isolation for an extended period of time and only merge their changes to the master branch once their work was completed. 
* This made merging code changes difficult and time-consuming, and also resulted in bugs accumulating for a long time without correction.

---

# Example Scenario
* Suppose , there is a mainline branch called dev, every developer pulls their code from this branch
* Bob is now working on a feture called f1 and Alice is now working on a feature called f2
* They created feature branches feature/f1-bob and feature/f2-alice
* For 3 months they worked on their features and after that they are now ready to merge
* What do you think would happen? 

---

![bg w:700 h:600](./merge_hell.png)

---

# CI Workflow
* Bob will pull a branch called feature/f1-bob from dev branch
* He will whatever he needs to do to work on his feature, the feature may be completed or not completed. His work may include modification of existing code, adding new code , changing existing test codes etc.
* Bob has done working, now he will run the tests. If a java project a `gradle clean build` will work.
* Meanwhile Alice has also pushed his code to dev , so Bob need to pull the latest code and build again.


---

* If all the tests are done, then Bob will push his code to dev branch, then an automated CI server will compile his code and run test, both unit and integration test.

* If something is wrong, then Bob will be notified and he needs to fix his code until all the tests are running.

---

# Common Practices
* Maintain a code repository, dah!
* Automate the build - (use a build tool, e.g. gradle, yarn, etc.)
* Make the build self-testing - build should run the tests - (no -x test anymore)
* Everyone commits to the baseline e.g. dev every day
* Every commit (to baseline e.g. dev) should be built
* Every bug-fix commit should come with a test case
* Keep the build fast
* Test in a clone of the production environment

---

* Make it easy to get the latest deliverables
* Everyone can see the results of the latest build
* Automate deployment


---
<!-- _class: lead -->
# Continuous Deployment

---



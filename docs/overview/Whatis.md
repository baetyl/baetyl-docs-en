# What is Baetyl

**[Baetyl](https://baetyl.io) is an open edge computing framework that extends cloud computing, data and service seamlessly to edge devices.** It can provide temporary offline, low-latency computing services, and include device connect, message routing, remote synchronization, function computing, video access pre-processing, AI inference, device resources report etc. The combination of Baetyl and the **Cloud Management Suite** of [BIE](https://cloud.baidu.com/product/bie.html)(Baidu IntelliEdge) will achieve cloud management and application distribution, enable applications running on edge devices and meet all kinds of edge computing scenario.

About architecture design, Baetyl takes **modularization** and **containerization** design mode. Based on the modular design pattern, Baetyl splits the product to multiple modules, and make sure each one of them is a separate, independent module. In general, Baetyl can fully meet the conscientious needs of users to deploy on demand. Besides, Baetyl also takes containerization design mode to build images. Due to the cross-platform characteristics of docker to ensure the running environment of each operating system is consistent. In addition, **Baetyl also isolates and limits the resources of containers**, and allocates the CPU, memory and other resources of each running instance accurately to improve the efficiency of resource utilization.

## Advantages

- **Shielding Computing Framework**: Baetyl provides two official computing modules(**Local Function Module** and **Python Runtime Module**), also supports customize module(which can be written in any programming language or any machine learning framework).
- **Simplified Application Production**: Baetyl combines with **Cloud Management Suite** of BIE and many other productions of Baidu Cloud(such as [CFC](https://cloud.baidu.com/product/cfc.html), [Infinite](https://cloud.baidu.com/product/infinite.html), [Jarvis](http://di.baidu.com/product/jarvis), [IoT EasyInsight](https://cloud.baidu.com/product/ist.html), [TSDB](https://cloud.baidu.com/product/tsdb.html), [IoT Visualization](https://cloud.baidu.com/product/iotviz.html)) to provide data calculation, storage, visible display, model training and many more abilities.
- **Quickly Deployment**: Baetyl pursues docker container mode, it make developers quickly deploy Baetyl on different operating system.
- **Deploy On Demand**: Baetyl takes modularization mode and splits functions to multiple independent modules. Developers can select some modules which they need to deploy.
- **Rich Configuration**: Baetyl supports X86 and ARM CPU processors, as well as Linux and Darwin operating systems.

## Contributing

Welcome to Baetyl of Baidu Open Source Project. To contribute to Baetyl, please follow the process below.

We sincerely appreciate your contribution. This document explains our workflow and work style.

### Workflow

Baetyl use this [Git branching model](https://nvie.com/posts/a-successful-git-branching-model/). The following steps guide usual contributions.

1. Fork

   Our development community has been growing fast, so we encourage developers to submit code. And please file Pull Requests from your fork. To make a fork, please refer to Github page and click on the ["Fork" button](https://help.github.com/articles/fork-a-repo/).

2. Prepare for the development environment

   ```bash
   go get github.com/baetyl/baetyl # clone baetyl official repository
   cd $GOPATH/src/github.com/baetyl/baetyl # step into baetyl
   git checkout master  # verify master branch
   git remote add fork https://github.com/<your_github_account>/baetyl  # specify remote repository
   ```

3. Push changes to your forked repository

   ```bash
   git status   # view current code change status
   git add .    # add all local changes
   git commit -c "modify description"  # commit changes with comment
   git push fork # push code changes to remote repository which specifies your forked repository
   ```

4. Create pull request

   You can push and file a pull request to Baetyl official repository [https://github.com/baetyl/baetyl](https://github.com/baetyl/baetyl). To create a pull request, please follow [these steps](https://help.github.com/articles/creating-a-pull-request/). Once the Baetyl repository reviewer approves and merges your pull request, you will see the code which contributed by you in the Baetyl official repository.

### Code Review

- About Golang format, please refer to [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments).
- Please feel free to ping your reviewers by sending them the URL of your pull request via email. Please do this after your pull request passes the CI.
- Please answer reviewers' every comment. If you are to follow the comment, please write "Done"; please give a reason otherwise.
- If you don't want your reviewers to get overwhelmed by email notifications, you might reply their comments by [in a batch](https://help.github.com/articles/reviewing-proposed-changes-in-a-pull-request/).
- Reduce the unnecessary commits. Some developers commit often. It is recommended to append a sequence of small changes into one commit by running `git commit --amend` instead of `git commit`.

### Merge Rule

- Please run command `govendor fmt +local` before push changes, more details refer to [govendor](https://github.com/kardianos/govendor)
- Must run command `make test` before push changes(unit test should be contained), and make sure all unit test and data race test passed
- Only the passed(unit test and data race test) code can be allowed to submit to Baetyl official repository
- At least one reviewer approved code can be merged into Baetyl official repository

## Contact us

As the first open edge computing framework in China, Baetyl aims to create a lightweight, secure, reliable and scalable edge computing community that will create a good ecological environment. In order to create a better development of Baetyl, if you have better advice about Baetyl, please contact us:

- If you want to participate in Baetyl's daily development communication, you are welcome to join [Wechat-for-Baetyl](https://baetyl.bj.bcebos.com/Wechat/Wechat-Baetyl.png)
- If you have better development advice about Baetyl, please send email to <baetyl@lists.lfedge.org>
- If you have more about feature requirements or bug feedback of Baetyl, please [submit an issue](https://github.com/baetyl/baetyl/issues)



# Useful notes for Dicty web development

* [List of projects Eric has worked on](https://github.com/dictybase-playground/dev-notes/blob/master/Eric's%20Projects.md)

### dicty specific

* [Documentation for frontend development](https://gist.github.com/cybersiddhu/bc9025e4413c3e3dbf748d3916c7039e#file-frontend-development-md) - both web apps and standalone components
* [Deployment guide](https://github.com/dictyBase/Migration/blob/master/deploy.md)
* [gdrive-image-uploadr](https://github.com/dictybase-playground/gdrive-image-uploadr/tree/develop) - has all of the standards to be used in React applications

### Creating new releases

1.  Make sure all tests are passing and write documentation as necessary
2.  Run `npm run build` to generate the library
3.  Merge changes into develop, then master
4.  Tag this release:

* Run `git tag -a v1.0.0 -m "version 1.0.0"` where the version number is the one that needs to be updated
* Push tag to branch, i.e. `git push origin v1.0.0`
* To delete local tag, run `git tag -d tagName`
* To delete remote tag, run `git push --delete origin tagName`

### Golang

* [Go by Example](https://gobyexample.com/)
* [Effective Go](https://golang.org/doc/effective_go.html)

### Testing

* [Guide on testing IE in VirtualBox/Vagrant on a Mac](https://bluegg.co.uk/writing/testing-ie-in-a-virtualbox-in-a-vagrant-in-a-mac-in-a-bluegg)
* [Updated Vagrantfile for IE testing](https://gist.github.com/anthonysterling/7cb85670b36821122a4a)
* [The Right Way to Test React Components](https://medium.freecodecamp.org/the-right-way-to-test-react-components-548a4736ab22)
* [Writing Tests - Redux](https://redux.js.org/recipes/writing-tests)

Questions to ask for test "contracts":
1) What does it render?
2) What props does it receive?
3) What state does it hold?
4) What does the component do when the user interacts with it?
5) What is the context the component is rendered in?
6) What side effects occur as part of the component lifecycle (i.e. componentDidMount)
7) Does my component render different things under different circumstances?
8) When I pass a function as a prop, what does my component use it for? Does it call it, or just give it to another component? If it calls it, what does it call it with?

### React concepts

* [React Patterns](https://reactpatterns.com/)
* [A gentle Introduction to React's Higher Order Components](https://www.robinwieruch.de/gentle-introduction-higher-order-components/)

### Node.js

* [A guide to create a NodeJS command-line package](https://medium.com/netscape/a-guide-to-create-a-nodejs-command-line-package-c2166ad0452e)
* [Node.js Best Practices](https://github.com/i0natan/nodebestpractices)

### Markdown

* [Markdown Cheatsheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)

### Documentation

* [React](https://reactjs.org/docs/hello-world.html)
* [Redux](https://redux.js.org/)

### Resources

* [Mark Erikson's react-redux-links](https://github.com/markerikson/react-redux-links)

### Git

* [GitFlow](https://datasift.github.io/gitflow/IntroducingGitFlow.html) - branching model for Git

- Delete last commit: `git reset --hard HEAD^`
- Delete last commit on remote branch: `git push origin +branchName`
- Delete local branch: `git branch -d branchName`
- Delete remote branch: `git push origin --delete branchName`

### DevOps

* [The Twelve-Factor App](https://12factor.net/)
* [Introduction to Docker](http://blog.brew.com.hk/introduction-to-docker/)
* [Kubernetes In-Browser Course](https://www.katacoda.com/courses/kubernetes)

* Running webapp locally in Docker:
- `docker run -it -p 9595:9595 [BUILD]`

**Workflow:**
1. Push changes to develop
2. Wait for Docker Hub to build it.
3. Upgrade your chart

*Quick workflow:*
1. Build it in your machine with any tag
2. Push it to the Minikube Docker daemon (activate with `eval $(minikube docker-env)`)
3. Upgrade your chart with that particular tag
- `helm install [CHART] --namespace dictybase --set [ARGS] --set image.tag=latest --setimage.pullPolicy=IfNotPresent`

**Dev/prod workflow:**
1. Push to `develop` -> new image -> pull and test in minikube
2. Rebase `develop` in `master`, create new tag and push both `master` and tag
3. New image -> Sidd deploys it to staging cloud

*Remember* update the staging dockerfile as necessary

### Specifications

* [JSON API](http://jsonapi.org/)

### HTTP Programming

* [List of HTTP response status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)

### Production concepts
* Analyze bundle size: `source-map-explorer build/static/js/main.*`

### Problem Solving Process

* Write down how you want to approach the problem
* Outline how you want to solve it
* How do you expect it to be solved?
* Do the problem and see how it is actually solved
* See if anything differs

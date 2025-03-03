---
author: alex
category:
  - devnet
  - study-material
date: "2024-01-17T12:12:31+00:00"
guid: https://netwithalex.blog/?p=81
title: 1.0 CI/CD Pipeline
url: /devops-ci-cd-pipeline/
draft: true

---
### CI/CD concepts

Continuous Integration/continuous delivery/deployment is a methodology to develop solutions. The idea is to automate as much as possible to move quickly and optimize the development process. The NetDevOps pipeline includes a building phase which we are going to focus on to better understand the concept of Continuous Integration.

![](/wp-content/uploads/2024/01/Screenshot-2024-01-18-at-18.45.53.png)NetDevOps pipeline

In **Continuous Integration (CI)**, a crucial idea is the frequent uploading of code into the code base, often multiple times a day. This involves the constant merging of developer work with the code base, and the early identification of issues through the use of testing.

![](/wp-content/uploads/2024/01/Screenshot-2024-01-17-at-15.23.10.png)CI Diagram

##### Benefits

- Reduce time to market
- Achieve revenue and results at a faster pace
- Smaller backlog
- Smaller code changes
- Fewer bugs

##### Build Process

Involves activities such as:

1. **Integration:** Merging code. If several people are working on a project, the changes of each of them, must be consolidated in a single place.
1. **Linting:** Analyze code to identify programming errors, software defects, style and incorrect amounts of spaces. Some linting tools to know:
   - [Pylint](https://pypi.org/project/pylint/)
   - [Pyflakes](https://pypi.org/project/pyflakes/)
   - [Bandit](https://bandit.readthedocs.io/en/latest/)
   - [Black](https://black.readthedocs.io/en/stable/)
1. **Unit Testing:** Test individual components of code, not the entire code base. Write tests first, then code - Test Driven Development (TDD). Some tools to know:
   - [Pytest](https://docs.pytest.org/en/7.4.x/)
   - [Unittest](https://docs.python.org/3/library/unittest.html)
1. **Build:** This stage implies that all linting and unit tests have been completed and code is ready to be deployed on staging environment. Here an "artifact" is built and stored independently.

**Suggestion:** Explore the different tools shown above and know the basics of them

### CI tools

These are some of the most common CI tools out there. In the next post we are going to explore them with more detail.

CI ToolsURL**_Travis CI_**[https://travis-ci.org](https://travis-ci.org)**_Jenkins_**[https://jenkins.io](https://jenkins.io)**_Drone_**[https://drone.io](https://drone.io)**_GitLab CI_**[https://gitlab.com](https://gitlab.com)

Click [here](/ci-tools/) to learn about them!

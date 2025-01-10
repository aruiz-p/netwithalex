---
author: alex
category:
  - sdwan
  - networking
date: "2025-01-05T14:20:31+00:00"
title: Enhanced Application Aware Routing
draft: true
url: /enhanced-aar/

---
### Introduction

In my previous [series of posts](/demystifying-aar-1-3-the-foundations), I talked in depth about Application Aware Routing (AAR), a technology used to direct traffic through best performing paths. AAR is a core functionality of SD-WAN and has been around for years. Naturally, new ideas to improve it came around and gave birth to a revamped implementation which Cisco called **_Enhanced_** Application Aware Routing (EAAR). Let´s take a look to it.

### AAR Limitations

Before diving into EAAR, let's understand why it was created.

The current AAR implementation meassures the path quality using BFD, sending packets at a defined interval, 1 second by default. Loss, latency and jitter are derived from those packtes and these values are placed into rotating buckets to calculate an average tunnel health metric. This process would typically take between 10 and 60 minutes and with some configuration tweaks we could achieve times of 2 - 10 mins.

- False Positive
- Tunnel Scale issues if bfd hello interval is reduced 
- Slow degradation detection and switch times  

### Enhanced AAR

In a nutshell these are the advantages 
- Improves performance measurements using **inline data**
- Steer traffic in seconds rather than minutes 10s by default
- Dapening implemented for stability purposes

#### Taking a deeper look 

Loss measurement is done per queue
Latency is the RTT
Jitter is unidirectional, which means it is measured at the receiver and reported to the sender




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

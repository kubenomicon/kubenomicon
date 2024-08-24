![](./src/images/kubenomicon_cropped.png)


# Check out The Kubenomicon
- [The Kubenomicon](https://kubenomicon.com)

# What is The Kubenomicon?
The Kubenomicon was born of a desire to understand more about Kubernetes from an offensive perspective. I found many great resources to aid in my journey, but I quickly realized:
1. I will never be able to solely document every offensive and defensive Kubernetes technique on my own.
2. Things in the Kubernetes world [move really fast](https://kubernetes.io/releases/release/) and there are constantly new attack surfaces to explore. 
My solution to this is to start the Kubenomicon -- a place where offensive security techniques and how to defend against them can easily be documented via pull requests to the Kubenomicon GitHub. 

This project was heavily inspired by the [Kubernetes Threat Matrix](https://microsoft.github.io/Threat-Matrix-for-Kubernetes/) from Microsoft which is a great starting point as it provides a framework to help understand some of the concepts in a [MITRE ATTACK](https://attack.mitre.org/) style framework. The Microsoft Threat Matrix was explicitly not designed to be a playbook offensive for security professionals and thus it lacks the details necessary to actually exploit (and remediate) each attack in Kubernetes cluster. 

# Contributing 
Would you like to contribute to this project? There is still lots to be done!
The entire offensive security landscape relives HEAVILY on the time and expertise of other offensive security professionals through the tools and techniques we use everyday.

This project is only as useful as we (the community) make it. Things will change, attacks will evolve, and new attacks will be discovered. This project aims to help keep up with the changing attack surface, but I cannot do it alone. This entire project is open source. Should you have something to contribute, please don't hesitate to open a pull request to the GitHub repository.

## Structure
This project is created using [mdbook](https://github.com/rust-lang/mdBook) and compiled using Github Actions. If you would like to make a change, here is what I would recommend:
1. Clone this repository: `git clone https://github.com/grahamhelton/grahamhelton.github.io`
2. Host a local site with mdbook `mdbook serve`
3. Make any changes to the files hosted in `./src`
    - If you wish to add a new item to the sidebar, you can do so by editing `./src/SUMMARY.md`
    - If you wish to add images, you can do so by placing them in `./src/images/`
4. Ensure the changes are properly rendered in the local site (by default it's hosted at `localhost:3000`
5. Submit a pull request for your changed files

# Prior work
I am far from the first person to come up with the idea to document this information. Many great projects exist that take a similar approach to this. Most notably what inspired this project was the [Microsoft Kubernetes Threat Matrix](https://microsoft.github.io/Threat-Matrix-for-Kubernetes/). Additionally, late into putting this project together I discovered this amazing [Threat matrix from RedGuard](https://kubernetes-threat-matrix.redguard.ch/). Some other projects that served as inspiration for this include:
- [Kubernetes Hacktricks](https://cloud.hacktricks.xyz/pentesting-cloud/kubernetes-security)


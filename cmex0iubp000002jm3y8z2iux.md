---
title: "Shifting Security Left in Data Platforms: Docker Image Scanning with Trivy"
datePublished: Fri Aug 29 2025 15:53:45 GMT+0000 (Coordinated Universal Time)
cuid: cmex0iubp000002jm3y8z2iux
slug: shifting-security-left-in-data-platforms-docker-image-scanning-with-trivy
tags: docker, devsecops, dataengineering, trivy, cloud-security, githubaction

---

For busy data team like ours, the priority has always been to keep the platform reliable and fast while delivering new features. But reliability is not just about performance, it is about security as well.

Parts of our data pipeline and live inference / scoring services runs inside containerised services on Azure, handling sensitive data. Without any guardrails, a vulnerable base image or outdated dependency could slip into production and put the entire platform at risk. For a team that needs to move quickly, manual checks weren’t realistic.

That is where we introduced automated container vulnerability scanning in the CI/CD pipeline. We used Trivy, an open-source scanner, together with GitHub Actions. It became a simple but powerful way to make sure that every Docker image we build and deploy is scanned before it ever reaches production.

---

## How It Works

The workflow is straightforward:

1. **Build** – Use Docker buildx in GitHub Actions to build the image.
    
2. **Scan with Trivy** – Run Trivy against the freshly built image. Check for critical/high vulnerabilities. Fail the build if any are found.
    
3. **Deploy** – Only if the scan passes.
    

Here’s the Trivy scan step in the pipeline:

```bash
- name: Run Trivy vulnerability scanner
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: ${{ env.CONTAINER_REGISTRY_LOGIN_SERVER }}/${{ env.PROJECT_NAME_FOR_DOCKER }}:${{ github.sha }}
    format: 'table'
    exit-code: '1'
    ignore-unfixed: true
    vuln-type: 'os,library'
    severity: 'CRITICAL,HIGH'
  env:
    TRIVY_USERNAME: ${{ secrets.CONTAINER_REGISTRY_USERNAME }}
    TRIVY_PASSWORD: ${{ secrets.CONTAINER_REGISTRY_PASSWORD }}
```

Where snippet above:

* Fails the build if any *Critical/High* vulnerabilities are found.
    
* Ignores unfixed issues to reduce noise.
    
* Scans both OS and app libraries.
    

This ensures that security scanning is built into the delivery pipeline, not something you rely on to remember manually.

---

## Benefits of This Approach

* **Shift security left** - issues are caught during the build, not after deployment.
    
* **Developer visibility** - vulnerabilities appear directly in PRs, making remediation faster.
    
* **Automated enforcement** - no reliance on manual checks or periodic scans.
    
* **Lightweight and fast** - Trivy is quick enough to run on every build without slowing down pipelines.
    

## How This Helps a Data Platform

Here’s where the real impact shows up:

* **Security at the source** - Vulnerabilities are caught before an image goes live.
    
* **Reliable pipelines** - Fewer runtime issues from broken OS libraries or insecure dependencies. That means ingestion and ML jobs run consistently.
    
* **Shift-left culture** - Developers see issues directly in their pull requests, making security everyone’s responsibility.
    
* **Scalable practice** - As the number of microservices and pipelines grows, automated scanning scales far better than manual checks.
    

---

## Better Solutions at Scale?

Larger organisations may want more than Trivy-in-CI/CD:

* **Centralized registry scanning** Azure Container Registry and AWS ECR can automatically scan images after push. This complements CI/CD scans by catching vulnerabilities in existing images over time.
    
* **Continuous monitoring** vulnerabilities evolve. An image built last month may be flagged today. A dedicated security tool (e.g., Aqua, Prisma, Anchore) provides continuous scanning and alerting.
    
* **Policy integration** tying scans into a broader DevSecOps workflow ensures vulnerabilities are prioritized, tracked, and remediated, not just blocked.
    

---

## Final Thoughts

For a small and busy team, this practice struck the right balance: low effort, high impact. By adding Trivy scanning into GitHub Actions, we made security part of our delivery pipeline and reduced the risk of shipping vulnerable containers into production.
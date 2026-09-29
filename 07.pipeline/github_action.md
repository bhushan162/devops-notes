# <Topic Name> (e.g., Kubernetes Deployments)

> **Tags:** #GITHUBACTION

## 📝 TL;DR (1-Minute Revision)
* **What is it:**  github action is pipeline tool


---

## 🧠 Core Concepts
* **runner **: GitHub provides Linux, Windows, and macOS virtual machines to run your workflows
* ** Workflows:** 
  * warkflow is YAML file that trigger at diffrent event on repo. or we can trgget it manually. 
  * one repo can have multiple workflow
  * 
* **How it fits into Edge DevOps:** 
  * <Explain how this specific tool/concept behaves in low-resource, disconnected, or embedded environments.>

---

## 🛠️ Example
```yaml
name: learn-github-actions
on: [push]

jobs:
 check-bats-version:
   runs-on: ubuntu-latest

   steps:

     — uses: actions/checkout@v4                    # market place action 

     — uses: actions/setup-node@v4                  # market place action 
       with:
         node-version: ‘20’
     — run: npm install -g bats                 
     — run: bats -v                                 
```

---

## 🐛 Troubleshooting & Gotchas (Issue Tracker)
* **Error / Symptom:** `<Paste the exact error message or crash log here for searchability>`
  * *Root Cause:* <Why did this happen?>
  * *Fix:* <Step-by-step fix or command to resolve it>
* **Edge Specific Gotcha:** <e.g., "Image pull fails because ARM64 architecture isn't supported by this image tag.">

---

## 🎤 Interview Scenarios & Q&A
**Q: <Insert a common interview question regarding this topic>**
* **A:** <Draft your ideal answer using the STAR method (Situation, Task, Action, Result) if applicable, or just clear bullet points.>

**Q: How would you design <System/Process> using <Topic>?**
* **A:** <Your architectural answer>

---
*Reference Links:*
* [Official Docs](<url>)
* [Helpful StackOverflow/Blog](<url>)
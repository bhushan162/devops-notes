# Cross Compiling

> **Status:** 🟢 Active Learning 
> **Tags:** #cross-compilation


## 📝 TL;DR (1-Minute Revision)
* **What is it:** <One sentence explaining what this technology/concept is>
* **Primary use:** <Why do we use this in Edge/Embedded DevOps?>
* **Key command:** `<Insert the most common command you use for this>`

---

## 🧠 Core Concepts
* **Compiling process:** 
  * preprocesing - source.c  --> source.i
  * compoling - gcc -S source.i       --> source.s
  * assembling   source.s  --> source.o
  * linking'   source.o --> source.o


* **gcc cpmmand and output:**
  * gcc  command invoke all four stages of the compilation process
  * basic commadd ''' gcc source.c -o BnaryName
  *  **gcc main command ** 
     *  -o set name of output file 
     *  -E stop after the preprocessing stage do not run compiler proper 
        *  result in .i(preprocessed) files
     *  -S Stop after the stage compilation proper; do not assemple
        *  result in .s (assembly) files.
     *  -c Compile and assemble the source file do not link
        *  result .o (object) files.
     *  -save-temps do not delete intermediate files 
  * **gcc main commadn to invoke compiler**
    * -l non-standered directeries to search for inclue files (.h)
    * -L non-standered directeries to serach for library file (.a, .so)
    * -l[libname] specific library to link in 
      * -lc             libc(satndered c)
      * -lm             libm(math librabry)
      * -lpthered       libpthread(pthread library)
    * 
* **How it fits into Edge DevOps:** 
  * <Explain how this specific tool/concept behaves in low-resource, disconnected, or embedded environments.>

---

## 🛠️ Playbook & Commands
**1. <Action 1 - e.g., Create a deployment>:**
```bash
# Add your copy-paste ready commands here
<command>
```

**2. <Action 2 - e.g., Check logs/status>:**
```bash
<command>
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
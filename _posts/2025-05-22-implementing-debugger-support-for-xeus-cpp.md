---
title: "Implementing Debugging Support for xeus-cpp"
layout: post
excerpt: "A GSoC 2025 project aiming at integration of debugger into the xeus-cpp kernel for Jupyter using LLDB and its Debug Adapter Protocol (lldb-dap)."
sitemap: false
author: Abhinav Kumar
permalink: blogs/gsoc25_abhinav_kumar_introduction_blog/
banner_image: /images/blog/gsoc-banner.png
date: 2025-05-22
tags: gsoc c++ debugger dap clang jupyter
---

### Introduction

I am Abhinav Kumar, a final-year Computer Science and Engineering undergraduate at Indian Institute of Technology(IIT) Indore. I'm thrilled to be working with CERN-HSF for Google Summer of Code 2025 on the project "Implementing Debugging Support for xeus-cpp."

**Mentors**: Anutosh Bhat, Vipul Cariappa, Aaron Jomy, Vassil Vassilev


### The Need for Seamless C++ Debugging in Jupyter

[Jupyter](https://jupyter.org/) notebooks have revolutionized interactive computing, and kernels like [xeus-cpp](https://github.com/compiler-research/xeus-cpp/) bring the power of C++ to this environment. However, while writing and executing C++ code incrementally is great, the debugging experience can be a hurdle. Developers often need to step through their code, inspect variables, and understand the program's state, especially when dealing with the complexities that C++ can present. This project aims to bridge that gap by integrating robust debugging capabilities directly into the xeus-cpp kernel.

### Project Goal: Interactive Debugging with LLDB and DAP
The core idea is to bring a full-fledged debugging experience to xeus-cpp users within Jupyter. This means enabling features like:

1. **Breakpoint Management**: Setting and removing breakpoints with ease.
2. **Variable Inspection**: Examining the values of variables at different execution stages.
3. **Step-Through Execution**: Controlling the flow of code execution (step in, step over, step out).
4. **Stack Tracing**: Understanding the call stack to pinpoint issues.

To achieve this, the project will leverage existing, powerful technologies:

1. **LLDB**: The [LLVM debugger](https://lldb.llvm.org/), which has excellent support for JIT-compiled code and Clang integration—perfect for how xeus-cpp executes code dynamically.
2. **Debug Adapter Protocol (DAP)**: A [standardized protocol](https://microsoft.github.io/debug-adapter-protocol//) by Microsoft that allows debuggers to communicate with development tools. This ensures compatibility with Jupyter's frontend and follows the successful model of [xeus-python](https://github.com/jupyter-xeus/xeus-python).

A key design principle is to run the debugger (specifically, [lldb-dap](https://lldb.llvm.org/resources/lldbdap.html), which acts as a DAP server for LLDB) as an external process. This is crucial for kernel stability, preventing the Jupyter kernel from freezing when a breakpoint is hit.

### Proposed Architecture Overview
The proposed system involves a few key components working together:
<img src="/images/blog/debugger_xeus_cpp_architecture.png" alt="Project Architecture" style="width: 90%; height: auto"/>

1. **Jupyter Environment (Notebook & Server)**: The user interface where DAP requests are initiated and responses are displayed.

2. **xeus-cpp Kernel**:
    - ``xcpp::debugger``: A new class that will manage the debugging session, inheriting from **xeus::xdebugger_base**. It will handle DAP messages and interact with the lldb-dap client.
    - ``xcpp::xlldb_dap_client``: This component will manage the actual DAP communication with the external lldb-dap process.
    - ``JIT Engine``: Compiles the C++ code from cells and notifies LLDB about new symbols.

3. **External Debugger (lldb-dap & LLDB)**:
    - ``lldb-dap``: Runs as a separate server, translating DAP messages into LLDB API calls and vice-versa.
    - ``LLDB``: Executes the debugging actions on the JIT-compiled code.

This modular design, inspired by [xeus-python](https://github.com/jupyter-xeus/xeus-python), aims for a lightweight integration that reuses existing xeus infrastructure where possible.

### Progress and Path Forward

I've already made some promising headway:

1. Successfully demonstrated that LLDB can attach to and debug JIT-compiled code generated by [CppInterOp](https://github.com/compiler-research/CppInterOp/) (which xeus-cpp relies on). This involved ensuring symbols are resolved correctly when ``plugin.jit-loader.gdb.enable`` is active in LLDB.
2. Set up debugging in VSCode using lldb-dap for JIT-compiled code, proving the viability of lldb-dap for this context.
3. Experimented with running the lldb-dap executable on a specific port and sending DAP requests (initialize, launch, setBreakpoints, continue) via a Python script, successfully hitting breakpoints and getting stack traces.

The roadmap includes:

1. **Implementing debugger class**: Defining the main debugger class, handling the debugger lifecycle, and managing LLDB-DAP settings. A key modification will be using a "launch" request instead of "attach" (as xeus-python does), because LLDB needs to launch our custom executable containing the JIT code. This might require minor adjustments in xeus-zmq.
2. **Developing lldb-dap client**: This class will inherit from ``xeus::xdap_tcp_client`` and manage the low-level DAP communication.
3. **Robust Testing**: Implementing a comprehensive testing framework using [GoogleTest](https://github.com/google/googletest).

### Expected Benefits and Future Scope 

Successfully completing this project will significantly enhance the C++ development experience within Jupyter notebooks. It will provide:

1. A seamless, interactive debugging workflow for xeus-cpp users.
2. Increased productivity by allowing developers to quickly identify and fix bugs in their C++ notebook code.
3. A more robust and feature-complete C++ kernel for the Jupyter ecosystem.

Looking ahead, the architecture is designed with future-proofing in mind, potentially supporting advanced features like remote debugging, alternative debuggers or debugger in xeus-cpp-lite.

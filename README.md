# BareMetalAI

[![Build Status](https://img.shields.io/github/actions/workflow/status/lachlan-jones5/baremetalai/ci.yml?branch=main)](https://github.com/lachlan-jones5/baremetalai/actions)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)

A full-stack AI serving platform and research testbed for exploring the distributed execution of large models using custom compilers, runtimes, and unikernels for complete, low-level control.

## 1. Vision & Mission

BareMetalAI is an intelligent, resource-aware platform for serving Large Language Models on a Kubernetes cluster. The core mission is to maximize the utilization of all available hardware, run multiple models in parallel, and enable the distributed execution of models that are too large for any single machine. It serves not only as a production-grade serving platform but also as a research testbed for exploring high-performance computing and compiler technologies. The system is designed to be a powerful backend for AI-powered clients like the `cline` VS Code extension. A key goal is to enable experimentation with heterogeneous cluster configurations, where different nodes can run entirely different software stacks—from a standard Arch Linux node managed by Kubernetes to a dedicated node running a specialized unikernel. This allows for direct, in-place comparison of various "below the host" execution strategies.

## 2. Status

**This project is currently in the initial design and infrastructure setup phase.** The core architecture is defined, and the immediate focus is on establishing the foundational Kubernetes cluster and automation scripts.

## 3. Core Architecture

The backend is designed with a clean separation between the user-facing API and the underlying orchestration logic. This is achieved through a well-defined internal "Backend Interface" that decouples the API Gateway from the specific implementation, ensuring the backend can be swapped out in the future.

The initial implementation consists of two main planes:
* **Control Plane:** A custom Kubernetes Operator that manages the entire lifecycle of model execution.
* **Data Plane:** The Kubernetes cluster nodes where containerized model pods are run.

For a detailed breakdown of the system design, please see the [**Architecture Design Document**](docs/architecture.md).

## 4. Key Features

The platform is designed with a rich set of features for intelligent and efficient model serving:
* **Swappable Backends:** The core design allows the standard Kubernetes Operator to be replaced with a separate, custom compiler/runtime for complete, low-level control over the cluster.
* **Tiered Deployment Strategy:** The scheduler automatically attempts to place models using the most efficient method available, from a single GPU to a multi-node distributed deployment using advanced techniques like Tensor Parallelism and Model Parallelism.
* **Auto-Tuning Model Loader:** Before scheduling, the system automatically determines the optimal model partitioning strategy.
* **Dynamic Resource Management:** The platform can dynamically discover hardware and implements an LRU Eviction Policy to manage resources.

## 5. Technology Stack

The platform is built on a modern, lightweight, and automatable infrastructure stack:

| Component | Technology |
| :--- | :--- |
| **Operating System** | Arch Linux (or Arch Linux ARM) |
| **Orchestrator** | K3s (Master node on a Raspberry Pi) |
| **Configuration**| Ansible for post-install management |
| **OS Deployment**| `archinstall` scripting for automation |
| **Container Runtime**| containerd |
| **Container Engine**| Docker (for development) |
| **Storage** | A distributed storage solution like Longhorn |

## 6. Getting Started

Detailed instructions for setting up the development environment and deploying the cluster can be found in the [**Getting Started Guide**](docs/getting-started.md).

The basic steps are:
1.  Clone the repository: `git clone https://github.com/lachlan-jones5/baremetalai.git`
2.  Install dependencies using the provided Ansible playbook.
3.  Build the project using CMake.

## 7. Design Philosophy

This project is being developed following a professional, design-first methodology (**Vision → Architecture → Interfaces → Implementation**). Major features are preceded by RFCs/Design Documents to ensure a robust and scalable architecture. These can be found in the `docs/` directory.

## 8. Project Roadmap

The project plan is managed via GitHub Issues, using a hierarchy of Epics, Features, and User Stories. The full project roadmap and current work can be viewed on the [**Project Board**](https://github.com/lachlan-jones5/baremetalai/projects).

## 9. License

This project is licensed under the MIT License. See the `LICENSE` file for details.

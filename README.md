# 🚀 ART (A Reporting Tool) Deployment Guide

This guide helps you deploy **ART (A Reporting Tool)** in multiple environments:

- 🐳 Docker
- 🔧 containerd (`ctr`)
- ☸️ Kubernetes

---

## 📁 Prerequisites

- Docker or containerd (`ctr`)
- Kubernetes cluster (Minikube, kind, EKS, etc.)
- `kubectl` installed and configured

---

## 📦 Step 1: Download ART

Download and extract the WAR file:

```bash
wget https://netix.dl.sourceforge.net/project/art/art/8.7/art-8.7.zip
unzip art-8.7.zip

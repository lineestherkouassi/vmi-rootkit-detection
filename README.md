# Framework for Rootkit and Malware Detection

Agentless rootkit and malware detection for virtualized cloud environments, based on Virtual Machine Introspection (VMI) and unsupervised machine learning.

The solution monitors virtual machines from outside the guest operating system. Memory is acquired directly from the hypervisor, analyzed with Volatility 3, reduced to a compact set of behavioral features, and scored using a One-Class SVM trained without labels on a predominantly benign corpus.

Because the observation point is located below the guest operating system, a kernel-mode rootkit that subverts in-guest monitoring tools cannot tamper with the memory acquisition process itself.

This repository accompanies a Master's thesis in Computer Security and Web Technologies (ESATIC / INPT, 2025–2026). It is a research proof of concept, not a production security product.

## Table of Contents

* [Responsible Use](#responsible-use)
* [How It Works](#how-it-works)
* [Features](#features)
* [Requirements](#requirements)
* [Extracted Features](#extracted-features)
* [Results](#results)
* [References](#references)
* [Citation](#citation)
* [Acknowledgments](#acknowledgments)

## Responsible Use

This repository contains defensive tooling only. No rootkit, exploit, or malicious payload is distributed here.

The evaluation described in the thesis used publicly available research rootkits, executed inside an isolated virtual machine on an air-gapped laboratory host. Those samples are not included in this repository, and no instructions for deploying them are provided.

If you reproduce this work:

* Run every experiment in an isolated environment with no network route to production systems.
* Never execute unknown kernel drivers on a host you rely on.
* Comply with the laws of your jurisdiction and the policies of your institution.

Memory acquisition captures the full contents of a virtual machine's RAM, including credentials, cryptographic keys, and personal data. Treat memory dumps as sensitive artifacts: store them with restricted permissions and delete them when the analysis is complete.

## How It Works

```text
┌──────────────┐   ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│      1       │   │      2       │   │      3       │   │      4       │
│ Introspection│──▶│   Forensic   │──▶│   Feature    │──▶│  Detection   │
│              │   │   Analysis   │   │  Extraction  │   │              │
│ virsh dump   │   │ Volatility 3 │   │  11 numeric  │   │ One-Class SVM│
│ --memory-only│   │  14 plugins  │   │   features   │   │  + threshold │
│ --live       │   │              │   │              │   │              │
└──────────────┘   └──────────────┘   └──────────────┘   └──────────────┘
       ▲                                                        │
       │                                                        ▼
   KVM/QEMU                                            CLEAN │ INFECTED
   hypervisor                                          + confidence index
       │
       └──────────── scheduler loop (periodic) ─────────────────┘
```

### 1 — Introspection

`virsh dump --memory-only --live` produces a raw memory image of a running instance without installing an agent inside the guest and without suspending it.

### 2 — Forensic Analysis

Volatility 3 reconstructs kernel structures from the raw memory image. Windows symbol tables are resolved automatically from Microsoft PDB files, so no manual profile configuration is required.

### 3 — Feature Extraction

Plugin outputs are normalized and cross-referenced to produce a fixed-length numerical feature vector. Several features require correlating outputs from multiple plugins, since a given driver may not be identified consistently across them.

### 4 — Detection

The feature vector is standardized and scored using a One-Class SVM. The anomaly score is compared with a threshold calibrated as an empirical quantile of the training score distribution.

## Features

* **Agentless:** Nothing is installed inside the monitored virtual machine.
* **Out-of-guest:** Memory acquisition cannot be directly subverted by a compromised guest kernel.
* **Unsupervised:** No labeled attack corpus is required for training.
* **Two observation layers:** Kernel-space and user-space artifacts are kept logically separate for interpretation while forming a single feature vector for the model.
* **Continuous monitoring:** A scheduler runs the full pipeline at fixed intervals and logs state transitions.
* **Web dashboard:** A Streamlit interface displays the verdict, per-feature values, and contributing artifacts.
* **Modular:** Each stage exchanges data in a standard format, allowing components to be replaced independently.

## Requirements

### Host — Supervision Node

| Component      | Version / Configuration                      |
| -------------- | -------------------------------------------- |
| OS             | Ubuntu 24.04 LTS                             |
| CPU            | x86-64 with Intel VT-x (`vmx` flag) or AMD-V |
| Hypervisor     | KVM / QEMU with libvirt                      |
| Cloud platform | OpenStack deployed via DevStack              |
| Python         | 3.10+                                        |

### Guest

* Windows 10 x64 (tested on Enterprise LTSC)
* Other Windows versions are untested; see [Limitations](#limitations).

## Extracted Features

Eleven features feed the final detector. They are grouped by observation layer. See the thesis for the feature-selection procedure.

### Volatility 3 Plugins Used

```text
windows.modules
windows.modscan
windows.driverscan
windows.driverirp
windows.devicetree
windows.callbacks
windows.timers
windows.ssdt
windows.thrdscan
windows.pslist
windows.psscan
windows.cmdline
windows.dlllist
windows.ldrmodules
```

## Results

The evaluation was conducted on a corpus of **358 memory dumps**:

* 262 clean
* 96 compromised

| Model                |   Accuracy |     Recall |       FPR |   ROC-AUC |
| -------------------- | ---------: | ---------: | --------: | --------: |
| Isolation Forest     |     98.52% |     95.86% |     0.00% |     1.000 |
| Local Outlier Factor |     96.79% |     91.03% |     0.00% |     1.000 |
| **One-Class SVM**    | **99.26%** | **97.93%** | **0.00%** | **0.987** |

These results should be interpreted with the following caveats.

The training corpus contained 214 unlabeled observations with a controlled contamination rate of 1.87%. Test metrics are averaged over five splits of a held-out set of 81 dumps.

With 29 positive observations in the test set, the Recall has a 95% Wilson confidence interval of approximately **[82.8%; 99.4%]**. The zero false-positive rate has a 95% confidence interval of **[0%; 6.9%]**. It was measured on a clean corpus with limited software diversity and should therefore not be interpreted as a production-level figure.

All three models reach a zero false positive rate. One-Class SVM was retained for its higher recall at the calibrated threshold (97.93 % against 95.86 % for Isolation Forest) and for its lower variance across splits. Isolation Forest reaches an AUC of 1.000, meaning a perfectly separating threshold exists for it that the calibration procedure does not select.

## References

* Wang, X., Zhang, J., Zhang, A., & Ren, J. (2019). *TKRD: Trusted kernel rootkit detection for cybersecurity of VMs based on machine learning and memory forensic analysis*. Mathematical Biosciences and Engineering, 16(4), 2650–2667. DOI: 10.3934/mbe.2019132.
* Zhang, T., & Lee, R. B. *CloudMonatt: An Architecture for Security Health Monitoring and Attestation of Virtual Machines in Cloud Computing*.
* Ligh, M. H., Case, A., Levy, J., & Walters, A. (2014). *The Art of Memory Forensics*. Wiley.
* Volatility 3 — Memory forensics framework.
* OpenStack — Cloud infrastructure platform.


## Citation

```bibtex
@mastersthesis{kouassi2026cloudguard,
  author  = {Kouassi, Moayé Line Esther},
  title   = {Conception et mise en œuvre d'un framework intelligent de détection
             des malwares et rootkits dans les environnements cloud virtualisés},
  school  = {École Supérieure Africaine des Technologies de l'Information et de
             la Communication (ESATIC) / Institut National des Postes et
             Télécommunications (INPT)},
  year    = {2026},
  type    = {Mémoire de Master}
}
```

## Acknowledgments

This work was carried out within the RAISS team (Networks, Architectures, Service Engineering and Security) of the STRS laboratory at INPT, under the supervision of Prof. El Mostafa Belmekki, as part of the ESATIC–INPT academic partnership.

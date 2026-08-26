Frameword of rootkit and malware detection.

Agentless rootkit and malware detection for virtualized cloud environments, based on Virtual Machine Introspection and unsupervised machine learning.
The solution monitors virtual machines from outside the guest operating system. Memory is acqueired directly from the hypervisor, analyzed with Volatility 3, reduced to a small set of behavioral features, and scored by a One-Class SVM trained without labels on a predominantly benign corpus.

Because the observation point sits below the guest, a kernel-mode rootkit that subverts in-guest monitoring tools cannot tamper with the collection process itslf.

This repository accompanies a Master's thesis in Computer Security and Web Technologies (ESATIC / INPT, 2025–2026). It is a research proof of concept, not a production security product.

Table of contents
Responsible use
How it works
Features
Requirements
Installation
Usage
Extracted features
Results
Limitations
Roadmap
Repository layout
Citation
References
License

Responsible use

This repository contains defensive tooling only. No rootkit, exploit, or malicious payload is distributed here.

The evaluation described in the thesis used publicly available research rootkits, executed inside an isolated virtual machine on an air-gapped laboratory host. Those samples are not included in this repository and no instructions for deploying them are provided.

If you reproduce this work:

run every experiment in an isolated environment with no network route to production systems;
never execute unknown kernel drivers on a host you rely on;
comply with the laws of your jurisdiction and the policies of your institution.

Memory acquisition captures the full contents of a virtual machine's RAM, including credentials, cryptographic keys and personal data. Treat memory dumps as sensitive artifacts: store them under restricted permissions and delete them when the analysis is complete.

How it works
┌──────────────┐   ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│      1       │   │      2       │   │      3       │   │      4       │
│ Introspection│──▶│   Forensic   │──▶│   Feature    │──▶│  Detection   │
│              │   │   analysis   │   │  extraction  │   │              │
│ virsh dump   │   │ Volatility 3 │   │  11 numeric  │   │ One-Class SVM│
│ --memory-only│   │  14 plugins  │   │   features   │   │  + threshold │
│ --live       │   │              │   │              │   │              │
└──────────────┘   └──────────────┘   └──────────────┘   └──────────────┘
       ▲                                                        │
       │                                                        ▼
   KVM/QEMU                                            CLEAN │ INFECTED
   hypervisor                                          + confidence index
       │
       └──────────────── scheduler loop (periodic) ─────────────────┘

1 — Introspection. virsh dump --memory-only --live produces a raw memory image of a running instance without installing an agent inside it and without suspending the guest.

2 — Forensic analysis. Volatility 3 reconstructs kernel structures from the raw image. Windows symbol tables are resolved automatically from Microsoft PDB files, so no manual profile configuration is required.

3 — Feature extraction. Plugin outputs are normalized and cross-referenced to produce a fixed-length numeric vector. Several features require correlating multiple plugins, since a given driver is not named identically across them.

4 — Detection. The vector is standardized and scored by a One-Class SVM. The anomaly score is compared to a threshold calibrated as an empirical quantile of the training score distribution.

Features
Agentless. Nothing is installed inside the monitored virtual machine.
Out-of-guest. Collection cannot be subverted by a compromised guest kernel.
Unsupervised. No labeled attack corpus is required for training.
Two observation layers. Kernel-space and user-space artifacts, kept logically separate for interpretation while forming a single feature vector for the model.
Continuous monitoring. A scheduler runs the full pipeline on a fixed interval and logs state transitions.
Web dashboard. A Streamlit interface displays the verdict, the per-feature values and the contributing artifacts.
Modular. Each stage exchanges data in a standard format, so components can be replaced independently.
Requirements
Host (supervision node)

Requirements
Host (supervision node)
Component	Version used
OS	Ubuntu 24.04 LTS
CPU	x86-64 with Intel VT-x (vmx flag) or AMD-V
Hypervisor	KVM / QEMU with libvirt
Cloud platform	OpenStack (deployed via DevStack)
Python	3.10+

Guest

Windows 10 x64 (tested on Enterprise LTSC). Other Windows versions are untested; see Limitations.

Extracted features
Eleven features feed the final detector. They are grouped below by observation layer. see the thesis for the selection procedure.

Volatility 3 plugins used

windows.modules · windows.modscan · windows.driverscan · windows.driverirp · windows.devicetree · windows.callbacks · windows.timers · windows.ssdt · windows.thrdscan · windows.pslist · windows.psscan · windows.cmdline · windows.dlllist · windows.ldrmodules

Results

Evaluated on a corpus of 358 memory dumps (262 clean, 96 compromised) collected on Windows 10 x64 Enterprise LTSC.

Model	Accuracy	Recall	FPR	ROC-AUC
Isolation Forest	98.52 %	95.86 %	0.00 %	1.000
Local Outlier Factor	96.79 %	91.03 %	0.00 %	1.000
One-Class SVM	99.26 %	97.93 %	0.00 %	0.987

Read these numbers with the following caveats.

The training corpus contained 214 unlabeled observations with a controlled contamination of 1.87 %. Test metrics are averaged over five splits of a held-out set of 81 dumps.

With 29 positives in the test set, the recall has a 95 % Wilson confidence interval of roughly [82.8 % ; 99.4 %]. The zero false positive rate has an interval of [0 % ; 6.9 %] — it is measured on a clean corpus of limited software diversity and should not be read as a production figure.

Isolation Forest reaches an AUC of 1.000, meaning a perfectly separating threshold exists for it. One-Class SVM was retained for the stability of its calibrated threshold across splits, not for superior separating power.

A kernel driver absent from the training corpus (a signed, vulnerable driver abused in a bring your own vulnerable driver scenario) was correctly flagged. This is a single observation and does not constitute statistical evidence of generalization.

References
Wang, X., Zhang, J., Zhang, A., Ren, J. (2019). TKRD: Trusted kernel rootkit detection for cybersecurity of VMs based on machine learning and memory forensic analysis. Mathematical Biosciences and Engineering, 16(4), 2650–2667. DOI: 10.3934/mbe.2019132
Zhang, T., Lee, R. B. CloudMonatt: An Architecture for Security Health Monitoring and Attestation of Virtual Machines in Cloud Computing.
Ligh, M. H., Case, A., Levy, J., Walters, A. (2014). The Art of Memory Forensics. Wiley.
Schölkopf, B. et al. (2001). Estimating the Support of a High-Dimensional Distribution. Neural Computation, 13(7).
Volatility 3 — memory forensics framework
OpenStack — cloud infrastructure platform
LibVMI — virtual machine introspection library

Citation
bibtex
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

Acknowledgments

Work carried out within the RAISS team (Networks, Architectures, Service Engineering and Security) of the STRS laboratory at INPT, under the supervision of Prof. El Mostafa Belmekki, as part of the ESATIC–INPT academic partnership.

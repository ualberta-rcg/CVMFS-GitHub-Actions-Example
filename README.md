<img src="https://www.ualberta.ca/en/toolkit/media-library/homepage-assets/ua_logo_green_rgb.png" alt="University of Alberta Logo" width="50%" />

# Use Compute Canada CVMFS Software Stack in GitHub Actions

[![CI/CD](https://github.com/ualberta-rcg/CVMFS-GitHub-Actions-Example/actions/workflows/main.yml/badge.svg)](https://github.com/ualberta-rcg/CVMFS-GitHub-Actions-Example/actions/workflows/main.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](./LICENSE)

**Maintained by:** Rahim Khoja ([khoja1@ualberta.ca](mailto:khoja1@ualberta.ca))

## 🧰 Description

This GitHub Actions workflow demonstrates how to **use the Compute Canada / Digital Research Alliance CVMFS software stack** directly inside a GitHub-hosted runner.

It mounts the official CVMFS repositories and enables you to load modules like `gcc`, `cmake`, or `python` — just like on a real HPC cluster.

This is especially useful for:

- Building software compatible with Compute Canada environments
- Testing reproducible workflows before deploying on Alliance clusters
- Accessing precompiled Alliance tools inside GitHub CI/CD

## 🗂️ Files

- `.github/workflows/main.yml`: The GitHub Actions workflow
- This `README.md`

## 🚀 How It Works

### 📁 GitHub Actions Workflow

```yaml
name: Use Compute Canada CVMFS Software Stack

on:
  workflow_dispatch: # Allows manual trigger from the Actions tab

jobs:
  build-with-alliance-stack:
    name: Build with Alliance/Compute Canada Software
    runs-on: ubuntu-latest

    steps:
      - name: 🛠️ Checkout Repository
        uses: actions/checkout@v4

      - name: 📂 Mount Compute Canada CVMFS Repositories
        uses: cvmfs-contrib/github-action-cvmfs@v5
        with:
          cvmfs_repositories: 'soft.computecanada.ca,containers.computecanada.ca'

      - name: ✅ Verify Mount & Initialize Environment
        run: |
          echo "--- Verifying CVMFS mounts are available ---"
          ls /cvmfs/
          if [ ! -d "/cvmfs/soft.computecanada.ca" ]; then
            echo "❌ ERROR: CVMFS repository 'soft.computecanada.ca' not found!"
            exit 1
          fi
          echo "✅ CVMFS mounts look good."

          echo "--- Sourcing the Alliance environment profile ---"
          source /cvmfs/soft.computecanada.ca/config/profile/bash.sh
          module --version

      - name: ⚙️ Use Software from CVMFS
        run: |
          source /cvmfs/soft.computecanada.ca/config/profile/bash.sh
          echo "--- Searching for available GCC modules ---"
          module avail gcc

          echo "--- Loading GCC 12.3 and checking version ---"
          module load gcc/12.3
          gcc --version

      - name: 🧪 Run CP2K Simulation (water molecule)
        run: |
          echo "--- Avoiding Intel module auto-load & glibc error ---"
          export LMOD_SYSTEM_NAME=generic
          export LMOD_VERSION=8.5

          echo "--- Sourcing Compute Canada CVMFS environment ---"
          source /cvmfs/soft.computecanada.ca/config/profile/bash.sh
          module --force purge
          module load StdEnv/2020 gcc/9.3.0 cp2k/8.2 openmpi/4.0.3 

          echo "--- Creating water molecule input file ---"
          cat <<EOF > water.inp
          &GLOBAL
            PROJECT water
            RUN_TYPE ENERGY
          &END GLOBAL
          &FORCE_EVAL
            METHOD QUICKSTEP
            &DFT
              BASIS_SET_FILE_NAME BASIS_MOLOPT
              POTENTIAL_FILE_NAME GTH_POTENTIALS
              &MGRID
                CUTOFF 280
              &END MGRID
              &XC
                &XC_FUNCTIONAL PBE
                &END XC_FUNCTIONAL
              &END XC
            &END DFT
            &SUBSYS
              &CELL
                ABC 10.0 10.0 10.0
              &END CELL
              &COORD
                O 0.000 0.000 0.000
                H 0.758 0.000 0.504
                H -0.758 0.000 0.504
              &END COORD
              &KIND H
                BASIS_SET DZVP-MOLOPT-SR-GTH
                POTENTIAL GTH-PBE-q1
              &END KIND
              &KIND O
                BASIS_SET DZVP-MOLOPT-SR-GTH
                POTENTIAL GTH-PBE-q6
              &END KIND
            &END SUBSYS
          &END FORCE_EVAL
          EOF

          echo "--- Running CP2K ---"
          cp2k.psmp -i water.inp > water.out

          echo "--- Printing final SCF energy ---"
          grep "ENERGY|" water.out | tail -1
````

## ✅ Example Output

```
✅ CVMFS mounts look good.
--- Sourcing the Alliance/Compute Canada environment profile ---
Modules based on Lua: Version 8.7.47 2024-07-22 10:04 -04:00

--- Searching for available GCC modules ---
-------------------------------- Core Modules ---------------------------------
   gcc/12.3 (L,t,D)    gcc/13.3 (t)

  Where:
   D:  Default Module
   L:  Module is loaded
   t:  Tools for development

Use "module spider" or "module keyword" to find additional modules.

--- Loading a specific GCC module and checking its version ---
gcc (Gentoo 12.3.1_p20230526 p2) 12.3.1 20230526

--- Avoiding Intel module auto-load & glibc error ---
--- Sourcing Compute Canada CVMFS environment ---
Lmod is automatically replacing "intel/2020.1.217" with "gcc/9.3.0"
Lmod is automatically replacing "boost/1.72.0" with "boost-mpi/1.72.0"

--- Creating water molecule input file ---
--- Running CP2K ---
--- Printing final SCF energy ---
 ENERGY| Total FORCE_EVAL ( QS ) energy [a.u.]:              -17.212010403238466
```

## 🧪 Try It Yourself

Fork this repo or copy the workflow into your own repo, then run it from the **Actions** tab in GitHub.

No runners, secrets, or billing needed — everything runs on the free `ubuntu-latest` GitHub-hosted runner.

## 🤝 Support

If you're deploying this as part of a broader research platform or CI/CD workflow for research computing, feel free to reach out.

Email **[khoja1@ualberta.ca](mailto:khoja1@ualberta.ca)** for U of A or Alliance-related questions.

## 📜 License

This project is licensed under the **MIT License**, meaning:

* ✅ You can use, modify, and redistribute it freely
* ✅ Use it in commercial or private projects
* ✅ Include it in closed-source or public tools

Just retain the copyright.

**Full license text:** [MIT License](./LICENSE)

## 🧠 About University of Alberta Research Computing

The [Research Computing Group](https://www.ualberta.ca/information-services-and-technology/research-computing/index.html) supports research infrastructure, software workflows, and advanced computing services for U of A and Canadian researchers.

We build scalable systems, enable reproducible science, and help accelerate discovery.

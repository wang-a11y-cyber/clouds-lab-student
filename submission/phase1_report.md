# Phase 1 — Comparison Report

Copy this file to `submission/phase1_report.md`, fill in every answer slot, and commit it:

```bash
cp phase-1-echo-bot/report_template.md submission/phase1_report.md
```

**How to fill it in.** Each answer sits between a pair of `<!--answer:...-->` markers. Replace
the `TODO` line with your answer — **leave the markers themselves untouched**, they are how the
grader finds your answers. Anything you write outside the markers is ignored, so add extra
prose, tables or images freely.

Numbers must **match `submission/phase1_report.json`** exactly — that file is produced by
`measure.py` and is what your figures are checked against. Copy them across; do not retype from
memory.

---

## 1. Setup facts

**Your image size**, in MB, from `gcloud artifacts docker images list` (number only, e.g. `142`):

<!--answer:image_size_mb-->
48.97276
<!--/answer-->

**The custom VPC and subnet you created** (names only):

<!--answer:vpc_name-->
echo-net
<!--/answer-->

<!--answer:subnet_name-->
echo-subnet
<!--/answer-->

---

## 2. Measurements

Copy these from `submission/phase1_report.json`. Cold is `cold_ms`; warm median is
`stats.median`. Numbers only, in milliseconds (e.g. `812.4`).

| Platform | Cold (ms) | Warm median (ms) |
|---|---|---|
| IaaS VM | <!--answer:vm_cold_ms-->232.8<!--/answer--> | <!--answer:vm_warm_median_ms-->210.8<!--/answer--> |
| Cloud Run | <!--answer:cloudrun_cold_ms-->2019.9<!--/answer--> | <!--answer:cloudrun_warm_median_ms-->632.0<!--/answer--> |
| Cloud Function | <!--answer:function_cold_ms-->1473.3<!--/answer--> | <!--answer:function_warm_median_ms-->132.5<!--/answer--> |

Include your histogram or plot of the warm distribution below (an image, or the ASCII chart
`measure.py` prints). This is read by a human, not the grader:

<!--answer:latency_plot-->
------------------------------------------------------------
[OK  ] vm        cold=  232.8ms  warm_median=  210.8ms
            #############
[OK  ] cloudrun  cold= 2019.9ms  warm_median=  632.0ms
            ########################################
[OK  ] function  cold= 1473.3ms  warm_median=  132.5ms
            ########
------------------------------------------------------------
<!--/answer-->

---

## 3. Per-platform analysis

**Scaling behaviour.** Imagine requests arriving one at a time, then ten at once, then a
hundred. For each of the three platforms, answer concretely:

- Does anything get created automatically to absorb the load — and if so, *what* exactly?
  (One platform here creates nothing at all.)
- What is the hard limit, and where does it come from — a flag you passed yourself, or the
  size of the machine you picked?

(~80 words. Your `--machine-type=e2-micro` and `--max-instances=3` are both concrete answers
to the second question.)

<!--answer:scaling_behaviour-->
The VM does not create anything automatically as load increases; all requests are handled by the single e2-micro VM, so its CPU and memory are the practical limit. Cloud Run automatically creates additional container instances to handle concurrent requests, but I limited it with --max-instances=3. The Cloud Function also scales automatically by creating additional function instances as demand increases. Unlike the fixed VM, the two serverless platforms manage instance creation and scaling automatically, subject to their configured platform limits.
<!--/answer-->

**Deployment effort.** What did you have to build, configure and maintain to get each of the
three running? (~100 words)

<!--answer:deployment_effort-->
For the VM, I created and configured the e2-micro instance, networking, firewall access, and the application process, so I am responsible for maintaining the machine and service. For Cloud Run, I built the Docker image, pushed it to Artifact Registry, and deployed the container while Google manages the underlying servers and scaling. For the Cloud Function, I deployed the function source and specified its runtime and entry point; Google handled the container and infrastructure. The VM therefore required the most infrastructure management, while the two serverless approaches required progressively less.
<!--/answer-->

---

## 4. Why do two of your three numbers look alike?

With Cloud Run and your Function both deployed, run `gcloud run services list` and paste the
**complete output** here, unedited:

<!--answer:run_services_list-->
✔
SERVICE: echo-bot
REGION: us-central1
URL: https://echo-bot-345870656110.us-central1.run.app
LAST DEPLOYED BY: wentao.wang@eurecom.fr
LAST DEPLOYED AT: 2026-10-13T20:56:38.644600Z

✔
SERVICE: echo-function
REGION: us-central1
URL: https://echo-function-345870656110.us-central1.run.app
LAST DEPLOYED BY: service-345870656110@gcf-admin-robot.iam.gserviceaccount.com
LAST DEPLOYED AT: 2026-10-13T21:46:36.283875Z
<!--/answer-->

Now explain what that output tells you about how your Function is actually being run, why its
cold start resembles Cloud Run's so closely, and what — if anything — genuinely differs between
the two deployment paths. (~120 words)

<!--answer:convergence_explanation-->
The output shows that both echo-bot and echo-function appear as Cloud Run services. This means the second-generation Cloud Function is ultimately executed on Cloud Run infrastructure rather than on a completely separate serving system. That explains why their cold-start times are relatively similar: both may need a managed container instance to be created and initialized before serving a request. The main difference is the deployment abstraction. For Cloud Run, I explicitly built and deployed a container image and configured the service. For the Function, I supplied function source code, runtime and entry point, while Google handled the build and underlying Cloud Run service automatically. Therefore the execution infrastructure converges, while the developer-facing deployment workflows differ.
<!--/answer-->

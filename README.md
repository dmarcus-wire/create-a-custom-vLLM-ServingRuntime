# Create a custom vLLM ServingRuntime

To update the vLLM image used for model serving in Red Hat OpenShift AI 3.2 to the custom CPU image quay.io/vllm/automation-vllm:0.18.0rc1.dev7_rhaiv.0.ge974742fb.cpu, create a custom ServingRuntime based on the built-in vLLM CPU one.
This is the recommended and supported approach (Red Hat does not support editing the built-in runtimes directly, and custom runtimes are your responsibility to maintain and license).
Prerequisites

You are logged in to the OpenShift AI dashboard as a user with OpenShift AI administrator privileges (cluster-admin role in OpenShift).
The target image quay.io/vllm/automation-vllm:0.18.0rc1.dev7_rhaiv.0.ge974742fb.cpu is accessible from your cluster (it must be pullable; if in a disconnected environment, mirror it first).
You have the vLLM CPU ServingRuntime for KServe available (it is included in OpenShift AI 3.2 for CPU-only or IBM Z/Power scenarios).

## Step-by-step procedure (via OpenShift AI UI – recommended)

In the OpenShift AI dashboard, go to Settings → Model resources and operations → Serving runtimes.
This page lists all installed and enabled runtimes.
Locate the built-in runtime named vLLM CPU ServingRuntime for KServe (or the closest vLLM runtime matching your hardware; the .cpu tag indicates you want the CPU variant).
Click the action menu (⋮) next to it and select Duplicate.
(Duplicating is the easiest way to start with a fully working base configuration including all vLLM-specific args, env vars, supported formats, etc.)
Configure the new runtime:
Model serving platform: Select Single-model serving platform.
API protocol: Select REST (vLLM typically uses the OpenAI-compatible REST endpoint).
In the embedded YAML editor, update the container image:YAMLspec:
  containers:
  - name: kserve-container
    image: quay.io/vllm/automation-vllm:0.18.0rc1.dev7_rhaiv.0.ge974742fb.cpu   # ← Change this line
You can also optionally update:
metadata.annotations.openshift.io/display-name to something like "Custom vLLM CPU 0.18.0rc1 (rhaiv)".
Any labels or opendatahub.io/runtime-version if you want to distinguish it.
Add/modify env or args if the new image requires extra vLLM flags.



Click Add (or Save).
The custom runtime appears in the list and is automatically enabled.
(Optional) Verify or further edit:
Click the ⋮ menu on your new runtime → Edit to make changes.

## Verification

The new runtime shows as enabled on the Serving runtimes page.
When you deploy a new model (Model Serving → Deploy model), it will appear in the Serving runtime dropdown. Select your custom one.

## Using the new runtime for models

For new model deployments: Choose your custom runtime in the deployment wizard or InferenceService.
For existing deployments: You will need to edit the existing InferenceService (or redeploy the model) and switch it to the new custom runtime. The underlying Deployment will then pull the updated image.

Alternative: CLI / YAML (if you prefer)
You can create the ServingRuntime directly with oc apply -f custom-vllm-runtime.yaml in the appropriate namespace (usually the project namespace or the OpenShift AI operator namespace). Start by exporting the built-in one with oc get servingruntime vllm-cpu-servingruntime -o yaml (adjust the name), modify the image field, and apply.
Important notes

IMPORTANTL Red Hat does not provide support for custom runtimes (just that you can create customize the runtime) — test thoroughly.
The new image must be compatible with vLLM’s expected entrypoint (vllm serve or equivalent) and any required arguments/env vars already in the duplicated runtime.
If you are on GPU hardware, duplicate the vLLM NVIDIA GPU ServingRuntime for KServe instead (the provided image is the CPU variant).
After updating, monitor the new InferenceService pods for image pull success and startup logs.

This process makes the custom vLLM image available across your OpenShift AI 3.2 environment without touching the built-in defaults. If you run into issues pulling the image or compatibility problems, check the pod logs or the Red Hat Knowledgebase for your exact OpenShift AI version.

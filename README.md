# gpu-info AI skill

This repository is a "skill" for agentic AI that adds a possibility for your agent to query for GPUs (graphics cards, integrated graphics chips) installed in your local system, their basic parameters (vendor, name, amount of video memory, graphics driver version), as well as all the detailed capabilities, properties, limits, extensions, formats supported by it as seen through Vulkan and DirectX 12 API.

## Installation

This is not a single command, sorry 🙂

1. Download the ZIP archive of this repository.
   - Unpack the `gpu-info` subdirectory to wherever you can place your agentic AI skills, whether in your repository or user directory. For example, when using Codex on Windows, unpack it to `c:\Users\YOUR_LOGIN\.codex\skills\`
   - To check if you did it correctly, validate that following file exists: `c:\Users\YOUR_LOGIN\.codex\skills\gpu-info\SKILL.md`.
2. (Windows only) Go to [the latest release of D3d12info app](https://github.com/sawickiap/D3d12info/releases/latest). Download `D3d12info.zip`.
   - Unpack it to the `scripts/` subdirectory of this skill.
   - To check if you unpacked it correctly, validate that following file exists: `c:\Users\YOUR_LOGIN\.codex\skills\gpu-info\scripts\D3d12info\D3d12info.exe`.

## Usage

You can use the \$gpu-info skill explicitly or just ask about the current GPU installed in the local system and any details about it, including Vulkan or DirectX 12 capabilities.

Example:

Q:

>Tell me about the support for 64-bit types and operations on my GPU from both DX12 and Vulkan perspective.

A:

>Your GPU has strong 64-bit support in both DX12 and Vulkan, for both floating-point and integer operations.
>
>DX12 (D3d12info.log)
>
>- DoublePrecisionFloatShaderOps = TRUE -> 64-bit float (double) shader ops supported
>- Int64ShaderOps = TRUE -> 64-bit integer shader ops supported
>- AtomicInt64OnTypedResourceSupported = TRUE
>- AtomicInt64OnGroupSharedSupported = TRUE
>- AtomicInt64OnDescriptorHeapResourceSupported = TRUE
>
>Vulkan (vulkaninfo.log)
>
>- shaderFloat64 = true -> 64-bit float shader ops supported
>- shaderInt64 = true -> 64-bit integer shader ops supported
>- shaderBufferInt64Atomics = true
>- shaderSharedInt64Atomics = true
>- shaderImageInt64Atomics = true
>- sparseImageInt64Atomics = true
>
>Practical takeaway:
>
>- You can use double and int64_t-class shader operations on both APIs.
>- 64-bit atomics are also broadly supported (buffers/shared/image in Vulkan; typed resources/groupshared/descriptor-heap resources in DX12).
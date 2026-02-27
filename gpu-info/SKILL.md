---
name: gpu-info
description: Reads information about GPUs installed in the local system, including vendor and model name, VendorID, DeviceID, amount of memory, driver version, all DirectX 12 and Vulkan capabilities, properties, limits, supported extensions, and formats.
---

# Steps to execute

1. Determine a directory path good for temporary files. Let's call it TEMP_DIR.
	- If you already know there is a directory in the current repository used for temporary files, output files, logs, or some kind of cache, use it as TEMP_DIR.
	- Otherwise, if the project in the repository uses CMake, use the directory that serves as the CMake binary directory as our TEMP_DIR.
	- Otherwise, just use the path to the root directory of the repository as our TEMP_DIR.
2. Determine which API you should use: DirectX 12 or Vulkan:
	- If you need information about "Vulkan" explicitly, like supported properties, limits, extensions, use Vulkan.
	- If you need information about "DirectX 12", "Direct3D 12", "DX12", "D3D12", use DirectX 12.
	- Only if you work on Windows:
		- If you need basic information about the current GPU, use DirectX 12.
		- If you need driver version, use DirectX 12.
	- Otherwise, use Vulkan.
3. For further steps, refer to appropriate file in this skill:
	- If using Vulkan, read file `references/api_vulkan.md` and follow instructions from it.
	- If using DirectX 12, read file `references/api_directx12.md` and follow instructions from it.

This file describes steps to follow when using Vulkan within this skill.

# Steps to follow

1. Determine which COMMAND you should execute based on following rules:
	- If you only need basic information about the GPU like the "vendor" and "model" name, "VendorID", "DeviceID", "type" (dedicated/integrated), the COMMAND is: `vulkaninfo --summary > "TEMP_DIR/vulkaninfo.log"`
	- If you need information about "formats", the COMMAND is: `vulkaninfo --show-formats > "TEMP_DIR/vulkaninfo.log"`
	- In all other cases, the command is: `vulkaninfo > "TEMP_DIR/vulkaninfo.log"`
2. Execute the COMMAND.
	- Note the COMMAND writes the output to a file. Let's call it OUTPUT_FILE. It has a text format where indentation marks nested structures and their members.
	- If you invoked the same COMMAND before, you can skip this step - you don't need to execute the COMMAND again. Reuse the existing OUTPUT_FILE only within current session and same machine state; re-run if user asks "current/latest" or OUTPUT_FILE missing.
	- If the COMMAND ended with an error/failure, report the issue to the user and stop the skill!
    - If you successfully executed the command, tell the user explicitly in your final response that the OUTPUT_FILE has been created.
3. Read fragments of the OUTPUT_FILE to find the information you need.
	- The OUTPUT_FILE may be long, hundreds of kilobytes in size, so don't read it entirely, but search for specific information and read just lines around it.

# Interpreting the OUTPUT FILE

There may be multiple GPUs present in the system, denoted by lines like this example: `GPU0:`. If there are more than 1 GPUs:

- If user specifies GPU/vendor/model/index, use that.
- Otherwise use GPU0.
- Always include GPU index (GPU0, GPU1) in the answer.

## Reading basic information

When you need basic information about the GPU, search for lines that look like in this example:

```
	apiVersion         = 1.4.334
	driverVersion      = 2.0.373
	vendorID           = 0x1002
	deviceID           = 0x7590
	deviceType         = PHYSICAL_DEVICE_TYPE_DISCRETE_GPU
	deviceName         = AMD Radeon RX 9060 XT
	driverID           = DRIVER_ID_AMD_PROPRIETARY
	driverName         = AMD proprietary driver
	driverInfo         = 26.2.2 (LLPC)
```

That's how you can find vendor ID, device ID, device type ("DISCRETE" versus "INTEGRATED"), device name, and driver version.

## Reading Vulkan properties/limits

When you need a specific property or limit or a set of those, coming from a specific Vulkan structure, search for the name of the structure (like `VkPhysicalDeviceSparseProperties`) and the parameter (like `residencyAlignedMipSize`). You will find text that looks like this example, listing parameters and their values, which may be of different types (boolean, integer, hexadecimal like `0x00000004`, floating-point like `0.125`):

```
VkPhysicalDeviceSparseProperties:
---------------------------------
	residencyStandard2DBlockShape            = true
	residencyStandard2DMultisampleBlockShape = false
	residencyStandard3DBlockShape            = true
	residencyAlignedMipSize                  = false
	residencyNonResidentStrict               = true
```

Some parameters are vectors, so you need to read following indented lines to get the full value, like:

```
	viewportBoundsRange: count = 2
		-32768
		32767
```

Some parameters are bit flags, which are shown in hexadecimal, as well as decoded into the names of the individual flags in the indented lines below it, like:

```
		propertyFlags = 0x00ce: count = 5
			MEMORY_PROPERTY_HOST_VISIBLE_BIT
			MEMORY_PROPERTY_HOST_COHERENT_BIT
			MEMORY_PROPERTY_HOST_CACHED_BIT
			MEMORY_PROPERTY_DEVICE_COHERENT_BIT_AMD
			MEMORY_PROPERTY_DEVICE_UNCACHED_BIT_AMD
```

## Reading Vulkan extensions

When you need to know if a specific extension is supported by the GPU or you need the list of all supported extensions, inspect the section listing device extensions, which looks like this example:

```
Device Extensions: count = 213
	VK_AMDX_dense_geometry_format               : extension revision 1
	VK_AMD_anti_lag                             : extension revision 1
	VK_AMD_buffer_marker                        : extension revision 1
	VK_AMD_device_coherent_memory               : extension revision 1
	VK_AMD_display_native_hdr                   : extension revision 1
	VK_AMD_draw_indirect_count                  : extension revision 2
	VK_AMD_gcn_shader                           : extension revision 1
```

Each line mentions one extension that is supported by the device.

There are also instance extensions listed at the beginning of the file, which you also need to inspect if searching instance extension is needed, like in this example:

```
Instance Extensions: count = 14
===============================
	VK_EXT_debug_report                    : extension revision 10
	VK_EXT_debug_utils                     : extension revision 2
	VK_EXT_swapchain_colorspace            : extension revision 5
	VK_KHR_device_group_creation           : extension revision 1
	VK_KHR_external_fence_capabilities     : extension revision 1
```

## Reading memory information

When you need information about memory, search for a section that looks like the following example:

```
VkPhysicalDeviceMemoryProperties:
=================================
memoryHeaps: count = 2
	memoryHeaps[0]:
		size   = 68530012160 (0xff4b50000) (63.82 GiB)
		budget = 67724752896 (0xfc4b5b800) (63.07 GiB)
		usage  = 331776 (0x00051000) (324.00 KiB)
		flags:
			None
	memoryHeaps[1]:
		size   = 17095983104 (0x3fb000000) (15.92 GiB)
		budget = 16177983488 (0x3c4487000) (15.07 GiB)
		usage  = 0 (0x00000000) (0.00 B)
		flags: count = 2
			MEMORY_HEAP_DEVICE_LOCAL_BIT
			MEMORY_HEAP_MULTI_INSTANCE_BIT
memoryTypes: count = 16
	memoryTypes[0]:
		heapIndex     = 1
		propertyFlags = 0x0001: count = 1
			MEMORY_PROPERTY_DEVICE_LOCAL_BIT
		usable for:
			IMAGE_TILING_OPTIMAL:
				color images
				FORMAT_D16_UNORM
				FORMAT_D32_SFLOAT
				FORMAT_S8_UINT
				FORMAT_D16_UNORM_S8_UINT
				FORMAT_D32_SFLOAT_S8_UINT
			IMAGE_TILING_LINEAR:
				color images
```

This is a 2-level hierarchy with:

1. Memory heaps.
2. Memory types, each referring to a specific memory heap by its `heapIndex`.

As the summary of memory:

- List heaps that have `MEMORY_HEAP_DEVICE_LOCAL_BIT` flag and their parameters as "GPU memory".
- List heaps that don't have `MEMORY_HEAP_DEVICE_LOCAL_BIT` flag and their parameters as "system memory".
- Check and report if there is a memory type that has both `MEMORY_PROPERTY_DEVICE_LOCAL_BIT` and `MEMORY_PROPERTY_HOST_VISIBLE_BIT` and what is its size.

This file describes steps to follow when using DirectX 12 within this skill.

# Steps to follow

1. Check if following executable file within this skill exists: `scripts\D3d12info\D3d12info.exe`. If not, report the problem to the user and stop this skill!
2. Determine which COMMAND you should execute based on following rules:
	- If you need information about "formats", the COMMAND is: `scripts\D3d12info\D3d12info.exe --Formats > "TEMP_DIR/D3d12info.log"`
	- In all other cases, the command is: `scripts\D3d12info\D3d12info.exe > "TEMP_DIR/D3d12info.log"`
2. Execute the COMMAND.
	- Note the COMMAND writes the output to a file. Let's call it OUTPUT_FILE. It has a text format where indentation marks nested structures and their members.
	- If you invoked the same COMMAND before, you can skip this step - you don't need to execute the COMMAND again. Reuse the existing OUTPUT_FILE only within current session and same machine state; re-run if user asks "current/latest" or OUTPUT_FILE missing.
	- If the COMMAND ended with an error/failure, report the issue to the user and stop the skill!
    - If you successfully executed the command, tell the user explicitly in your final response that the OUTPUT_FILE has been created.
3. Read fragments of the OUTPUT_FILE to find the information you need.
	- The OUTPUT_FILE may be long, hundreds of kilobytes in size, so don't read it entirely, but search for specific information and read just lines around it.

# Interpreting the OUTPUT FILE

There may be multiple GPUs present in the system, each denoted by lines like this example:

```
Adapter 0:
==========
```

If there are more than 1 GPUs:

- If user specifies GPU/vendor/model/index, use that.
- Otherwise use Adapter 0.
- Always include adapter index in the answer.

## Reading basic information

When you need basic information about the GPU, search for lines that look like in this example:

```
DXGI_ADAPTER_DESC3:
-------------------
    Description = AMD Radeon RX 9060 XT
    VendorId = AMD/ATI (0x1002)
    DeviceId = 0x7590
    SubSysId = Sapphire (0xA4931DA2)
    Revision = 0xC0
    DedicatedVideoMemory = 15.82 GiB (16987488256 B)
    DedicatedSystemMemory = 0 B
    SharedSystemMemory = 63.82 GiB (68530059264 B)
    AdapterLuid = 00000000-00012168
```

That's how you can find vendor ID, device ID, device description (name), and the amount of its dedicated memory.

## Reading driver version

When you need the version of the graphics driver, search the file for all lines containing "driver" or "version" (case-insensitive). D3d12info tool fetches the driver version in multiple ways. Analyze them together. As the result, use the following:

- "UMD Version" that looks like this example: `UMDVersion = 32.0.23027.2005` (always consists of 4 numbers).
- If the GPU is Nvidia, figure out their "user-facing version", which looks like for example "546.29".
- If the GPU is AMD, figure out their 2 versions:
    - "User-facing version" which looks like for example "23.12.1" ("year.month.ordinal" notation). Note it may not be present on some GPUs.
    - "Driver version", which looks like for example "23.30.13.01-231128a-398226C-AMD-Software-Adrenalin-Edition" (starting from 4 or sometimes 3 numbers separated by dots).

## Reading options

When you need a specific option coming from a specific D3D12 structure, search for the name of the structure (like `D3D12_FEATURE_DATA_D3D12_OPTIONS5`) and the parameter (like `RaytracingTier`). You will find text that looks like this example, listing parameters and their values, which may be of different types (boolean like `FALSE`, `TRUE`, integer, hexadecimal like `0x64`, an enum that specifies its name as well as numeric representation like `D3D12_RAYTRACING_TIER_1_1 (0xB)`):

```
D3D12_FEATURE_DATA_D3D12_OPTIONS5:
----------------------------------
    SRVOnlyTiledResourceTier3 = TRUE
    RenderPassesTier = D3D12_RENDER_PASS_TIER_0 (0x0)
    RaytracingTier = D3D12_RAYTRACING_TIER_1_1 (0xB)
```

Some parameters are bit flags, which are shown in hexadecimal, as well as decoded into the names of the individual flags in the indented lines below it, like:

```
    WriteBufferImmediateSupportFlags = 0xF
        D3D12_COMMAND_LIST_SUPPORT_FLAG_DIRECT
        D3D12_COMMAND_LIST_SUPPORT_FLAG_BUNDLE
        D3D12_COMMAND_LIST_SUPPORT_FLAG_COMPUTE
        D3D12_COMMAND_LIST_SUPPORT_FLAG_COPY
```

## Reading format support

When you need information about a specific "format", search for sections that look like this example:

```
    DXGI_FORMAT_R32_TYPELESS:
    -------------------------
        Support1 = 0x1010F0
            D3D12_FORMAT_SUPPORT1_TEXTURE1D
            D3D12_FORMAT_SUPPORT1_TEXTURE2D
            D3D12_FORMAT_SUPPORT1_TEXTURE3D
            D3D12_FORMAT_SUPPORT1_TEXTURECUBE
            D3D12_FORMAT_SUPPORT1_MIP
            D3D12_FORMAT_SUPPORT1_CAST_WITHIN_BIT_LAYOUT
        Support2 = 0x200
            D3D12_FORMAT_SUPPORT2_TILED
        SampleCount = 1: NumQualityLevels = 1
        SampleCount = 2: NumQualityLevels = 1
        SampleCount = 4: NumQualityLevels = 1
        SampleCount = 8: NumQualityLevels = 1
        PlaneCount = 1
```

It lists bit flags representing what that specific format supports.
